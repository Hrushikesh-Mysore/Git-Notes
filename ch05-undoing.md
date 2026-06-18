# Chapter 05 — Undoing Things

> **Time:** ~25 minutes  
> **Prerequisite:** Chapter 04 complete, `todo-app` pushed to GitHub.

---

## Contents

1. [The One Rule](#1-the-one-rule)
2. [Fix the Last Commit — amend](#2-fix-the-last-commit-amend)
3. [Unstage a File](#3-unstage-a-file)
4. [Discard Working Directory Changes](#4-discard-working-directory-changes)
5. [git reset — Move the Branch Pointer](#5-git-reset-move-the-branch-pointer)
6. [git revert — The Safe Undo](#6-git-revert-the-safe-undo)
7. [The Safety Net — git reflog](#7-the-safety-net-git-reflog)
8. [Project — Undo Mistakes in todo-app](#project-undo-mistakes-in-todo-app)

---

## 1. The One Rule

> **Note — Committed = recoverable. Unstaged = at risk.**  
> Anything you have **committed** is almost always recoverable (even after a bad reset). Anything you've only edited in your working directory and **not staged or committed** can be permanently lost if you discard it. Commit early and often — it's your safety net.

---

## 2. Fix the Last Commit — amend

Forgot to include a file? Typo in the message? `--amend` replaces the last commit entirely with a new one.

```bash
# Fix only the commit message (opens editor)
$ git commit --amend

# Fix the message inline
$ git commit --amend -m "Correct commit message"

# Add a forgotten file, keep the same message
$ git add forgotten-file.py
$ git commit --amend --no-edit
```

> **Warning — Don't amend pushed commits**  
> `--amend` creates a brand-new commit with a different hash. If you've already pushed the original and you amend + force-push, it rewrites history for everyone else on that branch. Only amend commits that haven't been pushed yet.

---

## 3. Unstage a File

You ran `git add` but changed your mind — remove it from the staging area without losing your edits.

```bash
# Modern way (Git 2.23+) — recommended
$ git restore --staged todo.py

# Older way — still works
$ git reset HEAD todo.py

# Unstage everything
$ git restore --staged .
```

Your file edits are preserved — only the staging is undone.

---

## 4. Discard Working Directory Changes

Throw away all edits in a file and go back to the last committed version.

```bash
# Discard changes in one file
$ git restore todo.py

# Discard all changes in the working directory
$ git restore .

# Older syntax (still works)
$ git checkout -- todo.py
```

> **Warning — This is permanent**  
> `git restore <file>` on an **unstaged, uncommitted** file is **irreversible**. Git never saved those changes anywhere. There is no undo. Double-check before running this.

---

## 5. git reset — Move the Branch Pointer

`git reset` moves the current branch pointer backward to an earlier commit. Three modes with different effects:

| Mode | History | Staging Area | Working Directory |
|------|---------|--------------|-------------------|
| `--soft` | Moves back | Keeps changes staged | Untouched |
| `--mixed` (default) | Moves back | Unstages changes | Untouched |
| `--hard` | Moves back | Cleared | **Reverted to that commit** |

```bash
# Undo last commit, keep changes staged (soft)
$ git reset --soft HEAD~1

# Undo last commit, unstage changes (mixed — default)
$ git reset HEAD~1

# Undo last commit, throw away changes completely (hard)
$ git reset --hard HEAD~1

# Reset to a specific commit hash
$ git reset --hard a4f3c2d

# HEAD~1 = one commit back
# HEAD~3 = three commits back
```

> **Warning — Hard reset on shared branches**  
> `git reset --hard` on pushed commits requires a force push (`git push --force`) which overwrites remote history. This breaks everyone else's local copy. Only use hard reset on unpushed local work.

---

## 6. git revert — The Safe Undo

`git revert` creates a **new commit** that undoes the changes of a previous commit. History is not rewritten — a new "undo" commit is added on top. This is the right way to undo pushed commits.

```bash
# Revert the most recent commit
$ git revert HEAD

# Revert a specific commit by hash
$ git revert a4f3c2d

# Revert without opening the editor (auto-generate message)
$ git revert --no-edit HEAD

# Revert but don't commit yet (stage the changes only)
$ git revert -n HEAD
```

### reset vs revert — which to use?

| | `git reset` | `git revert` |
|--|------------|--------------|
| Rewrites history | Yes | No |
| Safe on shared branches | No | Yes |
| Creates a new commit | No | Yes |
| Use when | Local, unpushed commits | Pushed / shared commits |

---

## 7. The Safety Net — git reflog

Git keeps a **reference log** of every position HEAD has been at — even when you delete branches or hard reset. This is your last resort for recovering "lost" commits.

```bash
$ git reflog
f5a9b12 HEAD@{0}: reset: moving to HEAD~2
d1c2e3f HEAD@{1}: commit: Add VERSION file
a4f3c2d HEAD@{2}: commit: Add todo.py
7e3a1d9 HEAD@{3}: commit (initial): Initial commit

# Recover a commit you reset away by resetting to its hash
$ git reset --hard d1c2e3f

# Or create a new branch at the "lost" commit
$ git branch rescue-branch d1c2e3f
```

> **Note — Reflog is local only**  
> The reflog lives only on your machine and expires after 90 days. It won't help you recover from someone else's force push to the remote, or recover changes you never committed.

---

## Project — Undo Mistakes in todo-app

### Step 1 — Make a typo in a commit message, then fix it

```bash
$ echo "# changelog" > CHANGELOG.md
$ git add CHANGELOG.md
$ git commit -m "addd changelog fiel"

# Fix the typo with amend
$ git commit --amend -m "Add CHANGELOG.md"
$ git log --oneline -3
```

### Step 2 — Stage a file accidentally, then unstage it

```bash
$ echo "debug stuff" > debug.log
$ git add debug.log
$ git status
# debug.log is staged

$ git restore --staged debug.log
$ git status
# debug.log is now untracked again

$ rm debug.log   # clean up
```

### Step 3 — Edit a file, then discard the change

```bash
$ echo "BROKEN CODE ####" >> todo.py
$ git diff     # see the bad change

$ git restore todo.py
$ cat todo.py  # the bad line is gone
```

### Step 4 — Soft reset — undo a commit but keep the work

```bash
$ echo "# work in progress" >> README.md
$ git commit -am "WIP: notes"

# Changed your mind — undo the commit but keep the edit staged
$ git reset --soft HEAD~1
$ git status
# Changes still staged, commit is gone

# Recommit properly
$ git commit -m "Add notes section to README"
```

### Step 5 — Safely undo a pushed commit with revert

```bash
# First push your current state
$ git push

# Now revert the last commit (safe for shared branches)
$ git revert --no-edit HEAD
[main abc1234] Revert "Add notes section to README"

$ git push   # push the revert commit
$ git log --oneline -4
# You'll see: original commit, then "Revert..." commit on top
```

---

## Summary

| Command | What it does |
|---------|-------------|
| `git commit --amend` | Fix last commit message or content |
| `git restore --staged <file>` | Unstage a file |
| `git restore <file>` | Discard working dir changes (permanent!) |
| `git reset --soft HEAD~1` | Undo commit, keep changes staged |
| `git reset HEAD~1` | Undo commit, unstage changes |
| `git reset --hard HEAD~1` | Undo commit, discard all changes |
| `git revert HEAD` | Safe undo — creates a new commit |
| `git reflog` | History of all HEAD movements |


---
**Previous:** [Chapter 04 — Remotes & GitHub](ch04-remotes-github.md)  
**Next:** [Chapter 06 — Stash & Clean](ch06-stash-clean.md)

---

**Reference:** [Cheat Sheet](CHEATSHEET.md) · [Glossary](GLOSSARY.md) · [Troubleshooting](TROUBLESHOOTING.md) · [Git Config](gitconfig.md) · [Home](README.md)
