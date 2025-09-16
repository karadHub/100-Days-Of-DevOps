# Day 24: Git — Create Branches

## 🎯 TASK

Create a new branch `xfusioncorp_ecommerce` from the `master` branch in the `/usr/src/kodekloudrepos/ecommerce` repository on the Storage server in the Stratos DC. Do not modify any code or files — this task only creates a branch reference.

---

## 🛠️ Solution (step-by-step)

Run the commands on the Storage server as a user with read/write access to the repository (do not edit code files):

1. SSH to the storage server (example):

```bash
ssh youruser@storage-server
```

2. Change to the repository directory:

```bash
cd /usr/src/kodekloudrepos/ecommerce
```

3. Ensure the repository is on the `master` branch and up to date:

```bash
git fetch --all --prune
git checkout master
git pull origin master
```

4. Create the new branch from `master` without modifying files:

```bash
git branch xfusioncorp_ecommerce master
```

This creates a local branch pointing to the same commit as `master` without checking it out. If you prefer to create and check it out (still non-destructive):

```bash
git checkout -b xfusioncorp_ecommerce master
```

5. (Optional) Push the new branch to the remote so others can access it:

```bash
git push origin xfusioncorp_ecommerce
```

6. Verify branch creation:

```bash
git branch --all
# or check remote
git ls-remote --heads origin | grep xfusioncorp_ecommerce || true
```

You should see `xfusioncorp_ecommerce` listed locally (and remotely if pushed).

---

## ✅ Acceptance Criteria

- A branch named `xfusioncorp_ecommerce` exists and points to the same commit as `master`.
- No source files were modified as part of this operation.

---

## 📚 Notes

- If `origin` is protected or you lack push permissions, skip the push step and document that the branch exists only locally.
- Use `git branch -vv` to view tracking information and the exact commit each branch points to.
