# Contributing

## Setup

```bash
mise trust
mise install
pnpm install
```

Work in an isolated worktree under the repo-root `.worktrees/` directory (e.g. `git worktree add .worktrees/<topic> -b <topic>`).

## Daily workflow

```bash
pnpm nx affected --target=fmt --configuration=fix
pnpm nx affected --target=build
pnpm nx affected --target=lint
pnpm nx affected --target=unit-test
pnpm nx affected --target=integration-test
```

`integration-test` depends on `build`, so the built output in `dist/` exists when integration tests run. `test` is an alias for `unit-test`.

## Conventions

- Formatting: [dprint](https://dprint.dev/) with the prettier plugin (`dprint.json`: single quotes, no trailing commas). Check with `fmt`, fix with `--configuration=fix`.
- Lint: [oxlint](https://oxc.rs/) (`.oxlintrc.json`); `oxlint --fix` via the `lint --configuration=fix` Nx configuration.
- Tests: [vitest](https://vitest.dev/) (`vitest.config.js`: no watch, pass with no tests). Unit tests live in `tests/unit/`, integration tests in `tests/integration/`, named `*.test.ts`. Import sources via relative `../../src/index.js` paths and use the `({ expect })` fixture-arg style.
- TypeScript: project references via `tsc -b`; packages extend the root `tsconfig.json` (`@tsconfig/node22`).
- Keep coverage high; prefer exercising behavior (pack/install/run) over asserting file contents.

## Release (maintainers)

Releases are automatic. Every push to `main` runs `release.yml`, which calls
the shared
[`auto-release.yml`](https://github.com/continuous-release/.github/blob/main/.github/workflows/auto-release.yml)
with `system: nx`:

1. compute the next version from conventional commits since the last `v*` tag;
2. run `nx release version <version>` (fixed group, both packages);
3. build and publish both packages to GitHub Packages;
4. tag `v<version>` and create a GitHub Release with the tarballs.

Do not version locally and do not push tags. Use a `Release-As: X.Y.Z` footer
on the head commit to force a version. Nothing is committed back to `main`;
the tag is the source of truth.

The shared workflow is the only place that knows the version rules and the
release notes format. If you need to change them, change them there, not in
this repo.

## Live E2E authentication

`e2e-release.yaml` is paused after the move to `continuous-release`. It runs
the `release-e2e` suite, which releases from namespaced `it-*` refs in the
`gh-testbed` repo, publishes `0.0.0-e2e.*` package versions, and deletes
everything afterwards. Authentication uses Octo STS: the job mints a
short-lived token scoped to the testbed repo (trust policy at
`gh-testbed/.github/chainguard/gha-testing-e2e.sts.yaml`).

To re-enable it, move `gh-testbed` to `continuous-release`, update the trust
policy to allow `continuous-release/nx`, and restore the schedule. Package
registry reads and version deletions still use the `TESTBED_PACKAGES_TOKEN`
secret (classic PAT, `read:packages` + `write:packages` + `delete:packages`),
because those endpoints only honor PATs.

## Pull requests

- Keep PRs small and focused on a single concern.
- Run formatter, linter, and affected tests before pushing (CI runs them anyway).
- Commit often to preserve progress; do not push unless asked (CI still validates the PR).
- Add the `ai:autofix` label so automation can fix up the branch (e.g. `gh pr edit --add-label "ai:autofix"`).
- Always open a PR for completed work; direct pushes to `main` are blocked by branch protection (linear history, signed commits).
