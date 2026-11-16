---
name: maintain-standards
description: Change or troubleshoot Idea2Crate shared instructions, skills, pre-commit profiles and synchronization. Use for managed-file drift or enrolling a repository, not ordinary application changes.
---

Shared rules are maintained in `Idea2Crate/standards`. In an enrolled repository,
read `.idea2crate/standards.lock.json` to identify the installed source revision.
Do not repair managed copies by hand or edit the lockfile to bless a local change.

In the central repository:

1. Read `docs/project.md` and `docs/automation.md`. Decide whether the requirement
   is universal, a language profile or project-specific documentation.
2. Edit the canonical file or `profiles.json`. For enrollment, add the exact
   repository and profile to `repositories.json`; keep target permissions scoped.
3. Run `python3 -m unittest discover -s tests -v` and the pre-commit hooks.
4. Run `python3 -m standards plan --repository OWNER/REPO` to inspect the generated
   file list. Publishing or merging central changes requires existing task
   authorization; a request to investigate drift alone does not grant it.

After an approved central merge, automation creates or updates one sync PR per
repository. A successful required check permits auto-merge. If blocked, inspect
the central Actions run and target PR: distinguish permissions, content mismatch,
CI failure and a stale base. Do not bypass protections, approve arbitrary bot PRs
or put App credentials in a downstream workflow.

For local verification, use the installed pre-commit `standards-integrity` hook.
It checks against its immutable central checkout; the independent GitHub App
check is the authoritative merge gate.
