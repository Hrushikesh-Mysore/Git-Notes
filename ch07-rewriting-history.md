# Chapter 07 — Rewriting History

> **Time:** ~30 minutes  
> **Prerequisite:** Chapter 06 complete. Comfortable with commits and branches.

---

## Contents

1. [Why Rewrite History?](#1-why-rewrite-history)
2. [Interactive Rebase — The Core Tool](#2-interactive-rebase-the-core-tool)
3. [Squashing Commits](#3-squashing-commits)
4. [Reordering & Dropping Commits](#4-reordering-dropping-commits)
5. [Editing a Commit Mid-history](#5-editing-a-commit-mid-history)
6. [Splitting a Commit](#6-splitting-a-commit)
7. [Cherry-pick — Grab a Single Commit](#7-cherry-pick-grab-a-single-commit)
8. [Project — Clean Up todo-app History](#project-clean-up-todo-app-history)

---

## 1. Why Rewrite History?

When you're working alone on a feature branch, your commit history is often messy — lots of "fix", "wip", "typo" commits. Before merging into `main`, you clean it up so the history tells a clear story.

> **Warning — Only rewrite local, unpushed commits**  
> All commands in this chapter change commit hashes. If you rewrite commits teammates have based their work on, you'll cause serious divergence. Rule: only rewrite commits that exist **only on your local machine** or on **branches only you own**.

---

## 2. Interactive Rebase — The Core Tool

Interactive rebase (`rebase -i`) opens a to-do list of commits. You edit the list to say what to do with each one, then Git executes your instructions.

```bash
# Edit the last 4 commits
$ git rebase -i HEAD~4
```

Git opens your editor:

```
pick a1b2c3d Add todo.py with add/remove/list
pick d4e5f6g Add count() function
pick h7i8j9k wip
pick l0m1n2o fix typo

# Commands:
# pick   = use this commit as-is
# reword = use commit, but edit the message
# edit   = pause here so you can amend the commit
# squash = meld into the previous commit (combines messages)
# fixup  = meld into the previous commit (discards this message)
# drop   = remove this commit entirely
#
# Reorder lines to reorder commits.
# Save and close to execute.
```

Change the keywords in front of the commits, save, and close. Git runs the rebase.

---

## 3. Squashing Commits

Combine multiple small commits into one meaningful commit.

**Before** (your messy feature branch):

```
pick a1b2c3d Add due-date feature
pick d4e5f6g fix
pick h7i8j9k typo
pick l0m1n2o final fix
```

**Change it to:**

```
pick a1b2c3d Add due-date feature
fixup d4e5f6g fix
fixup h7i8j9k typo
fixup l0m1n2o final fix
```

Save and close. Result: one clean commit with the first commit's message.

> **Tip — fixup vs squash**  
> `fixup` silently discards the squashed commit's message and keeps the first commit's message. `squash` opens an editor to combine all the messages. For cleanup commits ("fix typo", "wip"), use `fixup`. For commits with meaningful messages you want to merge, use `squash`.

---

## 4. Reordering & Dropping Commits

In the interactive rebase editor:

- **Reorder:** Just move the lines up or down. Git replays them in the new order.
- **Drop:** Delete the line entirely, or change `pick` to `drop`.

```
# Original order
pick a1b2c3d Add feature A
pick d4e5f6g Add feature B
pick h7i8j9k Add feature C

# Reordered + one dropped
pick h7i8j9k Add feature C
pick a1b2c3d Add feature A
drop d4e5f6g Add feature B
```

> **Warning — Reordering can cause conflicts**  
> If commit B depends on A's changes, swapping them will produce conflicts during the rebase. Git will pause at the conflict and let you resolve it, then `git rebase --continue`.

---

## 5. Editing a Commit Mid-history

Use `edit` in the interactive list to pause at a specific commit and amend it.

```
pick a1b2c3d Add feature A
edit d4e5f6g Add feature B    <- Git pauses here
pick h7i8j9k Add feature C
```

Git stops after applying commit `d4e5f6g`. Your working directory is at that point in history:

```bash
# Make changes to files
$ echo "extra line" >> todo.py

# Amend the paused commit
$ git add todo.py
$ git commit --amend --no-edit

# Resume the rebase
$ git rebase --continue
```

---

## 6. Splitting a Commit

A commit bundled too many unrelated changes. Break it apart:

```bash
# In rebase, mark the commit as "edit"
edit a1b2c3d "Add auth and update config and fix 3 bugs"
```

Git pauses there. Reset to un-commit the changes (but keep the file edits):

```bash
$ git reset HEAD^
# All the changes are now unstaged in your working directory

# Commit them as separate focused commits
$ git add auth.py
$ git commit -m "Add authentication module"

$ git add config.py
$ git commit -m "Update config for auth settings"

$ git add bugfix_a.py bugfix_b.py bugfix_c.py
$ git commit -m "Fix three null-pointer crashes"

# Resume — Git continues replaying the remaining commits
$ git rebase --continue
```

---

## 7. Cherry-pick — Grab a Single Commit

Copy one specific commit from anywhere in the history and apply it to your current branch. The original commit stays where it is — a new copy is created.

```bash
# Apply a specific commit hash to current branch
$ git cherry-pick a4f3c2d

# Cherry-pick without auto-committing (lets you review first)
$ git cherry-pick -n a4f3c2d

# Cherry-pick a range (exclusive start, inclusive end)
$ git cherry-pick a4f3c2d..d1c2e3f

# If a conflict occurs during cherry-pick
$ git cherry-pick --continue    # after resolving
$ git cherry-pick --abort       # bail out
```

**Common use case:** A bug fix was committed to `feature-branch` but you need it on `main` right now, without merging the whole feature branch.

> **Note — Cherry-pick creates a duplicate commit**  
> The cherry-picked commit gets a new hash on your branch. If you later merge the original branch, Git may apply the same change twice. For one-off backports this is fine; for ongoing work, a proper merge is usually cleaner.

---

## Project — Clean Up todo-app History

### Step 1 — Create a messy feature branch

```bash
$ cd todo-app
$ git switch -c feature/search

$ echo "" >> todo.py
$ git commit -am "start search"

$ cat >> todo.py << 'EOF'

def search(keyword):
    """Return todos containing keyword (case-insensitive)"""
    return [t for t in todos if keyword.lower() in str(t).lower()]
EOF
$ git commit -am "add search function"

$ echo "# search added" >> README.md
$ git commit -am "typo"

$ echo "" >> todo.py
$ git commit -am "fix"
```

```bash
$ git log --oneline
aaabbbc fix
999aaab typo
888aaab add search function
777aaab start search
...
```

Messy. Let's clean it up.

### Step 2 — Interactive rebase to squash

```bash
$ git rebase -i HEAD~4
```

In the editor, change to:

```
pick 777aaab start search
fixup 888aaab add search function
fixup 999aaab typo
fixup aaabbbc fix
```

Save and close.

```bash
$ git log --oneline
ccc1234 start search    <- one clean commit
...
```

### Step 3 — Reword the commit message

```bash
$ git rebase -i HEAD~1
```

Change `pick` to `reword`, save. Editor reopens for just the message:

```
Add search() function to todo.py
```

Save. Done.

```bash
$ git log --oneline
ddd5678 Add search() function to todo.py
```

### Step 4 — Cherry-pick it onto main

```bash
$ HASH=$(git log --oneline -1 | awk '{print $1}')
$ git switch main
$ git cherry-pick $HASH
$ git push
```

### Step 5 — Merge and clean up

```bash
$ git merge feature/search    # fast-forward since cherry-picked
$ git branch -d feature/search
```

---

## Summary

| Command | What it does |
|---------|-------------|
| `git rebase -i HEAD~n` | Interactive rebase of last n commits |
| `git rebase --continue` | Resume after resolving a conflict |
| `git rebase --abort` | Cancel rebase completely |
| `git cherry-pick <hash>` | Copy a commit to current branch |
| `git cherry-pick -n <hash>` | Copy without auto-committing |
| `pick` | Keep commit as-is |
| `reword` | Keep commit, edit message |
| `edit` | Pause to amend the commit |
| `squash` | Merge into previous (combine messages) |
| `fixup` | Merge into previous (discard message) |
| `drop` | Delete the commit |


---
**Previous:** [Chapter 06 — Stash & Clean](ch06-stash-clean.md)  
**Next:** [Chapter 08 — Tags & Releases](ch08-tags-releases.md)

---

**Reference:** [Cheat Sheet](CHEATSHEET.md) · [Glossary](GLOSSARY.md) · [Troubleshooting](TROUBLESHOOTING.md) · [Git Config](gitconfig.md) · [Home](README.md)
