# Tutorial: Nebuah con Google Drive — Guia para usuarios de la version anterior

## Que cambio?

Si usabas la version anterior de Nebuah, ya sabes que la experiencia era simple: escribias `/nebuah "tu objetivo"` y el sistema hacia todo automaticamente — creaba carpetas, descomponia la tarea en agentes, ejecutaba, consolidaba aprendizajes y te entregaba un reporte.

**La nueva version mantiene exactamente la misma experiencia.** La unica diferencia es que ahora todo tu trabajo tambien se sincroniza automaticamente con Google Drive. No tenes que aprender comandos nuevos ni cambiar tu flujo de trabajo.

---

## Comparacion rapida

| Aspecto | Version anterior | Nueva version con GDrive |
|---------|-----------------|--------------------------|
| Comando principal | `/nebuah "objetivo"` | `/nebuah "objetivo"` (igual) |
| Crear carpetas de proyecto | Automatico | Automatico (local + GDrive) |
| Documentos de entrada | Solo locales | Se descargan automaticamente desde GDrive |
| Entregables de salida | Solo locales | Se suben automaticamente a GDrive |
| Estrategias aprendidas | Solo en tu maquina | Se sincronizan a GDrive (accesibles desde cualquier maquina) |
| Memoria entre proyectos | Solo local | Busca automaticamente en proyectos pasados via GDrive |
| Setup de GDrive | N/A | Automatico (sin configuracion manual) |
| Pregunta al usuario | Ninguna | Maximo UNA vez (solo si hay ambiguedad) |

---

## Como empezar

### Paso 1: Instala el plugin

Asegurate de tener el plugin de Nebuah instalado en Claude Code. La carpeta del proyecto debe contener los archivos del sistema (`CLAUDE.md`, `nebuah/`, `system/`, etc.).

### Paso 2: Conecta Google Drive a Claude Code

Nebuah usa la integracion nativa de Google Drive de Claude Code (via MCP). Para habilitarla:

1. Abre Claude Code
2. Ve a la configuracion de integraciones
3. Habilita **Google Drive** como fuente de datos
4. Autoriza el acceso con tu cuenta de Google

> Esto solo se hace una vez. Una vez autorizado, Claude Code tiene acceso permanente a tu Drive.

### Paso 3: Usa Nebuah como siempre

```
/nebuah "Analizar documentos de due diligence para fusion corporativa"
```

Eso es todo. No hay paso 4.

---

## Que pasa por detras (automaticamente)

Cuando ejecutas `/nebuah "tu objetivo"`, el sistema ahora ejecuta este pipeline completo:

```
/nebuah "Analizar documentos de due diligence"
|
|-- Step 0: GDRIVE AUTO-BOOTSTRAP (solo la primera vez)
|   Busca o crea la carpeta "Nebuah" en tu Google Drive.
|   Crea las subcarpetas del sistema (projects/, system/memory/strategies/).
|   Guarda un registro local (system/gdrive_registry.json).
|   En ejecuciones futuras, este paso se salta completamente.
|
|-- Step 1: CONSULTA DE MEMORIA
|   Carga estrategias locales + busca automaticamente en GDrive
|   estrategias de proyectos anteriores que puedan ser relevantes.
|
|-- Step 2: ANALISIS Y PLAN
|   Descompone el objetivo en minimo 3 sub-tareas (Research, Quality, Integration).
|
|-- Step 3: CREAR ESTRUCTURA DE PROYECTO (Local + GDrive)
|   Crea carpetas locales:
|     projects/DueDiligence/{components/agents, input, output, memory}
|   Crea carpetas en GDrive:
|     Nebuah/projects/DueDiligence/{input, output, memory/long_term}
|   Descarga documentos de entrada desde GDrive input/ a local.
|
|-- Step 4: CREAR AGENTES ESPECIALIZADOS
|   Minimo 3 agentes por tarea.
|
|-- Step 5: EJECUTAR
|   Todo el trabajo es local. Sin llamadas a GDrive.
|
|-- Step 6: PRODUCIR OUTPUT + SUBIR A GDRIVE
|   Guarda entregables en projects/DueDiligence/output/
|   Los sube automaticamente a GDrive/Nebuah/projects/DueDiligence/output/
|
|-- Step 7: CONSOLIDAR + SINCRONIZAR APRENDIZAJES
|   Ejecuta Dream Engine (minimo 3 ciclos en paralelo).
|   Sube las nuevas estrategias/constraints a GDrive automaticamente.
|
|-- Step 8: REPORTE
|   Te muestra un resumen completo incluyendo el estado de GDrive.
```

---

## Trabajar con documentos de entrada desde Google Drive

Una de las funcionalidades mas utiles: podes subir documentos a Google Drive **antes** de ejecutar Nebuah, y el sistema los descarga automaticamente.

### Como funciona

1. Anda a tu Google Drive
2. Navega a `Nebuah/projects/[NombreDelProyecto]/input/`
3. Subi tus documentos ahi (PDF, Google Docs, .txt, .md, .docx, Excel)
4. Ejecuta `/nebuah "tu objetivo"`
5. Nebuah detecta los archivos y los descarga automaticamente a tu maquina local

### Formatos soportados

| Formato | Que pasa al descargar |
|---------|----------------------|
| Google Docs | Se convierte a Markdown (.md) |
| Google Sheets | Se convierte a tabla en Markdown (.md) |
| PDF | Se descarga como PDF (binario) |
| Word (.docx) | Se extrae el texto a Markdown (.md) |
| Excel (.xlsx) | Se extrae como tabla en Markdown (.md) |
| Texto plano (.txt) | Se descarga tal cual |
| Markdown (.md) | Se descarga tal cual |

### Tip: Pre-crear la carpeta del proyecto

Si sabes el nombre del proyecto de antemano, podes crear la estructura en GDrive manualmente:

```
Nebuah/projects/MiFusion/input/   <-- subi tus documentos aca
```

Cuando ejecutes `/nebuah "Analizar fusion MiFusion"`, el sistema va a detectar esa carpeta existente, reutilizarla, y descargar los documentos que hayas subido.

---

## Acceso a memoria entre proyectos

Esta es la funcionalidad mas poderosa de la integracion con GDrive. Cuando Nebuah planifica una nueva tarea:

1. Busca estrategias locales (como antes)
2. **Tambien busca en Google Drive** estrategias de proyectos anteriores
3. Si encuentra algo relevante, lo incorpora al plan

Esto significa que si trabajaste en un caso de M&A hace 3 meses en otra maquina, y ahora empezas un nuevo caso similar, Nebuah va a encontrar y reutilizar las estrategias que aprendio en aquel proyecto.

### Ejemplo practico

```
Proyecto 1 (Enero): /nebuah "Due diligence para adquisicion de empresa tech"
  -> Nebuah aprende: estrategia de revision de IP, checklist regulatorio, etc.
  -> Estrategias se suben a GDrive automaticamente.

Proyecto 2 (Abril, otra maquina): /nebuah "Due diligence para adquisicion de empresa biotech"
  -> Nebuah busca en GDrive, encuentra las estrategias del Proyecto 1.
  -> Las aplica como base, adaptandolas al nuevo contexto.
```

No tenes que hacer nada para que esto funcione. Es completamente automatico.

---

## La unica pregunta que Nebuah te puede hacer

En toda la integracion con Google Drive, hay **un solo escenario** donde Nebuah te hace una pregunta:

> **Cuando:** La primera vez que ejecutas `/nebuah` y existen **multiples** carpetas llamadas "Nebuah" en tu Google Drive.
>
> **Pregunta:** "Encontre N carpetas llamadas 'Nebuah'. Cual quiero usar?"
>
> **Cuando NO pregunta:** Si hay 0 carpetas (crea una nueva) o 1 carpeta (la usa directamente).

Despues de responder, la eleccion se guarda y nunca se vuelve a preguntar.

---

## Estructura en Google Drive

Despues de usar Nebuah, tu Google Drive va a tener esta estructura:

```
Nebuah/
|-- projects/
|   |-- DueDiligenceM&A/
|   |   |-- input/          <- Tus documentos de entrada
|   |   |-- output/         <- Entregables generados por Nebuah
|   |   |-- memory/
|   |       |-- long_term/  <- Aprendizajes del proyecto
|   |
|   |-- ContratoNDA/
|   |   |-- input/
|   |   |-- output/
|   |   |-- memory/
|   |       |-- long_term/
|   |
|   |-- (mas proyectos...)
|
|-- system/
    |-- memory/
        |-- strategies/
            |-- level_1_epics/          <- Patrones de engagement
            |-- level_2_architecture/   <- Estrategias de framework
            |-- level_3_tactical/       <- Tacticas de documentos
            |-- level_4_reactive/       <- Patrones de accion
            |-- _negative_constraints.md <- Lo que NO hacer
            |-- _dream_journal.md       <- Historial de consolidacion
```

### Que se sincroniza y que NO

| Contenido | Se sube a GDrive? | Razon |
|-----------|-------------------|-------|
| Estrategias aprendidas | Si | Son patrones abstractos, no datos sensibles |
| Restricciones negativas | Si | Lecciones de errores, aplicables en cualquier proyecto |
| Journal de suenos | Si | Historial de consolidacion |
| Entregables (output/) | Si | Para acceso compartido |
| Documentos de entrada | Permanecen en GDrive | GDrive es la fuente canonica |
| Trazas de ejecucion | NO | Contienen detalles sensibles del caso |
| Definiciones de agentes | NO | Contienen contexto especifico del caso |
| Memoria de corto plazo | NO | Datos de sesion, sensibles |
| Estrategias semilla (_seeds/) | NO | Son inmutables y locales |

---

## Comandos de Dream (sin cambios)

Los comandos de dream funcionan exactamente igual que antes:

```
/nebuah dream                    -- Consolidacion completa de todas las trazas
/nebuah dream contract review    -- Solo trazas relacionadas con contratos
/nebuah dream L3                 -- Solo trazas de nivel tactico
/nebuah dream --parallel A | B   -- Multiples dreams en paralelo
/nebuah dream status             -- Estado actual de la memoria
```

### Nuevo: Dream sincroniza a GDrive

La unica diferencia es que despues de cada dream, las estrategias nuevas o actualizadas se suben automaticamente a Google Drive. No tenes que hacer nada.

---

## Comandos manuales (para usuarios avanzados)

Si por alguna razon necesitas forzar una sincronizacion manualmente, estos comandos estan disponibles:

| Comando | Que hace |
|---------|----------|
| `/nebuah gdrive pull [proyecto]` | Fuerza la descarga de documentos de entrada |
| `/nebuah gdrive push [proyecto]` | Fuerza la subida de outputs y memorias |
| `/nebuah gdrive sync` | Sincronizacion bidireccional completa de memoria del sistema |
| `/nebuah gdrive status` | Muestra el estado actual de sincronizacion |

**En operacion normal, nunca necesitas usar estos comandos.** Todo es automatico. Estan disponibles como herramientas de diagnostico o para situaciones excepcionales.

---

## Trabajar en multiples maquinas

Uno de los beneficios principales de la integracion con GDrive es la continuidad entre maquinas:

### Escenario: Trabajas en tu laptop y luego en tu desktop

**Laptop:**
```
/nebuah "Redactar contrato NDA para cliente X"
  -> Nebuah crea el proyecto, ejecuta, sube todo a GDrive.
  -> Estrategias aprendidas se sincronizan a GDrive.
```

**Desktop (despues):**
```
/nebuah "Revisar clausulas de indemnizacion para cliente Y"
  -> Step 0: Detecta la carpeta "Nebuah" existente en GDrive (no pregunta).
  -> Step 1: Busca en GDrive, encuentra estrategias del proyecto NDA.
  -> Las aplica al nuevo proyecto automaticamente.
```

La primera vez que ejecutas Nebuah en una maquina nueva, el Step 0 descubre la estructura existente en GDrive y la reutiliza. No crea duplicados.

---

## Compartir con tu equipo

La carpeta `Nebuah/` en Google Drive se puede compartir con otros miembros de tu equipo:

1. En Google Drive, hace clic derecho en la carpeta `Nebuah/`
2. Selecciona "Compartir"
3. Agrega a los miembros de tu equipo

Cuando otro miembro ejecute `/nebuah` por primera vez:
- Step 0 detecta la carpeta compartida "Nebuah"
- La usa directamente (no crea una nueva)
- Tiene acceso a todas las estrategias y entregables existentes

> **Nota:** Las trazas de ejecucion y definiciones de agentes **nunca** se suben a GDrive, asi que cada miembro mantiene su privacidad operativa.

---

## FAQ

### Necesito crear la carpeta "Nebuah" en Google Drive antes de empezar?

No. Nebuah la crea automaticamente la primera vez que ejecutas `/nebuah`.

### Que pasa si ya tengo una carpeta "Nebuah" en Google Drive?

Nebuah la detecta y la usa. No crea una nueva.

### Que pasa si borro el archivo `system/gdrive_registry.json` local?

Nebuah re-ejecuta el bootstrap: busca la carpeta existente en GDrive, descubre los IDs de todas las subcarpetas, y recrea el registro. No se pierde nada.

### Que pasa si borro la carpeta en Google Drive?

Nebuah crea una nueva la proxima vez que ejecutes `/nebuah`. Las estrategias locales siguen intactas en tu maquina.

### Los documentos de entrada se borran de GDrive despues de descargarse?

No. GDrive es la fuente canonica de los inputs. Se descargan a tu maquina local para procesamiento pero el original permanece en GDrive.

### Puedo subir inputs despues de crear el proyecto?

Si. Subi los documentos a `Nebuah/projects/[Proyecto]/input/` en GDrive y ejecuta `/nebuah gdrive pull [proyecto]` para descargarlos, o simplemente ejecuta un nuevo `/nebuah` con el mismo nombre de proyecto.

### Las estrategias de otros proyectos se modifican?

No. El acceso a estrategias de proyectos pasados es de **solo lectura**. Nebuah las lee para informar su plan, pero nunca las modifica.

### Funciona sin internet?

Si. Nebuah trabaja local-first. Si Google Drive no esta disponible, el sistema sigue funcionando con las estrategias locales. La sincronizacion se reanuda cuando vuelva la conexion.

---

## Resumen: Lo que tenes que saber

1. **Usa `/nebuah "objetivo"` como siempre.** Nada cambio en tu flujo de trabajo.
2. **Google Drive se configura solo.** La primera vez, automaticamente.
3. **Tus documentos de entrada pueden estar en GDrive.** Se descargan solos.
4. **Tus entregables se suben a GDrive.** Automaticamente.
5. **Las estrategias se comparten entre proyectos y maquinas.** Via GDrive.
6. **Maximo una pregunta, una sola vez.** Solo si hay ambiguedad con la carpeta raiz.
7. **Comandos manuales de GDrive existen pero no los necesitas.** Son para casos excepcionales.
