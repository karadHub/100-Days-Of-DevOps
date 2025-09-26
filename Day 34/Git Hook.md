# Day 34: Git Hook — Auto-tag on master push

## 🎯 Task

- Merge `feature` into `master` in the working repo `/usr/src/kodekloudrepos/news`.
- In the bare repo `/opt/news.git`, create a post-update hook that, on pushes to `master`, creates a tag named `release-YYYY-MM-DD` (today’s date).
- Test the hook at least once so today’s tag is created.
- Perform actions as user `natasha` without changing existing permissions.

---

## 🛠 Steps (as natasha)

1. Merge feature into master (working clone)

```bash
sudo -iu natasha
cd /usr/src/kodekloudrepos/news
git fetch origin
git checkout master
git merge origin/feature   # or: git merge feature (if local branch exists)
# Resolve conflicts if any, then:
# git add <files> && git commit
```

2. Create the post-update hook in the bare repo

```bash
cd /opt/news.git/hooks
printf '%s\n' '#!/usr/bin/env bash
set -euo pipefail

DATE=$(date +%F)
TAG="release-$DATE"

# Only when master was updated
for ref in "$@"; do
  if [ "$ref" = "refs/heads/master" ]; then
    NEWREV=$(git rev-parse refs/heads/master)
    if git rev-parse -q --verify "refs/tags/$TAG" >/dev/null; then
      echo "Tag $TAG already exists; skipping."
    else
      git tag "$TAG" "$NEWREV"
      echo "Created tag: $TAG at $NEWREV"
    fi
  fi
done

# Keep server info updated (useful for dumb HTTP)
exec git update-server-info
' | tee post-update >/dev/null
chmod +x post-update
```

3. Push master to trigger the hook

```bash
cd /usr/src/kodekloudrepos/news
git push origin master
```

4. Verify the tag exists in the bare repo (origin)

```bash
# On the working clone
git ls-remote --tags origin | grep "release-$(date +%F)" || echo "Tag not found yet"

# Or inspect directly in the bare repo
cd /opt/news.git
git tag | grep "release-$(date +%F)"
git show "release-$(date +%F)" --no-patch --pretty=oneline
```

Notes

- The hook only tags when refs/heads/master is updated.
- If you push multiple times in a day, the tag already exists and is skipped.
- No ownership/permissions changed; actions run as `natasha`.
