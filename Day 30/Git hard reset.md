# Day 30 — Git Hard Reset (clean history)

Task summary

The `news` repository at `/usr/src/kodekloudrepos/news` contains a number of test commits. The goal is to reset the repository so the commit history only contains two commits: the initial commit and the commit with message `add data.txt file`. After resetting the local repository, force-push the cleaned history to the remote.

Steps (safe, step-by-step)

1. Change to the repo directory:

```bash
cd /usr/src/kodekloudrepos/news
```

2. Inspect the commit history and find the target commit hash for `add data.txt file`:

```bash
git fetch --all
git log --oneline --decorate --reverse
```

Look for the commit whose message is `add data.txt file` and copy its hash (e.g. `abc1234`).

3. Reset the branch to that commit (this will remove all commits that came after it):

```bash
git reset --hard <commit-hash>
```

4. Verify your working tree and history now contains only the initial commit and the `add data.txt file` commit:

```bash
git log --oneline --decorate -n 5
git status --porcelain
```

5. Force-push the cleaned history to the remote (this rewrites remote history):

```bash
git push origin HEAD --force
```

Notes and warnings

- This operation rewrites history. Coordinate with your team and ensure nobody else depends on the old commits.
- If the remote has branch protection preventing forced pushes, you'll need admin intervention or a different strategy (e.g., create a new branch and switch to it).
- Keep a backup before destructive operations: you can create a branch pointing to the current state for safekeeping:

```bash
git branch backup-before-reset
git push origin backup-before-reset
```

Troubleshooting

- If `git push --force` fails due to protected branches, create a new branch and push it, then open a PR or ask an admin to update protections.
- If you accidentally reset the wrong commit, you can recover using the reflog (if local history still contains it):

```bash
git reflog
git reset --hard <reflog-hash>
```

Deliverables

- After completion, `git log --oneline` should show only two commits: the initial commit and `add data.txt file`.
- The remote `master` (or target branch) should reflect the same cleaned history after the force push.
