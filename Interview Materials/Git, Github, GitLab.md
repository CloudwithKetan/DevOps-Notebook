# Git Scenario-Based Interview Questions & Answers
> Real-world problem-solving questions asked in interviews

---

## 🔴 SCENARIO 1: Accidental Commit to Wrong Branch

**Scenario:** You committed your changes directly to `main` instead of your feature branch. The commit hasn't been pushed yet. How do you fix it?

**Answer:**
```bash
# Step 1: Create the correct feature branch at current state
git branch feature/my-feature

# Step 2: Reset main back to before your commit
git reset --hard HEAD~1

# Step 3: Switch to the feature branch
git checkout feature/my-feature
```
Your commit is now only on `feature/my-feature` and `main` is clean.

---

## 🔴 SCENARIO 2: Pushed Sensitive Data (API Key/Password)

**Scenario:** You accidentally committed and pushed an API key to a public GitHub repo. What do you do?

**Answer:**

**Immediate steps (order matters):**

1. **Revoke the key first** — Go to your cloud provider/service and invalidate/rotate the key immediately. Assume it's already compromised.

2. **Remove from history using BFG Repo Cleaner (recommended):**
   ```bash
   # Download BFG jar, then:
   bfg --replace-text passwords.txt my-repo.git
   cd my-repo
   git reflog expire --expire=now --all
   git gc --prune=now --aggressive
   git push --force --all
   git push --force --tags
   ```

3. **Or use git filter-repo (modern approach):**
   ```bash
   git filter-repo --path secrets.env --invert-paths
   git push origin --force --all
   ```

4. **Notify all collaborators** to re-clone since history is rewritten.

5. **Add the file to `.gitignore`** to prevent recurrence.

> ⚠️ Even after removal, treat the key as permanently compromised — bots scrape GitHub in real time.

---

## 🔴 SCENARIO 3: Merge Conflict During Pull

**Scenario:** You ran `git pull` and got a merge conflict in `app.js`. Walk me through how you resolve it.

**Answer:**
```bash
# 1. See which files are conflicting
git status

# 2. Open app.js — Git marks conflicts like this:
<<<<<<< HEAD
  const port = 3000;   ← your local change
=======
  const port = 8080;   ← incoming remote change
>>>>>>> origin/main

# 3. Edit the file to keep the correct version (or combine both)
  const port = process.env.PORT || 3000;

# 4. Mark as resolved
git add app.js

# 5. Complete the merge
git commit

# 6. Push
git push origin main
```

**Tools that help:** `git mergetool`, VS Code's built-in merge editor, IntelliJ's 3-way merge view.

---

## 🔴 SCENARIO 4: Need to Undo the Last 3 Commits (Not Yet Pushed)

**Scenario:** You made 3 commits locally and realized all of them were wrong. You want to undo all 3 but keep your file changes in the working directory.

**Answer:**
```bash
# Keep changes in working directory (unstaged)
git reset --mixed HEAD~3

# OR: Keep changes staged
git reset --soft HEAD~3

# OR: Discard everything (DESTRUCTIVE)
git reset --hard HEAD~3
```

Use `--mixed` if you want to review and recommit selectively. Use `--soft` if you want to recommit as a single cleaner commit.

---

## 🔴 SCENARIO 5: Undo a Pushed Commit on a Shared Branch

**Scenario:** A bad commit was pushed to `main` which others are already using. You can't force push. How do you undo it safely?

**Answer:**
```bash
# Find the commit hash
git log --oneline

# Create a new commit that reverses the bad commit
git revert <bad-commit-hash>

# Push the revert commit normally
git push origin main
```

`git revert` is non-destructive — it adds a new commit that undoes the changes, preserving history. Safe for shared branches. Never use `git reset --hard` + force push on shared branches.

---

## 🔴 SCENARIO 6: Working on a Feature, Urgent Hotfix Needed

**Scenario:** You're halfway through a feature with uncommitted changes. A critical production bug is reported and you need to fix it right now on `main`. What do you do?

**Answer:**
```bash
# Save your in-progress work temporarily
git stash push -m "WIP: feature login form"

# Switch to main and create a hotfix branch
git checkout main
git checkout -b hotfix/critical-bug-fix

# Fix the bug, commit it
git add .
git commit -m "fix: resolve null pointer in payment processing"

# Merge hotfix to main
git checkout main
git merge --no-ff hotfix/critical-bug-fix
git push origin main

# Delete hotfix branch
git branch -d hotfix/critical-bug-fix

# Return to your feature branch and restore work
git checkout feature/login-form
git stash pop
```

---

## 🔴 SCENARIO 7: Recover a Deleted Branch

**Scenario:** You accidentally deleted a local branch that had 5 commits not yet merged. How do you recover it?

**Answer:**
```bash
# Step 1: Find the lost commit using reflog
git reflog
# Output shows: abc1234 HEAD@{2}: commit: last commit message on deleted branch

# Step 2: Recreate the branch from that commit
git checkout -b recovered-feature abc1234

# OR
git branch recovered-feature abc1234
```

`git reflog` is your safety net — it records every movement of HEAD for ~90 days by default. Commits aren't truly "deleted" until garbage collection runs.

---

## 🔴 SCENARIO 8: Your Feature Branch is Way Behind Main

**Scenario:** Your feature branch diverged from `main` 3 weeks ago. `main` has had 50+ commits since then. How do you catch up cleanly?

**Answer:**

**Option A — Rebase (cleaner, linear history):**
```bash
git checkout feature/my-feature
git fetch origin
git rebase origin/main
# Resolve any conflicts as they come up, then:
git rebase --continue
git push origin feature/my-feature --force-with-lease
```

**Option B — Merge (safer, preserves history):**
```bash
git checkout feature/my-feature
git fetch origin
git merge origin/main
# Resolve conflicts
git push origin feature/my-feature
```

**Prefer rebase** when your feature branch is personal and unshared. Use `--force-with-lease` instead of `--force` — it's safer as it won't push if someone else has pushed to the branch.

---

## 🔴 SCENARIO 9: Cherry-Pick a Bug Fix to a Release Branch

**Scenario:** A bug was fixed in `develop` but needs to be backported to the `release/v2.1` branch without merging all of develop. How?

**Answer:**
```bash
# Step 1: Find the exact commit hash of the bug fix
git log develop --oneline
# abc1234 fix: resolve login timeout issue

# Step 2: Switch to the release branch
git checkout release/v2.1

# Step 3: Cherry-pick just that commit
git cherry-pick abc1234

# Step 4: If conflict arises, resolve it, then:
git cherry-pick --continue

# Step 5: Push
git push origin release/v2.1
```

If you need multiple commits:
```bash
git cherry-pick abc1234 def5678 ghi9012
```

---

## 🔴 SCENARIO 10: Find Which Commit Introduced a Bug

**Scenario:** A bug exists in the current codebase but wasn't there 2 months ago. There are 300 commits between then and now. How do you find the exact commit that caused it?

**Answer:**
Use `git bisect` — binary search through commits:

```bash
# Start bisect session
git bisect start

# Mark current commit as bad
git bisect bad

# Mark a known good commit (2 months ago)
git bisect good v2.0.0
# OR by date: git bisect good HEAD@{2.months.ago}

# Git checks out a midpoint commit — test it
# Run your tests / reproduce the bug manually

# If bug is present:
git bisect bad

# If bug is absent:
git bisect good

# Git keeps narrowing down — repeat until it says:
# "abc1234 is the first bad commit"

# End session
git bisect reset
```

To **automate** bisect with a test script:
```bash
git bisect run npm test
```

---

## 🔴 SCENARIO 11: Accidentally Deleted a File and Committed

**Scenario:** You accidentally deleted `config/settings.json` and committed that deletion. You need it back.

**Answer:**
```bash
# Option 1: Revert just that commit
git revert <commit-hash>

# Option 2: Restore the file from a previous commit (surgical)
git checkout HEAD~1 -- config/settings.json
git commit -m "restore: accidentally deleted settings.json"

# Option 3: If you just committed and haven't pushed
git reset --soft HEAD~1    # Undo commit, keep changes staged
# Then un-delete the file (restore from stash/backup)
git restore config/settings.json  # from index if tracked
```

---

## 🔴 SCENARIO 12: Rewrite a Commit Message Already Pushed

**Scenario:** You pushed a commit with the message "asdfgh" and your team has a strict commit message policy. How do you fix it?

**Answer:**

⚠️ This rewrites history — coordinate with your team first.

```bash
# If it's the most recent commit:
git commit --amend -m "fix(auth): correct password validation logic"
git push --force-with-lease origin feature-branch

# If it's an older commit (interactive rebase):
git rebase -i HEAD~3   # Go back 3 commits
# In the editor, change 'pick' to 'reword' for the bad commit
# Save, then Git opens another editor for the new message
git push --force-with-lease origin feature-branch
```

> Only do this on feature branches you own. Never rewrite history on `main`.

---

## 🔴 SCENARIO 13: Squash All Commits Before Merging a PR

**Scenario:** Your feature branch has 15 messy "WIP" commits. Before merging, you want to squash them all into 1 clean commit.

**Answer:**

**Option A — Interactive Rebase:**
```bash
git rebase -i HEAD~15
# In editor, keep first as 'pick', change rest to 'squash' or 's'
# Save → Git opens editor for combined commit message
# Write clean message, save
git push --force-with-lease origin feature/my-feature
```

**Option B — Soft Reset:**
```bash
git reset --soft main    # Reset to where main is, keep all changes staged
git commit -m "feat(payments): add Stripe checkout integration"
git push --force-with-lease origin feature/my-feature
```

**Option C — Merge with --squash (done by reviewer):**
```bash
git checkout main
git merge --squash feature/my-feature
git commit -m "feat(payments): add Stripe checkout integration"
```

---

## 🔴 SCENARIO 14: Two Developers Edited the Same File

**Scenario:** You and a colleague both edited `UserController.java` on separate branches. Now both PRs are ready to merge. What's the correct process?

**Answer:**

1. **First PR merges cleanly** — whoever merges first has no conflict.
2. **Second PR needs to update from main:**
   ```bash
   git checkout feature/second-pr
   git fetch origin
   git rebase origin/main    # or git merge origin/main
   ```
3. **Conflict appears in UserController.java** — open the file:
   ```
   <<<<<<< HEAD (your changes)
   public User findByEmail(String email) { ... }
   =======
   public User findByUsername(String username) { ... }
   >>>>>>> origin/main
   ```
4. **Resolve** — keep both methods (or combine logic as needed).
5. **Stage and continue rebase:**
   ```bash
   git add UserController.java
   git rebase --continue
   git push --force-with-lease origin feature/second-pr
   ```

---

## 🔴 SCENARIO 15: You Need to Move Commits to a New Branch

**Scenario:** You've been making commits on `main` directly (forgetting to create a branch). You've made 4 commits. How do you move them to a new feature branch?

**Answer:**
```bash
# Step 1: Create a new branch at current position (captures the 4 commits)
git branch feature/new-feature

# Step 2: Move main back 4 commits (without touching working files)
git checkout main
git reset --hard HEAD~4

# Step 3: Switch to feature branch — commits are there
git checkout feature/new-feature

# Step 4: If main was already pushed, reset the remote too
git push origin main --force-with-lease
git push origin feature/new-feature
```

---

## 🔴 SCENARIO 16: View What Changed in a Specific Commit

**Scenario:** Your QA team says a bug appeared after a specific deploy. The deploy hash is `d4e5f6`. How do you inspect exactly what changed?

**Answer:**
```bash
# See full diff of that commit
git show d4e5f6

# See only file names that changed
git show d4e5f6 --stat

# See changes to a specific file in that commit
git show d4e5f6 -- src/auth/login.js

# Compare that commit to the one before it
git diff d4e5f6~1 d4e5f6

# See commit metadata only (no diff)
git show d4e5f6 -s
```

---

## 🔴 SCENARIO 17: Merge a Specific Folder from Another Branch

**Scenario:** Branch `design-overhaul` has new UI components in `src/components/`. You only want those files, not the whole branch merged.

**Answer:**
```bash
# Checkout (copy) specific folder from that branch
git checkout design-overhaul -- src/components/

# This stages those files from design-overhaul into your current branch
git status   # You'll see the files ready to commit

git commit -m "feat(ui): bring in redesigned components from design-overhaul"
```

For a single file:
```bash
git checkout design-overhaul -- src/components/Button.jsx
```

---

## 🔴 SCENARIO 18: Repo is Huge — Clone is Taking Forever

**Scenario:** A colleague joins and cloning the large monorepo takes 45 minutes. How do you speed this up?

**Answer:**

**Option A — Shallow clone (fastest):**
```bash
git clone --depth=1 https://github.com/org/repo.git
# Only clones the latest snapshot, no history
```

**Option B — Partial clone (skip large files):**
```bash
git clone --filter=blob:none https://github.com/org/repo.git
# Skips downloading file blobs until needed
```

**Option C — Sparse checkout (only needed directories):**
```bash
git clone --no-checkout https://github.com/org/repo.git
cd repo
git sparse-checkout init --cone
git sparse-checkout set src/frontend
git checkout main
```

**Option D — Use Git LFS** if the repo is large due to binary files.

---

## 🔴 SCENARIO 19: Detect Who Changed a Line of Code

**Scenario:** A production bug was traced to line 47 of `payment.js`. You need to know who wrote it, when, and in what commit.

**Answer:**
```bash
# Blame specific file
git blame payment.js

# Blame specific lines only
git blame -L 40,55 payment.js

# Output format:
# <hash> (<author> <date> <line>) <code>
# abc1234 (Jane Smith 2024-03-15 47)  return total * taxRate;

# See full commit details for that hash
git show abc1234

# See all commits that touched that line (follow history)
git log -L 47,47:payment.js
```

---

## 🔴 SCENARIO 20: Prevent Secrets from Being Committed

**Scenario:** You want to make sure no one on the team ever commits `.env` files or AWS keys. How do you enforce this?

**Answer:**

**Layer 1 — `.gitignore`:**
```
.env
.env.*
*.pem
secrets/
```

**Layer 2 — Pre-commit hook (`pre-commit` tool):**
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    hooks:
      - id: gitleaks
```

**Layer 3 — Server-side enforcement (GitHub/GitLab):**
- Enable **Secret Scanning** in GitHub (Settings → Security)
- GitLab has built-in **Secret Detection** in CI/CD

**Layer 4 — Git hook (manual `pre-commit`):**
```bash
#!/bin/bash
# .git/hooks/pre-commit
if git diff --cached | grep -E "(AKIA|api_key|secret|password)" ; then
  echo "Potential secret detected! Commit blocked."
  exit 1
fi
```

Use **Husky** to share hooks across the team via `package.json`.

---

## 🔴 SCENARIO 21: Multiple People Working on the Same Branch

**Scenario:** Three developers are working on the same `feature/checkout` branch and keep getting conflicts on every pull. How do you manage this?

**Answer:**

**Best practices:**
1. **Communicate ownership** — each dev owns specific files/modules.
2. **Pull with rebase** frequently:
   ```bash
   git pull --rebase origin feature/checkout
   ```
3. **Short-lived sub-branches:**
   ```bash
   git checkout -b feature/checkout-john
   # Work, then merge back to feature/checkout via PR
   ```
4. **Enable `rerere`** to auto-resolve repeated conflicts:
   ```bash
   git config rerere.enabled true
   ```
5. **Atomic commits** — commit and push small, focused changes often.

---

## 🔴 SCENARIO 22: Rollback a Merge That's Already in Main

**Scenario:** A feature was merged to `main` via a merge commit yesterday. It broke production. Rollback now.

**Answer:**
```bash
# Find the merge commit hash
git log --oneline --merges

# Revert the merge commit
# -m 1 means "keep parent 1" (main branch side, not the feature)
git revert -m 1 <merge-commit-hash>

git push origin main
```

The `-m 1` flag is critical for merge commits — it specifies which parent to revert to (1 = the branch you merged INTO, 2 = the branch you merged FROM).

---

## 🔴 SCENARIO 23: Tag a Specific Old Commit for a Release

**Scenario:** You forgot to tag commit `a1b2c3d` as `v1.5.0` when you released last week. How do you add the tag retroactively?

**Answer:**
```bash
# Create annotated tag on a specific past commit
git tag -a v1.5.0 a1b2c3d -m "Release v1.5.0 - payment module"

# Push the specific tag to remote
git push origin v1.5.0

# Push all tags
git push origin --tags

# Verify
git show v1.5.0
```

---

## 🔴 SCENARIO 24: Git Pull Rejected — Non-Fast-Forward Error

**Scenario:** You try to push and get: `error: failed to push some refs — non-fast-forward`. What does this mean and how do you fix it?

**Answer:**

This means the remote branch has commits your local branch doesn't have. Someone else pushed while you were working.

**Fix:**
```bash
# Option A — Pull and merge (creates merge commit)
git pull origin main
git push origin main

# Option B — Pull with rebase (linear history, cleaner)
git pull --rebase origin main
git push origin main

# Option C — Fetch first, then decide
git fetch origin
git log HEAD..origin/main --oneline   # See what's new
git rebase origin/main                # Then rebase
git push origin main
```

Never use `--force` unless you own the branch and know what you're doing.

---

## 🔴 SCENARIO 25: Repo Has a File That's Different on Every Machine

**Scenario:** `config/local.properties` needs to exist for the app to run but differs per developer. It keeps showing up as modified in `git status`.

**Answer:**

**Option A — `git update-index --assume-unchanged`:**
```bash
git update-index --assume-unchanged config/local.properties
# Git will ignore local changes to this file
# Undo: git update-index --no-assume-unchanged config/local.properties
```

**Option B — `git update-index --skip-worktree`** (better for config files):
```bash
git update-index --skip-worktree config/local.properties
```

**Option C — Use a template pattern (best practice):**
- Commit `config/local.properties.template` with placeholder values
- Add `config/local.properties` to `.gitignore`
- Developers copy the template: `cp config/local.properties.template config/local.properties`

---

## 🔴 SCENARIO 26: Comparing Your Branch to Main Before a PR

**Scenario:** Before opening a PR, you want to see exactly what commits and file changes your branch introduces compared to `main`.

**Answer:**
```bash
# See all commits your branch has that main doesn't
git log main..HEAD --oneline

# See all file changes
git diff main...HEAD

# See only which files changed (no diff content)
git diff main...HEAD --name-only

# See stat summary
git diff main...HEAD --stat

# See commits in a nice graph
git log --oneline --graph main..HEAD
```

Note: `..` = commits in HEAD not in main. `...` = diff from the common ancestor (what a PR shows).

---

## 🔴 SCENARIO 27: You Cloned a Repo But Can't Push

**Scenario:** You cloned a project, made changes, but `git push` returns `Permission denied (publickey)`.

**Answer:**

**Diagnosis:**
```bash
ssh -T git@github.com     # Test SSH connection
git remote -v              # Check if using SSH or HTTPS
```

**Fix 1 — No SSH key set up:**
```bash
# Generate key
ssh-keygen -t ed25519 -C "your@email.com"
# Add to agent
ssh-add ~/.ssh/id_ed25519
# Copy public key and add to GitHub/GitLab → SSH Keys
cat ~/.ssh/id_ed25519.pub
```

**Fix 2 — Wrong remote URL (HTTPS vs SSH):**
```bash
# Switch from HTTPS to SSH
git remote set-url origin git@github.com:user/repo.git
```

**Fix 3 — No write access:**
- Fork the repo and push to your fork
- Request collaborator access from the repo owner

---

## 🔴 SCENARIO 28: Clean Up Local Branches Already Merged to Main

**Scenario:** After months of work, you have 30+ local branches, most of which are already merged. How do you clean them up?

**Answer:**
```bash
# Delete all local branches already merged to main
git branch --merged main | grep -v "^\* main" | xargs git branch -d

# Alternatively, one at a time
git branch -d feature/old-feature

# Force delete unmerged branch (careful!)
git branch -D feature/abandoned

# Remove remote-tracking references that no longer exist on remote
git fetch --prune
# or
git remote prune origin

# Also prune automatically on every fetch
git config --global fetch.prune true
```

---

## 🔴 SCENARIO 29: Your `git log` Shows Duplicate Commits After Rebase

**Scenario:** After rebasing and pushing, reviewers say the PR history now shows the same commits twice. Why and how to fix?

**Answer:**

**Why it happens:** When you rebase a branch that someone else has already pulled, their copy still has the old commits. After your force-push, both old and new commits appear in their view.

**Fix:**
```bash
# Communicate with team: everyone on the branch must reset
git fetch origin
git reset --hard origin/feature-branch
```

**Prevention:**
- Only rebase branches that aren't shared with others
- Use `--force-with-lease` instead of `--force` — it fails if someone else has pushed
  ```bash
  git push --force-with-lease origin feature-branch
  ```

---

## 🔴 SCENARIO 30: Set Up a Repository Mirror (Backup to Another Remote)

**Scenario:** Your company wants to mirror a GitHub repo to a self-hosted GitLab instance for backup. How do you set this up?

**Answer:**

**Option A — Manual mirror push:**
```bash
# Add second remote
git remote add gitlab git@gitlab.company.com:team/repo.git

# Push all branches and tags
git push gitlab --mirror
```

**Option B — Automatic via GitHub Actions:**
```yaml
name: Mirror to GitLab
on: [push]
jobs:
  mirror:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - run: |
          git push --mirror git@gitlab.company.com:team/repo.git
```

**Option C — GitLab's built-in pull mirroring:**
Go to GitLab → Settings → Repository → Mirroring repositories → Add remote. GitLab will pull from GitHub on a schedule.

---

## 🔴 SCENARIO 31: Handle a Rebase That's Going Wrong

**Scenario:** You started an interactive rebase of 10 commits but now you have a massive conflict on commit 4 and you're not sure if you should continue. How do you abort and go back to where you started?

**Answer:**
```bash
# Abort the rebase entirely and return to original state
git rebase --abort

# This returns HEAD to where it was before you started the rebase
# You're back on your original branch with original commits
```

If you want to **skip** just the problematic commit instead:
```bash
git rebase --skip   # Skip current commit and continue
```

If you want to **pause and fix** the conflict:
```bash
# Resolve conflicts in the file
git add <resolved-file>
git rebase --continue
```

---

## 🔴 SCENARIO 32: Check if a Commit Exists in a Branch

**Scenario:** QA asks you to verify if a specific bug fix commit (`a1b2c3`) is included in the `release/v3.0` branch before they test.

**Answer:**
```bash
# Check if commit exists in a specific branch
git branch --contains a1b2c3

# If release/v3.0 appears in output → commit is included ✅
# If it doesn't appear → commit is NOT in that branch ❌

# More specific check
git log release/v3.0 --oneline | grep a1b2c3

# Or from the commit perspective
git merge-base --is-ancestor a1b2c3 release/v3.0
echo $?   # 0 = ancestor (included), 1 = not included
```

---

## 🔴 SCENARIO 33: Work with Git in a CI/CD Pipeline

**Scenario:** In your GitHub Actions pipeline, you need to (1) checkout code, (2) get the current git tag, (3) use it to version a Docker image. How?

**Answer:**
```yaml
name: Build and Push Docker Image

on:
  push:
    tags:
      - 'v*'

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0   # Important: fetch full history for tags

      - name: Get version from tag
        id: version
        run: echo "VERSION=${GITHUB_REF#refs/tags/}" >> $GITHUB_OUTPUT

      - name: Build Docker image
        run: docker build -t myapp:${{ steps.version.outputs.VERSION }} .

      - name: Push to registry
        run: docker push myapp:${{ steps.version.outputs.VERSION }}
```

Get git info in scripts:
```bash
git describe --tags        # v1.2.3-4-gabcdef7
git rev-parse HEAD         # Full SHA
git rev-parse --short HEAD # Short SHA
```

---

## 🔴 SCENARIO 34: Partial Stage — Only Commit Some Changes in a File

**Scenario:** You made two unrelated changes in `utils.js` — a bug fix and a new feature. You want to commit them separately for a clean history. How?

**Answer:**
```bash
# Interactively stage parts of a file (patch mode)
git add -p utils.js
# or
git add --patch utils.js
```

Git will show each "hunk" (change block) and ask:
```
Stage this hunk [y,n,q,a,d,s,?]?
  y = yes, stage this hunk
  n = no, skip this hunk
  s = split into smaller hunks
  e = manually edit the hunk
  q = quit
```

Stage only the bug fix hunks → commit → then stage feature hunks → commit separately.

You can also do this visually in VS Code (click the `+` gutter icon per line) or GitKraken/SourceTree.

---

## 🔴 SCENARIO 35: Check Out a Pull Request Locally Without a Branch

**Scenario:** You're reviewing a colleague's PR #42 on GitHub. You want to check out that PR's code locally to test it, but they haven't shared a branch name.

**Answer:**
```bash
# Fetch the PR ref
git fetch origin pull/42/head:pr-42

# Switch to it
git checkout pr-42

# Test the changes locally
# When done
git checkout main
git branch -d pr-42
```

For GitLab Merge Requests:
```bash
git fetch origin merge-requests/42/head:mr-42
git checkout mr-42
```

You can also set up automatic fetching of all PRs:
```bash
# Add to .git/config under [remote "origin"]
fetch = +refs/pull/*/head:refs/remotes/origin/pr/*
```

---

*© Git Scenario-Based Interview Q&A — 35 Real-World Scenarios*
*Covers: Accidental commits · Conflict resolution · History rewriting · Recovery · Collaboration · CI/CD*
