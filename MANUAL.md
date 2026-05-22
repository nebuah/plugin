# Manual de Configuracion — Nebuah Plugin

Guia completa para instalar el plugin de Nebuah en Claude Code y configurar el conector de Google Drive que Nebuah necesita para funcionar correctamente.

---

## Tabla de Contenidos

- [Requisitos Previos](#requisitos-previos)
- [Parte 1: Instalar o Actualizar el Plugin de Nebuah](#parte-1-instalar-o-actualizar-el-plugin-de-nebuah)
  - [Instalacion desde GitHub](#instalacion-desde-github)
  - [Actualizacion del Plugin](#actualizacion-del-plugin)
  - [Verificar la Instalacion](#verificar-la-instalacion)
  - [Desinstalar el Plugin](#desinstalar-el-plugin)
- [Parte 2: Configurar el Conector de Google Drive](#parte-2-configurar-el-conector-de-google-drive)
  - [Paso 1: Crear Credenciales OAuth 2.0](#paso-1-crear-credenciales-oauth-20)
  - [Paso 2: Agregar el Conector en Claude](#paso-2-agregar-el-conector-en-claude)
  - [Paso 3: Verificar la Conexion](#paso-3-verificar-la-conexion)
- [Parte 3: Primera Ejecucion de Nebuah](#parte-3-primera-ejecucion-de-nebuah)
- [Resolucion de Problemas](#resolucion-de-problemas)
- [Referencias](#referencias)

---

## Requisitos Previos

| Requisito | Detalle |
|-----------|---------|
| **Suscripcion Claude** | Pro, Max, Team o Enterprise |
| **Claude Desktop** | Descargar desde [claude.com/download](https://claude.com/download) (macOS o Windows) |
| **Cuenta de Google** | Necesaria para el conector de Google Drive |
| **Acceso a Google Cloud Console** | Para crear las credenciales OAuth 2.0 |

---

## Parte 1: Instalar o Actualizar el Plugin de Nebuah

### Instalacion desde GitHub

El repositorio del plugin de Nebuah esta en GitHub: `https://github.com/nebuah/plugin`

Hay dos formas de instalarlo. Ambas se ejecutan **dentro de Claude Code** (la pestaña Code de Claude Desktop).

#### Opcion A: Pedirle a Claude que lo instale (Recomendado)

Abre una sesion de Claude Code y escribe:

```
Instala el plugin de Nebuah desde el marketplace de GitHub nebuah/plugin
```

Claude ejecutara automaticamente los comandos necesarios para agregar el marketplace e instalar el plugin.

Si prefieres hacerlo tu mismo con comandos explicitos, sigue la Opcion B.

#### Opcion B: Comandos manuales paso a paso

**Paso 1 — Agregar el marketplace:**

Dentro de Claude Code, ejecuta:

```
/plugin marketplace add nebuah/plugin
```

Esto registra el repositorio `https://github.com/nebuah/plugin` como un marketplace de plugins. Claude Code descarga el catalogo y hace que los plugins del repositorio esten disponibles para instalar.

**Paso 2 — Instalar el plugin:**

```
/plugin install nebuah@nebuah-plugin
```

El nombre del marketplace se forma reemplazando `/` por `-` en el formato `owner/repo`, asi que `nebuah/plugin` se convierte en `nebuah-plugin`.

Claude Code te preguntara el alcance de la instalacion:

| Alcance | Descripcion |
|---------|-------------|
| **User** (recomendado) | Se instala para ti en todos los proyectos |
| **Project** | Se instala para todos los colaboradores del repositorio actual |
| **Local** | Se instala solo para ti en el repositorio actual |

**Paso 3 — Activar el plugin:**

Despues de instalar, ejecuta:

```
/reload-plugins
```

Esto recarga todos los plugins activos sin necesidad de reiniciar Claude Code.

### Actualizacion del Plugin

Cuando hay una nueva version del plugin en GitHub, puedes actualizar de dos formas:

#### Opcion A: Pedirle a Claude

```
Actualiza el plugin de Nebuah a la ultima version desde nebuah/plugin
```

#### Opcion B: Comandos manuales

```
/plugin marketplace update nebuah-plugin
```

Esto descarga la ultima version del catalogo. Si hay actualizaciones disponibles para plugins instalados, Claude Code te notificara. Luego ejecuta:

```
/reload-plugins
```

#### Actualizaciones automaticas

Puedes habilitar las actualizaciones automaticas para que Claude Code actualice el marketplace y sus plugins cada vez que inicia una sesion:

1. Ejecuta `/plugin` para abrir el gestor de plugins
2. Ve a la pestaña **Marketplaces**
3. Selecciona `nebuah-plugin`
4. Selecciona **Enable auto-update**

### Verificar la Instalacion

Despues de instalar, verifica que Nebuah esta activo:

```
/nebuah List all available seed strategies
```

Claude deberia reportar las 6 estrategias de arranque (seeds) que cubren: intake de casos, investigacion legal, redaccion de contratos, cumplimiento regulatorio, revision de documentos y analisis de jurisprudencia.

Tambien puedes verificar que el comando esta disponible con:

```
/plugin
```

En la pestaña **Installed**, deberias ver `nebuah` listado como activo.

### Desinstalar el Plugin

Si necesitas desinstalar el plugin:

```
/plugin uninstall nebuah@nebuah-plugin
```

Para remover el marketplace completamente:

```
/plugin marketplace remove nebuah-plugin
```

---

## Parte 2: Configurar el Conector de Google Drive

Nebuah **requiere** Google Drive para funcionar correctamente. Todos los archivos producidos (outputs, estrategias, agentes, memorias) se guardan automaticamente en Google Drive. Sin el conector, Nebuah funciona pero pierde la capacidad de persistir archivos en la nube, sincronizar entre sesiones y reutilizar agentes entre proyectos.

### Paso 1: Crear Credenciales OAuth 2.0

1. Ve a la **Google Cloud Console**: [console.cloud.google.com](https://console.cloud.google.com)

2. Crea un proyecto nuevo o selecciona uno existente

3. Habilita la **Google Drive API**:
   - Ve a **APIs & Services > Library**
   - Busca "Google Drive API"
   - Haz clic en **Enable**

4. Configura la pantalla de consentimiento OAuth:
   - Ve a **APIs & Services > OAuth consent screen** (o **Google Auth Platform > Branding**)
   - Selecciona **External** (o **Internal** si usas Google Workspace)
   - Completa los campos obligatorios (nombre de la app, email de soporte)
   - En **Scopes**, agrega: `https://www.googleapis.com/auth/drive`
   - Guarda

5. Crea las credenciales:
   - Ve a **APIs & Services > Credentials** (o **Google Auth Platform > Clients**)
   - Haz clic en **Create Credentials > OAuth client ID**
   - Tipo de aplicacion: **Web application**
   - Nombre: el que prefieras (ej: "Claude Drive Connector")
   - En **Authorized redirect URIs**, agrega exactamente esta URL:
     ```
     https://claude.ai/api/mcp/auth_callback
     ```
   - Haz clic en **Create**

6. **Guarda el Client ID y el Client Secret** — los necesitaras en el siguiente paso

### Paso 2: Agregar el Conector en Claude

1. Abre **Claude.ai** en el navegador o **Claude Desktop**

2. Ve a **Settings** (icono de engranaje):
   - En Claude.ai: Settings > Connectors
   - En Claude Desktop: Settings > Connectors
   - Si eres admin de un plan Team/Enterprise: Admin Settings > Connectors

3. Haz clic en **Add custom connector**

4. Completa los campos:

   | Campo | Valor |
   |-------|-------|
   | **Server name** | `Google Drive` |
   | **Remote MCP server URL** | `https://drivemcp.googleapis.com/mcp/v1` |

5. Expande **Advanced settings** e ingresa:
   - **Client ID**: el que obtuviste en el Paso 1
   - **Client Secret**: el que obtuviste en el Paso 1

6. Haz clic en **Add**

7. Claude te pedira autorizar el acceso a tu Google Drive. Sigue el flujo de autorizacion de Google y acepta los permisos.

### Paso 3: Verificar la Conexion

Una vez configurado el conector, verifica que funciona dentro de Claude Code:

```
Lista mis 5 archivos mas recientes de Google Drive
```

Claude deberia usar las herramientas MCP de Google Drive (`list_recent_files`) y mostrar tus archivos recientes. Si esto funciona, el conector esta correctamente configurado.

Tambien puedes verificarlo de forma mas tecnica:

```
Busca en Google Drive una carpeta llamada "Nebuah"
```

Si la carpeta no existe aun, Nebuah la creara automaticamente en la primera ejecucion.

---

## Parte 3: Primera Ejecucion de Nebuah

Con el plugin instalado y el conector de Google Drive configurado, Nebuah esta listo. En la primera ejecucion, el sistema realiza un **auto-bootstrap** automatico:

1. **Detecta el conector de GDrive**: Nebuah busca dinamicamente las herramientas MCP de Google Drive disponibles (no asume un prefijo fijo)

2. **Crea la estructura de carpetas en Google Drive**:
   ```
   Nebuah/
   ├── projects/
   └── system/
       └── memory/
           └── strategies/
               ├── level_1_epics/
               ├── level_2_architecture/
               ├── level_3_tactical/
               └── level_4_reactive/
   ```

3. **Escribe el registro local** (`system/gdrive_registry.json`) con los IDs de todas las carpetas creadas

Este proceso es completamente automatico. La unica situacion en la que Nebuah te hara una pregunta es si encuentra **multiples carpetas** llamadas "Nebuah" en tu Google Drive (para desambiguar cual usar).

### Probar el sistema completo

Ejecuta un objetivo simple para verificar que todo funciona de punta a punta:

```
/nebuah Analiza las clausulas de confidencialidad en el derecho contractual argentino
```

Nebuah deberia:
1. Cargar estrategias y restricciones de la memoria
2. Buscar estrategias previas en Google Drive (cross-project)
3. Crear la estructura del proyecto local y en Google Drive
4. Crear agentes especializados (minimo 3)
5. Ejecutar el plan delegando a los agentes
6. Producir el output localmente
7. Consolidar aprendizajes (dream cycles)
8. **Subir todo a Google Drive** (agentes, outputs, memorias, estrategias)
9. Reportar el resultado incluyendo el estado de GDrive

El reporte final debe incluir una seccion **Google Drive Sync** confirmando los archivos subidos.

---

## Resolucion de Problemas

### El plugin no se encuentra al instalar

```
/plugin marketplace update nebuah-plugin
```

Si eso no funciona, remueve y vuelve a agregar el marketplace:

```
/plugin marketplace remove nebuah-plugin
/plugin marketplace add nebuah/plugin
/plugin install nebuah@nebuah-plugin
```

### El comando /nebuah no aparece

Despues de instalar el plugin, ejecuta `/reload-plugins`. Si el problema persiste, reinicia Claude Code completamente.

### Google Drive no conecta

1. Verifica que el conector aparece en **Settings > Connectors** como activo
2. Verifica que el redirect URI es exactamente `https://claude.ai/api/mcp/auth_callback`
3. Revisa que la Google Drive API esta habilitada en tu proyecto de Google Cloud
4. Intenta remover y re-agregar el conector en Claude

### Nebuah no sube archivos a Google Drive

1. Verifica que el conector esta activo pidiendo a Claude: `Lista mis archivos recientes de Google Drive`
2. Si la herramienta no esta disponible, revisa la configuracion del conector (Parte 2)
3. Si `system/gdrive_registry.json` no existe, Nebuah deberia crearlo automaticamente. Si no lo hace, ejecuta:
   ```
   /nebuah gdrive sync
   ```

### Los archivos en Google Drive pierden el formato YAML

Esto ocurre cuando Google auto-convierte archivos de texto a Google Docs. Nebuah esta configurado para usar `disableConversionToGoogleType: true` en todas las subidas. Si ves archivos corruptos en GDrive, puede que estes usando una version anterior del plugin. Actualiza con:

```
/plugin marketplace update nebuah-plugin
/reload-plugins
```

---

## Referencias

- **Repositorio del plugin**: [github.com/nebuah/plugin](https://github.com/nebuah/plugin)
- **Documentacion oficial de plugins de Claude Code**: [code.claude.com/docs/en/discover-plugins](https://code.claude.com/docs/en/discover-plugins)
- **Configuracion del servidor MCP de Google Drive**: [developers.google.com/workspace/drive/api/guides/configure-mcp-server](https://developers.google.com/workspace/drive/api/guides/configure-mcp-server)
- **Google Cloud Console**: [console.cloud.google.com](https://console.cloud.google.com)
- **Descargar Claude Desktop**: [claude.com/download](https://claude.com/download)

---

Nebuah Labs
