# [1.1.0](https://github.com/cwaits6/.github/compare/v1.0.0...v1.1.0) (2026-03-16)


### Bug Fixes

* **container-build:** add docker login for cosign signature push ([ccebc90](https://github.com/cwaits6/.github/commit/ccebc90003203710b82ced067d5801e30aaa64e6))
* **container-build:** install cosign, sign by digest via --digestfile ([643d653](https://github.com/cwaits6/.github/commit/643d653f6720b9c4cdaae1a4637b781384ee35b8))
* **container-build:** use ubuntu-24.04-arm runner label ([d8b76ef](https://github.com/cwaits6/.github/commit/d8b76ef90a2651f74cc2a281089dc8b0ea83d846))
* detect new releases via tag comparison, gate features for private repos ([b7c20e1](https://github.com/cwaits6/.github/commit/b7c20e16f786c5e891bfd12a8516e7cf3bf25ac5))
* install QEMU for cross-platform container builds ([9479417](https://github.com/cwaits6/.github/commit/94794173e4ca6e1915470bc565f885ca80b66269))
* remove top-level permissions from reusable workflows ([38d5abd](https://github.com/cwaits6/.github/commit/38d5abd624bf35bec48371fd495f5657549c9d03))


### Features

* add Renovate config for GitHub Actions version updates ([0a02387](https://github.com/cwaits6/.github/commit/0a0238705660fa6a0ca9c8bef60bcbce3d88d500))
* **container-build:** add cosign keyless signing ([b65ef70](https://github.com/cwaits6/.github/commit/b65ef70c652e7b3fae54f86737171f6e93693cdb))
* **container-build:** use native ARM runners with QEMU fallback ([c7b1cb4](https://github.com/cwaits6/.github/commit/c7b1cb43e6078241edbf2fc2a39d5deca4089435))
* **release:** surface outputs for downstream job chaining ([a4b8543](https://github.com/cwaits6/.github/commit/a4b8543c04c98e19a51a3d7b876e5f79d810fcb3))
* support release and push triggers for container builds ([441ed9f](https://github.com/cwaits6/.github/commit/441ed9f0ae48edde8c5f666e358eeff90b9b30fb))
* tag container builds with branch name for PRs ([a29ef44](https://github.com/cwaits6/.github/commit/a29ef448a02b3126064efc5d025268e9fbb2b1d9))

# 1.0.0 (2026-03-16)


### Features

* add Go binary builds to release, add Buildah container workflow ([d7456c2](https://github.com/cwaits6/.github/commit/d7456c25f633f84a98bcb3f8a4e845d84dab9fe3))
* add shared reusable GitHub Actions workflows ([8f5252b](https://github.com/cwaits6/.github/commit/8f5252b3f95b5a9ea928f05a675bfa7b3a75996a))
