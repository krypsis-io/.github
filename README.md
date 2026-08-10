# .github

Shared reusable GitHub Actions workflows. Designed for use by `krypsis-io` repos but fully generic — private forks (mirrors) can use these workflows by replacing `krypsis-io` with their own org in the `uses:` references below.

All actions are SHA-pinned. Shell injection mitigations applied (env vars instead of direct `${{ }}` interpolation in `run:` blocks).

## Available Workflows

| Workflow | Description | Key Inputs |
|----------|-------------|------------|
| `release.yml` | Semantic-release with SBOM generation | `node-version`, `generate-sbom` |
| `release-please.yml` | release-please with a reviewable Release PR and pre-1.0 bump control | `release-type`, `bump-minor-pre-major`, `generate-sbom` |
| `goreleaser.yml` | GoReleaser binary builds on release | `go-version-file` |
| `container-build.yml` | Buildah multi-arch container build, push & cosign signing | `dockerfile`, `platforms`, `dockerhub-image` |
| `cleanup-container.yml` | Delete branch-tagged container images on branch deletion | `image-name`, `registry` |
| `dependency-review.yml` | PR dependency change review | `fail-on-severity` |
| `trivy.yml` | Filesystem vulnerability scan | `severity`, `scan-type` |
| `semgrep.yml` | Static analysis with autofix and PR comments | `semgrep-config` |
| `scorecard.yml` | OpenSSF Scorecard analysis with SARIF upload | `publish-results` |
| `cleanup-preview.yml` | Vercel preview deployment cleanup | `production-keep-count` |
| `renovate.yml` | Self-hosted Renovate dependency updates | `dry-run`, `log-level` |
| `sync-upstream.yml` | Auto-sync private mirrors from upstream | _(schedule/manual)_ |

## Usage

Create thin caller workflows in your repo.

### PR checks

```yaml
# .github/workflows/pr.yml
name: PR
on:
  pull_request:
    branches: [main]
permissions:
  contents: write
  issues: write
  pull-requests: write
jobs:
  dependency-review:
    uses: krypsis-io/.github/.github/workflows/dependency-review.yml@main
  trivy:
    uses: krypsis-io/.github/.github/workflows/trivy.yml@main
  semgrep:
    uses: krypsis-io/.github/.github/workflows/semgrep.yml@main
```

### Release

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]
permissions:
  contents: write
  issues: write
  pull-requests: write
jobs:
  release:
    uses: krypsis-io/.github/.github/workflows/release.yml@main
    secrets:
      APP_ID: ${{ secrets.APP_ID }}
      APP_PRIVATE_KEY: ${{ secrets.APP_PRIVATE_KEY }}
```

Runs semantic-release to determine version bumps from conventional commits, generates SBOM via Trivy, and creates a GitHub release. App credentials are optional — falls back to `GITHUB_TOKEN`.

### Release (release-please)

An alternative to `release.yml`. Both are supported; pick one per repo.

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]
permissions:
  contents: write
  issues: write
  pull-requests: write
jobs:
  release:
    uses: krypsis-io/.github/.github/workflows/release-please.yml@main
    with:
      release-type: go
    secrets:
      APP_ID: ${{ secrets.APP_ID }}
      APP_PRIVATE_KEY: ${{ secrets.APP_PRIVATE_KEY }}
```

Releases are two-phase. A push to the target branch opens or updates a **Release PR** whose title carries the pending version and whose body is the pending changelog. Nothing is tagged until you merge it, so the version is reviewable before it exists.

Choose this over `release.yml` when either matters:

- **The version needs a review gate.** semantic-release tags on merge to `main`, so a `BREAKING CHANGE:` footer buried in a squashed PR body bumps the major with no chance to catch it.
- **The repo is pre-1.0 and should stay there.** `bump-minor-pre-major` (default `true`) maps a breaking change onto a minor bump while the version is `0.x`. semantic-release has no equivalent and will promote `0.x` straight to `1.0.0`.

Outputs match `release.yml` (`new-release-published`, `new-release-version`), so downstream jobs like `container-build.yml` need no changes — they just fire on the run that merges the Release PR rather than on the feature merge. Two extra outputs are available: `new-release-tag` and `release-pr-created`.

#### Configuration

release-please reads `release-please-config.json` and `.release-please-manifest.json` **from the branch over the GitHub API**, not from the workflow's checkout, so both must be committed. On first run this workflow writes them for you from the org defaults, seeding the manifest from the repo's newest `v*` tag so an existing version line continues rather than restarting at `0.0.0`. After that the repo owns both files and may override any key; the `with:` inputs only ever affect bootstrap. Set `bootstrap-config: false` to require the repo to supply them itself.

Note that `release-type` is passed to bootstrap only and is deliberately **not** forwarded to the action — supplying it there would switch release-please to `Manifest.fromConfig()`, which ignores the committed config file entirely, and `bump-minor-pre-major` exists nowhere but that file.

#### GitHub App permissions

App credentials are effectively required here, unlike `release.yml`: the default `GITHUB_TOKEN` cannot trigger downstream workflows when the Release PR merges, so a `release: published` consumer such as `goreleaser.yml` would never fire.

The App needs more than `release.yml` did. semantic-release only ever pushed commits and tags, so a `Contents`-only App was sufficient; release-please additionally opens and labels a PR:

| App permission | Why |
|----------------|-----|
| `Contents: Read and write` | Push the release branch, create tags and releases, commit the bootstrap files |
| `Pull requests: Read and write` | Open and update the Release PR |
| `Issues: Read and write` | Apply the `autorelease: pending` / `autorelease: tagged` labels, which go through the issues API |

Granting `Contents` alone fails partway through, after the release branch already exists:

```text
✔ Successfully updated reference release-please--branches--main to <sha>
##[error]release-please failed: Resource not accessible by integration
         https://docs.github.com/rest/pulls/pulls#create-a-pull-request
```

Grant the permissions at `https://github.com/settings/apps/<app-slug>/permissions`, then approve the pending permission request on each installation at `https://github.com/settings/installations` — the grant does not take effect until the installation accepts it.

The job-level `permissions:` block in the caller does not help here. When the App token is present it supersedes `GITHUB_TOKEN`, and the App's own permissions govern.

The workflow mints the token with `permission-contents`, `permission-issues`, and `permission-pull-requests` set to `write` and nothing else, so an App that holds broader permissions for other workflows does not pass them to this job. The trade-off is that requesting a permission the App lacks fails token minting immediately — which is the better failure, since it names the missing permission up front instead of surfacing as `Resource not accessible by integration` after the release branch already exists.

The checkout runs with `persist-credentials: false`. The bootstrap step, the only one that pushes, authenticates that push on its own; nothing else in the job needs a git credential, since release-please works entirely over the API.

#### Changelog structure

New entries are inserted by matching this regex against the existing file:

```js
const DEFAULT_VERSION_HEADER_REGEX = '\n###? v?[0-9[]';
```

Note the leading `\n`. **A version heading on the very first line of the file has no newline before it and will not match**, so the new entry is inserted before the *second* heading instead of at the top — silently, and only visibly wrong once you have two entries.

A `CHANGELOG.md` must therefore start with a title line. release-please writes `# Changelog` when it creates the file itself; keep it:

```markdown
# Changelog

## [0.4.0](...) (2026-08-02)
...
```

This matters when migrating a repo off `release.yml`. semantic-release writes its newest entry at line 1 with no title, and formats minor/major entries as `#` rather than `##`. Before the first release-please run, prepend `# Changelog` and normalise any `# [x.y.z]` headings to `## [x.y.z]` — the regex only matches `##` or `###`.

Sections come from `changelog-sections`. Types default to hidden unless listed with `"hidden": false`, and **un-hiding a type also makes it release-triggering** — with the config below, a `ci:`-only change cuts a patch release, which is not the default behaviour.

#### Common config overrides

Everything below is a key in the repo's `release-please-config.json`. Root-level keys apply to all packages; per-package keys go under `packages["."]`.

| Key | Level | Effect |
|-----|-------|--------|
| `bump-minor-pre-major` | root | While `0.x`, a breaking change bumps minor instead of promoting to `1.0.0`. Org default `true`. |
| `bump-patch-for-minor-pre-major` | root | While `0.x`, a `feat` bumps patch instead of minor. Org default `false`. |
| `pull-request-title-pattern` | root | Release PR title. Org default `chore: release ${version}`; upstream default is `chore(${scope}): release ${version}`. |
| `pull-request-header` | root | First line of the Release PR body. Org default replaces upstream's `:robot: I have created a release *beep* *boop*`. |
| `pull-request-footer` | root | Last line of the body. Left at the upstream "generated with Release Please" credit. |
| `changelog-sections` | root | Section names, ordering, and which commit types appear at all. |
| `release-as` | package | Force an exact next version, ignoring what the commits imply. |
| `extra-files` | package | Stamp the version into arbitrary files. |
| `changelog-path` | package | Defaults to `CHANGELOG.md`. |

```json
{
  "$schema": "https://raw.githubusercontent.com/googleapis/release-please/main/schemas/config.json",
  "bump-minor-pre-major": true,
  "bump-patch-for-minor-pre-major": false,
  "pull-request-title-pattern": "chore: release ${version}",
  "pull-request-header": "Release notes below. Merge this PR to tag the release and publish it.",
  "changelog-sections": [
    { "type": "feat", "section": "Features" },
    { "type": "fix", "section": "Bug Fixes" },
    { "type": "perf", "section": "Performance" },
    { "type": "ci", "section": "CI/CD", "hidden": false },
    { "type": "docs", "section": "Documentation", "hidden": false },
    { "type": "chore", "section": "Chores", "hidden": true }
  ],
  "packages": {
    ".": {
      "release-type": "go",
      "changelog-path": "CHANGELOG.md",
      "extra-files": [{ "type": "generic", "path": "main.go" }]
    }
  }
}
```

`extra-files` with the `generic` updater rewrites any line carrying an `x-release-please-version` annotation, which covers the usual Go and shell cases:

```go
var version = "0.4.0" // x-release-please-version
```

**The header and footer are not templated.** `${version}` is substituted in `pull-request-title-pattern` but not in `pull-request-header` or `pull-request-footer`, which are emitted verbatim (`PullRequestBody.toString`) — put a literal `${version}` in the header and that is exactly what renders. The version is already visible in the PR title and in the changelog heading inside the body. Note also that both fall back to the upstream default on any falsy value (`options?.header || DEFAULT_HEADER`), so `""` restores the robot text rather than suppressing it; use a short string instead.

**`release-as` is sticky.** It forces that exact version on *every* subsequent run, including after that version is already tagged — you get a fresh Release PR proposing a version that exists. Remove the key once the forced release is cut; the next run then recovers to a computed version on its own.

Pushing more commits while a Release PR is open **updates that PR in place** rather than opening another, so the open PR always reflects the full set of unreleased changes.

### GoReleaser (Go projects)

```yaml
# .github/workflows/goreleaser.yml
name: GoReleaser
on:
  release:
    types: [published]
permissions:
  contents: write
jobs:
  goreleaser:
    uses: krypsis-io/.github/.github/workflows/goreleaser.yml@main
```

Requires a `.goreleaser.yml` in the repo root. Builds multi-arch Go binaries and uploads them to the GitHub release created by semantic-release.

Example `.goreleaser.yml`:

```yaml
version: 2
project_name: my-tool
builds:
  - main: ./cmd/my-tool
    binary: my-tool
    env:
      - CGO_ENABLED=0
    goos: [linux, darwin]
    goarch: [amd64, arm64]
    ldflags:
      - -s -w
      - -X main.version={{ .Version }}
      - -X main.commit={{ .ShortCommit }}
      - -X main.date={{ .Date }}
archives:
  - format: tar.gz
    name_template: "{{ .ProjectName }}_{{ .Version }}_{{ .Os }}_{{ .Arch }}"
checksum:
  name_template: checksums.txt
changelog:
  disable: true
```

### Container build (Buildah)

```yaml
# .github/workflows/container-build.yml
name: Container Build
on:
  release:
    types: [published]
permissions:
  contents: read
  packages: write
  id-token: write
jobs:
  build:
    uses: krypsis-io/.github/.github/workflows/container-build.yml@main
    with:
      dockerfile: deploy/docker/Dockerfile
      platforms: linux/amd64,linux/arm64
```

Rootless Buildah build, multi-arch manifest, pushes to GHCR, and signs with cosign.

#### With Docker Hub

```yaml
jobs:
  build:
    uses: krypsis-io/.github/.github/workflows/container-build.yml@main
    with:
      dockerfile: deploy/docker/Dockerfile
      platforms: linux/amd64,linux/arm64
      dockerhub-image: docker.io/user/repo
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

### Container cleanup

```yaml
# .github/workflows/cleanup-container.yml
name: Cleanup Container Images
on:
  delete:
jobs:
  cleanup:
    if: github.event.ref_type == 'branch'
    uses: krypsis-io/.github/.github/workflows/cleanup-container.yml@main
```

Deletes branch-tagged container images from GHCR when a branch is deleted.

### OpenSSF Scorecard

```yaml
# .github/workflows/scorecard.yml
name: Scorecard
on:
  push:
    branches: [main]
  schedule:
    - cron: "0 6 * * 1"
permissions:
  contents: read
  security-events: write
  id-token: write
  actions: read
jobs:
  scorecard:
    uses: krypsis-io/.github/.github/workflows/scorecard.yml@main
```

### Vercel cleanup

```yaml
# .github/workflows/cleanup-preview.yml
name: Cleanup Deployments
on:
  pull_request:
    types: [closed]
jobs:
  cleanup:
    uses: krypsis-io/.github/.github/workflows/cleanup-preview.yml@main
    secrets:
      VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
      VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
```

### Renovate (self-hosted)

For orgs that prefer not to use the public Renovate GitHub App (e.g., to avoid private repo access by third parties), this workflow runs Renovate entirely within your own GitHub Actions runner.

```yaml
# .github/workflows/renovate.yml
name: Renovate
on:
  schedule:
    - cron: "0 4 * * *"
  workflow_dispatch:
jobs:
  renovate:
    uses: krypsis-io/.github/.github/workflows/renovate.yml@main
    secrets: inherit
```

**How it works:**

- When run from a `.github` repo, Renovate autodiscovers all repos in the org (excluding `.github` itself)
- When called from a single repo, it scans only that repo
- Repos without a Renovate config receive an onboarding PR
- Skipped in `krypsis-io/.github` — only activates in downstream orgs

**GitHub App permissions required:**

The app referenced by `APP_ID` / `APP_PRIVATE_KEY` must have these **repository permissions**:

| Permission | Access | Why |
|------------|--------|-----|
| Contents | Read & Write | Read dependency files, create update branches |
| Pull requests | Read & Write | Open and manage dependency update PRs |
| Issues | Read & Write | Onboarding issues and dependency notices |
| Checks | Read | Read CI status before automerging |
| Metadata | Read | Repository discovery (always granted) |

The app must be installed on every repo Renovate should manage.

### Upstream sync (private mirrors)

The `sync-upstream.yml` workflow automatically keeps private mirrors in sync with this repo. It runs weekly (Mondays at 6am UTC) and supports manual trigger.

- Skipped in the upstream repo (`krypsis-io/.github`) — only activates in mirrors
- Uses `git reset --hard` and force push to ensure the mirror is an exact copy of upstream
- Requires a GitHub App token (`APP_ID` and `APP_PRIVATE_KEY` secrets) with Contents and Workflows write permissions to push workflow file changes

No configuration needed — it's included automatically when you mirror the repo.

## Private mirrors

GitHub doesn't allow private forks of public repos. To use these workflows in a private org, create a mirror:

```bash
# 1. Create an empty private repo in your org
gh repo create your-org/.github --private --description "Shared GitHub Actions workflows"

# 2. Bare clone and mirror push
git clone --bare https://github.com/krypsis-io/.github.git /tmp/.github-bare
cd /tmp/.github-bare
git push --mirror https://github.com/your-org/.github.git
rm -rf /tmp/.github-bare

# 3. Clone a working copy and add upstream for manual syncs
gh repo clone your-org/.github /tmp/.github
cd /tmp/.github
git remote add upstream https://github.com/krypsis-io/.github.git
```

Your repos then reference the mirror instead of the upstream:

```yaml
jobs:
  release:
    uses: your-org/.github/.github/workflows/release.yml@main
```

The sync and Renovate workflows require a GitHub App installed on your org. At minimum it needs **Contents** and **Workflows** write permissions (sync), plus **Pull requests**, **Issues**, and **Checks** read permissions (Renovate). Add the app credentials as repo secrets:

- `APP_ID` — the GitHub App's Client ID
- `APP_PRIVATE_KEY` — the GitHub App's private key (`.pem` file contents)

The included `sync-upstream.yml` workflow will keep the mirror up to date automatically (weekly on Mondays). You can also trigger it manually from the Actions tab.

> **Note:** The sync does a hard reset to upstream, so any changes made directly to the mirror's `.github` repo will be overwritten. Place org-specific workflows in individual repos instead.

## Private repo compatibility

Workflows that require public repo features are automatically gated:

| Workflow | Behavior in private repos |
|----------|--------------------------|
| `dependency-review.yml` | Skipped |
| `release.yml` (SBOM step) | Skipped |
| `release-please.yml` (SBOM step) | Skipped |
| `scorecard.yml` | Skipped |
| `release-self.yml` | Skipped (only runs in `krypsis-io/.github`) |

All other workflows work in both public and private repos.

## Overriding Defaults

All workflows accept optional inputs with sensible defaults:

```yaml
jobs:
  scan:
    uses: krypsis-io/.github/.github/workflows/trivy.yml@main
    with:
      severity: "CRITICAL"
```

## Testing Changes

1. Push to a feature branch in this repo
2. Point a consuming repo at the branch: `@my-branch` instead of `@main`
3. Open a PR in the consuming repo to trigger it
4. After validating, merge here, revert the consuming repo back to `@main`
