# Commits — how to push and open a PR

## Branch naming

One short-lived branch per unit of work, prefixed by the commit type:

```
<type>/<short-description>
```

Examples: `feat/loans-endpoint`, `docs/spec-php-translation`, `refactor/hexagonal-structure`,
`fix/isbn-unique-index`.

## Commit message convention

```
<type>(<scope>): <short summary in imperative mood>

<optional body: what and why>
```

| Type | When |
|---|---|
| `feat` | New behavior: an endpoint, a service, a migration that adds a table |
| `fix` | A bug fix |
| `refactor` | Restructuring with no behavior change |
| `docs` | Anything under `docs/`, `DECISIONS.md`, ADRs, READMEs |
| `test` | Adding or fixing tests only |
| `chore` | Tooling, `.gitignore`, dependency bumps |

Scope is the layer touched: `api`, `app`, `infra`, `tool`, `docs`, or narrower (`auth`, `loans`, `books`).

## Push and PR

1. Create the branch from the fork's default branch.
2. Commit on that branch (small, working-state commits — one coherent step each).
3. Push the branch to `origin` (your fork).
4. Open a PR **against `main`**.

**Why `main` and not `develop`:** each of the 6 forks was created with a single branch, `main` — there is no
`develop` branch unless one is created on purpose, and the challenge doesn't ask for one. Introducing a
`develop` branch here would add a second integration point nobody asked for and nothing downstream reads.
Target `main`.

5. Merge the PR into `main` once it's green (or once you've reviewed it yourself — there's no CI gate on
   these forks). Delete the branch after merging.
