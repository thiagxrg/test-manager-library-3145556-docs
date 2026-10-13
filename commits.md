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

## Branches per repo

Each fork keeps two long-lived branches:

- **`develop`** — where work actually lands. Every feature/fix/refactor branch is created from `develop` and
  PRs target `develop`.
- **`main`** — the stable/release line. Only updated by merging `develop` into it (periodically, and for sure
  before the 5pm deadline, since the test is graded on what's on the forks — merge `develop` into `main` at
  least once near the end so the graded content isn't sitting only on `develop`).

If a fork was created with only `main` (the GitHub default), create `develop` from it once, push it, and from
then on treat `develop` as the integration branch.

## Push and PR

1. Create the branch from `develop` (not `main`).
2. Commit on that branch (small, working-state commits — one coherent step each).
3. Push the branch to `origin` (your fork).
4. Open a PR **against `develop`**.
5. Merge the PR into `develop` once it's green (or once you've reviewed it yourself — there's no CI gate on
   these forks). Delete the branch after merging.
6. Before the deadline, merge `develop` into `main` so the graded branch has everything too.

> Note: the `docs` fork's `spec-php/` translation was pushed straight to `main` before this `develop`
> convention was set — left as-is rather than rewritten, since the content is correct and rewriting history
> would just cost time. Everything from here on (in every repo, `docs` included if it changes again) follows
> the `develop`-first flow above.
