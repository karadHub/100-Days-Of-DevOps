# Day 22: Clone Git Repository on Storage Server

## 🎯 TASK

The DevOps team created a Git repository at `/opt/games.git`. The Nautilus application team needs a working copy of this repository on the Storage Server at `/usr/src/kodekloudrepos`.

Requirements:

- Perform the clone as the `natasha` user.
- Do not modify repository contents or change permissions on existing directories.
- Clone `/opt/games.git` into `/usr/src/kodekloudrepos` in a way that results in a subdirectory named after the repo (e.g., `/usr/src/kodekloudrepos/games`).

---

## 🛠 Solution (step-by-step)

Log in or switch to the `natasha` user on the Storage Server. You must run the commands as `natasha`.

1. Ensure the target parent directory exists (do NOT change existing permissions):

```bash
sudo -u natasha mkdir -p /usr/src/kodekloudrepos # create parent if missing
```

2. Change into the target directory as `natasha` and clone the bare repo so that a `games` subdirectory is created:

```bash
sudo -u natasha bash -c 'cd /usr/src/kodekloudrepos && git clone /opt/games.git'
```

Explanation: Running `git clone /opt/games.git` from inside `/usr/src/kodekloudrepos` creates `/usr/src/kodekloudrepos/games` (a working copy) which matches the requirement that a subdirectory be present. Avoid cloning directly with `git clone /opt/games.git /usr/src/kodekloudrepos` because that would place the repository contents directly in the target directory without creating the `games` subdirectory.

3. Verify the clone as `natasha`:

```bash
sudo -u natasha ls -la /usr/src/kodekloudrepos
sudo -u natasha test -d /usr/src/kodekloudrepos/games && echo "OK: games cloned"
sudo -u natasha bash -c 'cd /usr/src/kodekloudrepos/games && git status --porcelain'
```

`git status --porcelain` should show a clean working tree if no changes were made.

---

## ✅ Notes & troubleshooting

- If you see permission errors when running git as `natasha`, ensure `/opt/games.git` is world-readable for git or readable by the `natasha` user (do not change repo permissions unless directed).
- If `/usr/src/kodekloudrepos` already contains files, cloning will create a `games` directory alongside them — this is intended.
- Use `git clone --depth 1 /opt/games.git` if you only need the latest snapshot.

---

## 📚 Acceptance criteria

- `/usr/src/kodekloudrepos/games` exists and contains a working copy cloned from `/opt/games.git`.
- The clone operation was performed by the `natasha` user.
- No changes were made to `/opt/games.git` or permissions on existing directories.
