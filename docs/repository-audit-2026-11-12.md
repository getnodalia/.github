# getnodalia repository engineering audit

Snapshot: 12 November 2026. Scope: all 23 repositories returned by GitHub organization search, including public repositories and the accessible private demo repository. Workflow and tree inventory was read through GitHub APIs; branch protection endpoints were unavailable to the connected GitHub App (403), so an empty ruleset response is not proof that no legacy protection exists.

## Inventory and baseline

| Repository | Default branch | Stack / role | Existing quality and release surface | Priority / observations |
|---|---|---|---|---|
| [lienzo](https://github.com/getnodalia/lienzo) | `nodalia` | Rust desktop/image editor; multi-platform | Extensive Cargo tests; CI, packaging lint, SonarCloud, Linux/macOS/Windows releases | Critical active app. Linux release succeeded; Windows/macOS artifacts built and smoke-tested, but release attachment failed because tag `v0.1.0` points to a different commit. Do not change release flow as part of security rollout. |
| [vinilo](https://github.com/getnodalia/vinilo) | `main` | Rust + Swift desktop audio app | CI, deterministic gates, custom release | Linux/macOS Rust jobs pass on main. A macOS Swift-native job failed on a feature branch: `DaemonSupervisor.observer` is non-Sendable in a nonisolated deinit. |
| [pluma](https://github.com/getnodalia/pluma) | `master` | Rust desktop publishing app | Extensive tests; CI, deterministic gates, SonarCloud, release/packaging/platform workflows | Active app; current dependency-update runs were in progress during audit. |
| [nodalix-os](https://github.com/getnodalia/nodalix-os) | `main` | OS scripts and apps: Python, JavaScript, C/C++, shell | CI, deterministic gates, kernel, NxPM, package repository publication, release | **Critical.** Default CI and quality gates passed. Existing ISO/kernel/NxPM/publication flows were left untouched. |
| [vortia](https://github.com/getnodalia/vortia) | `main` | Rust + scientific/desktop components | Rust CI, deterministic gates, AppImage release | **Critical scientific app.** Default Rust/CI checks passed. Numerical reference/convergence testing is a documented follow-up; do not mix benchmarks with correctness tests. |
| [forja](https://github.com/getnodalia/forja) | `main` | Rust desktop CAD app | CI, SonarCloud, packaging and multi-platform release | Active app. SonarCloud and CI passed on a recent dependency branch; main release pipeline failed in shared Git-dependency setup. A separate feature branch failed `cargo xtask ci` at fmt. |
| [lumbre](https://github.com/getnodalia/lumbre) | `main` | Rust local AI/model agent | One CI workflow; unit/API tests | Experimental/new project. Main CI passed. No large-model download should be added to CI. |
| [celdas](https://github.com/getnodalia/celdas) | `main` | Rust spreadsheet app | CI, SonarCloud, release/packaging/platform workflows | Active app; dependency update checks were in progress. |
| [nodalia-web](https://github.com/getnodalia/nodalia-web) | `main` | JavaScript/TypeScript static sites | Validate/deploy workflow and SonarCloud; no test files found in tree inventory | Site deployment checks passed on main. Keep deployment restricted to its current default-branch flow. |
| [papel](https://github.com/getnodalia/papel) | `main` | Rust desktop/PDF app | CI, SonarCloud, release/packaging/platform workflows | Active app; current main CI history included successful dependency-update jobs. |
| [pulso](https://github.com/getnodalia/pulso) | `main` | Rust desktop audio app | Linux/macOS CI, deterministic gates, SonarCloud, release/packaging | Active app; latest default-branch dependency checks were mixed/in progress. |
| [tienda](https://github.com/getnodalia/tienda) | `main` | C/C++ Meson desktop app | Build workflow; small test footprint; Sonar config present | Build passed on main. Confirm Sonar project onboarding and run its tests in CI before treating them as gates. |
| [torno](https://github.com/getnodalia/torno) | `main` | Large Rust desktop app/workspace | CI, PR checks, packaging lint, claim/triage automation, extensive tests and release files | Active app. Root is a Cargo workspace. A recursive tree fetch failed due response size, so deep file-by-file timing/test audit remains incomplete. |
| [demo-repository](https://github.com/getnodalia/demo-repository) | `main` | Private HTML/JavaScript demo | Proof-HTML and auto-assign workflows | Demo, no tests found. Auto-assign failed on a workflow context without issue data and passes an obsolete `numOfAssignee` input. Private repository rulesets are not available on GitHub Free. |
| [getnodalia/.github](https://github.com/getnodalia/.github) | `main` | Organization profile/community metadata | No CI or tests | This repository is being used for the shared standard and audit docs. |
| [luz](https://github.com/getnodalia/luz) | `main` | Rust desktop image editor | Linux/Windows CI, deterministic gates, SonarCloud, release/packaging | Active app. A Dependabot cargo update failed because the dependency manifest asks for nonexistent feature `sha2/std`; this is distinct from test results. |
| [nodalix-icon-theme](https://github.com/getnodalia/nodalix-icon-theme) | `main` | Icon/cursor assets and shell installers | No regular CI or tests found | Theme/asset project. Validate install scripts and metadata; CodeQL is limited to its workflow files. |
| [letra](https://github.com/getnodalia/letra) | `main` | Rust desktop/word-processing app | CI, SonarCloud, packaging/release/platform workflows | Active app; routine dependency jobs were active during audit. |
| [nodalia-design](https://github.com/getnodalia/nodalia-design) | `main` | Rust + Node tooling/design system | CI, SonarCloud, release | Main CI passed; retain its current design/package publishing behavior. |
| [pliego](https://github.com/getnodalia/pliego) | `main` | Rust desktop document app | CI, deterministic gates, SonarCloud, packaging/release/platform workflows | Active app. Packaging lint failed on a dependency branch; investigate reusable packaging-lint/hardening dependency before making it required. |
| [brujula](https://github.com/getnodalia/brujula) | `main` | Rust app/workspace | Rust CI and deterministic gates | Main CI and deterministic gates passed. |
| [toma](https://github.com/getnodalia/toma) | `main` | Rust desktop app | CI, deterministic gates, SonarCloud, packaging/release/platform workflows | Active app; dependency checks were mixed/in progress during audit. |
| [lamina](https://github.com/getnodalia/lamina) | `main` | Rust desktop presentation app | CI, SonarCloud, release/packaging/platform workflows | Active app. Packaging lint failed on a GitHub Actions dependency-update branch, with the reusable lint/harden-runner service timing out. |

The large Rust applications share many workflow conventions and reference shared `craft-common-actions`; this indicates a related family of projects, but does not by itself establish their upstream/fork status. Existing releases remain custom/tag-based or cargo-dist-like. No Release Please workflow was added because it would create competing version/tag/changelog publication paths.

## What was verified

- Recent default-branch CI/quality runs were green for Nodalix OS, Vortia, Lumbre, Nodalia Web, Tienda, Nodalia Design, and Brújula.
- Vinilo's latest main CI and deterministic gates were green.
- Forja's latest CI and SonarCloud runs on a dependency branch were green. Its separate release failure is not a test failure.
- Actions for the newly proposed CodeQL workflows are now present on 21 public code/asset repositories. Their first PR runs are being checked; they are not required.
- Scorecard workflows are scheduled and run on default-branch pushes, not ordinary PRs, to keep PR permissions minimal.
- No Codecov workflow or real coverage upload was present in the audited workflow inventory. No coverage thresholds were added.
- SonarCloud workflow/config files exist in multiple projects, but account tokens, PR decoration, project bindings, and Quality Gate status cannot be proven solely from repository files.
- GitHub ruleset list responses were empty for the accessible public repositories. The connected GitHub App cannot read legacy branch protection due admin-scope 403s. Web settings access was unavailable in this session, so no ruleset or branch-protection change is claimed.
- Default-branch workflow durations and step-level timing were not collected consistently before this rollout. Do not claim performance improvements from this snapshot.

## PR rollout

These PRs add CodeQL and OpenSSF Scorecard workflows only. They do not change tests, publication, or required-check settings.

| Repository | PR | Repository | PR |
|---|---|---|---|
| lienzo | [#24](https://github.com/getnodalia/lienzo/pull/24) | vinilo | [#38](https://github.com/getnodalia/vinilo/pull/38) |
| pluma | [#21](https://github.com/getnodalia/pluma/pull/21) | nodalix-os | [#91](https://github.com/getnodalia/nodalix-os/pull/91) |
| vortia | [#92](https://github.com/getnodalia/vortia/pull/92) | forja | [#12](https://github.com/getnodalia/forja/pull/12) |
| lumbre | [#3](https://github.com/getnodalia/lumbre/pull/3) | celdas | [#9](https://github.com/getnodalia/celdas/pull/9) |
| nodalia-web | [#25](https://github.com/getnodalia/nodalia-web/pull/25) | papel | [#11](https://github.com/getnodalia/papel/pull/11) |
| pulso | [#13](https://github.com/getnodalia/pulso/pull/13) | tienda | [#8](https://github.com/getnodalia/tienda/pull/8) |
| torno | [#8](https://github.com/getnodalia/torno/pull/8) | luz | [#13](https://github.com/getnodalia/luz/pull/13) |
| nodalix-icon-theme | [#7](https://github.com/getnodalia/nodalix-icon-theme/pull/7) | letra | [#15](https://github.com/getnodalia/letra/pull/15) |
| nodalia-design | [#15](https://github.com/getnodalia/nodalia-design/pull/15) | pliego | [#11](https://github.com/getnodalia/pliego/pull/11) |
| brujula | [#7](https://github.com/getnodalia/brujula/pull/7) | toma | [#14](https://github.com/getnodalia/toma/pull/14) |
| lamina | [#9](https://github.com/getnodalia/lamina/pull/9) |  |  |

## Manual configuration and remaining work

1. **Codecov:** Onboard only active code projects with genuine coverage output. Add Codecov to the same test job that produces LCOV/Cobertura; tokenless upload availability and free OSS eligibility must be confirmed in Codecov. For private `demo-repository`, do not enable a paid plan. No coverage check is currently required.
2. **SonarQube Cloud:** Confirm project onboarding/free OSS terms, token/connection, project keys, PR decoration, and Quality Gate for each existing Sonar workflow. Add supported repositories lacking a Sonar workflow only after a valid scan can be produced.
3. **Rulesets:** Repository rulesets/branch protection are free for public repositories, but organization-wide rulesets require GitHub Team/Enterprise. Create public repo rulesets individually after review of the default branch, test checks, release exceptions, and existing protections. The connector used here lacks admin scope and browser settings were unavailable, so no ruleset was changed. Private demo protection requires a paid GitHub plan; do not upgrade.
4. **Secret scanning/dependency alerts:** Check each repository's GitHub security settings and enable free features where available. These account settings are not exposed by the connected app.
5. **Renovate/Dependabot:** Several repositories show Renovate branches and Dependabot config/runs. Choose one bot per ecosystem per repository, then set grouping and PR limits in the repository/app configuration.
6. **Coverage and CI timing:** Gather baseline job durations and coverage from stable runs before optimizing caches or setting thresholds. Current new-PR runs must finish before declaring CodeQL successful.
7. **Nodalix OS, Vortia, Lumbre:** Add project-specific work only in dedicated PRs after this foundational review: safe reuse of OS release artifacts; Vortia's validated numerical references/tolerances; Lumbre's tiny-fixture resource/model lifecycle, CPU fallback and graceful degradation.
