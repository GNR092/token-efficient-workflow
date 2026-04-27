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

## Recuperacion rapida

Si `codesearch` falla:

1. Ejecutar `codesearch doctor` y `codesearch index --list`.
2. Revisar procesos duplicados `codesearch mcp`.
3. Si hay bloqueo confirmado, detener solo esos procesos y ejecutar `codesearch index`.
4. Validar con una busqueda CLI y una busqueda MCP antes de continuar el analisis.
