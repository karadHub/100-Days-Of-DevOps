# Day 32 — Git Rebase (integrate master into feature branch)

Task summary

A developer is working on a feature branch in the repository cloned at `/usr/src/kodekloudrepos`. New changes have been pushed to the master branch. The goal is to rebase the feature branch onto master to integrate the latest master changes without losing feature branch data and without creating a merge commit.

What rebasing does

- Takes commits from the feature branch and "replays" them on top of the latest master branch.
- Results in a linear history without merge commits.
- Preserves all feature branch work while incorporating master updates.

Step-by-step instructions

1. Navigate to the cloned repository

```bash
cd /usr/src/kodekloudrepos
```

2. Fetch the latest changes from the remote

```bash
git fetch origin
```

3. Check which branch you're currently on

```bash
git branch
```

4. Switch to the feature branch (if not already on it)

```bash
git checkout feature
```

5. Rebase the feature branch onto the latest master

```bash
git rebase origin/master
```

Alternative (if master is up-to-date locally):

```bash
git rebase master
```

6. Handle conflicts if they occur

If Git encounters conflicts during rebase:

- Edit the conflicted files to resolve the issues
- Stage the resolved files:

```bash
git add <resolved-file>
```

- Continue the rebase:

```bash
git rebase --continue
```

- Repeat until all conflicts are resolved

7. Force-push the rebased feature branch (if it was previously pushed)

```bash
git push origin feature --force
```

⚠️ **Note**: Use `--force` carefully and ensure no one else is working on the same feature branch.

Alternative commands

- To abort the rebase if something goes wrong:

```bash
git rebase --abort
```

- To skip a commit during rebase (use with caution):

```bash
git rebase --skip
```

Verification

- Check the commit history to ensure the feature branch commits are now on top of master:

```bash
git log --oneline --graph -n 10
```

- Verify no merge commit was created (the history should be linear).

Benefits of rebasing

- Clean, linear project history
- No unnecessary merge commits
- Feature branch commits appear as if they were developed on the latest master
- Easier to understand project evolution
