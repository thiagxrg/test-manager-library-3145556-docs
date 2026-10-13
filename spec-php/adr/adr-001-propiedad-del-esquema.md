# ADR-001 · El esquema es propiedad de las migraciones de la aplicación

- **Estado:** aceptada
- **Contexto:** el repositorio `infra` levanta un motor de base de datos. Alguien podría poner el DDL en un script de arranque del
  contenedor y otro en las migraciones de la aplicación; con dos dueños, nadie sabe qué esquema corre en producción.
- **Decisión:** **todo el DDL vive en las migraciones nativas de Laravel** (`database/migrations/`): tablas, índices, restricciones
  y semilla de categorías. `infra` solo crea el motor vacío (`docker-compose.yml`, servicio `db` sin volumen de inicialización SQL),
  el usuario y el volumen.
- **Alternativas descartadas:** scripts SQL en `infra/db/init/` (segunda fuente de verdad); `Schema::create` disparado al arrancar
  la app en vez de versionado en migraciones (no deja historial ni reversa).
- **Consecuencias:** un índice o un `CHECK` nuevo se crea en la **misma** migración que lo usa. Un esquema cambiado a mano en el
  motor es un defecto, no una variante.
- **Cómo se verifica:** `infra` no contiene `CREATE TABLE`; las consultas de `data-model.md` §10 coinciden con lo que crea
  `php artisan migrate`.
