# MoneyOS

The public, open-source home of [MoneyOS](https://moneyos.cc), a personal finance assistant
that explains your money in plain English. The hosted product at moneyos.cc lives in a
separate private repository; this one is where code that anyone can read, run and
contribute to is built.

## What is here today

- A Next.js 16 scaffold (App Router, React 19, TypeScript, Tailwind CSS 4) in `src/app/`.
  The page is still the create-next-app placeholder.
- The engineering gates: formatter, linter, typecheck, production build, Markdown link
  checker and a public-repository guardrail, run by git hooks locally and by GitHub
  Actions on every push and pull request. See [`docs/engineering-practices.md`](docs/engineering-practices.md)
  for the full audit of what is applied and what was deliberately left out.

No product code has landed yet. This README describes what exists; it is updated as the
layout changes.

## Getting started

Requires Node 24 (see `.nvmrc`) and pnpm (version pinned in `package.json`).

```bash
pnpm install
pnpm dev
```

Open <http://localhost:3000>.

## Checks

| Command                 | What it does                                                                   |
| ----------------------- | ------------------------------------------------------------------------------ |
| `pnpm run ci`           | Everything CI runs except the build: typecheck, lint, format, docs, guardrails |
| `pnpm run build`        | Production build, the same way the host builds it                              |
| `pnpm run lint:fix`     | ESLint with auto-fix. Run by a person, never by a hook or CI                   |
| `pnpm run format`       | Prettier, writing. The pre-commit hook runs this on staged files only          |
| `pnpm run check:docs`   | Relative links and heading anchors in every Markdown file                      |
| `pnpm run check:public` | No private paths, env files or credential patterns in the tree                 |

Each hand-written checker ships with a test that proves it can fail
(`check:docs:test`, `check:public:test`); CI runs the proof before the check.

`pnpm install` installs two git hooks: pre-commit formats the staged files, pre-push runs
`pnpm run ci` and the production build. The workflow in `.github/workflows/ci.yml` runs
the same checks on a hosted runner, one named job each, so they can be required on `main`.

## Layout

```
.github/          CI workflow, shared setup action, Dependabot, pull request template
.husky/           pre-commit (format staged files) and pre-push (ci + build)
docs/             Living documents, link-checked in CI
scripts/          Hand-written checkers and their proof-of-failure tests
src/app/          Next.js App Router: layout, page, global styles
```
