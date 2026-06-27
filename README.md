# actions-bump-first-party-dep

Reusable workflow that bumps a single published first-party dependency to
`@latest` in one directory (direct dep → rewrite the pinned range; transitive →
refresh the lockfile) and opens/updates a single rolling PR. Lockfile-only — CI
does the real install.

## Inputs

| input | required | default | description |
|---|---|---|---|
| `package` | yes | — | Package to bump, e.g. `@comapeo/core` |
| `path` | no | `.` | Directory containing the `package.json` + lockfile to update |
| `node-version-file` | no | `package.json` | File to read the Node version from (tooling) |

Secret: `PR_BOT_PRIVATE_KEY`. Also needs variables `PR_BOT_APP_ID`,
`PR_BOT_USER_ID` and the PR bot app installed with Contents + Pull-requests write
(so the bump PR triggers the repo's required checks — the default `GITHUB_TOKEN`
would not).

## Usage — single package tree

```yaml
on:
  repository_dispatch:
    types: [first-party-release]
  workflow_dispatch:
    inputs:
      package: { description: "Package to bump", required: true }

jobs:
  bump:
    uses: digidem/actions-bump-first-party-dep/.github/workflows/bump.yml@v1
    with:
      package: ${{ github.event.client_payload.package || inputs.package }}
    secrets:
      PR_BOT_PRIVATE_KEY: ${{ secrets.PR_BOT_PRIVATE_KEY }}
```

## Usage — multiple package trees (matrix)

A repo with more than one tree (e.g. a root + an embedded `backend`) fans out
over directories itself; the rolling PR + concurrency group land them in one PR:

```yaml
jobs:
  bump:
    strategy:
      matrix:
        path: [".", "backend"]
    uses: digidem/actions-bump-first-party-dep/.github/workflows/bump.yml@v1
    with:
      package: ${{ github.event.client_payload.package || inputs.package }}
      path: ${{ matrix.path }}
    secrets:
      PR_BOT_PRIVATE_KEY: ${{ secrets.PR_BOT_PRIVATE_KEY }}
```

Pin to `@v1` (moving major) or a SHA.