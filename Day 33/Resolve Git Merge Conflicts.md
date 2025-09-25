# Day 33: Resolve Git Merge Conflicts

## 🎯 Task

SSH into storage server as user `max` (password `Max_pass123`). In `/home/max/story-blog`, push local changes and resolve merge conflicts so that:

- `story-index.txt` lists titles for all 4 stories.
- Fix the typo “Mooose” to “Mouse”.
  You can verify via Gitea UI (login as `sarah`/`Sarah_pass123` or `max`/`Max_pass123`).

---

## 🛠️ Solution (step-by-step)

1. SSH and navigate to repo

```bash
ssh max@<storage-server-ip>
# Password: Max_pass123
cd /home/max/story-blog
```

2. Check status and remotes

```bash
git status
git remote -v
```

3. Update local and rebase on latest master (reduces merge commits)

```bash
git fetch origin
git pull --rebase origin master
```

- If conflicts appear, Git will pause and mark conflicted files (e.g., `story-index.txt`).

4. Fix conflicts and required edits

- Open the file and resolve conflict markers.
- Ensure all 4 titles exist and fix the typo:

```
The Lion and the Mouse
The Frogs and the Ox
The Fox and the Grapes
The Tortoise and the Hare
```

```bash
nano story-index.txt
```

5. Stage and continue (if rebasing) or commit (if no rebase)

```bash
git add story-index.txt
git rebase --continue || git commit -m "Fix story-index list and typo (Mouse)"
```

6. Push to origin

```bash
git push origin master
```

7. Verify in Gitea UI

- Login as `max` (`Max_pass123`) or `sarah` (`Sarah_pass123`).
- Open `story-blog` and confirm latest commit and corrected `story-index.txt`.

---

## ✅ Notes

- If rebase gets complicated:

```bash
git rebase --abort
git pull origin master
```

Resolve, then:

```bash
git add story-index.txt
git commit -m "Fix story-index list and typo (Mouse)"
git push origin master
```
