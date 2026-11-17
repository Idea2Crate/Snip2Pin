# Snip2Pin 开发与验证

## Development

Follow [the workflow](workflow.md) and [CONTRIBUTING.md](../CONTRIBUTING.md).
Use short branches and focused PRs; `main` requires all native CI checks and
dependency checks, resolved discussions, an up-to-date base, and squash merge.
Do not bypass protections or push directly to `main`. Single-maintainer PRs do
not require a second reviewer. Describe user-visible acceptance criteria before
implementation; separate automatic coverage from actual native interaction.
Keep repository rules in `.github/rulesets/` consistent with GitHub settings.
After work is merged or superseded, promptly remove its unused local/remote
branches and worktrees after checking for uncommitted changes, active work and
retained build artifacts. Verify squash/cherry-pick replacements before deleting
old branches; preserve ongoing validation and release inputs. Keep machine-local
workspace paths out of the repository.

PR titles and every branch commit subject must use `type(scope)!: description`;
scope and `!` are optional. Types: feat, fix, docs, refactor, perf, test, build,
ci, chore, revert. Use lowercase ASCII types/scopes (scope may contain digits
and hyphens), a nonempty Chinese or English description, and at most 100 characters.
Examples: `fix(pin): 修复顶部按钮位置`, `ci: automate release signing`.
Use `!` and a `BREAKING CHANGE:` body/footer when changing incompatible behavior.
Explain why in the body when useful; avoid vague subjects such as `update` or `fix`.
Keep PR titles short and descriptions/progress comments to problem and fix lists;
omit conversations and local debugging details. Link issues when applicable.
`messages.yml` checks open PR titles on edits and
all branch commits on updates; fix/reword invalid commits before merging.
Squash commits use the validated PR title and PR body. Do not bypass the required
`messages` check. Conventional formatting never authorizes a version bump/release.

Use `cargo xtask check` locally and in CI. Shared development/packaging logic
belongs in the Rust `xtask` workspace member; native scripts only adapt platform
tools. Read the development toolchain from rust-toolchain.toml and verify the
minimum Rust version declared in Cargo.toml with `cargo xtask check-msrv`.
Install the `.pre-commit-config.yaml` hooks for development. Commit-time checks
run `cargo xtask lint` (Rust format, Clippy and repository metadata), official
YAML/TOML, large-file and merge-conflict hooks, and `cargo deny --locked check`.
Official whitespace and end-of-file fixers may edit files and block that commit;
review and stage their changes before retrying. Large files above 500 KiB must use
Git LFS; enforce the size check on all supplied files, with LFS files skipped by
the official hook. Other failures block the commit without rewriting files; never
skip missing Rust/cargo-deny tools once hooks run. Keep the local Rust hook as a thin
call to xtask and run the official general hooks in CI too. CI reuses the same lint entry point
through `check`; full tests/builds remain separate from the faster commit checks.
Preserve hooks managed by other tools; use their supported integration rather
than overwriting `core.hooksPath` or disabling security/LFS hooks. Update
CONTRIBUTING.md and docs/workflow.md together when changing hook setup or policy.
Keep xtask's own tests in `#[cfg(test)]` modules: Cargo builds the normal binary
for integration-test targets, which cannot replace a running `xtask.exe` on Windows.
Dependency updates go through PRs and `cargo deny check`; do not auto-merge or
globally suppress advisories/licenses to obtain a green check. Any scoped
exception needs an advisory ID, impact analysis and a dated follow-up.
The project and contributions use MIT; include LICENSE in application packages
and preserve required third-party license notices.
Generate notices with the pinned cargo-about tool over the application's normal
desktop dependencies. Preserve bundled font and installer-runtime notices too.
Keep upstream clarifications and reviewed fallbacks bound to source versions and
checksums; unknown attribution gaps block packaging. Never invent copyright
holders or present a standard-license reference as an upstream copyright notice.

CI uses pinned Action commit SHAs, least-privilege tokens, finite timeouts and
platform-specific caches. Release validation, draft creation and publication run
only xtask and use the shared setup action with `desktop: 'false'`; native builds,
package preparation and native verification retain their platform dependencies.
PRs cannot access signing credentials. Daily checks
may build unsigned debug packages and verify final unverified package layout
without GUI activation, uploading distribution files, using signing credentials,
tagging or creating Releases. Installation checks use only disposable hosted
runners or containers and temporary destinations; never install on a developer's
machine or alter an existing installation. macOS debug bundle integrity checks
may use an ad-hoc seal; never use the local development identity in CI.
For explicitly requested isolated native interaction tests, manual CI may retain
unsigned archives, their prepared installers and source manifests for one day via
`validation_artifacts`. The `validation_profile` choice defaults to debug; an
explicit manual test may select release optimization to distinguish runtime
performance from functional failures. Ordinary push/PR checks always use debug
packages. Compiler optimization is not publication; these test artifacts are not
formal release candidates. Keep retention disabled on push/PR events, without
signing, tags, Releases or release Environment access; never publish these builds.
Formal publication requires a complete set of four native targets, matching
version/source SHA, checksums, native checks for the explicitly selected
distribution mode, recorded
installation/upgrade acceptance and approval of the `release` Environment.
Read paginated CI workflow history and match the full SHA, main branch, trusted
workflow path, push/manual event and success locally. Bound the scan to its initial
record count; missing/invalid records or API errors must block publication.
Do not run candidate, signing, stage or publish commands without an explicit
release request. Implementing/testing these workflows is not such a request.
Production signing credentials may be provisioned once through the protected
`release` Environment Secrets, using the names documented in docs/workflow.md.
Do not export/upload the local development identity. Use GitHub-hosted ephemeral
runners, single-step secret exposure, temporary keychains/certificate imports and
cleanup on failure. No credentials in caches/artifacts; no signing on PRs or
automatic push/merge events. Do not attach a public-PR self-hosted runner to the
user's everyday computer. Missing/invalid production credentials must fail closed;
never fall back to ad-hoc/development signatures in `signed` mode. No automatic
version bump/release. `publish.yml` defaults to the `prepare` operation and explicit `unverified` distribution,
without secrets; it validates candidate run/source/digests, prepares packages
without rebuilding, checks native integrity and creates only a draft. Manifests
record the distribution mode. Never mix modes or publish unverified files using
signed names/checks. Preserve the bilingual trust notice in unverified Releases.
Actual publication remains a separate manual request with native acceptance and
Environment approval. Windows PFX support requires a provider-approved exportable
code-signing certificate; nonexportable/hardware identities need a cloud signing
adapter instead of attempts to export them.
Use Cargo.toml as the single version source; generate package metadata and bind
manifests to a full commit SHA. Publish the verified files without rebuilding;
never overwrite published tags/assets. Fix a release with a new patch version.
Keep one unsigned candidate archive per native target. Schema 3 manifests list
typed artifacts: final macOS DMG, Windows portable ZIP plus per-user NSIS Setup.exe,
and Linux tar.gz plus Debian package. Require the complete exact format set and
preserve the original candidate hash for every target. Signed Windows mode signs
the application, embedded uninstaller and installer; signed macOS mode signs and
notarizes the outer DMG after the app, without later content changes. Debian
dependencies derive from ELF plus runtime-loaded libraries on Ubuntu 24.04; do
not claim untested Debian compatibility. AUR generation is a separate manual
post-publication command, verifies the public tarball, uses makepkg for .SRCINFO,
and never pushes or uploads to AUR. Daily CI may verify temporary silent Windows
installs and generated upgrade fixtures on hosted runners; no desktop activation
or real user settings.

Use the pinned Rust toolchain and commit `Cargo.lock`. Keep dependencies small.
Use published crates instead of unpinned Git branches. Avoid unsafe code,
production unwrap, hidden background work, and blocking the UI.
Capture runs on a worker; only the UI thread creates or changes windows.
Cancellation and closing a pin must release image data and textures.
Use events and state transitions instead of idle polling. Keep empty selection
layers transparent and mouse-pass-through outside capture mode. Never use native
fullscreen for macOS selection; it triggers desktop/Space animations.
Shortcut changes must retain the previous binding if registration fails. Demo
mode must not register a global shortcut, save settings, or read the desktop.
macOS selection windows must join Spaces without creating a fullscreen Space,
cover the menu bar, and release keyboard focus when capture finishes.
macOS pins stay at floating window level, below the Dock and menu bar. Never
replace the runtime class of an observed window: AppKit owns KVO subclasses and
their observation bookkeeping. Test creation, movement, appearance changes,
and destruction using real winit windows, invisible with activation prohibited;
plain NSWindow geometry tests alone cannot cover backend observation teardown.
After raising macOS selection windows above system UI, restore `NSScreen.frame`
instead of the work area. Warm inactive layers too, and defer image painting
until the viewport reports the complete display size; never stretch a desktop
capture into a menu-bar-constrained window. Native geometry tests must keep
windows invisible and prohibit application activation. An ordered-window
regression must use small fully transparent, mouse-pass-through windows and
release them before exit; hidden-window checks alone do not cover ordering.
Use egui's theme preference to follow system appearance without polling. Theme
and icon changes must preserve unrelated settings and existing shortcut files.
Keep unsafe code denied by default. A platform API that requires unsafe may have
one narrowly scoped exception, with documented preconditions and owned memory.
macOS Dock icons must decode PNG in Rust, avoiding AppKit PNG loading and borrowed
pixel buffers. Updating an icon must not modify the signed application bundle.
At startup use eframe's explicit empty-icon sentinel to preserve the bundle icon;
omitting it selects eframe's fallback. Auto icons wait for a known system theme,
including logic-only background startup; never briefly install a light placeholder.
Select Finder/Launchpad bundle resources before signing during local installation.
Do not set a custom file icon on a signed app: its Finder metadata/resource fork
fails strict signature verification. Runtime icon choices update the Dock; document
the bundle-icon installation requirement rather than silently invalidating signing.

Use dedicated file patch tools for text edits. Programmatic transformation is
appropriate only when the transformation itself is complex; state the reason and
show the resulting diff. Keep code, comments, README, and this file consistent.

## Validation

```sh
cargo xtask check
cargo xtask check-msrv
cargo deny check
```

Cover reverse drag, DPI conversion, empty and out-of-bounds selections, image
release, and capture/cancel transitions. Never make CI capture a real desktop or
require privacy permission. Use generated demo images for visual checks.
For input-to-window changes, cover the production deferred callback through
action dispatch and root logic to the final window command, including stale
root geometry, DPI/UI zoom, cross-viewport continuation and lock checks.
Distinguish compile/test coverage from interactive checks on Windows, macOS,
Linux X11, and Wayland. Document compositor-controlled pinning and shortcuts.

Do not commit private screenshots, credentials, toolchains, caches, or build output.
Explicitly requested README illustrations may use generated public sample images
from the shared UI sandbox; label their source and keep binary assets in Git LFS.
Keep the release macOS bundle ID `io.github.Snip2Pin.Snip2Pin` stable. Local certificate
signing is for development; it does not mean an app is notarized for distribution.
Settings use the platform configuration root plus `Snip2Pin`; local macOS signing
material uses `~/Library/Application Support/Snip2Pin/signing`, outside the repo.
Reuse the existing certificate; do not regenerate credentials during a rename.
An explicitly requested identifier change uses `--install --migrate-identity`,
which preserves the certificate requirement; later updates use ordinary `--install`.
`build.sh --install` enables `local-dev` and updates only `Snip2Pin Dev.app` with
bundle ID `io.github.Snip2Pin.Snip2Pin.dev`, settings under `Snip2Pin/dev`, and
build artifacts under `target/local-dev`. The runtime identity, localized names,
bundle metadata, signing requirement, registration and login entry must agree.
macOS packaging checks the binary identity and rejects a development executable
in a release bundle. The release installation and its permissions stay intact.
Formal distribution uses the manual workflows documented in docs/workflow.md, production signing
and notarization only when `signed` is explicitly selected; default `unverified`
packages are not Apple-notarized. Never use the local development certificate to release.
