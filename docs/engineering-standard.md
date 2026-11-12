# Nodalia Engineering Standard

Status: proposed baseline, 12 November 2026. Apply per project type; never turn a new integration into a required check until a relevant PR has a successful, stable run.

## Nodalia product and infrastructure independence

Nodalia owns and maintains its application identity, documentation, release decisions, security response, and engineering standards. A previous open-source project's authorship is acknowledged where its licensing requires it, but upstream repositories **must not control Nodalia's active development pipeline**.

- The desired end state is **no required CI workflows, reusable actions, release services, governance files, fonts, translations, updater endpoints, or Git source dependencies fetched from `storytold/*`**. Replace each with a reviewed Nodalia-maintained component or a properly licensed independent upstream alternative.
- Move workflow code to `getnodalia` repositories or local workflows, pin actions to immutable commits, and use **only standard free GitHub-hosted runners** for public repositories. Never silently enable paid runners or self-hosted infrastructure.
- Do not break existing releases by globally replacing repository URLs. Migrate and verify one dependency at a time, including its license, source provenance, reproducible build, distribution configuration, and security/update process.
- Repository `AGENTS.md` and quality rules are authored and maintained by Nodalia. Do not inherit `storytold/craft-contrib` as a live source of authority.
- Preserve applicable MIT/Apache-2.0 license notices, source attribution and third-party font and asset licenses. Provenance documentation is not an operational dependency and must not be erased to imply original authorship.
- Use the [independence migration inventory](independence-migration.md) to coordinate the transition; a repo is not "independent" until its build, tests, releases and delivered binaries have been verified without `storytold/*` access.

## Principles

- Prefer existing working project workflows over a universal workflow that hides important platform differences.
- Keep PR validation, deep validation, and publication distinct. Never let a test-only change publish artifacts.
- Use GitHub-hosted standard runners only. Do not require paid services, paid runner minutes, home infrastructure, or large model downloads.
- Give each workflow explicit least-privilege permissions. Pin third-party actions to full commit SHAs and explain the release tag in a comment; update pins through grouped dependency PRs.
- Use concurrency cancellation for obsolete PR/branch CI only. Never cancel release or signing workflows that are producing artifacts.
- A required check must run for every applicable PR, use a unique stable name, avoid path filters that leave it pending, and have a verified success on the default branch and on a representative PR.
- Keep human review as a separate control. Do not give broad bypass rights to automation.

## By stack

| Stack | Minimum fast PR checks | Deeper/periodic checks |
|---|---|---|
| Rust | `cargo fmt --all -- --check`, workspace clippy with the project's supported feature set, unit/integration tests, `cargo check` or the production build | platform matrix, dependency/license audit, coverage via llvm-cov only if the same real test suite can produce an accurate report |
| Python | formatter/linter (ruff or configured equivalents), mypy/pyright where types are maintained, pytest, package/build check | supported Python versions, coverage from pytest-cov, dependency and license audit |
| JavaScript/TypeScript | lockfile-frozen install, formatter/linter, typecheck, tests, production build | supported Node versions, coverage from the existing test runner, dependency audit |
| C/C++ | CMake/Meson configure and build, project formatter/linter, unit tests, compiler warnings appropriate to the project | supported compilers/platforms, sanitizer jobs where stable, license/dependency audit |
| Shell/config/assets | shellcheck and syntax/metadata validation where applicable; validate packaged assets and install scripts without installing to a host | packaging smoke tests and reproducible checksums |
| Documentation/static sites | link/markup/build validation and tests that exist | deploy only from the protected default branch or release path |

Do not create a test target merely to satisfy a badge. For numerical software (Vortia), add documented reference cases, justified tolerances, convergence checks, and separate benchmarks. For model software (Lumbre), exercise resource cleanup, CPU-only fallback, model lifecycle using tiny fixtures, privacy boundaries, and graceful failure; do not download large models in CI. Desktop apps should build and smoke-test their supported artifacts when a free hosted runner and maintainable fixture exist.

## Coverage and quality services

- Coverage must be produced by a real test run. Start with reporting and a baseline; set a threshold only after measuring the repository's current branch coverage and exclude generated/vendor code explicitly. Raise thresholds gradually; never claim coverage from a synthetic report.
- Codecov can be used for public repositories at zero cost only if its free OSS terms remain available. Use tokenless uploads where Codecov allows them; if the account requires a token, add it as an Actions secret manually. Do not require the upload status until the report is verified.
- SonarQube Cloud workflows are appropriate for supported source projects when the project is onboarded and the free OSS tier is confirmed. Verify project key, token/connection, PR decoration, and Quality Gate. A workflow file alone does not prove the service is configured.
- CodeQL and OpenSSF Scorecard are free for public repositories. Keep their results informational until successful runs are verified. Do not enable paid security features for private repositories.
- Use Renovate or Dependabot, not both for the same ecosystem/directory. Group routine updates, separate security updates, cap concurrent PRs, and avoid major upgrades without review.
- Enable repository secret scanning/dependency alerts where GitHub makes them available at no charge. Never add credentials to repository variables or files.

## Versioning and releases

Keep one authoritative release mechanism per repository. Release Please is suitable only where Conventional Commits and a compatible manifest/version model are already adopted and no other tool creates tags, changelogs, packages, or GitHub Releases. Do not add it beside cargo-dist, tag-driven packaging, custom release workflows, or distribution publication pipelines. Release jobs should use immutable source revisions, test artifacts before publication, and preserve signature/checksum verification where already present.

Nodalix OS is critical: changes to ISO/kernel/GNOME/NxPM/package repositories, signing, Secure Boot, installers, or updaters require explicit compatibility review. Keep its current release and repository publishing workflows intact unless a dedicated change proves compatibility.

## Pull requests and branch protection

- Protect each repository's actual default branch, not a hard-coded `main`.
- Require pull requests. Require human approval where there is another maintainer available; never make the author approve their own PR.
- Initially require only a stable fast CI check after it has passed on a PR. Add security/coverage/Sonar checks only after they are verified and do not make them intermittently skip.
- Do not require release, deployment, platform-specific, scheduled, or informational checks on ordinary PRs.
- For public repositories on GitHub Free, use repository-level rulesets/branch protections. Organization-wide rulesets and rulesets for private repos require a paid GitHub plan; do not upgrade to enable them.

## Dependency and permissions hygiene

- Prefer read-only `GITHUB_TOKEN` at workflow level and elevate permissions on the single job that needs them.
- Use `persist-credentials: false` for read-only checkouts; do not pass write tokens into untrusted PR code.
- Pin Actions to commit SHA, run dependency updates on a predictable cadence, and preserve lockfiles.
- Cache package/build directories using lockfile and toolchain keys. Do not restore writable caches across untrusted forks into protected builds.
- Use immutable artifact uploads between jobs; build once and promote the verified artifact instead of rebuilding for publication when the current pipeline supports it.

## Exceptions and rollout

Project-specific exceptions belong in the repo's `QUALITY_GATES.md` or CI documentation with a reason, owner, and review date. Roll out in this order: audit baseline; propose CI/security additions; verify PR runs; configure branch requirements; then consider gradual coverage gates. A failed existing check must be investigated separately from a change that introduced it.
