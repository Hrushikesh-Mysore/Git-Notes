# Troubleshooting

The 15 errors people Google most when learning Git — with exact fix commands.

---

## Table of Contents

1. [fatal: refusing to merge unrelated histories](#1-fatal-refusing-to-merge-unrelated-histories)
2. [rejected — non-fast-forward](#2-rejected--non-fast-forward)
3. [You are in detached HEAD state](#3-you-are-in-detached-head-state)
4. [Could not read Username for HTTPS](#4-could-not-read-username-for-https)
5. [Permission denied (publickey) — SSH](#5-permission-denied-publickey--ssh)
6. [Merge conflict — stuck mid-merge](#6-merge-conflict--stuck-mid-merge)
7. [Changes not staged — nothing to commit](#7-changes-not-staged--nothing-to-commit)
8. [fatal: not a git repository](#8-fatal-not-a-git-repository)
9. [My commit is on the wrong branch](#9-my-commit-is-on-the-wrong-branch)
10. [I accidentally committed to main](#10-i-accidentally-committed-to-main)
11. [I need to undo a pushed commit](#11-i-need-to-undo-a-pushed-commit)
12. [fatal: pathspec did not match any files](#12-fatal-pathspec-did-not-match-any-files)
13. [HEAD~1 went too far — I lost commits](#13-head1-went-too-far--i-lost-commits)
14. [My branch is behind origin — push rejected](#14-my-branch-is-behind-origin--push-rejected)
15. [gitignore is not working](#15-gitignore-is-not-working)

---

## 1. fatal: refusing to merge unrelated histories

**Full error:**
```
fatal: refusing to merge unrelated histories
```

**Why it happens:** You initialised a repo locally (`git init`) and also created one on GitHub with a README. Now you're trying to pull — but Git sees them as two completely separate histories with no common ancestor.

**Fix:**
```bash
$ git pull origin main --allow-unrelated-histories
# Resolve any conflicts, then commit
```

**Better: avoid it next time** — when creating a repo on GitHub, don't tick "Add a README". Let your local repo be the single source of truth from the start.

---

## 2. rejected — non-fast-forward

**Full error:**
```
! [rejected]        main -> main (non-fast-forward)
error: failed to push some refs to 'origin'
hint: Updates were rejected because the remote contains work that you do not have locally.
```

**Why it happens:** Someone else pushed to the same branch since your last pull. Your history has diverged.

**Fix:**
```bash
# Pull remote changes first (rebasing keeps history clean)
$ git pull --rebase origin main

# If there are conflicts, resolve them, then:
$ git rebase --continue

# Then push
$ git push
```

> **Never do** `git push --force` on a shared branch. It overwrites other people's commits.

---

## 3. You are in detached HEAD state

**Full message:**
```
You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.
```

**Why it happens:** You checked out a specific commit hash, a tag, or used `git bisect`. HEAD now points directly to a commit instead of a branch.

**If you just want to look around — no action needed.** Switch back when done:
```bash
$ git switch main    # or any branch name
```

**If you already made commits here and want to keep them:**
```bash
# Create a branch at your current position before switching away
$ git switch -c my-new-branch
# Your commits are now safely on my-new-branch
```

---

## 4. Could not read Username for HTTPS

**Full error:**
```
fatal: could not read Username for 'https://github.com': terminal prompts disabled
```
or a password prompt that fails with your GitHub password.

**Why it happens:** GitHub removed password authentication for HTTPS in 2021. You need a Personal Access Token (PAT) or SSH.

**Fix A — Use SSH (recommended):**
```bash
$ ssh-keygen -t ed25519 -C "you@example.com"
$ cat ~/.ssh/id_ed25519.pub
# Paste the output into: GitHub → Settings → SSH Keys → New SSH key

# Change your repo's remote to SSH
$ git remote set-url origin git@github.com:USERNAME/REPO.git

# Test
$ ssh -T git@github.com
```

**Fix B — Use a Personal Access Token:**
```
GitHub → Settings → Developer settings → Personal access tokens → Generate new token
```
Give it `repo` scope. Use it as your password when Git prompts.

**Fix C — Cache credentials so you're not prompted every time:**
```bash
$ git config --global credential.helper store       # saves to disk (less secure)
$ git config --global credential.helper cache       # saves in memory for 15 min
```

---

## 5. Permission denied (publickey) — SSH

**Full error:**
```
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.
```

**Why it happens:** Your SSH key is not added to GitHub, or the SSH agent doesn't have it loaded.

**Fix:**
```bash
# Check if your key is loaded in the SSH agent
$ ssh-add -l

# If "The agent has no identities", add your key
$ ssh-add ~/.ssh/id_ed25519

# Check that your public key is on GitHub
$ cat ~/.ssh/id_ed25519.pub
# Compare with: GitHub → Settings → SSH Keys

# If the key isn't there, add it, then test
$ ssh -T git@github.com
Hi username! You've successfully authenticated.

# If you have multiple SSH keys, check ~/.ssh/config
$ cat ~/.ssh/config
# Should have a Host github.com entry pointing to the right key
```

---

## 6. Merge conflict — stuck mid-merge

**Symptoms:** `git status` shows "You have unmerged paths" and conflict markers (`<<<<<<<`) are in your files. You're not sure how to get out.

**Option A — Resolve and continue:**
```bash
# 1. See which files have conflicts
$ git status

# 2. Open each conflicted file, find and fix the markers:
#    <<<<<<< HEAD       <- your version
#    =======
#    >>>>>>> branch     <- incoming version
#    (Edit to the final correct state, delete all marker lines)

# 3. Stage each resolved file
$ git add resolved-file.py

# 4. Once all conflicts are resolved
$ git commit
```

**Option B — Abort and go back to before the merge:**
```bash
$ git merge --abort
# You're back to exactly where you were before starting the merge
```

---

## 7. Changes not staged — nothing to commit

**Symptoms:** You edited files but `git commit` says "nothing to commit" or `git status` shows files under "Changes not staged for commit".

**Why it happens:** You forgot to `git add` before committing.

**Fix:**
```bash
# Stage your changes first
$ git add .

# Then commit
$ git commit -m "Your message"

# Or stage and commit tracked files in one step
$ git commit -am "Your message"
# Note: -a does NOT add brand-new untracked files
```

---

## 8. fatal: not a git repository

**Full error:**
```
fatal: not a git repository (or any of the parent directories): .git
```

**Why it happens:** You're running a `git` command outside a Git repository, or in a subdirectory above where the repo was initialised.

**Fix:**
```bash
# Check where you are
$ pwd

# Navigate to your project
$ cd /path/to/your/project

# Confirm the .git folder is here
$ ls -a | grep .git

# If there's genuinely no repo, initialise one
$ git init
```

---

## 9. My commit is on the wrong branch

You committed to `main` but it should have gone on a feature branch.

**Fix — move the commit to the right branch:**
```bash
# 1. Create the intended branch at the current position (before moving anything)
$ git branch feature/my-work

# 2. Remove the commit from main (soft reset keeps your changes staged)
$ git reset --soft HEAD~1

# 3. Switch to the feature branch (your changes are still staged)
$ git switch feature/my-work

# 4. Commit there
$ git commit -m "Your message"
```

---

## 10. I accidentally committed to main

You pushed a commit directly to `main` that should have been on a feature branch.

**If NOT pushed yet:**
```bash
# Move the commit to a new branch and reset main
$ git branch feature/oops-work    # save the commit on a new branch
$ git reset --hard HEAD~1          # remove it from main
$ git switch feature/oops-work     # continue working there
```

**If already pushed:**
```bash
# Safe option: revert (creates a new undo commit, doesn't rewrite history)
$ git revert HEAD
$ git push

# Then cherry-pick onto the right branch
$ git switch -c feature/real-branch
$ git cherry-pick <hash-of-original-commit>
```

---

## 11. I need to undo a pushed commit

**Safe way — revert (does not rewrite history, safe for shared branches):**
```bash
$ git revert HEAD             # undo the most recent commit
$ git revert HEAD~2           # undo a commit 2 back
$ git push
```

**Dangerous way — only if you are the only person using this branch:**
```bash
$ git reset --hard HEAD~1
$ git push --force-with-lease   # safer than --force, checks remote hasn't changed
```

> **Warning:** Force pushing rewrites remote history. Anyone who has already pulled will have diverged history. Only do this on branches you own exclusively.

---

## 12. fatal: pathspec did not match any files

**Full error:**
```
fatal: pathspec 'filename.txt' did not match any files
```

**Why it happens:** The file doesn't exist, is in a different directory, or the filename/path is wrong.

**Fix:**
```bash
# Check the exact filename (case-sensitive on Linux/macOS)
$ ls
$ git status

# If trying to restore a deleted file from the last commit
$ git restore HEAD -- filename.txt

# If trying to checkout a file from another branch
$ git restore --source=main -- path/to/file.txt

# For git log -- file, make sure you're in the right directory
$ git log --oneline -- src/todo.py
```

---

## 13. HEAD~1 went too far — I lost commits

You ran `git reset --hard HEAD~3` and now commits seem to be gone.

**Fix — use reflog to recover them:**
```bash
# See the history of where HEAD has been
$ git reflog
abc1234 HEAD@{0}: reset: moving to HEAD~3
def5678 HEAD@{1}: commit: The commit you want back
...

# Reset to the commit you want to recover
$ git reset --hard def5678

# Or, safer — create a new branch there first
$ git branch recovered-work def5678
```

> **Note:** Reflog only exists locally and entries expire after 90 days.

---

## 14. My branch is behind origin — push rejected

**Full error:**
```
error: failed to push some refs
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart.
```

**Fix:**
```bash
# Pull remote changes and rebase your local commits on top
$ git pull --rebase

# If there are conflicts during rebase:
$ git rebase --continue    # after resolving each conflict
$ git rebase --abort       # to cancel and go back

# Then push
$ git push
```

---

## 15. .gitignore is not working

You added a file to `.gitignore` but Git still tracks it.

**Why it happens:** Git only ignores files it has never tracked. If a file was already committed, adding it to `.gitignore` does nothing — Git keeps tracking it.

**Fix — untrack the file without deleting it:**
```bash
# Stop tracking a single file (removes from Git, keeps on disk)
$ git rm --cached filename.txt
$ git commit -m "Stop tracking filename.txt"

# Stop tracking an entire folder
$ git rm --cached -r foldername/
$ git commit -m "Stop tracking foldername/"
```

Now that Git has "forgotten" the file, `.gitignore` will prevent it from being tracked again.

**If `.gitignore` itself is being ignored** — make sure it's saved in the root of the repository and there's no trailing whitespace or Windows line endings in the filename patterns.

```bash
# Check for encoding issues
$ cat -A .gitignore | head -5
# Lines should end with $ (Unix) not ^M$ (Windows)

# Fix Windows line endings if needed
$ sed -i 's/\r//' .gitignore
```

---

## Quick Bail-out Reference

| Situation | Command |
|-----------|---------|
| Cancel a merge | `git merge --abort` |
| Cancel a rebase | `git rebase --abort` |
| Cancel a cherry-pick | `git cherry-pick --abort` |
| Cancel a revert | `git revert --abort` |
| Discard all uncommitted changes | `git restore . && git clean -fd` |
| Get back to a clean state | `git reset --hard HEAD` |
| Recover "lost" commits | `git reflog` then `git reset --hard <hash>` |

---

*Back to [README](README.md)*
