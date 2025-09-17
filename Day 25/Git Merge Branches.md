# Day 25: Git — Merge Branches

## 🎯 TASK

Perform the following Git workflow in the storage server clone at `/usr/src/kodekloudrepos/apps`:

- Create a new branch `xfusion` from `master`.
- Copy `/tmp/index.html` (on the storage server) into the repository root.
- Add and commit the file on the `xfusion` branch.
- Merge `xfusion` back into `master`.
- Push both `xfusion` and `master` to `origin`.

> Note: This guide assumes you have the necessary permissions and `origin` is configured. Do not modify unrelated files.

---

## 🛠 Solution (step-by-step)

1. Change to the repository on the storage server:

```bash
cd /usr/src/kodekloudrepos/apps
```

2. Ensure master is up-to-date:

```bash
git fetch --all --prune
git checkout master
git pull origin master
```

3. Create and switch to the new branch `xfusion`:

```bash
git checkout -b xfusion
```

4. Copy `/tmp/index.html` into the repository root (non-destructive):

```bash
cp /tmp/index.html .
```

5. Stage and commit the file:

```bash
git add index.html
git commit -m "Add index.html on xfusion branch"
```

6. Switch back to `master` and merge `xfusion` into it:

```bash
git checkout master
git merge --no-ff xfusion -m "Merge xfusion into master: add index.html"
```

7. Push both branches to the remote origin:

```bash
git push origin master
git push origin xfusion
```

8. Verification (optional):

```bash
git log --oneline --graph --decorate -n 10
git branch -a
```

---

## ✅ Acceptance Criteria

- `index.html` exists in the repo and was added on the `xfusion` branch.
- `xfusion` was merged into `master` and both branches are pushed to `origin`.

---

## ⚠️ Safety Notes

- If `origin` rejects pushes (protected branches), you'll need proper permissions or a PR workflow.
- If the working tree is dirty and you cannot alter local files, stash changes first: `git stash --include-untracked` and restore after.
