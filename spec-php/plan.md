# Plan técnico — Manager Library (React + Laravel)

Cómo se construye lo que dicen [`spec.md`](../spec-python/spec.md) y [`constitution.md`](constitution.md) (ambos
idénticos en negocio al spec de referencia). El esquema vive en [`data-model.md`](../spec-python/data-model.md)
(también idéntico: el modelo de datos no depende del lenguaje). La forma del sistema está en [`architecture.md`](architecture.md).

## 1. Decisiones técnicas

| # | Decisión | Elección (Laravel) | Equivalente en el spec de referencia (Python) | Por qué |
|---|---|---|---|---|
| D-01 | Lenguaje y runtime | PHP 8.2 | Python 3.12 | Stack del reto |
| D-02 | Framework HTTP | Laravel 11 | FastAPI | Contrato en `api-contract.md` |
| D-03 | Persistencia | Eloquent + migraciones nativas sobre PostgreSQL 16 | SQLAlchemy 2 + Alembic | El esquema es propiedad de las migraciones (ADR-001) |
| D-04 | Concurrencia | `CHECK` de rango en `book` + bloqueo de fila (`lockForUpdate`) dentro de la transacción de préstamo | Testigo de versión + `CHECK` | Último ejemplar: uno 201, el otro 409 (RNF-2, ADR-002) |
| D-05 | Fechas | Instantes UTC (`timestamptz`); reloj inyectable para pruebas | Puerto `Clock` | Probar vencimientos sin esperar 14 días |
| D-06 | Reporte | Consulta agregada única sobre `loan_item`, agrupada por valores congelados | Puerto de lectura agregado | Estable y barato (ADR-004) |
| D-07 | Objetos de valor | `App\Domain\ValueObjects\{Isbn,Quantity,DateRange}`: PHP puro, sin Eloquent | Igual | No tienen identidad; viven en la fila de su dueño |
| D-08 | Portadas | Disco local (`storage/app/covers`) en desarrollo; la base guarda solo `cover_key` | Disco/S3 detrás de `CoverStorage` | Clave opaca, nunca ruta |
| D-09 | Contraseñas | Hash nativo de Laravel (bcrypt), `password_hash` oculto en el modelo | Argon2id detrás de `PasswordHasher` | Artículo IX |
| D-10 | Arranque | Comando/seeder crea el admin inicial desde `ADMIN_USERNAME`/`ADMIN_PASSWORD` | Igual | La base no puede producir el hash |
| D-11 | Autenticación | **Laravel Sanctum** (tokens de acceso personal), expiración 8 h | JWT HS256, clave en `JWT_SECRET` | Mismo contrato (`Authorization: Bearer <token>`); Sanctum es el estándar de Laravel y evita reinventar firma/verificación |
| D-12 | Pruebas | PHPUnit/Pest: unitarias de dominio sin BD, *feature* contra Postgres real | pytest con *fakes* + integración | Misma pirámide de pruebas |

## 2. Configuración por entorno

| Variable | Obligatoria | Ejemplo |
|---|---|---|
| `DB_CONNECTION`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD` | sí | Postgres 16, esquema `library` |
| `DB_SCHEMA` | no (default `library,public`) | `search_path` de la conexión |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | sí | definidos en `.env`, nunca en el repo |
| `COVERS_DIR` | no | `storage/app/covers` |
| `CORS_ORIGIN` | sí | `http://localhost:5173` |

## 3. Estrategia de pruebas

| Nivel | Qué prueba | Contra qué |
|---|---|---|
| Dominio | `Isbn` (13 dígitos), `Quantity` (> 0), `DateRange` (fin ≥ inicio) | Nada: PHP puro |
| Aplicación/servicio | Casos de uso: prestar, devolver, reporte | Modelos Eloquent contra una BD de pruebas |
| Integración | Migraciones, constraints, búsqueda paginada | PostgreSQL real |
| Contrato | Cuerpo de error único y códigos de estado de `api-contract.md` | La API levantada |
| Concurrencia | Dos préstamos del último ejemplar | PostgreSQL real, dos peticiones simultáneas |

## 4. Orden de construcción

Esqueleto y `/health` → migración inicial con semilla → objetos de valor de dominio → autenticación (Sanctum) →
libros → socios → préstamos (transacción crítica) → devoluciones y vencidos → reporte → pruebas de contrato y
concurrencia → frontend React. El detalle con evidencia está en [`tasks.md`](tasks.md).
