# Day 28 — Git Cherry-Pick a Commit

Task:

You need to take a single commit with the message `Update info.txt` from the `feature` branch of the repository cloned at `/usr/src/kodekloudrepos` and apply it to `master` without merging the entire feature branch. After applying the commit, push the updated `master` branch to the remote.

Solution (step-by-step):

1. Change to the repository working copy on the storage server:

```bash
cd /usr/src/kodekloudrepos
```

2. Ensure the local repo is up to date and switch to `master`:

```bash
git fetch --all
git checkout master
git pull origin master
```

3. Locate the commit hash on the `feature` branch with the message `Update info.txt`:

```bash
git log --oneline feature
```

Look for the commit whose message is `Update info.txt` and copy its short or full hash (for example `abcd1234`).

4. Cherry-pick that specific commit onto `master`:

```bash
git cherry-pick <commit-hash>
```

If the cherry-pick leads to conflicts, resolve them, then:

```bash
git add <file(s)>
git cherry-pick --continue
```

If you decide to abort the cherry-pick:

```bash
git cherry-pick --abort
```

5. Push the updated `master` branch to the remote:

```bash
git push origin master
```

Verification:

- Run `git log --oneline -n 5` on `master` and confirm the `Update info.txt` commit is present at the top of the history.
- Inspect the file `info.txt` to confirm the expected changes are present.

Notes:

# 🍒 What Does It Do?

Imagine you have a branch called feature with several commits, but you only want to bring one of those commits into your master branch. Instead of merging the whole feature branch, you can cherry-pick just that one commit.

Good luck — the requested commit should now be selectively merged into `master`.
