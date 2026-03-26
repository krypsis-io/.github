# .github

Shared reusable GitHub Actions workflows for `krypsis-io` repos.

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
