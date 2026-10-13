# ADR-002 · Concurrencia sobre los ejemplares: bloqueo de fila + CHECK como última barrera

- **Estado:** aceptada
- **Contexto:** dos bibliotecarios pueden prestar el **último ejemplar** del mismo libro al mismo tiempo (RNF-2). Si ambos leen
  `available_copies = 1` y ambos escriben, el sistema presta un libro que no tiene.
- **Decisión:** dentro de la transacción de `LendBooksService`, se bloquea la fila del libro con `lockForUpdate()` (Eloquent) antes
  de descontar el ejemplar; la segunda petición espera el bloqueo y relee `available_copies` ya actualizado, por lo que falla con
  **409 `BOOK_NOT_AVAILABLE`** si ya no hay ejemplares. La **última barrera**, igual que en el spec de referencia, es el `CHECK`
  `ck_book_available_range` (`0 <= available_copies <= total_copies`): si algo escribe fuera de este servicio, la base rechaza el
  dato antes de corromperse.
- **Alternativas descartadas:** concurrencia optimista con testigo de versión (como el spec Python): más fiel al original, pero
  `lockForUpdate` es el patrón idiomático de Eloquent para este caso y da la misma garantía observable (una petición 201, la otra
  409) con menos código nuevo.
- **Consecuencias:** la biblioteca no se serializa globalmente: el bloqueo es por fila de libro, no por tabla. Devolver dos veces
  la misma línea no suma dos veces porque `returned_at` ya está lleno (CA-07.2).
- **Cómo se verifica:** prueba con dos peticiones HTTP simultáneas sobre un libro con un solo ejemplar disponible: exactamente una
  recibe 201, la otra 409.
