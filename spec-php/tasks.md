# Tareas — Manager Library (React + Laravel)

Traducción de [`spec-python/tasks.md`](../spec-python/tasks.md). Cada tarea termina cuando su **evidencia**
existe y se puede ejecutar (artículo XII). Estado inicial de todas: **pendiente**.

| # | Tarea | Depende de | Evidencia (Laravel/React) |
|---|---|---|---|
| T-01 | Esqueleto Laravel y `GET /api/v1/health` | — | `curl /api/v1/health` → `{"status":"ok"}`; el proyecto arranca con `php artisan serve` o el contenedor |
| T-02 | Migración inicial: seis tablas, FK, índices únicos, `CHECK` y las cinco categorías sembradas | T-01 | `php artisan migrate`; las consultas de `data-model.md` §10 devuelven lo documentado; ninguna columna con `DEFAULT` |
| T-03 | Objetos de valor: `Isbn`, `Quantity`, `DateRange` (PHP puro, sin Eloquent) | T-01 | PHPUnit: ISBN de 13 dígitos, cantidad > 0, fin ≥ inicio |
| T-04 | Modelo `Book` (descuento/restock, baja lógica) | T-03 | Pruebas: nunca < 0, nunca > total |
| T-05 | Modelo `Loan`/`LoanItem` (estado calculado, 14 días, máx. 3) | T-04 | Pruebas con reloj fijo: `open`, `overdue`, `closed` |
| T-06 | Modelo `Member` y `User` (normalización de nombre, rol válido) | T-03 | `"Ana "` queda `ana`; rol `admin` por API rechazado |
| T-07 | **Autenticación**: `LoginService`, `CreateLibrarianService` con Sanctum | T-06 | HU-01 y HU-02 pasan; token Bearer válido 8 h |
| T-08 | **Libros**: crear, buscar, editar, baja, portada | T-04 | HU-03 y HU-04 pasan |
| T-09 | **Prestar** (`LendBooksService`, transacción + `lockForUpdate`) | T-05, T-08 | HU-06 completa: límites, vencidos, sin ejemplares |
| T-10 | **Devolver** y **vencidos** | T-09 | HU-07 y HU-09; devolver dos veces → `ITEM_ALREADY_RETURNED` |
| T-11 | **Reporte de más prestados** | T-09 | HU-10; el título viejo se conserva tras renombrar (ADR-004) |
| T-12 | Rutas HTTP: todas las de [`api-contract.md`](../spec-python/api-contract.md), cuerpo de error único, autorización por rol (`middleware`) | T-07, T-08, T-09, T-10, T-11 | Prueba de contrato recorre todas las rutas y todos los códigos de §2 |
| T-13 | Arranque: admin inicial desde `.env` (`artisan` command o seeder) | T-12 | El sistema arranca con la base vacía y se puede iniciar sesión como admin |
| T-14 | Prueba de concurrencia: dos préstamos del último ejemplar | T-09 | Dos peticiones simultáneas: una 201, otra 409 (RNF-2) |
| T-15 | `infra`: `docker compose up` levanta Postgres vacío, API y frontend | T-13 | Un comando; la base arranca sin tablas hasta correr las migraciones |
| T-16 | `tool`: sembrador de demostración **por la API** (≥ 40 libros, 10 socios, 15 préstamos, 3 vencidos) | T-13 | Tras correrlo, `/loans/overdue` devuelve 3 líneas y el reporte tiene filas |
| T-17 | `app` (React): login, catálogo, socios, prestar, devolver, vencidos, reporte | T-12 | Recorrido manual de punta a punta |
| T-18 | `page`: sitio público estático de presentación | — | Se abre sin la API; correcto en móvil y escritorio |

## Definición de terminado de la prueba

Quien llegue a T-13 con evidencia tiene el **núcleo** hecho (equivale a T-17 del spec de referencia: auth +
libros + préstamos + devoluciones + reporte + arranque con admin). T-14 a T-18 son lo que separa un trabajo
bueno de uno sobresaliente. Se evalúa el desempeño con SDD, no la cantidad de tareas.
