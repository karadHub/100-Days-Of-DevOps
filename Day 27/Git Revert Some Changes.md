# Day 27 — Git Revert Some Changes

Task:

The Nautilus application development team reported an issue with the recent commits pushed to the repository `/usr/src/kodekloudrepos/official` on the Storage server. Your job is to revert the repository HEAD to the previous commit by creating a new commit that undoes the latest commit and uses the commit message `revert official` (all lowercase).

Solution (step-by-step):

1. Switch to the target repository directory:

```bash
cd /usr/src/kodekloudrepos/official
```

2. Check the current status and recent commits (optional but recommended):

```bash
git status --porcelain
git log --oneline -n 5
```

3. Revert the latest commit (HEAD). This creates a new commit that undoes the changes from the latest commit.

```bash
git revert HEAD --no-edit
```

4. Amend the automatically generated revert commit message to the required message `revert official`:

```bash
git commit --amend -m "revert official"
```

5. Push the new revert commit to the origin (if remote push is allowed/required):

```bash
git push origin HEAD
```

Verification:

- Run `git log --oneline -n 3` and ensure the top commit message is `revert official` and the commit below it is the original bad commit that was reverted.
- Inspect files or run application tests to confirm the unwanted changes were undone.

Notes and Troubleshooting:

- If you get merge conflicts while running `git revert`, resolve them by editing the files, then `git add <file>` and run `git revert --continue`.
- If the repository is set up with branch protections that prevent pushes, either coordinate with your team to allow the push or create a PR from a branch that contains the revert commit.
- If you must reset the branch (destructive) instead of reverting, use `git reset --hard <previous-commit-hash>` and then force-push, but this rewrites history and should be avoided unless explicitly allowed.

Good job — the repository HEAD is now reverted and a clear revert commit was created.
