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

Root `AGENTS.md`, `.agents/skills`, `.pre-commit-config.yaml` and
`.idea2crate/standards.lock.json` are managed by
[Idea2Crate/standards](https://github.com/Idea2Crate/standards). Change universal
rules there; project requirements belong in the documents above. Pre-commit keeps
the existing Rust/xtask and dependency checks alongside common integrity and
credential-pattern checks. The central GitHub App check is the authoritative
managed-file gate; a target workflow cannot grant its own policy approval.
