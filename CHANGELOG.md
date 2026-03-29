# [1.7.0](https://github.com/krypsis-io/.github/compare/v1.6.3...v1.7.0) (2026-03-29)


### Bug Fixes

* install conventional commits preset for releases ([#16](https://github.com/krypsis-io/.github/issues/16)) ([7e8ac13](https://github.com/krypsis-io/.github/commit/7e8ac135572240fd147dc422957375a112b2b67d))
* use local paths for self-test workflows ([#17](https://github.com/krypsis-io/.github/issues/17)) ([4250b67](https://github.com/krypsis-io/.github/commit/4250b67984b37f4621b6c1d7187502da170cb96c))


### Features

* generate app token for Homebrew tap in goreleaser workflow ([#15](https://github.com/krypsis-io/.github/issues/15)) ([fb2e7a6](https://github.com/krypsis-io/.github/commit/fb2e7a6cd35feffe083017dbdfdc29042b8e99c0))

## [1.6.3](https://github.com/krypsis-io/.github/compare/v1.6.2...v1.6.3) (2026-03-24)


### Bug Fixes

* pass App token to checkout for git push bypass ([#12](https://github.com/krypsis-io/.github/issues/12)) ([3a6cb0d](https://github.com/krypsis-io/.github/commit/3a6cb0d5173b43e2f206cad16cab26dc200c0061))

## [1.6.2](https://github.com/krypsis-io/.github/compare/v1.6.1...v1.6.2) (2026-03-24)


### Bug Fixes

* use App token for git push in release workflow ([#11](https://github.com/krypsis-io/.github/issues/11)) ([ded1387](https://github.com/krypsis-io/.github/commit/ded1387aa497110003ee1195ec8a9f6417cbcd69))

## [1.6.1](https://github.com/krypsis-io/.github/compare/v1.6.0...v1.6.1) (2026-03-24)


### Bug Fixes

* grant semgrep required permissions in pr.yml ([4640192](https://github.com/krypsis-io/.github/commit/46401924479c0acae9bd0b493b10535581ef8dd5))
* pin all actions to SHA commits and suppress Node.js 20 warnings ([635eac4](https://github.com/krypsis-io/.github/commit/635eac485bd1a7787cf4dec53737faf1673016fe))
* remove skip ci from semantic-release commit message ([a53b2b0](https://github.com/krypsis-io/.github/commit/a53b2b0e96f23614a13effbe4232b3cbac8494b2))
* use env vars instead of direct interpolation in run steps ([5a57340](https://github.com/krypsis-io/.github/commit/5a573404aacc399c0488f0bdd0e9d26b03b36ecf))

# [1.6.0](https://github.com/krypsis-io/.github/compare/v1.5.2...v1.6.0) (2026-03-24)


### Features

* remove Go build steps from release workflow ([#9](https://github.com/krypsis-io/.github/issues/9)) ([ecb699d](https://github.com/krypsis-io/.github/commit/ecb699df270990be809b39bfcb8b9c08b786ff88))

## [1.5.2](https://github.com/krypsis-io/.github/compare/v1.5.1...v1.5.2) (2026-03-19)


### Bug Fixes

* move scorecard write permissions to job level ([#7](https://github.com/krypsis-io/.github/issues/7)) ([1bdc765](https://github.com/krypsis-io/.github/commit/1bdc76598a12bc9c4831de0def4da141908599df))

## [1.5.1](https://github.com/krypsis-io/.github/compare/v1.5.0...v1.5.1) (2026-03-19)


### Bug Fixes

* pin scorecard-action to v2.4.3 ([#6](https://github.com/krypsis-io/.github/issues/6)) ([6785415](https://github.com/krypsis-io/.github/commit/6785415118f11f24f1a29f4cc247b476de92406f))

# [1.5.0](https://github.com/krypsis-io/.github/compare/v1.4.0...v1.5.0) (2026-03-19)


### Features

* add reusable OpenSSF Scorecard workflow ([#5](https://github.com/krypsis-io/.github/issues/5)) ([5b69976](https://github.com/krypsis-io/.github/commit/5b6997646658146953d52205e809e62ae0459b47))

# [1.4.0](https://github.com/krypsis-io/.github/compare/v1.3.0...v1.4.0) (2026-03-19)


### Features

* add optional Docker Hub publishing to container build workflow ([#4](https://github.com/krypsis-io/.github/issues/4)) ([ac16452](https://github.com/krypsis-io/.github/commit/ac164521e1ae6ca88b4a4de43c93cd2709763d61))

# [1.3.0](https://github.com/krypsis-io/.github/compare/v1.2.1...v1.3.0) (2026-03-18)


### Bug Fixes

* use env var check for app credentials instead of secrets context ([#3](https://github.com/krypsis-io/.github/issues/3)) ([2f46251](https://github.com/krypsis-io/.github/commit/2f46251c3e730de758b945f8ed5e1b0ec61d2d49))


### Features

* support GitHub App token for triggering downstream workflows ([#2](https://github.com/krypsis-io/.github/issues/2)) ([2b1360b](https://github.com/krypsis-io/.github/commit/2b1360b800dc67e120d286c122480ec0a37a64df))

## [1.2.1](https://github.com/krypsis-io/.github/compare/v1.2.0...v1.2.1) (2026-03-16)


### Bug Fixes

* **release:** use single Trivy step for SBOM dependency submission ([1e8b592](https://github.com/krypsis-io/.github/commit/1e8b5920fa72c05beb9fd7ba43495c4fd0d38c2d))

# [1.2.0](https://github.com/krypsis-io/.github/compare/v1.1.2...v1.2.0) (2026-03-16)


### Features

* add container image cleanup workflow for deleted branches ([1e1869c](https://github.com/krypsis-io/.github/commit/1e1869c3e15699a9da93072989d5762ed8a7779a))

## [1.1.2](https://github.com/krypsis-io/.github/compare/v1.1.1...v1.1.2) (2026-03-16)


### Bug Fixes

* **release:** disable go cache when go.sum missing ([2d3a59b](https://github.com/krypsis-io/.github/commit/2d3a59bbda43924b91f5596e76c87c53782efada))

## [1.1.1](https://github.com/krypsis-io/.github/compare/v1.1.0...v1.1.1) (2026-03-16)


### Bug Fixes

* **release:** bump setup-go to v6, sbom-action to v0.23.1 ([8adfbdf](https://github.com/krypsis-io/.github/commit/8adfbdfd8f5aea63f66ae0d8f250cff35d885bd4))

# [1.1.0](https://github.com/krypsis-io/.github/compare/v1.0.0...v1.1.0) (2026-03-16)


### Bug Fixes

* **container-build:** add docker login for cosign signature push ([ccebc90](https://github.com/krypsis-io/.github/commit/ccebc90003203710b82ced067d5801e30aaa64e6))
* **container-build:** install cosign, sign by digest via --digestfile ([643d653](https://github.com/krypsis-io/.github/commit/643d653f6720b9c4cdaae1a4637b781384ee35b8))
* **container-build:** use ubuntu-24.04-arm runner label ([d8b76ef](https://github.com/krypsis-io/.github/commit/d8b76ef90a2651f74cc2a281089dc8b0ea83d846))
* detect new releases via tag comparison, gate features for private repos ([b7c20e1](https://github.com/krypsis-io/.github/commit/b7c20e16f786c5e891bfd12a8516e7cf3bf25ac5))
* install QEMU for cross-platform container builds ([9479417](https://github.com/krypsis-io/.github/commit/94794173e4ca6e1915470bc565f885ca80b66269))
* remove top-level permissions from reusable workflows ([38d5abd](https://github.com/krypsis-io/.github/commit/38d5abd624bf35bec48371fd495f5657549c9d03))


### Features

* add Renovate config for GitHub Actions version updates ([0a02387](https://github.com/krypsis-io/.github/commit/0a0238705660fa6a0ca9c8bef60bcbce3d88d500))
* **container-build:** add cosign keyless signing ([b65ef70](https://github.com/krypsis-io/.github/commit/b65ef70c652e7b3fae54f86737171f6e93693cdb))
* **container-build:** use native ARM runners with QEMU fallback ([c7b1cb4](https://github.com/krypsis-io/.github/commit/c7b1cb43e6078241edbf2fc2a39d5deca4089435))
* **release:** surface outputs for downstream job chaining ([a4b8543](https://github.com/krypsis-io/.github/commit/a4b8543c04c98e19a51a3d7b876e5f79d810fcb3))
* support release and push triggers for container builds ([441ed9f](https://github.com/krypsis-io/.github/commit/441ed9f0ae48edde8c5f666e358eeff90b9b30fb))
* tag container builds with branch name for PRs ([a29ef44](https://github.com/krypsis-io/.github/commit/a29ef448a02b3126064efc5d025268e9fbb2b1d9))

# 1.0.0 (2026-03-16)


### Features

* add Go binary builds to release, add Buildah container workflow ([d7456c2](https://github.com/krypsis-io/.github/commit/d7456c25f633f84a98bcb3f8a4e845d84dab9fe3))
* add shared reusable GitHub Actions workflows ([8f5252b](https://github.com/krypsis-io/.github/commit/8f5252b3f95b5a9ea928f05a675bfa7b3a75996a))
