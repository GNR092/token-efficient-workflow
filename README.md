# Token Efficient Workflow

Skill orquestador que activa `code-search` primero, ahorra tokens y aplica política de mínimo contexto en todas las búsquedas, análisis y documentación.

## Uso

Carga el skill:

```
skill({ name: "token-efficient-workflow" })
```

Se activa automáticamente cuando el usuario pide: **buscar**, **analizar**, **explorar**, **revisar código**, **documentar**, **generar DOCX**, **hacer commit** o **ahorrar tokens**.

## Flujo principal

```
token-efficient-workflow → code-search → docx-generator / commit-msg
```

1. **code-search** primero para ubicar código, símbolos y referencias.
2. **docx-generator** solo si pide `.docx`.
3. **commit-msg** solo si pide mensaje de commit.

## Reglas clave

- No leer archivos completos como primer paso.
- Siempre usar `filter_path` si el índice es global o padre.
- Limitar resultados: 5–8 en exploración, 3–5 en refinamiento.
- Descartar rutas excluidas: `.git/`, `node_modules/`, `vendor/`, `dist/`, `build/`, `coverage/`, `__pycache__/`, `.codesearch.db/`.

## Detección de sistema operativo

- **Windows** (`Platform: win32`): usar PowerShell, `$env:USERPROFILE`.
- **Linux/macOS** (POSIX): usar Bash, `$HOME`.

## Preflight obligatorio

Antes de buscar en una sesión nueva:

```bash
codesearch_index_status()
codesearch_find_databases()
```

Estado esperado: `indexed=true`, `status="ready"`, `total_chunks > 0`.

## Recuperación de errores comunes

| Error | Solución |
|---|---|
| `Index not built` | `codesearch index` |
| `LockBusy` | Detener procesos `codesearch mcp` duplicados, luego reindexar |
| `Multiple git repositories found` | Indexar cada repo por separado, no desde carpeta padre |
| `FileAlreadyExists("*.del")` | Reindexar con `codesearch index --force` |

## Indexación multi-repositorio (Windows)

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

## Indexación multi-repositorio (Linux/macOS)

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

## Paquete de evidencia compacta

Antes de pasar a `docx-generator` o `commit-msg`, resumir en este formato:

```md
## Evidencia compacta

### Hallazgo 1
- Archivo: `ruta/al/archivo.php`
- Simbolo/metodo: `Nombre::metodo`
- Que hace: resumen de 1 a 2 lineas
- Evidencia: fragmento corto o lineas relevantes
- Confianza: alta/media/baja

### Pendientes
- Punto que falta validar, si aplica.
```

## Archivos

- `SKILL.md` — skill principal
- `references/token_policy.md` — política de ahorro de tokens y anti-patrones
- `.gitattributes` — fuerza finales de linea `LF` en archivos `.md`
- `.gitignore` — ignora `.codesearch.db`

## Notas

- El skill se cachea al iniciar OpenCode; si no aparece, reiniciar el servidor o sesión.
- Verificar con `opencode debug skill`.
- La ruta de skills para OpenCode es: `~/.agents/skills/`.
