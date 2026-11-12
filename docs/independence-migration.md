# Nodalia independence migration — initial audit

Status: **migration required**, 12 November 2026. This is an operational-dependency audit, not a claim that all inherited source code is original or that migrations are finished.

## Goal

The Nodalia apps are independently branded, operated, maintained and released by Nodalia. Remove required access to `storytold/*` for builds, releases, runtime updates, assets, repository governance, and security contacts, without suppressing licenses or historical attribution.

## Confirmed dependencies (sampled directly from default branches)

| Repository | Active coupling found | Required migration |
| --- | --- | --- |
| `getnodalia/forja` | CI, house-rules, FreeBSD, packaging lint and release call `storytold/craft-common-actions@main`; `Cargo.toml` fetches `craft-libs`, `storytold/egui` and `storytold/winit`; signing and fonts are tied to the shared workflow | Build first-party CI/release workflows, replace or vendor licensed source with immutable provenance, own font and update pipeline |
| `getnodalia/lienzo` | CI, house-rules and release call shared actions; CI fetches `storytold/craft-fonts` and custom apt install actions; release delegates platform builds | Replace CI/release/actions and assets in a tested rollout; keep standalone `release-macos.yml` working |
| `getnodalia/letra` | Security reporting previously pointed to `storytold/wordcraft`; shared Craft infrastructure references | Own security contacts, workflows, dependencies and documentation |
| `getnodalia/toma` | Security contact referenced the ArtCraft community; shared infrastructure references | Own security contacts, workflows, dependencies and documentation |
| `getnodalia/pluma`, `papel`, `pulso`, `pliego`, `celdas`, `luz`, `lamina` | Existing shared Craft CI/asset/dependency references in repository files | Audit each project's actual dependency graph, artifacts, updater and workflows |

Other Nodalia apps may also have dependencies; this table does not certify coverage of all repositories.

## Sequence (no pay-as-you-go services)

1. **Control and security:** point security reporting, default documentation and policies at Nodalia, never upstream. Preserve historical credits and licenses.
2. **CI independence:** implement first-party workflow definitions in `getnodalia` or per-repo; make lint/tests and branch rules run only on free standard GitHub Actions. Do not point a workflow to a new repo until the target workflows/actions exist and their compatibility is verified.
3. **Release independence:** own macOS, Windows, Linux, Flatpak and other release jobs, without `macos-15-xlarge` or upstream shared release workflows. Keep signing, notarization, QA, checksums, provenance and protected draft-publish behavior. Verify reproducible artifacts.
4. **Source dependencies:** systematically identify `git+https://github.com/storytold/*` in `Cargo.toml`, `Cargo.lock`, patches and scripts. Move licensed code to reviewed Nodalia-controlled repos, vendor it with records, or replace it with published independently maintained alternatives. Test ABI/features, license metadata, checksum and lockfile resolution.
5. **Assets and distribution:** migrate pinned font, translation, corpus, registry, website, AppImageUpdate/zsync, release and updater endpoints. Preserve authors and license files; use `ghcr.io/getnodalia` and Nodalia-controlled URLs only where artifacts actually exist.
6. **Policy and automation:** replace generated upstream house rules and `AGENTS.md` blocks with Nodalia-owned rules, remove automated upstream sync jobs once intentional one-time migrations are complete, and inspect all default-branch workflows for external dependencies.
7. **Verification:** run standard CI on clean hosted runners, test installation and update on supported platforms, scan artifacts for old URLs, verify signature and metadata, then merge individually. Do not delete provenance archives and do not advertise independence before testing.

## Explicit non-goals

- Do **not** strip copyright notices, third-party acknowledgements or legally required attribution.
- Do **not** claim a derivative codebase was independently written from scratch.
- Do **not** replace upstream URLs blindly without source/asset compatibility and security checks.
- Do **not** activate billable runners, external services, GitHub larger runners, personal machines or local infrastructure for CI.

## Definition of done per application

- No required network access to `storytold/*` in CI, builds, releases, runtime, update feeds or asset imports.
- Build/release succeeds on standard free GitHub-hosted runners using an immutable lockfile and source inputs controlled by Nodalia or independent upstream projects.
- No public security report goes to another project's maintainers.
- All original copyright, license and asset provenance notices remain intact.
- Reinstall, update and release artifacts validated and documented for each supported OS.
