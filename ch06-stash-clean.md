# Chapter 06 — Stash & Clean

> **Time:** ~20 minutes  
> **Prerequisite:** Chapter 05 complete.

---

## Contents

1. [Why Stash?](#1-why-stash)
2. [Stash Basics](#2-stash-basics)
3. [Multiple Stashes & Advanced Options](#3-multiple-stashes-advanced-options)
4. [git clean — Remove Untracked Files](#4-git-clean-remove-untracked-files)
5. [Project — Mid-feature Context Switch](#project-mid-feature-context-switch)

---

## 1. Why Stash?

**The scenario:** You're halfway through a new feature. Your manager asks you to drop everything and fix an urgent bug on `main`. Your work isn't ready to commit. You can't switch branches cleanly because Git will complain about uncommitted changes (or carry them over into the wrong branch).

**Solution: stash it.** Git stash saves your working directory and staging area to a temporary stack, gives you a clean slate, and lets you restore everything exactly as it was — later.

---

## 2. Stash Basics

```bash
# Save current WIP (tracked modified files + staged files)
$ git stash
Saved working directory and index state WIP on main: d1c2e3f Add VERSION file

# Save with a descriptive message (recommended)
$ git stash push -m "half-done due-date feature"

# List all stashes (newest first)
$ git stash list
stash@{0}: On main: half-done due-date feature
stash@{1}: WIP on feature/x: a4b5c6d Add tests

# Apply the most recent stash (keeps it in the stash list)
$ git stash apply

# Apply AND remove from the stash list (most common)
$ git stash pop

# Apply a specific stash by index
$ git stash apply stash@{1}

# Drop (delete) a specific stash
$ git stash drop stash@{0}

# Delete ALL stashes
$ git stash clear

# See what's in a stash without applying it
$ git stash show -p stash@{0}
```

> **Note — Stash doesn't save untracked files by default**  
> New files that have never been `git add`-ed are NOT stashed unless you ask explicitly:
> ```bash
> $ git stash -u        # include untracked files
> $ git stash -a        # include untracked + .gitignore'd files
> ```

---

## 3. Multiple Stashes & Advanced Options

```bash
# Include new (untracked) files in the stash
$ git stash -u
$ git stash --include-untracked   # same thing

# Stash only the staging area, keep working dir changes visible
$ git stash --keep-index

# Interactively pick which changed hunks to stash (patch mode)
$ git stash -p

# Create a new branch from a stash and apply it there
# Useful when applying the stash would conflict with current branch
$ git stash branch new-feature stash@{0}
# This: creates the branch, checks it out, applies the stash, drops it
```

> **Tip — Stash conflicts on pop**  
> If `git stash pop` causes a conflict, Git leaves the conflict markers in the file just like a merge conflict. Resolve them, `git add` the file. Note that the stash is **not** automatically dropped after a conflict — do `git stash drop` manually when you're done.

---

## 4. git clean — Remove Untracked Files

`git clean` permanently deletes files from the working directory that Git is not tracking — build output, temp files, generated artifacts, etc.

> **Warning — git clean is irreversible**  
> Unlike almost every other Git operation, `git clean` **permanently deletes files**. They are not staged or committed anywhere. There is no recovery. **Always do a dry run first.**

```bash
# DRY RUN — see what WOULD be deleted (no actual deletion)
$ git clean -n
Would remove build/output.js
Would remove temp.log

# Delete untracked files (but not directories)
$ git clean -f

# Delete untracked files AND directories
$ git clean -fd

# Delete untracked files, directories, AND .gitignore'd files
# (use this to get a completely pristine working directory)
$ git clean -fdx

# Interactive mode — choose file by file what to delete
$ git clean -i
```

| Flag | Meaning |
|------|---------|
| `-n` | Dry run — show what would be removed |
| `-f` | Force — required to actually delete |
| `-d` | Include untracked directories |
| `-x` | Also remove .gitignore'd files |
| `-i` | Interactive mode |

---

## Project — Mid-feature Context Switch

**Goal:** Simulate being interrupted mid-feature, stash, fix a bug on `main`, then resume.

### Step 1 — Start a new feature (don't finish it)

```bash
$ cd todo-app
$ git switch -c feature/due-dates

# Start writing code — not done yet
$ cat >> todo.py << 'EOF'

def set_due(task, date):
    """Assign a due date to a task. date format: YYYY-MM-DD"""
    pass   # TODO: implement
EOF

# Also create a new file (untracked)
$ echo '{"dues": {}}' > due_dates.json
```

### Step 2 — Stash everything (including the new file)

```bash
$ git stash -u -m "WIP: due-date feature, set_due not implemented yet"
Saved working directory and index state On feature/due-dates: WIP: due-date ...

$ git status
nothing to commit, working tree clean
```

### Step 3 — Fix the urgent bug on main

```bash
$ git switch main

# The bug: remove() crashes if the task doesn't exist
# It's already been fixed in our todo.py, but let's add a test note
$ echo "# Bug fix: remove() now safe on missing tasks" >> CHANGELOG.md
$ git add CHANGELOG.md
$ git commit -m "Document remove() safety fix in CHANGELOG"
$ git push
```

### Step 4 — Return to the feature and restore stash

```bash
$ git switch feature/due-dates

$ git stash list
stash@{0}: On feature/due-dates: WIP: due-date feature, set_due not implemented yet

$ git stash pop
On branch feature/due-dates
Changes not staged for commit:
        modified:   todo.py
Untracked files:
        due_dates.json
Dropped stash@{0}
```

Everything is back exactly as you left it. Continue working.

### Step 5 — Finish and commit the feature

```bash
# Replace the TODO with real implementation
$ cat >> todo.py << 'EOF'

due_dates = {}

def set_due(task, date):
    """Assign a due date. date format: YYYY-MM-DD"""
    due_dates[task] = date
    print(f"Due date for '{task}' set to {date}")
EOF

$ rm due_dates.json   # no longer needed as a file
$ git add todo.py
$ git commit -m "Implement set_due() with in-memory due_dates dict"
```

### Step 6 — Merge into main

```bash
$ git switch main
$ git merge feature/due-dates
$ git push
$ git branch -d feature/due-dates
```

---

## Summary

| Command | What it does |
|---------|-------------|
| `git stash` | Save WIP to stash stack |
| `git stash push -m "msg"` | Stash with a description |
| `git stash -u` | Stash including untracked files |
| `git stash list` | Show all stashes |
| `git stash pop` | Apply latest stash + remove it |
| `git stash apply stash@{n}` | Apply a specific stash |
| `git stash drop stash@{n}` | Delete a stash |
| `git stash branch <name>` | New branch from stash |
| `git clean -n` | Dry run — see what would be deleted |
| `git clean -fd` | Delete untracked files + dirs |
| `git clean -fdx` | Delete everything Git doesn't track |


---
**Previous:** [Chapter 05 — Undoing Things](ch05-undoing.md)  
**Next:** [Chapter 07 — Rewriting History](ch07-rewriting-history.md)

---

**Reference:** [Cheat Sheet](CHEATSHEET.md) · [Glossary](GLOSSARY.md) · [Troubleshooting](TROUBLESHOOTING.md) · [Git Config](gitconfig.md) · [Home](README.md)
