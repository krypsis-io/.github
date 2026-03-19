# [1.5.0](https://github.com/cwaits6/.github/compare/v1.4.0...v1.5.0) (2026-03-19)


### Features

* add reusable OpenSSF Scorecard workflow ([#5](https://github.com/cwaits6/.github/issues/5)) ([5b69976](https://github.com/cwaits6/.github/commit/5b6997646658146953d52205e809e62ae0459b47))

# [1.4.0](https://github.com/cwaits6/.github/compare/v1.3.0...v1.4.0) (2026-03-19)


### Features

* add optional Docker Hub publishing to container build workflow ([#4](https://github.com/cwaits6/.github/issues/4)) ([ac16452](https://github.com/cwaits6/.github/commit/ac164521e1ae6ca88b4a4de43c93cd2709763d61))

# [1.3.0](https://github.com/cwaits6/.github/compare/v1.2.1...v1.3.0) (2026-03-18)


### Bug Fixes

* use env var check for app credentials instead of secrets context ([#3](https://github.com/cwaits6/.github/issues/3)) ([2f46251](https://github.com/cwaits6/.github/commit/2f46251c3e730de758b945f8ed5e1b0ec61d2d49))


### Features

* support GitHub App token for triggering downstream workflows ([#2](https://github.com/cwaits6/.github/issues/2)) ([2b1360b](https://github.com/cwaits6/.github/commit/2b1360b800dc67e120d286c122480ec0a37a64df))

## [1.2.1](https://github.com/cwaits6/.github/compare/v1.2.0...v1.2.1) (2026-03-16)


### Bug Fixes

* **release:** use single Trivy step for SBOM dependency submission ([1e8b592](https://github.com/cwaits6/.github/commit/1e8b5920fa72c05beb9fd7ba43495c4fd0d38c2d))

# [1.2.0](https://github.com/cwaits6/.github/compare/v1.1.2...v1.2.0) (2026-03-16)


### Features

* add container image cleanup workflow for deleted branches ([1e1869c](https://github.com/cwaits6/.github/commit/1e1869c3e15699a9da93072989d5762ed8a7779a))

## [1.1.2](https://github.com/cwaits6/.github/compare/v1.1.1...v1.1.2) (2026-03-16)


### Bug Fixes

* **release:** disable go cache when go.sum missing ([2d3a59b](https://github.com/cwaits6/.github/commit/2d3a59bbda43924b91f5596e76c87c53782efada))

## [1.1.1](https://github.com/cwaits6/.github/compare/v1.1.0...v1.1.1) (2026-03-16)


### Bug Fixes

* **release:** bump setup-go to v6, sbom-action to v0.23.1 ([8adfbdf](https://github.com/cwaits6/.github/commit/8adfbdfd8f5aea63f66ae0d8f250cff35d885bd4))

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
