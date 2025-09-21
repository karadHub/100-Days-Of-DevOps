# Day 29 — Manage Git Pull Requests (Gitea)

Task summary

Max has pushed a branch `story/fox-and-grapes` to the remote Gitea repository. Your job is to create a Pull Request (PR) to merge `story/fox-and-grapes` into `master`, assign `tom` as the reviewer using the Gitea UI, then log in as `tom` to review and merge the PR.

Credentials and hosts

- SSH into the storage server as `max`:
  - Username: max
  - Password: Max_pass123
- Gitea web UI credentials:
  - max / Max_pass123
  - tom / Tom_pass123

Steps (CLI + UI)

1. SSH into the storage server as `max` and inspect the cloned repo

```bash
ssh max@<storage-server-ip>
# enter password: Max_pass123

cd ~/
ls -la
cd <repository-folder>   # change to the cloned repo directory in Max's home
git status
git log --oneline --decorate -n 10
```

Validate the commit history and author information. You should see commits from `sarah` and the commit(s) related to Max's story.

2. Open Gitea web UI

- Click the Gitea UI button in the lab environment (or open the Gitea URL in a browser). Log in as `max` (Max_pass123).
- Navigate to the repository and open the `Pull Requests` tab.

3. Create a new Pull Request from `story/fox-and-grapes` into `master`

- Click `New Pull Request`.
- Set:
  - Title: `Added fox-and-grapes story`
  - Pull from (source): `story/fox-and-grapes`
  - Merge into (destination): `master`
- Add a short description if desired and click `Create Pull Request`.

4. Assign `tom` as a reviewer

- On the PR page, find the `Reviewers` panel on the right-hand side.
- Search for `tom` and add him as a reviewer.
- Take a screenshot of the PR page showing `tom` listed under Reviewers (recommended for grading).

5. Log out of `max` and log in as `tom`

- Click `Sign out` in Gitea or use the UI logout control.
- Log back in with `tom` / `Tom_pass123`.

6. Review and approve the PR as `tom`

- Navigate to the repository > Pull Requests > open PR titled `Added fox-and-grapes story`.
- Click `Files changed` to inspect Max's changes and verify the story content.
- Use the `Review` or `Approve` button to leave an approval and optionally a comment.

7. Merge the PR

- After approval, click `Merge` on the PR page (choose `Merge` or `Squash and Merge` as per repo policy). Confirm the merge.
- Take a screenshot of the merged PR and the merge confirmation.

8. Verify merge in the storage server

Log back into the storage server (either as `max` or another user with access) and update the local clone:

```bash
cd ~/<repository-folder>
git checkout master
git pull origin master
git log --oneline -n 5
ls -la
cat info.txt   # or the file containing the fox story to verify content
```

Notes & troubleshooting

- If you can't access the Gitea UI from the lab, capture CLI evidence: `git log` showing the remote branch and the new commits pushed by Max.
- If branch protection prevents merging from `tom`, ask an admin to merge or adjust protections. Document the issue with a screenshot.
- Screenshots or a short screen recording are recommended for UI tasks to provide proof of completion.

Deliverables

- Screenshot(s) of:
  - PR created by `max` showing `story/fox-and-grapes` -> `master`.
  - `tom` listed as reviewer.
  - `tom` approving and merging the PR.
  - `git log` on the storage server showing the merge on `master`.
