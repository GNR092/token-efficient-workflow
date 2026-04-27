# Política de bajo consumo de tokens

## Prioridades

1. Buscar antes de leer.
2. Limitar resultados.
3. Filtrar por ruta.
4. Leer fragmentos, no archivos completos.
5. Resumir evidencia una sola vez.
6. Reutilizar evidencia compacta para documentación y commits.

## Preflight obligatorio

Antes de buscar codigo en una sesion nueva, confirmar que `codesearch` esta usable:

```text
codesearch_index_status()
codesearch_find_databases()
```

Estado esperado:

- `indexed=true`
- `status="ready"`
- `total_chunks > 0`

Si el indice esta en un directorio padre o global, usar siempre `filter_path`.

Si `codesearch index` falla con `Multiple git repositories found in subdirectories`, no crear un indice padre. Entrar a cada repositorio Git e indexarlo por separado, esperando a que termine cada proceso antes de continuar.

Antes de ejecutar comandos de shell, detectar el sistema operativo:

- Windows/OpenCode `Platform: win32`: usar PowerShell, `$env:USERPROFILE` y `Join-Path`.
- Linux/macOS/OpenCode POSIX: usar Bash, `$HOME` y rutas con `/`.
- No mezclar comandos PowerShell y Bash en el mismo flujo.

## Presupuesto recomendado

- Exploración inicial: 5 a 8 resultados.
- Refinamiento: 3 a 5 resultados.
- Lectura directa: máximo 1 a 3 archivos, preferentemente fragmentos.
- Documentación: generar desde resumen compacto, no desde código completo.

## Anti-patrones

- Abrir todo el repositorio.
- Copiar archivos enteros al contexto.
- Generar documentación desde suposiciones.
- Usar búsqueda textual masiva si `code-search` puede resolverlo.
- Pedir diffs completos si `git diff --stat` alcanza.
- Buscar sin `filter_path` cuando existe un indice global.
- Ignorar errores `Index not built`, `Vector store empty` o `LockBusy` y continuar como si la busqueda fuera confiable.
- Ejecutar `codesearch index` desde una carpeta que contiene varios repos Git y asumir que OpenCode usara un indice unico.

## Recuperacion rapida

Si `codesearch` falla:

1. Ejecutar `codesearch doctor` y `codesearch index --list`.
2. Revisar procesos duplicados `codesearch mcp`.
3. Si hay bloqueo confirmado, detener solo esos procesos y ejecutar `codesearch index`.
4. Validar con una busqueda CLI y una busqueda MCP antes de continuar el analisis.

## Repos multiples

Detectar primero el sistema operativo y luego usar solo el bloque correspondiente.

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
    codesearch index "$repo" || exit 1
  fi
done
```

Despues ejecutar OpenCode desde el repositorio concreto que se va a analizar, usando la sintaxis del sistema operativo detectado.

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
