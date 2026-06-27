# actions-bump-first-party-dep

Reusable workflow that bumps a single published first-party dependency to
`@latest` wherever it appears (direct → rewrite the pinned range; transitive →
refresh the lockfile), in `/` and `/backend`, and opens/updates a single rolling
PR. Lockfile-only — CI does the real install.

## Usage

```yaml
# .github/workflows/bump-first-party-deps.yml in a downstream repo
on:
  repository_dispatch:
    types: [first-party-release]
  workflow_dispatch:
    inputs:
      package:
        description: "Package to bump (e.g. @comapeo/core)"
        required: true

jobs:
  bump:
    uses: digidem/actions-bump-first-party-dep/.github/workflows/bump.yml@v1
    with:
      package: ${{ github.event.client_payload.package || inputs.package }}
    secrets:
      PR_BOT_PRIVATE_KEY: ${{ secrets.PR_BOT_PRIVATE_KEY }}
```

Requires (in the caller repo): variables `PR_BOT_APP_ID`, `PR_BOT_USER_ID`,
secret `PR_BOT_PRIVATE_KEY`, and the PR bot app installed with Contents +
Pull-requests write. The PR bot (not the default `GITHUB_TOKEN`) is required so
the bump PR triggers the repo's required checks.

Pin to `@v1` (moving major) or a SHA.