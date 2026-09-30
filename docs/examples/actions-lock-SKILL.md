---
name: actions-lock
description: Keep GitHub Actions dependencies locked whenever workflows or local actions are created, modified, upgraded, or removed.
---

<!--
Example skill. Install as .github/skills/actions-lock/SKILL.md.
See docs/repository-developer-experience.md for the rationale.
-->

When changing files under `.github/workflows/`, or changing an `action.yml` or
`action.yaml` used by those workflows:

1. Do not edit `.github/workflows/actions.lock` manually.
2. Ensure the `github/gh-actions-lock` CLI extension is installed. This is safe
   to run when it is already present:

   ```bash
   gh extension install github/gh-actions-lock
   ```

3. After editing workflows or local actions, update the lockfile:

   ```bash
   gh actions-lock --no-interactive
   ```

4. Expect the command to edit source files, not just the lockfile. It narrows
   `uses:` refs to a full semver tag and rewrites same-repo `./…` action
   references to `$/…`. Both are intended; do not revert them.
5. Include every generated workflow, local action, and
   `.github/workflows/actions.lock` change in the resulting change set.
6. Verify the result without modifying files:

   ```bash
   gh actions-lock --verify
   ```

Do not consider the task complete unless verification succeeds. If locking
fails, report the exact finding instead of leaving a stale or incomplete
lockfile. A report of `local path actions are not yet supported` usually means a
`./…` reference does not resolve — those paths are relative to the repository
root, not to the directory of the file containing them.

Job-level reusable workflow calls
(`jobs.<id>.uses: owner/repo/.github/workflows/x.yml@ref`) are not read by the
CLI yet, and a fix run deletes a correct lockfile entry for one
(github/gh-actions-lock#129, under review). If this repository calls remote
reusable workflows, check `git diff` on the lockfile before committing and
restore any entry the run removed.

Background on how locking is automated in this repository:
`docs/repository-developer-experience.md`.
