# nx

Nx + pnpm testbed for
[continuous-release](https://github.com/continuous-release).

Every push to `main` cuts a release: CI computes the version from conventional
commits, bumps both packages with `nx release version`, builds, publishes to
GitHub Packages, and creates a GitHub Release with the tarballs.

Bazel counterpart: [continuous-release/bazel](https://github.com/continuous-release/bazel).

## Packages

| Package                           | Description                                                         |
| --------------------------------- | ------------------------------------------------------------------- |
| `packages/typescript-library`     | Publishable `hello()` library. See its README.                      |
| `packages/typescript-application` | Runnable app printing `hello('World')`. See its README.             |
| `packages/release-smoke`          | Hermetic release smoke tests (never published). See its README.     |
| `packages/release-e2e`            | Live release E2E in ephemeral repos (never on PRs). See its README. |

## Prerequisites

[mise](https://mise.jdx.dev/) provides Node and pnpm (see `mise.toml`). Run
`mise trust` once, then `mise install`.

## Commands

```bash
pnpm install
pnpm nx affected --target=build          # tsc -b, with ^build deps
pnpm nx affected --target=lint           # oxlint
pnpm nx affected --target=unit-test      # vitest --dir=tests/unit
pnpm nx affected --target=integration-test  # vitest --dir=tests/integration (builds first)
pnpm nx affected --target=fmt            # dprint check (root only)
pnpm nx affected --target=fmt --configuration=fix  # dprint fmt
```

Target defaults live in `nx.json`. Per-package `nx.targets` entries inherit them.

## CI

`pull.yaml` runs on pull requests and delegates to reusable `shared.yaml`,
which builds a plan matrix over `fmt/build/lint/unit-test/integration-test` and
fans out to `pnpm nx affected --target=<targets>` with `nx-set-shas` so only
affected projects run.

## Release

Push to `main`. `release.yml` calls the shared
[`auto-release.yml`](https://github.com/continuous-release/.github/blob/main/.github/workflows/auto-release.yml)
with `system: nx`:

1. compute the next version from conventional commits since the last `v*` tag;
2. run `nx release version <version>` (fixed group, both packages);
3. build and publish both packages to GitHub Packages;
4. tag `v<version>` and create a GitHub Release with the tarballs.

Nothing is committed back to `main`; tags are the source of truth. Force a
version with a `Release-As: X.Y.Z` footer on the head commit.

## Automation

- Renovate: uses the shared `jonathanmorley/renovate-config`.
- Dependabot security updates: enabled.
- CodeQL: default setup.

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md).
