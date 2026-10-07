# Git Cheat Sheet

Notes from the *Git and GitHub for Beginners - Crash Course* video, organized by what you are trying to do.

> **Quick mental model**
> Working directory -> (`git add`) -> Staging area -> (`git commit`) -> Local repository -> (`git push`) -> GitHub (remote)

---

## 1. Setup and starting a repository

| Command | What it does |
|---|---|
| `git config --global user.name "Your Name"` | Sets the name attached to your commits (once per machine) |
| `git config --global user.email "you@example.com"` | Sets the email attached to your commits |
| `git init` | Initializes a new repository in the current folder |
| `git clone <url>` | Copies a remote repository to your machine |

---

## 2. Checking the state

| Command | What it does |
|---|---|
| `git status` | Shows changed, staged and untracked files (use it all the time) |
| `git log` | Shows the commit history |
| `git log --oneline` | Compact history, one line per commit |
| `git diff` | Shows unstaged changes (working directory vs staging area) |
| `git diff --staged` | Shows staged changes (what the next commit will contain) |
| `git diff <commit1> <commit2>` | Compares two commits (branches work too) |
| `git show <commit>` | Shows the details of one commit |

---

## 3. Staging changes (`git add`)

| Command | What it stages |
|---|---|
| `git add <file>` | One specific file |
| `git add .` | All changes (new, modified, deleted) in the current directory and below |
| `git add -A` or `git add --all` | All changes in the entire repository |
| `git add *` | Whatever the shell expands `*` to in the current directory: skips hidden files and deleted files. Prefer `.` or `-A` |

### Unstaging

| Command | What it does |
|---|---|
| `git reset <file>` | Unstages a file (the changes stay in your working directory) |
| `git restore --staged <file>` | Same, the modern command |

---

## 4. Saving changes (`git commit`)

| Command | What it does |
|---|---|
| `git commit -m "title"` | Commits with a title |
| `git commit -m "title" -m "description"` | The second `-m` adds a longer description |
| `git commit -am "title"` | Stages **modified and deleted tracked files** and commits. It does **not** include new (untracked) files |
| `git commit --amend` | Edits the last commit (message or content). Do not use it after pushing |

**Good commit messages:** short title in the imperative ("Add login route"), details in the description.

---

## 5. Removing files

| Command | What it does |
|---|---|
| `git rm <file>` | Deletes the file and stages the deletion |
| `git rm -f <file>` | Forces the removal (even with unsaved changes) |
| `git rm --cached <file>` | Stops tracking the file but keeps it on your disk |

Use `.gitignore` to keep files like `.env` and `__pycache__/` out of Git completely.

---

## 6. Undoing things

| Goal | Command | Notes |
|---|---|---|
| Discard local changes in a file | `git restore <file>` | Cannot be undone |
| Undo the last commit, keep the changes **staged** | `git reset --soft HEAD~1` | Safe |
| Undo the last commit, keep the changes **unstaged** | `git reset HEAD~1` (same as `--mixed`) | Safe |
| Undo the last commit **and delete the changes** | `git reset --hard HEAD~1` | Dangerous |
| Undo a commit safely by creating a new commit | `git revert <commit>` | Best for commits already pushed |
| Find lost commits | `git reflog` | A history of where HEAD has been |

**Rule:** `reset` rewrites history (use it on local commits only). `revert` adds history (safe for shared branches).

---

## 7. Branches

| Command | What it does |
|---|---|
| `git branch` | Lists branches |
| `git branch <name>` | Creates a branch |
| `git branch -d <name>` | Deletes a merged branch |
| `git checkout <branch>` | Switches to a branch |
| `git checkout -b <branch>` | Creates a branch and switches to it |
| `git switch <branch>` | Modern command for switching |
| `git switch -c <branch>` | Modern command for create + switch |

### Merging

```bash
git checkout main
git merge feature-login
```

If Git reports a **merge conflict**:
1. Open the conflicted files. Git marks the conflict with `<<<<<<<`, `=======`, `>>>>>>>`.
2. Edit the file to keep the correct code and delete the markers.
3. Run `git add <file>`, then `git commit`.
4. To cancel the merge: `git merge --abort`.

### Rebase

`git rebase <branch>` re-applies your commits on top of another branch, which gives a straight, clean history.
`git rebase -i HEAD~3` lets you squash, reorder or edit your last 3 commits.

**Golden rule:** never rebase commits that you already pushed and others use.

---

## 8. Working with GitHub (remotes)

| Command | What it does |
|---|---|
| `git remote add origin <url>` | Connects your local repo to a GitHub repo |
| `git remote -v` | Shows the connected remotes |
| `git push -u origin main` | First push (sets the upstream branch) |
| `git push` | Uploads your commits to GitHub |
| `git fetch` | Downloads new commits from GitHub **without** merging |
| `git pull` | `fetch` + `merge`: downloads and merges |

### Pull requests (PRs)

A pull request is a **GitHub feature**, not a Git command. You push a branch, then ask on GitHub for it to be reviewed and merged into another branch (usually `main`).

Typical flow: create a branch -> commit -> push the branch -> open a PR -> review -> merge.

---

## 9. Stash (save work temporarily)

Use it when you must switch branches but are not ready to commit.

| Command | What it does |
|---|---|
| `git stash` | Saves uncommitted changes and cleans the working directory |
| `git stash list` | Lists saved stashes |
| `git stash pop` | Restores the latest stash and removes it from the list |
| `git stash apply` | Restores the latest stash and keeps it in the list |
| `git stash drop` | Deletes a stash |

By default `git stash` does not save untracked files. Add `-u` to include them.

---

## 10. Daily workflow

```bash
git status                       # what changed?
git add .                        # stage
git commit -m "Describe change"  # save
git push                         # upload
```

Feature workflow:

```bash
git checkout -b feature/my-feature
# work, add, commit
git push -u origin feature/my-feature
# open a Pull Request on GitHub
```
