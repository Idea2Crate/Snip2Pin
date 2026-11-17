# Snip2Pin

Canonical repository: <https://github.com/Idea2Crate/Snip2Pin>.
Snip2Pin is a Rust screenshot-and-pin application for macOS, Windows and Linux.
Its application identity and configuration paths are independent of the GitHub
organization name; keep the existing bundle IDs and user settings stable.

Read the project documents appropriate to the change:

- [Product and platform rules](product-rules.md): product behavior, native window
  constraints, screenshots, input, permissions and user-visible acceptance.
- [Implementation and structure](implementation.md): shared UI logic, platform
  boundaries, geometry, lifecycle and workspace responsibilities.
- [Development and validation rules](development-rules.md): contribution checks,
  dependency/licensing rules, packaging and release boundaries.
- [Workflow](workflow.md) and [contributing](../CONTRIBUTING.md): executable setup,
  local commands, CI and native acceptance procedures.

Use the pinned Rust toolchain. Run `cargo xtask lint` and
`cargo deny --locked check` for commit-time checks, `cargo xtask check` for the
full project checks and `cargo xtask check-msrv` for the declared minimum version.
Desktop interaction tests require task authorization; use generated WASM data
when desktop control is not authorized. Do not publish or install an application
as part of a documentation or governance update.

This repository maintains its own `AGENTS.md`, skills, pre-commit configuration
and CI. Change contribution rules here through the normal project review process.
Pre-commit runs text metadata checks, the local credential-pattern guard, Rust
format/Clippy/repository checks and dependency checks. GitHub secret scanning and
push protection provide additional credential checks.

Repository setup skills may be installed locally in `.agents/skills/idea2crate-*`
and ignored by Git. A human invokes them when creating a project or updating its
rules; the resulting files remain project-owned. Ordinary application work and
new skill versions do not trigger rule updates. There is no synchronization
lockfile, central managed-file check or cross-repository credential dependency.
