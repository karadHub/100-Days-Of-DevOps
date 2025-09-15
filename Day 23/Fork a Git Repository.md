# Day 23: Fork a Git Repository

## 🎯 TASK

A new developer, Jon, needs to fork an existing repository in the project's Gitea server so he can start working on it.

Requirements:

1. Open the Gitea UI via the top-bar Gitea button.
2. Log in to the Gitea server as the `jon` user using password `Jon_pass123`.
3. Locate the repository `sarah/story-blog` and fork it into Jon’s account.
4. Also find the repository named `Tork` (if present) and fork it into Jon’s account.

> Note: For web UI tasks, include screenshots showing (a) successful login, (b) repository page, and (c) the fork confirmation. A short screen recording is also recommended.

---

## ✅ Solution (step-by-step)

1. Open Gitea UI

- Click the Gitea button on the top bar (or open the Gitea URL provided by your environment).

2. Log in as `jon`

- On the Gitea login page enter:

  - Username: `jon`
  - Password: `Jon_pass123`

- Click **Sign In**.

3. Fork `sarah/story-blog`

- Use the search bar or navigate to the repository owner `sarah` and then to `story-blog`.
- On the repository page, click the **Fork** button (usually top-right).
- Choose the target account as `jon` (the UI will prompt where to fork). Confirm.
- Wait for the fork to complete and capture a screenshot of the forked repo under `jon/story-blog`.

4. Fork `Tork` (if present)

- Locate the `Tork` repository via search.
- Click **Fork** and select `jon` as the target account. Capture a screenshot after forking.

5. Verification

- After forking, visit `jon/story-blog` (or `jon/Tork`) in Gitea and confirm the default branch exists and you can view files.
- Optionally, clone the fork locally as `jon` to verify:

```bash
git clone git@<gitea-host>:jon/story-blog.git
cd story-blog
git branch -a
```

Replace `<gitea-host>` with the actual hostname.

---

## 📸 Screenshots & Recording

- Capture these steps:

  - Login screen showing `jon` is logged in.
  - The original repository page (`sarah/story-blog`).
  - The action of clicking **Fork** and the fork confirmation.
  - The forked repo under `jon` (e.g., `jon/story-blog`).

- A short Loom or screen recording helps reviewers confirm the UI steps.

---

## Notes & Assumptions

- The Gitea server URL and UI button are available in your environment.
- If the `jon` user does not exist, create the user first or ask the evaluator for credentials.
- If the repository names differ or `Tork` is absent, document what you found and proceed with available repos.
