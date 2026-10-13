# ADR-003 · Los libros se dan de baja, no se borran

- **Estado:** aceptada
- **Contexto:** una línea de préstamo (`loan_item`) apunta a un libro. Si el libro se borra físicamente, el historial y el reporte
  quedan rotos.
- **Decisión:** el modelo `Book` lleva `deleted_at`. Dar de baja lo llena; el catálogo filtra por `deleted_at IS NULL` en un único
  lugar (*scope* del modelo, no `SoftDeletes` de Laravel porque esa trait asume columnas `created_at`/`updated_at` que este
  proyecto no tiene — artículo VIII de la constitución). Un libro con préstamos abiertos **no** se da de baja (409
  `BOOK_HAS_OPEN_LOANS`). La unicidad del ISBN es un índice **parcial** sobre libros activos (`WHERE deleted_at IS NULL`), así un
  ISBN dado de baja puede volver a registrarse.
- **Alternativas descartadas:** borrado físico con cascada (destruye el histórico); `SoftDeletes` de Laravel tal cual (trae
  columnas de auditoría que el modelo de datos prohíbe); marcar `active = false` (no dice cuándo).
- **Consecuencias:** `loan_item.book_id` lleva una FK `RESTRICT` como barrera de última instancia: un `DELETE` manual falla
  ruidosamente en vez de corromper el histórico. El binario de la portada es el único dato que sí se borra físicamente.
- **Cómo se verifica:** consultar un libro dado de baja devuelve 404; un `DELETE` por `psql` sobre un libro con líneas falla por
  la FK.
