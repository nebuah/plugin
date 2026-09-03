# Unificación con nebuah-engine

Qué cambió en este repo el 2026-09-02 y por qué. Punto de retorno: tag
`pre-unificacion-2026-09-02`.

## La decisión

El engine y este plugin escribían dos formatos de proyecto distintos. **Gana el
del engine y este plugin se muda.** No hay simetría que negociar:

| | Engine | Plugin |
|---|---|---|
| Proyectos reales en disco | 20 | 2, con sólo `output/` |
| Estado persistido | máquina de 12 fases | ninguno |
| Costo de migrarlo | el negocio | ≈ 0 |

La especificación canónica es `docs/PROJECT_FORMAT.md` en el repo del engine.

## Cambios

### `CLAUDE.md` viaja con el plugin

Estaba sólo en la raíz del repo. Instalado desde el marketplace, el usuario
obtenía `/nebuah` y los tres subagentes **sin el contrato del kernel**: sin la
convención de niveles, sin el formato de traces, sin las reglas de memoria.

Ahora vive en `nebuah/CLAUDE.md` y la raíz lo importa con `@nebuah/CLAUDE.md`,
así que las dos formas de usarlo funcionan.

### El directorio de trabajo es explícito

Todas las rutas (`system/memory/…`, `projects/…`) son relativas al CWD y el
manual no decía cuál. Ahora el contrato arranca verificando que exista un
workspace Nebuah y **pregunta antes de crear uno** dentro de un repo ajeno.

### Formato de proyecto del engine

`project.json` con `schema_version`, `phase` y `phase_data.plan_tasks`;
`ROSTER.md` que ata tarea↔agente **por nombre**; traces **dentro del
proyecto** (`traces/<agente>/…` + `_index.jsonl`) en vez del
`trace_YYYY-MM-DD.md` global que el engine nunca lee.

Dos reglas nuevas y no negociables:

* `schema_version` mayor al conocido → **no escribir**, preguntar.
* **Nunca borrar una clave que no se entiende.** El engine hace lo mismo con
  las nuestras: hasta este cambio, una sola clave desconocida en
  `project.json` hacía que el proyecto desapareciera de su radar sin un log.

### Reanudar es el default

`Step 2.5` busca el proyecto antes de crear nada: lee `phase`, calcula qué
tareas quedaron sin `output_file` y ejecuta sólo esas, con el equipo que ya
está en el ROSTER. Crear un proyecto nuevo pasó a ser la excepción.

### Lock cooperativo

El engine hace tick sobre estas mismas carpetas. Antes de escribir `phase` o
`phase_data` hay que tomar `locked_by` / `locked_at`, y soltarlos al terminar
**incluso si la corrida falla**. El lock vence a los 30 minutos: un proceso que
muere no puede congelar un expediente.

Leer y escribir a `output/`, `traces/` y `components/agents/` nunca necesita
lock.

### Niveles: L1 es el más alto

El engine usaba el mismo prefijo `level_N_` con la altura **invertida**
(`level_1_reflexive` era lo más bajo). Sus directorios estaban vacíos y se
renombraron a esta convención. La traducción del resto de la memoria va por
`services/memory_adapter.py` en el engine — nunca a mano.

### Un solo árbol en Drive

Había dos bajo la misma raíz: `projects/` por la API y `proyectos/` por Drive
para Escritorio. Gana `proyectos/`: ya tiene datos y clientes mirándolo. El
estado del proyecto va a `_sistema/` dentro de cada carpeta, que es lo que
permite retomar desde Drive sin ensuciar la vista del cliente.

### La regla 7, resuelta

Tres fuentes decían dos cosas opuestas sobre subir definiciones de agentes.
Queda una sola regla:

* **traces**: nunca salen de la máquina.
* **definiciones de agentes**: sí se suben — es lo que hace reutilizable un
  expediente pasado, y el control de acceso son los permisos de la carpeta.
* **`confidential: true`**: no sube **nada**, y eso pisa todo lo anterior.

### `model` es un rol lógico

`gemini-3.5-flash` en un ROSTER lo vuelve imposible de portar: el engine corre
`ollama/gemma4:26b` y este plugin corre Claude. Ahora se escribe
`razonamiento-largo`, `redaccion` o `verificacion` y cada runtime lo mapea.
