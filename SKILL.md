---
name: token-efficient-workflow
description: Activa SIEMPRE este skill antes de buscar, explorar, analizar codigo, revisar archivos, entender modulos, generar documentacion, crear DOCX, preparar mensajes de commit o hacer commits. Orquesta code-search, docx-generator y commit-msg con una politica estricta de ahorro de tokens: buscar primero, leer solo fragmentos necesarios y evitar abrir archivos completos salvo que sea indispensable.
---

# Token Efficient Workflow

## Objetivo

Coordinar los skills `code-search`, `docx-generator` y `commit-msg` usando una estrategia de mínimo contexto: buscar primero, leer poco, resumir evidencia y solo abrir archivos completos cuando sea estrictamente necesario.

## Regla principal

Carga este skill como primer paso para cualquier solicitud que incluya intenciones como: busca, buscar, encuentra, localizar, explora, analiza, revisar codigo, entender modulo, documentar, generar docx, commit, mensaje de commit, cambios de git o ahorrar tokens.

Si el skill no aparece en la lista disponible, continua con `code-search` y reporta el problema como cache/catalogo de skills. Verifica despues con `opencode debug skill` y confirma que exista en esta ruta de OpenCode:

- Windows: `%USERPROFILE%\.agents\skills\token-efficient-workflow\SKILL.md`
- Linux/macOS: `$HOME/.agents/skills/token-efficient-workflow/SKILL.md`

OpenCode puede cachear el catalogo al iniciar la sesion. Si `opencode debug skill` lo muestra pero `skill({ name: "token-efficient-workflow" })` falla, reiniciar la sesion o servidor de OpenCode es la solucion esperada.

Antes de leer archivos grandes o explorar carpetas completas, usa `code-search` para ubicar el código relevante.

No hagas esto como primera opción:

```bash
cat archivo_grande.php
find . -type f
ls -R
leer directorios completos
abrir muchos archivos completos
```

Haz esto primero:

```text
skill({ name: "code-search" })
```

Después usa búsqueda semántica, referencias y filtros por ruta para obtener resultados compactos.

## Skills usados

Debes cargar los skills en este orden según la tarea:

1. `code-search` para ubicar archivos, símbolos, referencias y comportamiento.
2. `docx-generator` solo si el usuario pidió generar un `.docx`.
3. `commit-msg` solo si el usuario pidió mensaje de commit o hacer commit.

## Flujo recomendado

Usa este orden para ahorrar tokens y evitar cargar contexto innecesario:

```text
token-efficient-workflow -> code-search -> docx-generator -> commit-msg
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

Preflight MCP recomendado:

```text
codesearch_index_status()
codesearch_find_databases()
```

El estado esperado es:

- `indexed=true`
- `status="ready"`
- `total_chunks > 0`
- `dimensions=384` para `minilm-l6-q`

Comandos sugeridos:

```bash
codesearch --version
codesearch doctor
codesearch index --list
```

Si MCP está disponible, prefiere MCP sobre CLI.

Si `codesearch_index_status()` falla pero CLI funciona, usa CLI como fallback y reporta que el MCP necesita reiniciarse.

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

Cuando `codesearch_find_databases()` indique que la base esta en un directorio padre o global, nunca busques sin filtro porque puede mezclar resultados de varios proyectos.

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

## Recuperacion de code-search

Usa esta seccion cuando las busquedas fallen con errores como `Index not built`, `Vector store empty`, `LockBusy`, `Failed to acquire index lock` o `FileAlreadyExists("*.del")`.

Si `codesearch index` se ejecuta desde una carpeta padre y falla con `Multiple git repositories found in subdirectories`, no intentes forzar un indice unico. Primero detecta el sistema operativo, luego ejecuta el bloque correspondiente. `codesearch` debe ejecutarse dentro de cada repositorio Git por separado y esperar a que termine cada indexacion antes de iniciar la siguiente.

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

Si el usuario pide indexar todos los repositorios, ejecuta el patron anterior y espera a que finalice. No respondas como completado hasta validar cada repositorio con `codesearch doctor` o una busqueda de prueba desde su carpeta.

1. Diagnostica el estado:

```bash
codesearch doctor
codesearch index --list
```

2. Si hay procesos `codesearch mcp` duplicados bloqueando el indice, identificalos antes de detenerlos:

```powershell
Get-CimInstance Win32_Process | Where-Object { $_.Name -match 'codesearch|opencode|node' } | Select-Object ProcessId,Name,CommandLine
```

3. Si el bloqueo es confirmado y no hay indexacion intencional en curso, detener solo procesos `codesearch.exe` con `codesearch mcp` y reindexar:

```powershell
Get-CimInstance Win32_Process | Where-Object { $_.Name -eq 'codesearch.exe' -and $_.CommandLine -eq 'codesearch mcp' } | ForEach-Object { Stop-Process -Id $_.ProcessId -Force }
codesearch index
```

4. Verifica que quede listo:

```bash
codesearch doctor
codesearch search "consulta de prueba" --filter-path "Documents/Proyectos/" --max-results 5 --scores
```

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
4. Usa `scripts/docx_skill.py` para generar el documento.
5. No incluyas en el documento archivos no validados.

Comando base:

```bash
python scripts/docx_skill.py --output reporte.docx --title "Reporte técnico" --input reporte.md --lang es-ES
```

## Flujo C: generar mensaje de commit con mínimo contexto

Usa este flujo si el usuario pide mensaje de commit.

1. Carga `commit-msg`.
2. Revisa primero:

```bash
git status --short
git diff --cached --name-only
git diff --cached --stat
```

3. Solo si el stat no basta, revisa diffs puntuales por archivo.
4. Si el cambio afecta código y no queda claro, usa `code-search` para entender el módulo sin leer de más.
5. Genera un mensaje en español, sin prefijos tipo `feat:` o `fix:`.

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

## Respuesta final esperada

Entrega siempre:

1. Qué se buscó.
2. Qué se encontró.
3. Archivos o símbolos clave.
4. Qué se evitó leer para ahorrar tokens.
5. Siguiente acción sugerida.

Si se generó DOCX, incluye la ruta del archivo.
Si se generó commit, incluye el mensaje sugerido.
