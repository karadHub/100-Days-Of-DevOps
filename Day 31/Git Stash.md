# Day 31 — Git Stash (restore specific stash)

Task summary

One of the developers stashed some in-progress changes in the repository `/usr/src/kodekloudrepos/apps`.
Your job is to restore the stash with identifier `stash@{1}`, commit the restored changes, and push them to the origin.

Action plan (short)

- Navigate to the repo
- Confirm the stash exists
- Apply `stash@{1}` (use `apply` to keep the stash intact)
- Stage, commit, and push the changes

Step-by-step instructions

1. Change to the repository directory

```bash
cd /usr/src/kodekloudrepos/apps
```

2. List all stashes and confirm `stash@{1}` exists

```bash
git stash list
```

Example output:

```
stash@{0}: WIP on main: abc123 Commit message
stash@{1}: WIP on main: def456 Another commit message
```

3. Apply the specific stash (keeps the stash entry intact)

```bash
git stash apply stash@{1}
```

4. Stage the restored changes

```bash
git add .
```

5. Commit with a clear message

```bash
git commit -m "Restored changes from stash@{1}"
```

6. Push to origin (current branch)

```bash
git push origin HEAD
```

Optional cleanup (only if you're sure)

```bash
git stash drop stash@{1}
```

Troubleshooting

- If `git stash apply` causes conflicts, resolve the conflicts, `git add` the files, then run `git commit` to finalize.
- If pushing fails due to permissions or branch protection, create a branch, push it, and open a PR instead.

Verification

- `git stash list` should still show `stash@{1}` (unless you dropped it).
- `git log --oneline -n 3` should show your new commit message `Restored changes from stash@{1}` at the top.
