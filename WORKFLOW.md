# Git workflow — how this gets pushed

Decision: **direct commits to `main`** on each of the 6 forks, no feature branches / PRs. Reason: the test's
own rule is "lo que quede subido a los forks antes de las 5:00 p.m." — what's graded is the state of `main`
on each fork at the deadline, not the elegance of the git flow. A branch + PR cycle (branch → push → open PR →
merge) adds steps that cost time without adding evidence, and risks something sitting unmerged at 5pm.

## Commit message convention

```
<type>(<scope>): <short summary in imperative mood>

<optional body: what and why, not a diff restated in prose>
```

| Type | When |
|---|---|
| `feat` | New behavior: an endpoint, a service, a migration that adds a table |
| `fix` | A bug fix |
| `refactor` | Restructuring with no behavior change (e.g. moving domain code to `src/Library`) |
| `docs` | Anything under `docs/`, `DECISIONS.md`, ADRs, READMEs |
| `test` | Adding or fixing tests only |
| `chore` | Tooling, `.gitignore`, dependency bumps |

Scope is the layer touched: `api`, `app`, `infra`, `tool`, `docs`, or a narrower one like `auth`, `loans`,
`books` when it helps. Example: `feat(loans): lend endpoint with lockForUpdate concurrency`.

## Commit frequency

**Small and often**, not one giant commit at the end. Each commit should be a single coherent step that leaves
the repo in a working state (migration runs, endpoint responds, test passes) — matching constitution Article
XII: a task counts as done only when its evidence exists and runs, and that evidence is what a commit should
capture.

## Push frequency

Push right after each commit, or in small batches of 2-4 related commits. Never hold everything until the end
— if something goes wrong in the last 15 minutes, whatever was already pushed still counts; what's only local
does not.

## Per-repo breakdown

| Repo | What gets committed there |
|---|---|
| `docs` | `spec-php/` (this fork's translation of the spec), this `WORKFLOW.md` |
| `api` | Laravel backend: migrations, `src/Library/` (domain, application, adapters, bootstrap), `app/Http`, `DECISIONS.md` |
| `app` | React frontend |
| `infra` | `docker-compose.yml`, any Dockerfiles |
| `tool` | The demo seeder script (calls the API, never touches the DB directly) |
| `page` | Static public site |

## Order of operations today

1. `docs`: spec-php translation — done, pushed.
2. `infra`: `docker-compose.yml` — done, pending push.
3. `api`: skeleton, migration, domain, hexagonal adapters — in progress.
4. `api`: auth, books, members, loans, returns, overdue, report — next.
5. `app`: React screens, once the API has real endpoints to call.
6. `tool`, `page`: only if time remains after the core (per `tasks.md`'s definition of done).
