# Day 26: Git — Manage Remotes

## 🎯 TASK

Update the `/usr/src/kodekloudrepos/media` repository to add a new remote and push changes:

a. Add a new remote `dev_media` pointing to `/opt/xfusioncorp_media.git`.
b. Copy `/tmp/index.html` into the repo, add and commit it to the `master` branch.
c. Push the `master` branch to the new remote `dev_media`.

---

## 🛠 Solution (step-by-step)

1. Change to the repository directory:

```bash
cd /usr/src/kodekloudrepos/media
```

2. Add the new remote `dev_media` pointing to the local bare repo:

```bash
git remote add dev_media /opt/xfusioncorp_media.git
git remote -v
```

Expected output shows `dev_media` for fetch and push.

3. Ensure you're on `master` branch:

```bash
git checkout master
git pull origin master || true
```

4. Copy `/tmp/index.html` into the repo and commit:

```bash
cp /tmp/index.html .
git add index.html
git commit -m "Add index.html to master branch"
```

If commit fails because nothing changed, inspect `git status` to verify.

5. Push `master` to the new remote `dev_media`:

```bash
git push dev_media master
```

6. Verification:

```bash
git remote -v
git log -n 5 --oneline
git ls-remote dev_media refs/heads/master
```

---

## ✅ Acceptance Criteria

- `dev_media` is configured as a remote and points to `/opt/xfusioncorp_media.git`.
- `/tmp/index.html` is committed on `master`.
- `master` is pushed to `dev_media`.

---

## 📚 Notes

- If `dev_media` already exists, use `git remote set-url dev_media /opt/xfusioncorp_media.git`.
- If you cannot push because of permissions, coordinate with the repo owner or use `sudo` where appropriate (but avoid changing repo ownership unnecessarily).
