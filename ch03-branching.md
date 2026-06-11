# Chapter 03 — Branching

> **Time:** ~30 minutes  
> **Prerequisite:** Chapter 02 complete, `todo-app` repo with 3 commits.

---

## Contents

1. [What is a Branch?](#1-what-is-a-branch)
2. [Creating & Switching Branches](#2-creating-switching-branches)
3. [Merging](#3-merging)
4. [Merge Conflicts](#4-merge-conflicts)
5. [Rebase vs Merge](#5-rebase-vs-merge)
6. [Project — Feature Branch on todo-app](#project-feature-branch-on-todo-app)

---

## 1. What is a Branch?

A branch is a **lightweight pointer to a commit**. When you commit, the pointer moves forward automatically. Creating a branch costs almost nothing — it's just a 41-byte file containing a commit hash.

`HEAD` is a special pointer that tracks which branch you're currently on. Switch branches → HEAD moves.

```
main:    A -- B -- C          <- HEAD points here
                    \
feature:             D -- E   <- a new branch
```

> **Note — Branches are free**  
> In Git, creating a branch is instant no matter how large your repo is. There's no file copying. Use branches liberally — one per feature, one per bug fix.

---

## 2. Creating & Switching Branches

```bash
# List local branches (* = current branch)
$ git branch
* main
  feature-login

# List all branches including remote-tracking
$ git branch -a

# Create a new branch (doesn't switch to it)
$ git branch feature-auth

# Switch to a branch (modern syntax — Git 2.23+)
$ git switch feature-auth

# Switch — older syntax, still works
$ git checkout feature-auth

# CREATE and switch in one command (most common)
$ git switch -c new-feature
$ git checkout -b new-feature    # older equivalent

# Delete a branch (safe — refuses if unmerged)
$ git branch -d old-feature

# Force delete (even if unmerged)
$ git branch -D old-feature

# Rename a branch
$ git branch -m old-name new-name
```

> **Tip — Prefer `git switch` over `git checkout`**  
> Git 2.23 split `git checkout` into `git switch` (change branches) and `git restore` (discard file changes). `checkout` still works but these newer commands are less ambiguous.

---

## 3. Merging

Merging brings work from one branch into another. Typical flow: work on a feature branch, then merge it into `main`.

```bash
# 1. Switch to the branch you want to merge INTO
$ git switch main

# 2. Merge the feature branch
$ git merge feature-auth
Updating a4f3c2d..9b1e4f2
Fast-forward
 auth.py | 30 ++++++++++++++++++++++++++++++
```

### Fast-forward vs Three-way merge

**Fast-forward** — happens when `main` hasn't moved since you branched off. Git just moves the `main` pointer forward. No new commit, linear history.

**Three-way merge** — happens when both branches have new commits since they diverged. Git creates a **merge commit** with two parents. The history shows that a merge happened.

```bash
# Force a merge commit even when fast-forward is possible
$ git merge --no-ff feature-auth

# Abort a merge in progress
$ git merge --abort

# Preview what would change without actually merging
$ git diff main..feature-auth
```

---

## 4. Merge Conflicts

A conflict occurs when the same lines in the same file were changed differently on both branches. Git can't decide which version to keep — it marks the file and asks you to resolve it.

```bash
$ git merge feature-auth
CONFLICT (content): Merge conflict in config.py
Automatic merge failed; fix conflicts and then commit the result.
```

Inside the conflicted file:

```python
<<<<<<< HEAD
DEBUG = False
=======
DEBUG = True
>>>>>>> feature-auth
```

- Everything between `<<<<<<< HEAD` and `=======` is your current branch's version
- Everything between `=======` and `>>>>>>>` is the incoming branch's version

**To resolve:**

1. Open the file and edit it to the correct final state (delete all the `<<<<`, `====`, `>>>>` markers too)
2. Stage the resolved file
3. Commit

```bash
# After manually editing the file
$ git add config.py
$ git commit -m "Merge feature-auth into main"

# Abort if you want to give up and go back
$ git merge --abort
```

> **Warning — Always test after resolving conflicts**  
> Picking one side of a conflict may break logic from the other side. Run your tests after every conflict resolution before committing.

> **Tip — Visual merge tools**  
> `git mergetool` opens a 3-way diff GUI. Popular tools: VS Code (built-in), IntelliJ, vimdiff. Configure with:  
> `git config --global merge.tool vscode`

---

## 5. Rebase vs Merge

Both integrate changes from one branch into another. The difference is in the history they produce.

| | Merge | Rebase |
|--|-------|--------|
| History | Shows what actually happened (merge commits) | Linear, cleaner-looking |
| Creates new commits? | Yes (merge commit) | Yes (replayed commits with new hashes) |
| Safe on shared branches? | Yes | **No** |
| Good for | Final integration into main | Cleaning up local work before sharing |

```bash
# Replay your feature branch on top of the latest main
$ git switch feature-auth
$ git rebase main

# If conflicts during rebase:
$ git rebase --continue     # after resolving
$ git rebase --skip         # skip this commit
$ git rebase --abort        # bail out completely

# Then merge (will be a clean fast-forward)
$ git switch main
$ git merge feature-auth
```

> **Warning — The Golden Rule of Rebasing**  
> **Never rebase commits that have been pushed to a shared remote branch.**  
> Rebase rewrites commit hashes. If teammates have based work on those commits, their history will diverge badly. Only rebase your local, unpublished commits — or branches only you own.

---

## Project — Feature Branch on todo-app

**Goal:** Add a priority system using a feature branch, then merge it into `main`.

### Step 1 — Create a feature branch

```bash
$ cd todo-app
$ git switch -c feature/priority
Switched to a new branch 'feature/priority'
```

### Step 2 — Add priority support

```bash
$ cat >> todo.py << 'EOF'

def add_priority(task, level="low"):
    """Add a todo with a priority level: low, medium, high"""
    if level not in ("low", "medium", "high"):
        print("Priority must be low, medium, or high")
        return
    todos.append({"task": task, "priority": level})
    print(f"Added [{level}]: {task}")
EOF

$ git add todo.py
$ git commit -m "Add priority levels to todos"
[feature/priority a4b5c6d] Add priority levels to todos
```

### Step 3 — Switch back to main and make a different change

```bash
$ git switch main

# Simulate ongoing work on main
$ echo "" >> README.md
$ echo "## Usage" >> README.md
$ echo "Run \`python todo.py\` to manage your todos." >> README.md
$ git commit -am "Update README with usage section"
```

### Step 4 — View the diverged branches

```bash
$ git log --oneline --graph --all
* b9c8d7e (main) Update README with usage section
| * a4b5c6d (feature/priority) Add priority levels to todos
|/
* c3a9b12 Add count() function to todo.py
```

Both branches have new commits — this will be a three-way merge.

### Step 5 — Merge the feature branch

```bash
$ git merge feature/priority
Merge made by the 'ort' strategy.
 todo.py | 9 +++++++++

$ git log --oneline --graph
*   d1c2e3f (HEAD -> main) Merge branch 'feature/priority'
|\
| * a4b5c6d Add priority levels to todos
* | b9c8d7e Update README with usage section
|/
* c3a9b12 Add count() function to todo.py
```

### Step 6 — Clean up

```bash
$ git branch -d feature/priority
Deleted branch feature/priority (was a4b5c6d).
```

---

## Summary

| Command | What it does |
|---------|-------------|
| `git branch` | List branches |
| `git branch <name>` | Create a branch |
| `git switch <name>` | Switch to a branch |
| `git switch -c <name>` | Create + switch |
| `git branch -d <name>` | Delete (safe) |
| `git branch -D <name>` | Delete (force) |
| `git merge <branch>` | Merge branch into current |
| `git merge --no-ff` | Force a merge commit |
| `git merge --abort` | Cancel a merge |
| `git rebase <branch>` | Replay commits on top of branch |
| `git rebase --abort` | Cancel a rebase |


---
**Previous:** [Chapter 02 — Git Basics](ch02-git-basics.md)  
**Next:** [Chapter 04 — Remotes & GitHub](ch04-remotes-github.md)

---

**Reference:** [Cheat Sheet](CHEATSHEET.md) · [Glossary](GLOSSARY.md) · [Troubleshooting](TROUBLESHOOTING.md) · [Git Config](gitconfig.md) · [Home](README.md)
