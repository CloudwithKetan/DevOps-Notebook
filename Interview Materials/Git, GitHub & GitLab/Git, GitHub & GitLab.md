# Git, GitHub & GitLab Interview Questions & Answers

---

## 🟢 BASICS & FUNDAMENTALS

---

### 1. What is Git?
**Answer:**
Git is a distributed version control system (DVCS) that tracks changes in source code during software development. It allows multiple developers to collaborate on a project simultaneously, maintain a history of changes, and revert to previous versions if needed. Created by Linus Torvalds in 2005, Git stores data as snapshots of the project rather than file-based differences.

---

### 2. What is the difference between Git and GitHub/GitLab?
**Answer:**
- **Git** is a version control tool installed locally on your machine. It manages source code history.
- **GitHub** is a cloud-based hosting service for Git repositories. It adds features like pull requests, issues, actions (CI/CD), and project management.
- **GitLab** is similar to GitHub but can be self-hosted and includes built-in CI/CD pipelines, container registry, and DevOps lifecycle tools out of the box.

---

### 3. What is a repository in Git?
**Answer:**
A repository (repo) is a directory that contains your project files along with the entire history of changes made to those files. It includes a hidden `.git` folder that stores all metadata and version history. Repositories can be **local** (on your machine) or **remote** (on a server like GitHub/GitLab).

---

### 4. What is the difference between `git init` and `git clone`?
**Answer:**
- `git init` — Creates a brand new, empty Git repository in the current directory. Used when starting a project from scratch.
- `git clone <url>` — Creates a copy of an existing remote repository on your local machine. It copies all files, branches, and commit history.

```bash
git init              # Initialize new repo
git clone https://github.com/user/repo.git   # Clone existing repo
```

---

### 5. What are the three states of a file in Git?
**Answer:**
1. **Modified** — The file has been changed but not yet staged.
2. **Staged** — The file is marked to be included in the next commit (added to the staging area).
3. **Committed** — The data is safely stored in the local repository.

These map to three sections: **Working Directory → Staging Area → Repository**.

---

### 6. What is the staging area (index)?
**Answer:**
The staging area (also called the index) is an intermediate area where changes are collected before committing. It lets you craft commits precisely — you can stage only certain files or even certain lines within a file, giving you fine-grained control over what goes into each commit.

```bash
git add file.txt       # Stage a specific file
git add .              # Stage all changes
git status             # See staged/unstaged files
```

---

### 7. What is `HEAD` in Git?
**Answer:**
`HEAD` is a special pointer that refers to the current snapshot you're working on — usually the latest commit on the currently checked-out branch. It tells Git "this is where I am right now." When you switch branches, `HEAD` moves to point to that branch's latest commit. A **detached HEAD** occurs when `HEAD` points directly to a commit rather than a branch.

---

### 8. What is the difference between `git fetch` and `git pull`?
**Answer:**
- **`git fetch`** — Downloads changes from the remote repository into your local repo but does **not** merge them into your working branch. It's safe and non-destructive.
- **`git pull`** — Fetches changes AND automatically merges them into your current branch. It's essentially `git fetch` + `git merge`.

```bash
git fetch origin        # Download without merging
git pull origin main    # Download and merge
```

---

### 9. What is `git push`?
**Answer:**
`git push` uploads your local commits to a remote repository. It updates the remote branch with your local changes.

```bash
git push origin main          # Push to main branch
git push -u origin feature    # Push and set upstream
git push --force              # Force push (use with caution!)
```

---

### 10. What is a `.gitignore` file?
**Answer:**
A `.gitignore` file specifies intentionally untracked files that Git should ignore. Patterns defined in this file tell Git not to track files like build artifacts, environment files, logs, or IDE-specific files.

```
node_modules/
*.log
.env
dist/
.DS_Store
```

---

## 🔵 BRANCHING & MERGING

---

### 11. What is a branch in Git?
**Answer:**
A branch is a lightweight, movable pointer to a specific commit. It allows you to diverge from the main line of development and work independently without affecting other branches. The default branch is typically called `main` (or `master` in older repos). Branches are cheap to create and delete in Git.

```bash
git branch feature-login    # Create branch
git checkout feature-login  # Switch to branch
git checkout -b feature-login  # Create and switch
```

---

### 12. What is the difference between `git merge` and `git rebase`?
**Answer:**
- **`git merge`** — Combines two branches by creating a new **merge commit** that has two parents. It preserves the full history and context of both branches. Non-destructive.
- **`git rebase`** — Moves or "replays" commits from one branch onto another. It rewrites commit history, producing a **linear history** without merge commits. Cleaner but rewrites history.

**Rule of thumb:** Use `merge` for shared/public branches; use `rebase` to clean up local feature branches before merging.

---

### 13. What is a fast-forward merge?
**Answer:**
A fast-forward merge happens when the branch being merged into has not diverged — the target branch is directly behind the incoming branch. Git simply moves the pointer forward without creating a merge commit. This results in a linear history.

```bash
git merge feature-branch     # Fast-forward if no divergence
git merge --no-ff feature    # Force a merge commit even if fast-forward is possible
```

---

### 14. How do you resolve a merge conflict?
**Answer:**
1. Run `git merge` or `git pull` — Git will flag conflicting files.
2. Open the conflicting files. Git marks conflicts like:
   ```
   <<<<<<< HEAD
   your changes
   =======
   incoming changes
   >>>>>>> feature-branch
   ```
3. Edit the file to resolve the conflict manually.
4. Stage the resolved file: `git add <file>`
5. Complete the merge: `git commit`

Tools like `git mergetool` or IDE integrations can assist visually.

---

### 15. What is `git cherry-pick`?
**Answer:**
`git cherry-pick` applies the changes introduced by a specific commit from one branch to another. Useful when you want to port a bug fix from one branch to another without merging the entire branch.

```bash
git cherry-pick <commit-hash>          # Apply a single commit
git cherry-pick abc123 def456          # Apply multiple commits
git cherry-pick feature~2..feature    # Apply a range
```

---

### 16. What is the difference between `git reset`, `git revert`, and `git checkout`?
**Answer:**
| Command | Effect | History |
|---|---|---|
| `git reset` | Moves HEAD and optionally modifies index/working tree | Rewrites history |
| `git revert` | Creates a new commit that undoes a previous commit | Safe, keeps history |
| `git checkout` | Switches branches or restores files | No history change |

Use `revert` on shared/public branches; `reset` only on local/private branches.

---

### 17. What are the types of `git reset`?
**Answer:**
- `--soft` — Moves HEAD to specified commit; staged changes are kept.
- `--mixed` (default) — Moves HEAD; changes are unstaged but kept in working directory.
- `--hard` — Moves HEAD; all changes in staging and working directory are **discarded**. Dangerous!

```bash
git reset --soft HEAD~1    # Undo last commit, keep staged
git reset --mixed HEAD~1   # Undo last commit, keep unstaged
git reset --hard HEAD~1    # Undo last commit, discard changes
```

---

### 18. What is `git stash`?
**Answer:**
`git stash` temporarily shelves (saves) changes you've made to your working directory so you can work on something else. The stash is like a clipboard for uncommitted changes.

```bash
git stash             # Stash current changes
git stash list        # View all stashes
git stash pop         # Apply last stash and remove it
git stash apply       # Apply last stash but keep it
git stash drop        # Delete a stash
git stash branch new-branch  # Create branch from stash
```

---

### 19. What is the difference between `git stash pop` and `git stash apply`?
**Answer:**
- `git stash pop` — Applies the most recent stash to the working directory and **removes** it from the stash list.
- `git stash apply` — Applies the most recent stash but **keeps** it in the stash list for later use.

---

### 20. What is a detached HEAD state?
**Answer:**
A detached HEAD state occurs when `HEAD` points directly to a commit hash instead of a branch name. This happens when you check out a specific commit, tag, or remote branch directly. Any commits made in this state are not attached to a branch and may be lost when you switch branches.

```bash
git checkout abc1234    # Enters detached HEAD
git checkout -b new-branch  # Save work by creating a new branch
```

---

## 🟡 ADVANCED GIT COMMANDS

---

### 21. What is `git rebase -i` (interactive rebase)?
**Answer:**
Interactive rebase lets you rewrite, squash, reorder, or remove commits before sharing them. It opens an editor where you can mark commits with actions:
- `pick` — keep the commit
- `squash` / `fixup` — combine with previous commit
- `reword` — change commit message
- `drop` — delete the commit
- `edit` — pause to amend

```bash
git rebase -i HEAD~5   # Interactively edit last 5 commits
```

---

### 22. What is `git bisect`?
**Answer:**
`git bisect` helps you find the commit that introduced a bug using binary search. You mark a known good commit and a known bad commit; Git automatically checks out commits in between for you to test.

```bash
git bisect start
git bisect bad              # Current commit is bad
git bisect good v1.0        # v1.0 was good
# Git checks out a midpoint — test it, then:
git bisect good / git bisect bad
git bisect reset            # End session
```

---

### 23. What is `git reflog`?
**Answer:**
`git reflog` shows a log of all reference updates in your local repo — including commits, checkouts, merges, and resets — even those no longer referenced by any branch. It's a safety net for recovering "lost" commits after a bad reset or rebase.

```bash
git reflog              # See all reference history
git checkout HEAD@{3}   # Go back to a specific state
```

---

### 24. What is `git tag` and when would you use it?
**Answer:**
Tags are references to specific commits, commonly used to mark release points (e.g., `v1.0.0`). Unlike branches, tags don't move — they're permanent markers.

- **Lightweight tag** — Just a pointer to a commit.
- **Annotated tag** — Stores extra metadata (tagger name, date, message).

```bash
git tag v1.0.0                          # Lightweight tag
git tag -a v1.0.0 -m "Release 1.0.0"   # Annotated tag
git push origin v1.0.0                  # Push specific tag
git push origin --tags                  # Push all tags
```

---

### 25. What is `git log` and how can you customize its output?
**Answer:**
`git log` shows the commit history. It's highly customizable:

```bash
git log --oneline              # One line per commit
git log --graph --all          # ASCII graph of all branches
git log -p                     # Show diff for each commit
git log --author="John"        # Filter by author
git log --since="2 weeks ago"  # Filter by date
git log --grep="fix"           # Search commit messages
git log --oneline --graph --decorate --all  # Popular combo
```

---

### 26. What is the difference between `git diff` options?
**Answer:**
```bash
git diff                    # Working directory vs staging area
git diff --staged           # Staging area vs last commit
git diff HEAD               # Working directory vs last commit
git diff branch1..branch2   # Differences between two branches
git diff abc123 def456      # Differences between two commits
```

---

### 27. What is `git blame`?
**Answer:**
`git blame` shows who last modified each line of a file and in which commit. Useful for tracking down who introduced a specific line of code or bug.

```bash
git blame filename.js
git blame -L 10,20 filename.js   # Blame specific lines 10-20
```

---

### 28. How do you squash commits?
**Answer:**
Squashing combines multiple commits into one, often to clean up history before merging. You can do this via:

1. **Interactive rebase:**
   ```bash
   git rebase -i HEAD~3
   # Mark commits as 'squash' or 's'
   ```

2. **Merge with squash flag:**
   ```bash
   git merge --squash feature-branch
   git commit -m "Single clean commit"
   ```

---

### 29. What is `git worktree`?
**Answer:**
`git worktree` lets you check out multiple branches simultaneously into separate directories from a single repository. Useful when you need to work on two branches at the same time without stashing or committing.

```bash
git worktree add ../hotfix hotfix-branch   # Check out branch in a new folder
git worktree list                          # List all worktrees
git worktree remove ../hotfix              # Remove worktree
```

---

### 30. What is `git submodule`?
**Answer:**
Git submodules allow you to embed one Git repository inside another. Useful when your project depends on another project's source code (e.g., a shared library). The parent repo stores a reference to a specific commit of the submodule.

```bash
git submodule add https://github.com/lib/repo libs/repo
git submodule update --init --recursive   # Initialize and update submodules
```

---

## 🟠 WORKFLOWS & COLLABORATION

---

### 31. What is the Git Flow branching strategy?
**Answer:**
Git Flow is a popular branching model with defined branch types:
- **`main`** — Production-ready code
- **`develop`** — Integration branch for features
- **`feature/*`** — New features (branch from develop)
- **`release/*`** — Release preparation (branch from develop, merge to main + develop)
- **`hotfix/*`** — Emergency fixes (branch from main, merge to main + develop)

It provides structure but can feel heavy for smaller teams.

---

### 32. What is GitHub Flow?
**Answer:**
GitHub Flow is a simpler, lightweight workflow:
1. Create a branch from `main`
2. Add commits
3. Open a Pull Request
4. Discuss and review
5. Deploy and test
6. Merge to `main`

It suits teams that deploy frequently and prefer simplicity over formality.

---

### 33. What is a Pull Request (PR)?
**Answer:**
A Pull Request is a feature of GitHub/GitLab that lets you notify team members about changes you've pushed to a branch. It opens a discussion thread where code can be reviewed, commented on, and approved before merging into the target branch. PRs are central to collaborative code review workflows.

---

### 34. What is a Merge Request in GitLab?
**Answer:**
A Merge Request (MR) in GitLab is GitLab's equivalent of GitHub's Pull Request. It serves the same purpose — proposing and reviewing changes before merging. GitLab's MRs also integrate tightly with CI/CD pipelines, showing test results and deployment status directly in the MR interface.

---

### 35. What is the difference between `git merge --squash` and a regular merge?
**Answer:**
- **Regular merge** — Brings all individual commits from the feature branch into the target branch, preserving full commit history.
- **`--squash` merge** — Combines all commits from the feature branch into a single staged change, which you then commit as one clean commit. The original feature branch commits don't appear in the target branch history.

---

### 36. What is code review and why is it important?
**Answer:**
Code review is the process of having teammates examine your code changes (via PR/MR) before merging. Benefits:
- Catches bugs and logic errors early
- Enforces coding standards
- Spreads knowledge across the team
- Improves code quality and design
- Documents why changes were made (in comments/discussions)

---

### 37. What are protected branches?
**Answer:**
Protected branches in GitHub/GitLab prevent unauthorized or accidental changes to critical branches like `main` or `production`. Rules can include:
- Require pull request reviews before merging
- Require status checks (CI) to pass
- Restrict who can push directly
- Prevent force pushes or branch deletion

---

### 38. What is a fork and how is it different from a branch?
**Answer:**
- **Fork** — A complete copy of a repository under your own GitHub/GitLab account. Used in open-source workflows where you don't have write access to the original repo. You fork, make changes, and submit a PR back.
- **Branch** — A pointer within the same repository. Used within a team that has access to the same repo.

---

### 39. What is `git remote` and how do you manage remotes?
**Answer:**
A remote is a version of your repository hosted on the internet or a network. `git remote` manages remote connections.

```bash
git remote -v                          # List remotes with URLs
git remote add origin <url>            # Add a remote
git remote rename origin upstream      # Rename remote
git remote remove origin               # Remove remote
git remote set-url origin <new-url>    # Change remote URL
```

---

### 40. What is an upstream repository?
**Answer:**
In a forked workflow, the **upstream** is the original repository you forked from. You typically add it as a remote to sync changes from the original project into your fork.

```bash
git remote add upstream https://github.com/original/repo.git
git fetch upstream
git merge upstream/main
```

---

## 🔴 GITHUB & GITLAB SPECIFIC

---

### 41. What is GitHub Actions?
**Answer:**
GitHub Actions is a CI/CD and automation platform built into GitHub. Workflows are defined in YAML files inside `.github/workflows/`. They can trigger on events like push, PR, schedule, or manual dispatch, and can run tests, build Docker images, deploy to cloud, and more.

```yaml
name: CI
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm test
```

---

### 42. What is GitLab CI/CD?
**Answer:**
GitLab CI/CD is GitLab's built-in continuous integration and delivery system. Pipelines are defined in a `.gitlab-ci.yml` file at the root of the repo. GitLab Runners execute jobs in stages (e.g., build → test → deploy).

```yaml
stages:
  - build
  - test
  - deploy

test:
  stage: test
  script:
    - npm test
```

---

### 43. What is a GitHub/GitLab webhook?
**Answer:**
Webhooks are HTTP callbacks sent to a specified URL when certain events happen in a repository (e.g., push, PR open, issue created). They allow external systems (like CI servers, Slack bots, or deployment tools) to react to repository events in real time.

---

### 44. What are GitHub Issues?
**Answer:**
GitHub Issues are a built-in issue tracking system for managing bugs, feature requests, tasks, and discussions. Issues can be labeled, assigned to team members, linked to milestones, and referenced in commits/PRs using `#issue-number`. Closing keywords like `Fixes #42` in a commit message or PR can auto-close linked issues.

---

### 45. What is a GitLab Runner?
**Answer:**
A GitLab Runner is an agent that executes CI/CD jobs defined in `.gitlab-ci.yml`. Runners can be:
- **Shared runners** — Provided by GitLab, available to all projects
- **Group runners** — Scoped to a group of projects
- **Specific/Project runners** — Registered for a single project

Runners support different executors: shell, Docker, Kubernetes, etc.

---

### 46. What is GitHub Packages?
**Answer:**
GitHub Packages is a package hosting service integrated with GitHub. It lets you publish and consume packages (npm, Maven, NuGet, Docker, RubyGems, etc.) directly within GitHub, keeping your source code and packages in one place with unified access control.

---

### 47. What is GitLab's container registry?
**Answer:**
GitLab's container registry is a built-in Docker image registry. It allows you to store and manage Docker images tied directly to your GitLab project. CI/CD pipelines can push images to it, and deployments can pull from it — all within the same platform, without needing Docker Hub.

---

### 48. What is a CODEOWNERS file?
**Answer:**
The `CODEOWNERS` file (in GitHub/GitLab) defines individuals or teams that are responsible for specific parts of the codebase. When a PR touches files owned by someone, they are automatically added as reviewers.

```
# CODEOWNERS example
*.js @frontend-team
/src/api/ @backend-team
README.md @docs-team
```

---

### 49. What is semantic versioning in the context of Git tags?
**Answer:**
Semantic Versioning (SemVer) is a versioning scheme: `MAJOR.MINOR.PATCH`
- **MAJOR** — Breaking changes
- **MINOR** — New backward-compatible features
- **PATCH** — Backward-compatible bug fixes

Git tags are used to mark release versions following SemVer (e.g., `v2.3.1`). Tools like `semantic-release` can automate version bumping based on commit message conventions.

---

### 50. What is the difference between SSH and HTTPS for Git remotes?
**Answer:**
- **HTTPS** — Authenticates with username/password or personal access token. Easier to set up, works through firewalls. Credentials can be cached.
- **SSH** — Authenticates with an SSH key pair. More secure, no password prompts once configured. Requires SSH key setup on the machine and added to your GitHub/GitLab account.

```bash
# HTTPS remote
git remote add origin https://github.com/user/repo.git

# SSH remote
git remote add origin git@github.com:user/repo.git
```

---

## 🟣 COMMIT BEST PRACTICES & HISTORY

---

### 51. What makes a good commit message?
**Answer:**
A good commit message follows this structure:
```
<type>(<scope>): <short summary>   ← Subject line (50 chars max)

<longer explanation if needed>     ← Body (wrap at 72 chars)

<footer: issue refs, breaking changes>
```

**Types (Conventional Commits):** `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

Example:
```
feat(auth): add JWT token refresh logic

Implemented token refresh endpoint to handle expiry gracefully.
Users are now automatically re-authenticated.

Closes #89
```

---

### 52. What is the Conventional Commits specification?
**Answer:**
Conventional Commits is a standard for commit message format that enables automated tooling. The format is:
```
<type>[optional scope]: <description>
```

Benefits:
- Auto-generated changelogs
- Automated semantic version bumps
- Better readability and filtering
- Works with tools like `semantic-release`, `commitlint`, `standard-version`

---

### 53. How do you amend the last commit?
**Answer:**
Use `git commit --amend` to modify the most recent commit — change its message or add/remove staged changes.

```bash
git add forgotten-file.js
git commit --amend                      # Opens editor for message
git commit --amend -m "New message"     # Directly change message
git commit --amend --no-edit            # Keep message, just add staged files
```

⚠️ Never amend commits that have already been pushed to a shared branch.

---

### 54. What is `git clean`?
**Answer:**
`git clean` removes untracked files from the working directory. Useful for cleaning up build artifacts or generated files not tracked by Git.

```bash
git clean -n     # Dry run — show what would be deleted
git clean -f     # Force delete untracked files
git clean -fd    # Delete untracked files and directories
git clean -fdx   # Also delete ignored files (.gitignore'd)
```

---

### 55. How do you remove a file from Git tracking without deleting it?
**Answer:**
Use `git rm --cached` to stop tracking a file while keeping it in the working directory. Then add it to `.gitignore`.

```bash
git rm --cached secrets.env
echo "secrets.env" >> .gitignore
git commit -m "Stop tracking secrets.env"
```

---

## 🔶 SECURITY & CONFIGURATION

---

### 56. How do you configure Git globally?
**Answer:**
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global core.editor "code --wait"   # VS Code
git config --global init.defaultBranch main
git config --list                                # View all settings
```

Config levels: `--system` (all users) > `--global` (current user) > `--local` (current repo).

---

### 57. How do you set up SSH keys for GitHub/GitLab?
**Answer:**
1. Generate key pair: `ssh-keygen -t ed25519 -C "your@email.com"`
2. Add private key to SSH agent: `ssh-add ~/.ssh/id_ed25519`
3. Copy public key: `cat ~/.ssh/id_ed25519.pub`
4. Paste into GitHub → Settings → SSH Keys / GitLab → Preferences → SSH Keys
5. Test: `ssh -T git@github.com`

---

### 58. What is a Personal Access Token (PAT)?
**Answer:**
A PAT is a token used in place of a password for Git operations over HTTPS and for API authentication. It's more secure than passwords because:
- Can be scoped to specific permissions
- Can be revoked individually
- Can have expiration dates

Used when HTTPS authentication is needed (especially in CI/CD scripts).

---

### 59. How do you remove sensitive data accidentally committed to Git?
**Answer:**
1. **`git filter-branch`** (legacy) or **BFG Repo Cleaner** (recommended):
   ```bash
   bfg --delete-files secrets.env
   git push --force --all
   ```
2. Use `git filter-repo` (modern tool).
3. Rotate/invalidate the exposed credentials immediately regardless of cleanup.
4. Force push all branches and tags after rewriting history.
5. Ask all collaborators to re-clone or rebase.

---

### 60. What is `git gc` (garbage collection)?
**Answer:**
`git gc` runs housekeeping tasks to optimize your repository:
- Compresses file revisions
- Removes unreachable objects
- Packs loose objects into pack files
- Prunes old reflog entries

Git runs this automatically over time, but you can run it manually to reduce repo size.

```bash
git gc           # Standard cleanup
git gc --prune=now   # Remove all unreachable objects immediately
```

---

## ⚫ SCENARIO-BASED QUESTIONS

---

### 61. How do you undo a pushed commit without force pushing?
**Answer:**
Use `git revert` to create a new commit that undoes the changes. This is safe for shared branches.

```bash
git revert <commit-hash>   # Creates a new "undo" commit
git push origin main       # Push the revert commit normally
```

---

### 62. How do you merge only specific files from another branch?
**Answer:**
Use `git checkout` (or `git restore`) to bring specific files from another branch:

```bash
git checkout feature-branch -- path/to/file.js
git checkout feature-branch -- src/components/
git commit -m "Bring specific file from feature-branch"
```

---

### 63. How do you recover a deleted branch?
**Answer:**
Use `git reflog` to find the last commit hash of the deleted branch, then recreate it:

```bash
git reflog                         # Find the commit SHA
git checkout -b recovered-branch <SHA>
# or
git branch recovered-branch <SHA>
```

---

### 64. What happens when you run `git pull --rebase`?
**Answer:**
Instead of merging remote changes into your local branch (creating a merge commit), `git pull --rebase` replays your local commits on top of the fetched remote commits. This keeps a linear history and avoids unnecessary merge commits.

```bash
git pull --rebase origin main
```

You can set this as default: `git config --global pull.rebase true`

---

### 65. How do you rename a branch?
**Answer:**
```bash
# Rename locally
git branch -m old-name new-name

# If you're on the branch you want to rename
git branch -m new-name

# Push new name to remote and delete old
git push origin -u new-name
git push origin --delete old-name
```

---

### 66. How do you compare two branches?
**Answer:**
```bash
git diff main..feature-branch           # Show all differences
git log main..feature-branch            # Commits in feature not in main
git log --left-right main...feature     # Commits in either, not both
```

---

### 67. How do you list all branches (local and remote)?
**Answer:**
```bash
git branch         # Local branches only
git branch -r      # Remote branches only
git branch -a      # All branches (local + remote)
git branch -vv     # Local branches with tracking info
```

---

### 68. What is `git shortlog`?
**Answer:**
`git shortlog` groups commit messages by author, making it useful for generating changelogs or seeing contribution summaries.

```bash
git shortlog -sn          # Count commits per author, sorted by count
git shortlog -sn --all    # Include all branches
```

---

### 69. How do you apply a patch file in Git?
**Answer:**
```bash
# Create a patch
git format-patch -1 <commit-hash>    # Single commit
git diff > changes.patch             # From diff

# Apply a patch
git apply changes.patch              # Apply without committing
git am fix.patch                     # Apply as a commit (preserves author)
```

---

### 70. What is `git archive`?
**Answer:**
`git archive` creates an archive (zip or tar) of files from a specific tree or commit, without the `.git` history. Useful for creating release packages.

```bash
git archive --format=zip HEAD > release.zip
git archive --format=tar.gz v1.0.0 > v1.0.0.tar.gz
```

---

## 🏆 EXPERT & INTERNALS

---

### 71. How does Git store data internally?
**Answer:**
Git stores all data as objects in `.git/objects/`. There are four object types:
- **Blob** — File content
- **Tree** — Directory structure (references blobs and other trees)
- **Commit** — Points to a tree + metadata (author, message, parent commit)
- **Tag** — Points to a commit with extra metadata

All objects are content-addressed: their filename is the SHA-1 hash of their content.

---

### 72. What is the difference between SHA-1 and SHA-256 in Git?
**Answer:**
Git has traditionally used SHA-1 for hashing objects. Due to theoretical collision vulnerabilities in SHA-1, Git 2.29+ introduced experimental support for **SHA-256** (referred to as `sha256` or `hash2`). SHA-256 provides stronger cryptographic guarantees. New repos can opt into SHA-256, but migration for existing repos is complex.

---

### 73. What is `git rerere`?
**Answer:**
`git rerere` (Reuse Recorded Resolution) records how you resolved merge conflicts and automatically reapplies those resolutions if the same conflict appears again. Extremely useful during long rebases.

```bash
git config rerere.enabled true   # Enable globally
```

---

### 74. What is `git notes`?
**Answer:**
`git notes` allows you to add extra information to a commit object without modifying the commit itself. Notes are stored separately and can be pushed/pulled independently. Useful for adding review notes, build results, or annotations.

```bash
git notes add -m "Reviewed by John" <commit>
git notes show <commit>
git push origin refs/notes/*
```

---

### 75. What is the `ORIG_HEAD`, `FETCH_HEAD`, and `MERGE_HEAD`?
**Answer:**
- **`ORIG_HEAD`** — Records the previous state of `HEAD` before a merge, rebase, or reset. Lets you undo with `git reset --hard ORIG_HEAD`.
- **`FETCH_HEAD`** — Records the branch/commit that was last fetched from a remote.
- **`MERGE_HEAD`** — The commit being merged into `HEAD` during an in-progress merge.

---

### 76. What is a bare repository?
**Answer:**
A bare repository has no working directory — it contains only the `.git` folder contents at the root level. It's used as a remote/central repository that people push to and pull from. GitHub/GitLab servers store bare repositories.

```bash
git init --bare repo.git    # Create a bare repo
```

---

### 77. How does `git bisect` use binary search?
**Answer:**
Given a known good commit and a known bad commit, `git bisect` calculates the midpoint commit and checks it out. You test and mark it good or bad. This halves the search space each time. For 1000 commits, it finds the culprit in ~10 steps (log₂(1000) ≈ 10).

---

### 78. What is the difference between `origin/main` and `main`?
**Answer:**
- **`main`** — Your local branch tracking local commits.
- **`origin/main`** — A remote-tracking branch that represents the state of `main` on the remote (`origin`) as of your last `fetch` or `pull`. It's a local snapshot of the remote state.

---

### 79. What is `git sparse-checkout`?
**Answer:**
`git sparse-checkout` lets you check out only a subset of files/directories from a repository, rather than the entire tree. Useful for very large monorepos where you only need part of the codebase.

```bash
git sparse-checkout init --cone
git sparse-checkout set src/frontend
```

---

### 80. What is `git lfs` (Large File Storage)?
**Answer:**
Git LFS is an extension that replaces large files (videos, images, datasets, binaries) in your repository with small pointer files, while storing the actual file content on a remote server. This keeps the Git history lightweight.

```bash
git lfs install
git lfs track "*.psd"        # Track Photoshop files
git lfs ls-files             # List tracked LFS files
```

---

### 81. What is the `--orphan` flag in `git checkout`?
**Answer:**
`git checkout --orphan <branch>` creates a new branch with no commit history — it starts completely fresh with no parents. Commonly used for creating a `gh-pages` branch or separate documentation branches.

```bash
git checkout --orphan gh-pages
git rm -rf .    # Clear all files
```

---

### 82. What are Git hooks and give examples?
**Answer:**
Git hooks are scripts that run automatically before or after Git events. They live in `.git/hooks/`.

| Hook | Trigger |
|---|---|
| `pre-commit` | Before a commit is created |
| `commit-msg` | Validate/edit commit message |
| `post-commit` | After commit completes |
| `pre-push` | Before pushing |
| `post-merge` | After a merge |
| `pre-rebase` | Before a rebase |

Tools like **Husky** make managing hooks in teams easy.

---

### 83. What is the difference between `git pull --ff-only` and `git pull`?
**Answer:**
- `git pull` — Will merge if fast-forward isn't possible, creating a merge commit.
- `git pull --ff-only` — Fails if fast-forward isn't possible instead of creating a merge commit. Forces you to explicitly decide how to integrate diverged history.

```bash
git config --global pull.ff only   # Make ff-only the default
```

---

### 84. How do you find which branch a commit is on?
**Answer:**
```bash
git branch --contains <commit-hash>     # Local branches
git branch -r --contains <commit-hash>  # Remote branches
git branch -a --contains <commit-hash>  # All branches
```

---

### 85. What is `git rev-parse`?
**Answer:**
`git rev-parse` is a low-level command to parse Git references into raw SHA hashes and extract repo metadata. Used heavily in scripts.

```bash
git rev-parse HEAD              # Get current commit hash
git rev-parse --abbrev-ref HEAD # Get current branch name
git rev-parse --show-toplevel   # Get repo root directory
```

---

### 86. How do you count the number of commits in a branch?
**Answer:**
```bash
git rev-list --count HEAD              # Total commits in current branch
git rev-list --count main..feature    # Commits in feature not in main
git log --oneline | wc -l             # Alternative
```

---

### 87. What is the purpose of `git cat-file`?
**Answer:**
`git cat-file` is a low-level plumbing command to inspect Git objects (blobs, trees, commits, tags).

```bash
git cat-file -t <hash>    # Show object type
git cat-file -p <hash>    # Pretty-print object content
git cat-file -s <hash>    # Show object size
```

---

### 88. What is the `--no-ff` flag in git merge?
**Answer:**
`--no-ff` (no fast-forward) forces Git to always create a merge commit even when a fast-forward is possible. This preserves the topological evidence that a feature branch existed and was merged.

```bash
git merge --no-ff feature-branch
```

Many teams enforce `--no-ff` for feature branch merges so the history clearly shows feature groupings.

---

### 89. How do you set up Git aliases?
**Answer:**
Git aliases let you create shortcuts for commands.

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.lg "log --oneline --graph --all"

# Usage
git st        # runs git status
git lg        # runs the log command
```

---

### 90. What is `git grep`?
**Answer:**
`git grep` searches through tracked files in the repository for a string or pattern. Faster than regular `grep` for large repos since it only searches tracked files.

```bash
git grep "TODO"                     # Search all tracked files
git grep -n "function login"        # Show line numbers
git grep "pattern" v1.0..v2.0      # Search between tags
```

---

### 91. What is the difference between `git show` and `git log`?
**Answer:**
- **`git log`** — Shows a list of commits and their metadata (author, date, message).
- **`git show`** — Shows the detailed contents (diff) of a specific commit, tag, or object.

```bash
git show HEAD             # Show last commit with diff
git show v1.0.0           # Show tag details
git show HEAD:file.txt    # Show file contents at HEAD
```

---

### 92. How do you configure multiple remote repositories?
**Answer:**
```bash
git remote add origin https://github.com/user/repo.git
git remote add backup https://gitlab.com/user/repo.git

git push origin main     # Push to GitHub
git push backup main     # Push to GitLab

# Push to all remotes with one command (via alias or all-remote config)
git remote add all https://github.com/user/repo.git
git remote set-url --add all https://gitlab.com/user/repo.git
git push all main
```

---

### 93. What is `git describe`?
**Answer:**
`git describe` gives a human-readable name for a commit based on the nearest tag. Useful for versioning build artifacts.

```bash
git describe                  # e.g., v1.2.0-14-gabcdef7
git describe --tags           # Use lightweight tags too
git describe --abbrev=0       # Show only the tag name
```
Output format: `<tag>-<commits-since-tag>-g<short-hash>`

---

### 94. What is the Git object model?
**Answer:**
Git's object model has four types stored in `.git/objects/`:
1. **Blob** — Raw file content (no filename info)
2. **Tree** — Maps filenames to blobs/other trees (directory)
3. **Commit** — References a tree + parent commit(s) + metadata
4. **Tag** — Named reference to a commit with a message

All objects are immutable and identified by their SHA hash. This makes Git's history tamper-evident.

---

### 95. What is the `COMMIT_EDITMSG` file?
**Answer:**
`.git/COMMIT_EDITMSG` is a temporary file Git uses to store the commit message currently being edited. After a commit is finalized, it retains the last commit message. Tools like `git commit --reuse-message` can reference it.

---

### 96. What is `git format-patch` used for?
**Answer:**
`git format-patch` generates patch files from commits that can be shared via email and applied elsewhere with `git am`. This is the traditional open-source contribution workflow (used by the Linux kernel project).

```bash
git format-patch -3              # Last 3 commits as patch files
git format-patch origin/main     # All commits not in main
git am *.patch                   # Apply patches from files
```

---

### 97. What are the differences between centralized and distributed VCS?
**Answer:**
| Feature | Centralized (SVN) | Distributed (Git) |
|---|---|---|
| Repository | Single central server | Every developer has full copy |
| Offline work | Limited | Full functionality |
| Speed | Slower (server round-trips) | Faster (local operations) |
| Branching | Heavy | Lightweight |
| History | On server only | Full local history |
| Risk | Single point of failure | Redundant by nature |

---

### 98. How do you handle large monorepos in Git?
**Answer:**
Strategies for large monorepos:
- **`git sparse-checkout`** — Check out only needed subdirectories
- **`git lfs`** — Store large binary files externally
- **Shallow clones** — `git clone --depth=1` for faster cloning
- **Partial clone** — `git clone --filter=blob:none` to skip blobs
- **Tools** — Nx, Turborepo, Bazel for build optimization
- **Virtual filesystem** — Microsoft's VFS for Git (GVFS) used for Windows repo

---

### 99. What is the difference between `git switch` and `git checkout`?
**Answer:**
`git switch` (introduced in Git 2.23) is a more focused replacement for the branch-switching part of `git checkout`. `git restore` handles file restoration.

```bash
# Old way
git checkout feature-branch
git checkout -b new-branch
git checkout -- file.txt      # Restore file

# New way
git switch feature-branch
git switch -c new-branch
git restore file.txt          # Restore file
```

`git checkout` still works but `switch`/`restore` are clearer in intent.

---

### 100. What are some Git best practices you follow in a team environment?
**Answer:**
1. **Commit often, push regularly** — Small, focused commits are easier to review and revert.
2. **Write meaningful commit messages** — Use Conventional Commits or similar standards.
3. **Use feature branches** — Never commit directly to `main`/`master`.
4. **Keep branches short-lived** — Long-lived branches drift and cause painful merges.
5. **Review code via PRs/MRs** — Never self-merge important changes.
6. **Protect main branches** — Enforce reviews, CI checks, and no force-push policies.
7. **Use `.gitignore`** — Keep secrets and build artifacts out of repos.
8. **Never commit secrets** — Use environment variables and secret managers.
9. **Rebase before merging** — Keep history clean with `git pull --rebase`.
10. **Tag releases** — Use SemVer tags for every production release.

---

*© Git Interview Q&A — 100 Questions | Covers Basics · Branching · Advanced · GitHub/GitLab · Internals*
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
