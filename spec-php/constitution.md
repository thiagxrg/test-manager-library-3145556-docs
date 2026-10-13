# Constitución — Manager Library (React + Laravel)

Traducción de [`spec-python/constitution.md`](../spec-python/constitution.md) al stack real del reto. Los principios
son los mismos; lo que cambia es **cómo se verifican** en Laravel/PHP. Donde el stack obliga a un compromiso
pragmático, se declara explícitamente — no se esconde.

---

## Artículo I — El dominio no depende de nada

Las invariantes puras (`Isbn`, `Quantity`, `DateRange`) viven en `app/Domain/ValueObjects/`: PHP puro, sin
Eloquent, sin Laravel, sin librerías de terceros. Se prueban con PHPUnit sin base de datos y sin framework.

**Compromiso declarado:** a diferencia del spec Python (que aísla *todo* el dominio, incluidos `Book`/`Loan`/`Member`/`User`
como agregados sin ORM), aquí `Book`, `Loan`, `Member` y `User` son **modelos Eloquent** con la lógica de invariantes
en métodos propios del modelo o en clases de servicio. Es una concesión de tiempo/pragmatismo del framework, no
una negación del principio: las invariantes siguen centralizadas y probadas, no dispersas en los controladores.

## Artículo II — El núcleo declara sus puertos; el mundo los implementa

Los casos de uso (`app/Services/*`) no conocen HTTP: reciben datos simples, devuelven resultados simples o lanzan
excepciones de dominio propias (`BookNotAvailableException`, `MemberHasOverdueException`, …). Los controladores
(`app/Http/Controllers`) son los únicos que traducen esas excepciones al cuerpo de error único de `api-contract.md`.

## Artículo III — Un único punto de composición

Laravel resuelve dependencias por su *service container*. Los *bindings* explícitos (interfaz → implementación,
p. ej. `CoverStorage` → `DiskCoverStorage`) se declaran en un único proveedor: `app/Providers/AppServiceProvider.php`.
Fuera de ahí no se instancia infraestructura a mano.

## Artículo IV — SOLID no es decorativo

| Principio | Aplicación concreta en Laravel |
|---|---|
| **SRP** | Un servicio de aplicación = un caso de uso (`LendBooksService`, `ReturnLoanItemService`, …) |
| **OCP** | Cambiar el almacenamiento de portadas (disco → S3) implementa la misma interfaz `CoverStorage` |
| **LSP** | Los *fakes* en tests sustituyen la implementación real sin cambiar la expectativa del test |
| **ISP** | Interfaces pequeñas: `CoverStorage`, `Clock` — ningún implementador deja métodos sin usar |
| **DIP** | Los servicios dependen de interfaces, inyectadas por el contenedor de Laravel |

## Artículo V — El esquema es propiedad de las migraciones

El repositorio `infra` levanta un motor **vacío** (`docker-compose.yml`, sin `CREATE TABLE`). Todo el DDL vive en
las migraciones nativas de Laravel (`database/migrations/`), equivalente a Alembic en el spec de referencia.
Ver [ADR-001](adr/adr-001-propiedad-del-esquema.md).

## Artículo VI — Las invariantes que la base puede imponer, la base las impone

`CHECK`, `UNIQUE` y `FK` se declaran en las migraciones, no solo en el código: `ck_book_available_range`,
`ck_book_total_positive`, `ck_user_role`, índice único parcial de `isbn` sobre libros activos. Ya están en la
migración inicial — no se dejaron pendientes, a diferencia del spec de referencia que los marca T-18.

## Artículo VII — Lo derivado se calcula, no se almacena

El estado del préstamo (`open`, `overdue`, `closed`) y los días de retraso se calculan en `Loan::status()` y en
la consulta de vencidos. No hay columna para ellos.

## Artículo VIII — Los valores congelados no siguen al catálogo

`loan_item` guarda `book_title`, `isbn` y `category_name` del instante del préstamo. El reporte y el listado de
vencidos leen de `loan_item`, nunca de `book`/`category` en vivo.

## Artículo IX — Ningún secreto entra al repositorio ni a un log

`.env` real en `.gitignore`; solo se versiona `.env.example`. El hash de contraseña (`password_hash`) está en
`$hidden` del modelo `User` y nunca se registra en logs. Verificación: `grep` de patrones en CI y prueba de
contrato que confirma que ninguna respuesta incluye `password` ni `password_hash`.

## Artículo X — Gana el motor

Si `data-model.md` y la base discrepan, se corrige el documento, no la base.

## Artículo XI — Idioma

Código, columnas y rutas en inglés (`loan_item`, `due_at`, `/api/v1/loans`). Documentación y mensajes de error
de la API al usuario, en español.

## Artículo XII — Nada se da por hecho sin evidencia

Una tarea de [`tasks.md`](tasks.md) está terminada cuando su evidencia existe y se puede ejecutar — no cuando
«ya está programada».
