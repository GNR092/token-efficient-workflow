---
name: token-efficient-workflow
description: Activa SIEMPRE este skill antes de buscar, explorar, analizar codigo, revisar archivos, entender modulos, generar documentacion, crear DOCX, preparar mensajes de commit o hacer commits. Orquesta code-search, docx-generator y commit-msg con una politica estricta de ahorro de tokens: buscar primero, leer solo fragmentos necesarios y evitar abrir archivos completos salvo que sea indispensable.
---

# Token Efficient Workflow

## Objetivo

Coordinar los skills `code-search`, `docx-generator` y `commit-msg` usando una estrategia de mínimo contexto: buscar primero, leer poco, resumir evidencia y solo abrir archivos completos cuando sea estrictamente necesario.

## Regla principal

Carga este skill como primer paso para cualquier solicitud que incluya intenciones como: busca, buscar, encuentra, localizar, explora, analiza, revisar codigo, entender modulo, documentar, generar docx, commit, mensaje de commit, cambios de git o ahorrar tokens.

Si el skill no aparece en la lista disponible, confirma su disponibilidad y ruta efectiva con `opencode debug skill`; no asumas un directorio fijo, porque depende de la configuración. Si el diagnóstico lo encuentra pero la sesión no lo carga, reinicia la sesión o el servidor.

**Fallback con find-skills**: si el skill sigue sin aparecer, usa `find-skills` si está disponible para localizarlo.

## Tabla de contenidos

- [Activación rápida](#activación-rápida-ejemplos) — Prompts que disparan este skill
- [Regla principal](#regla-principal) — Qué hacer antes de leer archivos completos
- [Qué NO hacer](#qué-no-hacer-primeras-opciones)
- [Skills usados](#skills-usados) — Orden de carga según la tarea
- [Flujo A: análisis de código](#flujo-a-análisis-de-código-con-mínimo-contexto)
- [Flujo B: generar DOCX](#flujo-b-generar-docx-desde-análisis-de-código)
- [Flujo C: generar mensaje de commit](#flujo-c-generar-mensaje-de-commit-con-mínimo-contexto)
- [Flujo D: búsqueda web y caché](#flujo-d-búsqueda-web-y-caché)
- [Troubleshooting express](#troubleshooting-express)
- [Recuperación de code-search](#recuperacion-de-code-search)
- [Skills opcionales recomendados](#skills-opcionales-recomendados)
- [Reglas para evitar gasto innecesario](#reglas-para-evitar-gasto-innecesario)
- [Respuesta final esperada](#respuesta-final-esperada)

## Activación rápida (ejemplos)

Este skill se activa automáticamente cuando el usuario escriba o diga algo como:

```
"busca dónde se valida el rol de admin"
"encuentra el método que aprueba solicitudes"
"explora el módulo de compras"
"analiza cómo funciona la autenticación"
"busca código relacionado con pagos"
"qué archivos tratan sobre requisiciones"
"localiza la función que genera órdenes"
"revisar código del controlador X"
"entender el modelo de usuarios"
"documentar el flujo de approval"
"generar docx del análisis"
"hacer commit de los cambios"
"preparar mensaje de commit"
"qué cambios hay en git"
"ahorrar tokens en esta búsqueda"
```

Antes de leer archivos grandes o explorar carpetas completas, usa `code-search` para ubicar el código relevante.

## Qué NO hacer (primeras opciones)

Evita estas acciones como primera opción — siempre hay una alternativa que gasta menos tokens:

```
❌ cat archivo_grande.php
❌ find . -type f
❌ ls -R
❌ leer directorios completos
❌ abrir muchos archivos completos
❌ copiar archivos completos en la respuesta
❌ pegar resultados largos de búsquedas completas
❌ abrir documentación extensa salvo que sea necesaria
❌ repetir el mismo hallazgo en varias secciones
❌ documentar comportamiento inferido como si fuera confirmado
```

**En su lugar:** carga `code-search` si está disponible y usa búsqueda semántica, referencias y filtros por ruta para obtener resultados compactos.

## Skills opcionales recomendados

Estos skills complementan el flujo principal según el tipo de tarea. Se cargan bajo demanda, no automáticamente; comprueba que cada skill esté instalado antes de cargarlo. Si no está disponible, sigue el flujo normal sin él.

### Por tipo de tarea

| Tarea | Skill a cargar |
|-------|---------------|
| Análisis de código | `code-search` |
| Cambios de código | `lint-format` + `test-runner` + `security-check` |
| Commit / PR | `commit-msg` → `pr-writer` → `changelog-release` |
| Datos / Infraestructura | `postgresqldb` + `migration-safety` + `ci-triage` |
| Preguntas generales / web | Usa la herramienta MCP `searchmcp` si está disponible |
| Diseño de interfaces | `interface-design` |

### Integración con el flujo principal

```
Búsqueda/análisis:
  token-efficient-workflow → code-search

Si hay cambios de código:
  token-efficient-workflow → code-search → lint-format → test-runner → security-check

Si hay entrega:
  token-efficient-workflow → code-search → commit-msg → pr-writer → changelog-release

Si toca datos/infra:
  token-efficient-workflow → code-search → postgresqldb → migration-safety → ci-triage
```

### Descripción breve de cada skill opcional

- **`lint-format`**: reglas de calidad automáticas (`ruff/eslint/prettier/php-cs-fixer`) con política "arregla solo lo tocado".
- **`test-runner`**: estandariza qué pruebas correr por stack (`pytest`, `npm test`, `phpunit`), estrategia rápida→completa, y cómo reportar fallos.
- **`security-check`**: escaneo de secretos, dependencias vulnerables y revisión básica de riesgos antes de commit/PR.
- **`pr-writer`**: genera descripción de PR con resumen, impacto, riesgos, checklist de validación y notas de rollback.
- **`changelog-release`**: crea notas de versión y clasificación semántica de cambios.
- **`postgresqldb`**: guía para conectar Node.js a PostgreSQL en modo solo lectura, consultas seguras y patrón de uso del toolkit MCP.
- **`migration-safety`**: checklist para migraciones seguras (backups, compatibilidad, pasos de rollback).
- **`ci-triage`**: guía para diagnosticar fallos de CI rápido (tests flaky, lint, permisos, paths).
- **`interface-design`**: dashboards, admin panels, apps, tools y productos interactivos.

### Reglas de activación

Carga un skill opcional cuando detectes intención explícita:

La disponibilidad o carga de `test-runner` no autoriza a ejecutar pruebas: hazlo solo si el usuario lo pide o las instrucciones aplicables lo requieren.

- `lint-format`: usuario menciona "lint", "formatear", "calidad", "estilo"
- `test-runner`: usuario menciona "test", "prueba", "correr tests", "pytest"
- `security-check`: usuario menciona "seguridad", "secretos", "vulnerabilidad", "scan"
- `pr-writer`: usuario menciona "PR", "pull request", "descripción de cambios"
- `changelog-release`: usuario menciona "changelog", "notas de versión", "release"
- `postgresqldb`: usuario menciona "PostgreSQL", "postgres", "base de datos", "DB", "consulta SQL", "MCP postgres"
- `migration-safety`: usuario menciona "migración", "DB", "database", "rollback"
- `ci-triage`: usuario menciona "CI falló", "tests rotos", "pipeline"
- `interface-design`: usuario menciona "dashboard", "UI", "interfaz", "panel"

## Skills usados

Debes cargar los skills en este orden según la tarea:

1. `code-search` para ubicar archivos, símbolos, referencias y comportamiento.
2. `postgresqldb` cuando se requiera conexión a PostgreSQL o consultas SQL.
3. `docx-generator` solo si el usuario pidió generar un `.docx`.
4. `commit-msg` solo si el usuario pidió mensaje de commit o hacer commit.

Para búsquedas web, usa la herramienta MCP `searchmcp` si está disponible; no es un skill que debas cargar.

## Flujo recomendado

Usa este orden para ahorrar tokens y evitar cargar contexto innecesario:

```text
token-efficient-workflow -> code-search
  -> docx-generator (solo para DOCX)
  -> commit-msg (solo para mensajes de commit)
```

`code-search` debe ser el primer paso para ubicar archivos, símbolos y referencias. `docx-generator` y `commit-msg` deben consumir un resumen compacto de evidencia, no el repositorio completo.

## Compatibilidad de sistema operativo

Antes de ejecutar comandos de shell, detectar el sistema operativo y elegir la sintaxis correcta. No mezcles PowerShell con Bash en una misma instruccion.

Deteccion recomendada:

```text
Si el entorno reporta Platform: win32, usa Windows/PowerShell.
Si el entorno reporta Linux, macOS, darwin o una shell POSIX, usa Bash.
```

Fallback desde terminal:

```powershell
$IsWindows
$env:OS
```

```bash
uname -s
```

Rutas base por sistema:

- Windows: usar `$env:USERPROFILE` en PowerShell y `%USERPROFILE%` solo en texto/documentacion.
- Linux/macOS: usar `$HOME`.
- Para rutas de `codesearch` en respuestas, preferir rutas relativas al repositorio cuando sea posible.
- Para `filter_path`, usar separadores `/` porque funcionan mejor como filtros portables: `Documents/Proyectos/ticket/`.

## Política de ahorro de tokens

### 1. Preflight una sola vez por sesión

Valida `code-search` una vez al inicio de la sesión o cuando cambie el repositorio.

Usa la herramienta de estado disponible en `code-search` y confirma que el índice esté listo y tenga contenido. Los nombres de las herramientas pueden variar entre versiones.

Si la herramienta devuelve estos campos, el estado esperado es:

- `indexed=true`
- `status="ready"`
- `total_chunks > 0`

Comandos sugeridos:

```bash
codesearch --version
codesearch doctor
codesearch index --list
```

Si MCP está disponible, prefiere MCP sobre CLI.

Si el MCP no está disponible pero la CLI funciona, usa la CLI como fallback y señala esa limitación.

### 1.1 Exclusiones obligatorias

Ignora por completo estas carpetas durante busquedas, lecturas, indexacion, resumenes y generacion de documentacion:

- `.git/`
- `__pycache__/`
- `.pytest_cache/`
- `.mypy_cache/`
- `.ruff_cache/`
- `node_modules/`
- `vendor/`
- `dist/`
- `build/`
- `coverage/`
- `.codesearch.db/`

Si algun resultado aparece dentro de una ruta excluida, descartalo y repite la busqueda con `filter_path` mas especifico o filtros de exclusion.

Ejemplo de fallback textual solo cuando `code-search` no pueda resolverlo:

```bash
rg "texto_a_buscar" \
  --glob '!**/.git/**' \
  --glob '!**/__pycache__/**' \
  --glob '!**/node_modules/**' \
  --glob '!**/vendor/**' \
  --glob '!**/dist/**' \
  --glob '!**/build/**' \
  --glob '!**/coverage/**' \
  --glob '!**/.codesearch.db/**'
```

### 2. Siempre limitar resultados

Usa límites pequeños por defecto:

- búsqueda amplia: `limit=5` a `limit=8`
- búsqueda refinada: `limit=3` a `limit=5`
- referencias de símbolo: solo las más relevantes

Evita pedir 20, 50 o más resultados salvo que el usuario lo solicite.

### 3. Siempre usar filtro de ruta

Si conoces la carpeta del proyecto o módulo, usa `filter_path`.

Ejemplos:

```text
filter_path="app/Controllers/"
filter_path="app/Models/"
filter_path="public/js/"
```

Si el índice es global o padre, el `filter_path` es obligatorio.

Si el índice es global o está en un directorio padre, nunca busques sin filtro porque puede mezclar resultados de varios proyectos.

Ejemplos para este entorno:

```text
filter_path="Documents/Proyectos/"
filter_path="Documents/Proyectos/MB_Cyn/"
filter_path="Documents/Proyectos/skills/"
filter_path="Documents/Proyectos/Residentes/"
```

### 4. Buscar por intención, no por carpetas

Prefiere consultas semánticas de intención:

```text
"dónde se aprueba una solicitud de compra"
"controlador que genera orden de compra"
"función que registra recepción de material"
"validación de roles para pagos"
"dónde se cambia el estado de una requisición"
```

Evita leer archivos solo porque el nombre parece relevante.

### 5. Leer solo lo necesario

Después de `code-search`, lee únicamente:

- el fragmento necesario
- la función específica
- el método del controlador
- el modelo relacionado
- la ruta o configuración puntual

No leas archivos completos si el fragmento de búsqueda ya es suficiente.

### 6. Crear un paquete compacto de evidencia

Antes de pasar a `docx-generator` o `commit-msg`, resume la evidencia en este formato:

```md
## Evidencia compacta

### Hallazgo 1
- Archivo: `ruta/al/archivo.php`
- Símbolo/método: `Nombre::metodo`
- Qué hace: resumen de 1 a 2 líneas
- Evidencia: fragmento corto o líneas relevantes
- Confianza: alta/media/baja

### Pendientes
- Punto que falta validar, si aplica.
```

Ese resumen compacto es el contexto que deben consumir los demás skills.

## Flujo A: análisis de código con mínimo contexto

1. Carga `code-search`.
2. Ejecuta preflight si no se ha hecho en la sesión.
3. Haz una búsqueda amplia con límite bajo.
4. Refina por ruta, símbolo o término de dominio.
5. Usa referencias si aparece una clase, función o método clave.
6. Lee solo los fragmentos necesarios.
7. Entrega hallazgos compactos.

Si `code-search` no esta sano, no avances con exploracion amplia. Primero corrige el indice o usa una busqueda textual puntual con `grep`/`Glob` solo como fallback temporal.

## Troubleshooting express

Errores comunes y solución inmediata (antes de ir a recuperación larga):

| Error | Solución |
|-------|----------|
| `Index not built` / `Vector store empty` | `codesearch index` |
| `LockBusy` / `Failed to acquire index lock` | Inspeccionar procesos y pedir autorización antes de detener uno |
| `FileAlreadyExists("*.del")` | `codesearch index --force` o reindexar |
| `Multiple git repositories found` | Ejecutar `codesearch index` dentro de cada repositorio por separado |
| MCP no responde pero CLI sí | Usar CLI como fallback temporal, reportar que MCP necesita reiniciarse |
| El estado MCP no está disponible | Verificar con `codesearch doctor` y `codesearch index --list` si la CLI está instalada |

## Recuperacion de code-search

Usa esta seccion cuando las busquedas fallen con errores como `Index not built`, `Vector store empty`, `LockBusy`, `Failed to acquire index lock` o `FileAlreadyExists("*.del")`.

Si `codesearch index` se ejecuta desde una carpeta padre y falla con `Multiple git repositories found in subdirectories`, no intentes forzar un indice unico. Primero detecta el sistema operativo, luego ejecuta el bloque correspondiente. `codesearch` debe ejecutarse dentro de cada repositorio Git por separado y esperar a que termine cada indexacion antes de iniciar la siguiente.

Las listas siguientes son ejemplos fijos, no autodetectan todos los repositorios. Si el usuario pide indexarlos todos, confirma primero la carpeta y los repositorios que entran en el alcance.

Windows/PowerShell:

```powershell
$base = Join-Path $env:USERPROFILE "Documents\Proyectos"
$repos = @(
  (Join-Path $base "ticket"),
  (Join-Path $base "Timbrado"),
  (Join-Path $base "Updaters")
)

foreach ($repo in $repos) {
  if (Test-Path (Join-Path $repo ".git")) {
    Write-Host "Indexando $repo"
    codesearch index $repo
    if ($LASTEXITCODE -ne 0) { throw "Fallo la indexacion de $repo" }
  }
}
```

Linux/macOS/Bash:

```bash
base="$HOME/Documents/Proyectos"
repos=(
  "$base/ticket"
  "$base/Timbrado"
  "$base/Updaters"
)

for repo in "${repos[@]}"; do
  if [ -d "$repo/.git" ]; then
    printf 'Indexando %s\n' "$repo"
    codesearch index "$repo" || exit 1
  fi
done
```

Despues abre o sirve OpenCode desde el repositorio que se va a analizar, no desde la carpeta padre `Proyectos`. Usa el bloque que corresponda al sistema operativo detectado.

Windows/PowerShell:

```powershell
cd (Join-Path $env:USERPROFILE "Documents\Proyectos\ticket")
opencode serve
```

Linux/macOS/Bash:

```bash
cd "$HOME/Documents/Proyectos/ticket"
opencode serve
```

Si el usuario pide indexar varios repositorios, usa solo las rutas confirmadas y espera a que termine cada índice. No respondas como completado hasta validar cada repositorio con `codesearch doctor` o una búsqueda de prueba desde su carpeta.

1. Diagnostica el estado:

```bash
codesearch doctor
codesearch index --list
```

2. Si hay procesos `codesearch mcp` duplicados bloqueando el indice, identificalos antes de detenerlos:

```powershell
Get-CimInstance Win32_Process | Where-Object { $_.Name -match 'codesearch|opencode|node' } | Select-Object ProcessId,Name,CommandLine
```

3. Si el bloqueo es confirmado y no hay indexación en curso, identifica solo el proceso que lo causa. Pide autorización antes de detenerlo o reindexar, porque puede interrumpir trabajo activo:

```powershell
Get-CimInstance Win32_Process | Where-Object { $_.Name -eq 'codesearch.exe' -and $_.CommandLine -eq 'codesearch mcp' } | ForEach-Object { Stop-Process -Id $_.ProcessId -Force }
codesearch index
```

4. Verifica que quede listo desde el repositorio que vas a analizar:

```bash
codesearch doctor
codesearch search "consulta de prueba" --max-results 5 --scores
```

Si el índice es global, añade un `--filter-path` que corresponda a la ruta real del repositorio.

5. Criterio de exito:

- `Vector index searchable`
- `total_chunks > 0`
- busqueda MCP devuelve resultados
- busqueda CLI devuelve resultados

## Flujo B: generar DOCX desde análisis de código

Usa este flujo si el usuario pide documentación `.docx`.

1. Ejecuta primero el Flujo A.
2. Construye un Markdown temporal con evidencia compacta.
3. Carga `docx-generator`.
4. Sigue las instrucciones del skill `docx-generator`; no asumas que su script está dentro del proyecto analizado.
5. No incluyas en el documento archivos no validados.

## Flujo C: generar mensaje de commit con mínimo contexto

Usa este flujo si el usuario pide mensaje de commit.

1. Carga `commit-msg`.
2. Revisa primero:

```bash
git status --short
git log -3 --oneline
git diff --name-only
```

3. Solo si es necesario, revisa diffs puntuales por archivo.
4. Si el cambio afecta código y no queda claro, usa `code-search` para entender el módulo sin leer de más.
5. Genera un mensaje en español, sin prefijos tipo `feat:` o `fix:`.

### Ejecutar un commit

Este flujo solo prepara el mensaje. Si el usuario pide ejecutar el commit, usa el agente o proceso Git disponible y sigue sus reglas de inspección y confirmación; no stages ni crees commits desde este flujo.

### Archivos no relacionados

Si existen archivos en el working directory que no pertenecen al commit (ej: build scripts, spec files, archivos de configuracion de empaquetado), no incluirlos automáticamente. Solo hacer commit de lo que el usuario confirme explícitamente.

## Flujo D: búsqueda web y caché

Usa este flujo cuando quieras reutilizar resultados web guardados por `searchmcp`. La caché puede evitar búsquedas repetidas, pero no implica que `codesearch` pueda acceder a esos resultados.

### Principio

**Busca una vez y reutiliza los resultados guardados cuando estén disponibles.**

### Pasos

1. Usa `searchmcp` para consultar la web. Si quieres conservar los resultados, usa `search_and_save` si está disponible y toma como referencia la ruta que devuelve.

### Recomendación

- Para resultados guardados, usa las herramientas de caché de `searchmcp` si están disponibles.
- No asumas que los resultados se guardaron en `.search/` ni que `codesearch` puede encontrarlos o indexarlos.

## Reglas para evitar gasto innecesario

- No pegues resultados largos de búsquedas completas.
- No copies archivos completos en la respuesta.
- No abras documentación extensa salvo que sea necesaria.
- No repitas el mismo hallazgo en varias secciones.
- No documentes comportamiento inferido como si fuera confirmado.
- Marca como pendiente cualquier punto no validado en código.
- Para reportes, usa resumen técnico compacto y evidencia mínima suficiente.

## Criterio de escalamiento

Solo aumenta el contexto cuando ocurra una de estas condiciones:

- hay resultados contradictorios
- el símbolo aparece en muchos lugares
- se requiere modificar código
- se requiere generar documentación formal con trazabilidad
- el usuario pide explicación detallada

## Múltiples preguntas en un solo mensaje

Cuando el usuario haga varias preguntas en un mismo mensaje:

1. **Identifica la intención principal** — Responde primero la más relevante o la que defina el alcance.
2. **Divide las demás en sub-tareas** — Si son 3+ preguntas relacionadas, enumera las pendientes y resuelve las que tengan alcance claro.
3. **Usa el formato compacto** — Cada hallazgo en 2-3 líneas máximo.
4. **Sugiere siguiente paso** — Pregunta solo si queda una decisión pendiente.

**Ejemplo:**

> "busca el modelo de Usuario, dónde se usa en Controllers y si hay tests"

```
✅ Respuesta:
1. Modelo: app/Models/Usuario.php — maneja CRUD de usuarios
2. Controllers: 3 referencias en app/Controllers/ (Login, Perfil, Admin)
3. Tests: no hay tests directos para ese modelo

Siguiente: ¿quieres que revise los tests del Login controller o profundizo en el modelo?
```

## Contexto insuficiente: cuándo detenerse

Usa este criterio antes de forzar una búsqueda o inventar una respuesta:

**Detente y pide más información cuando:**

- la consulta es demasiado vaga (`busca algo de pagos`)
- el símbolo no aparece en el índice y no hay fallback claro
- los resultados son contradictorios sin poder discriminarlos
- el usuario pide modificar código pero no hay evidencia clara de dónde

**Qué reportar en ese caso:**

```text
❌ No tengo suficiente contexto para responder con confianza.
Necesito:
1. Más detalles sobre qué buscas (módulo, funcionalidad, archivo)
2. O el nombre exacto de un símbolo/método/controlador

Alternativa: puedo buscar con términos más amplios, pero los resultados pueden ser imprecisos.
```

## Respuesta final esperada

Entrega siempre:

1. Qué se buscó.
2. Qué se encontró.
3. Archivos o símbolos clave.
4. Qué se evitó leer para ahorrar tokens.
5. Siguiente acción sugerida, si aplica.

Si se generó DOCX, incluye la ruta del archivo.
Si se generó commit, incluye el mensaje sugerido.
