# .github

Shared reusable GitHub Actions workflows for `cwaits6` repos.

## Available Workflows

| Workflow | Description | Key Inputs |
|----------|-------------|------------|
| `dependency-review.yml` | PR dependency change review | `fail-on-severity` |
| `trivy.yml` | Filesystem vulnerability scan | `severity`, `scan-type` |
| `release.yml` | Semantic-release + SBOM + Go binary builds | `go-main-package`, `go-ldflags-prefix`, `go-binary-name` |
| `container-build.yml` | Buildah multi-arch container build & push | `dockerfile`, `platforms`, `build-args` |
| `cleanup-preview.yml` | Vercel preview deployment cleanup | `production-keep-count` |
| `semgrep.yml` | Static analysis with autofix and PR comments | `semgrep-config` |

## Usage

Create thin caller workflows in your repo:

### PR checks

```yaml
# .github/workflows/pr.yml
name: PR
on:
  pull_request:
    branches: [main]
jobs:
  dependency-review:
    uses: cwaits6/.github/.github/workflows/dependency-review.yml@main
  trivy:
    uses: cwaits6/.github/.github/workflows/trivy.yml@main
  semgrep:
    uses: cwaits6/.github/.github/workflows/semgrep.yml@main
```

### Release (Node.js project)

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]
jobs:
  release:
    uses: cwaits6/.github/.github/workflows/release.yml@main
```

### Release (Go project)

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]
jobs:
  release:
    uses: cwaits6/.github/.github/workflows/release.yml@main
    with:
      go-binary-name: my-tool
      go-main-package: ./cmd/my-tool
      go-ldflags-prefix: github.com/cwaits6/my-tool/cmd/my-tool/cmd
```

Auto-detects `go.mod` — if present, builds multi-arch binaries (linux/darwin, amd64/arm64) and uploads them to the GitHub release. If no `go.mod`, runs semantic-release only.

### Container build (Buildah)

```yaml
# .github/workflows/container-build.yml
name: Container Build
on:
  push:
    tags: ["v*"]
jobs:
  build:
    uses: cwaits6/.github/.github/workflows/container-build.yml@main
    with:
      dockerfile: deploy/docker/Dockerfile
      platforms: linux/amd64,linux/arm64
```

Rootless Buildah build, multi-arch manifest, pushes to GHCR by default.

### Vercel cleanup

```yaml
# .github/workflows/cleanup-preview.yml
name: Cleanup Deployments
on:
  pull_request:
    types: [closed]
jobs:
  cleanup:
    uses: cwaits6/.github/.github/workflows/cleanup-preview.yml@main
    secrets:
      VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
      VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
```

## Overriding Defaults

```yaml
jobs:
  scan:
    uses: cwaits6/.github/.github/workflows/trivy.yml@main
    with:
      severity: "CRITICAL"
```

## Testing Changes

1. Push to a feature branch in this repo
2. Point a consuming repo at the branch: `@my-branch` instead of `@main`
3. Open a PR in the consuming repo to trigger it
4. After validating, merge here, revert the consuming repo back to `@main`
