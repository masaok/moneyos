# Engineering practices, applied here

An audit of this repository against
[Engineering practices that survive the stack](https://www.masao.site/code-quality/engineering-practices),
32 techniques in seven themes. Each row names the invariant the technique provides, then
what this repository does about it. The rule is what transfers; the tool named is one
instantiation.

Audited 2026-10-11 against a repository that was still the create-next-app scaffold: four
source files, no tests, no database, no server modules, no deploy target, and no `.github/`
directory. Several items therefore shrink to "the rule is recorded; the mechanism lands with
the first thing it guards." Those are marked **with the first** below and are not deferred by
choice.

Status: ✅ applied · ⏳ with the first · ➖ not applicable · 🚫 not adopted (reason in the
section at the end).

## Summary

| #   | Technique                                       | Priority | Status | Here                                                                                                                                                                           |
| --- | ----------------------------------------------- | -------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.1 | Hosted CI pipeline                              | P0       | ✅     | `.github/workflows/ci.yml`: seven named jobs on push to `main` and on pull requests, read-only permissions, cancel-in-progress, SHA-pinned actions                             |
| 1.2 | Staged-files-only pre-commit hook               | P0       | ✅     | `husky` + `lint-staged`; `.husky/pre-commit` formats staged files only. The expensive work is in `.husky/pre-push` and CI                                                      |
| 1.3 | Read-only lint in every automated path          | P0       | ✅     | `lint` is `eslint`; `lint:fix` exists for people and is wired to nothing                                                                                                       |
| 1.4 | Required status checks on `main`                | P0       | ✅     | Branch protection requires the seven job names below, bound to the GitHub Actions app, strict (up to date with `main`)                                                         |
| 1.5 | Host-faithful production build before deploy    | P1       | ✅     | `build` job on Node 24 from `.nvmrc` with a frozen-lockfile install; `.husky/pre-push` builds before every push. There is no deploy target yet, so no host build CLI to mirror |
| 2.1 | Compiler-enforced client/server boundary        | P0       | ⏳     | No module reads a credential or opens a connection. The first one starts with `import "server-only"`, declared in `dependencies`                                               |
| 2.2 | Every committed config has its tool installed   | P0       | ✅     | Prettier installed with the import-sort and Tailwind plugins, `format` job in CI. The one-off reformat is its own commit, recorded in `.git-blame-ignore-revs`                 |
| 2.3 | Guides that describe the code that exists       | P0       | ✅     | `README.md` replaced; `AGENTS.md` names the verification command and the push rule; one-line pull request template                                                             |
| 3.1 | One declared runtime version                    | P1       | ✅     | `.nvmrc` = 24, `engines.node` = 24.x, `@types/node` ^24, Dependabot holds the types major                                                                                      |
| 3.2 | One composite setup step                        | P1       | ✅     | `.github/actions/setup/action.yml` reads Node from `.nvmrc` and pnpm from `packageManager`                                                                                     |
| 3.3 | Derived categorizations from one constant       | P1       | ➖     | No data layer                                                                                                                                                                  |
| 3.4 | Dependency automation that respects pins        | P1       | ✅     | `.github/dependabot.yml`: weekly grouped minors and patches, majors separate, actions grouped, `@types/node`/`typescript`/`eslint` majors held                                 |
| 4.1 | Typegen inside the typecheck command            | P1       | ✅     | `typecheck` is `next typegen && tsc --noEmit`                                                                                                                                  |
| 4.2 | Proof-of-failure test for every custom check    | P0       | ✅     | `check:docs:test` and `check:public:test` build fixtures that must fail; CI runs each before its check                                                                         |
| 4.3 | Import smoke test over real entry points        | P1       | ⏳     | No scripts or daemons. Lands with the first one, run under the same launcher production uses                                                                                   |
| 4.4 | Fail-safe changed-files gate                    | P2       | 🚫     | See below                                                                                                                                                                      |
| 4.5 | Job-level skip, never workflow path filters     | P2       | ✅     | No path filters on the trigger; every job always reports                                                                                                                       |
| 5.1 | Numbered migration files                        | P1       | ➖     | No database. The rule applies the day one is added: migration 0001 captures the schema, CI applies from empty                                                                  |
| 5.2 | Language-neutral migration format               | P1       | ➖     | One language                                                                                                                                                                   |
| 5.3 | Harvest the accidental migration log            | P1       | ➖     | Nothing to harvest                                                                                                                                                             |
| 5.4 | Retire the replaced mechanism                   | P1       | ➖     | Nothing replaced                                                                                                                                                               |
| 5.5 | Advisory lock around the migration runner       | P1       | ➖     | No runner                                                                                                                                                                      |
| 6.1 | Name the invariant before the vendor            | P2       | ✅     | This table names the invariant per row                                                                                                                                         |
| 6.2 | Evaluate tools by failure mode                  | P2       | ⏳     | No tool choice pending. Applies to the first one, with a destructive-default column                                                                                            |
| 6.3 | Score bundled benefits one at a time            | P2       | ⏳     | Same                                                                                                                                                                           |
| 6.4 | A rejected-items section inside the audit       | P2       | ✅     | Below                                                                                                                                                                          |
| 7.1 | Infra-dependent suites on a schedule            | P0       | ⏳     | No suite touches live infrastructure. Rule recorded: such a suite runs on `schedule` and `workflow_dispatch`, never required                                                   |
| 7.2 | Port and resource registry                      | P2       | 🚫     | See below                                                                                                                                                                      |
| 7.3 | Shared environment preflight                    | P2       | ⏳     | Nothing reads an environment variable. The first entry point that does asserts its variables by name at the top                                                                |
| 7.4 | One living document per subsystem, link-checked | P2       | ✅     | `docs/` and every Markdown file pass `check:docs` in the `docs` job; relative links and anchors only, nothing fetched                                                          |
| 7.5 | Remediation ordered by leverage                 | P1       | ✅     | This table is in the practice's order; everything applied landed in two commits on the day of the audit                                                                        |
| 7.6 | A pull request only when it buys a review       | P1       | ✅     | `AGENTS.md`: push directly to `main`; CI runs on push; pull requests for reviewed work, outside contributions and Dependabot                                                   |

## Required status checks

Branch protection on `main` names these job contexts, each bound to the GitHub Actions
app so no other integration can satisfy a check by reporting the same name:

```
typecheck · lint · format · build · docs · guardrails · gitleaks
```

Admins can still bypass the rule, so for a direct push to `main` the pre-push hook is the
gate, not the setting.

## Not yet a check

- **Unit tests.** There is no test runner because there is nothing to test. A `test` job
  with "pass with no tests" would be a check that cannot fail, which is worse than no job.
  Vitest and a `test` job (required) arrive with the first unit under test.

## Explicitly not adopted

- **4.4 Fail-safe changed-files gate.** Every job here finishes in about a minute. A
  changed-files gate is untested logic running outside the gate, and it would be more code
  than the minutes it saves. Revisit if the repository grows a large docs or research
  directory.
- **7.2 Port and resource registry.** One developer, one dev server. The convention it
  replaces (a table of ports per worktree) does not exist here either. Revisit when two
  sessions run at once.
