# .github

Shared reusable GitHub Actions workflows. Designed for use by `krypsis-io` repos but fully generic — private forks (mirrors) can use these workflows by replacing `krypsis-io` with their own org in the `uses:` references below.

All actions are SHA-pinned. Shell injection mitigations applied (env vars instead of direct `${{ }}` interpolation in `run:` blocks).

## Available Workflows

| Workflow | Description | Key Inputs |
|----------|-------------|------------|
| `release.yml` | Semantic-release with SBOM generation | `node-version`, `generate-sbom` |
| `goreleaser.yml` | GoReleaser binary builds on release | `go-version-file` |
| `container-build.yml` | Buildah multi-arch container build, push & cosign signing | `dockerfile`, `platforms`, `dockerhub-image` |
| `cleanup-container.yml` | Delete branch-tagged container images on branch deletion | `image-name`, `registry` |
| `dependency-review.yml` | PR dependency change review | `fail-on-severity` |
| `trivy.yml` | Filesystem vulnerability scan | `severity`, `scan-type` |
| `semgrep.yml` | Static analysis with autofix and PR comments | `semgrep-config` |
| `scorecard.yml` | OpenSSF Scorecard analysis with SARIF upload | `publish-results` |
| `cleanup-preview.yml` | Vercel preview deployment cleanup | `production-keep-count` |
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

### Upstream sync (private mirrors)

The `sync-upstream.yml` workflow automatically keeps private mirrors in sync with this repo. It runs weekly (Mondays at 6am UTC) and supports manual trigger.

- Skipped in the upstream repo (`krypsis-io/.github`) — only activates in mirrors
- Uses `--ff-only` so it fails loudly if the mirror has diverged

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

The included `sync-upstream.yml` workflow will keep the mirror up to date automatically (weekly on Mondays). You can also trigger it manually from the Actions tab or sync manually:

```bash
git fetch upstream && git merge upstream/main --ff-only && git push origin main
```

## Private repo compatibility

Workflows that require public repo features are automatically gated:

| Workflow | Behavior in private repos |
|----------|--------------------------|
| `dependency-review.yml` | Skipped |
| `release.yml` (SBOM step) | Skipped |
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
