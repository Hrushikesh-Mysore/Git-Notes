# Chapter 08 — Tags & Releases

> **Time:** ~20 minutes  
> **Prerequisite:** Chapter 07 complete, `todo-app` on GitHub.

---

## Contents

1. [What is a Tag?](#1-what-is-a-tag)
2. [Lightweight vs Annotated Tags](#2-lightweight-vs-annotated-tags)
3. [Creating Tags](#3-creating-tags)
4. [Pushing Tags to Remote](#4-pushing-tags-to-remote)
5. [Deleting Tags](#5-deleting-tags)
6. [Checking Out a Tag](#6-checking-out-a-tag)
7. [Semantic Versioning](#7-semantic-versioning)
8. [Project — Release v1.0.0 of todo-app](#project-release-v100-of-todo-app)

---

## 1. What is a Tag?

A tag is a **permanent bookmark on a specific commit**. Unlike a branch (which moves forward with every new commit), a tag always points to the same commit — forever.

Tags are used to mark official release points: `v1.0.0`, `v2.3.1`, `2024-Q1-release`.

---

## 2. Lightweight vs Annotated Tags

| Type | Command | Stores | Use for |
|------|---------|--------|---------|
| **Lightweight** | `git tag v1.0` | Just a pointer to a commit | Quick private bookmarks |
| **Annotated** | `git tag -a v1.0 -m "..."` | Tagger, date, message, GPG-signable | Official releases |

For anything shared publicly or pushed to a remote, use **annotated tags** — they carry full metadata and can be verified.

---

## 3. Creating Tags

```bash
# Lightweight tag on the current commit
$ git tag v1.0-beta

# Annotated tag on the current commit (recommended for releases)
$ git tag -a v1.0.0 -m "Version 1.0.0 — first stable release"

# Tag a specific past commit by hash
$ git tag -a v0.9.0 a4f3c2d -m "Beta release"

# List all tags
$ git tag

# List tags matching a pattern
$ git tag -l "v1.*"

# Show full details of an annotated tag
$ git show v1.0.0
tag v1.0.0
Tagger: Your Name <you@example.com>
Date:   Mon Jan 15 14:32:10 2024 +0530

Version 1.0.0 — first stable release

commit d1c2e3f...
Author: Your Name ...
```

---

## 4. Pushing Tags to Remote

Tags are **not pushed automatically** with `git push`. You must push them explicitly.

```bash
# Push a single tag
$ git push origin v1.0.0

# Push ALL local tags at once
$ git push origin --tags

# Push only annotated tags (skips lightweight) — recommended
$ git push --follow-tags
```

> **Note — GitHub Releases**  
> When you push an annotated tag to GitHub, you can create a Release from it:  
> Repository → Tags → click the tag → Create release  
> This lets you write release notes and attach downloadable binaries. Many teams automate this with GitHub Actions.

---

## 5. Deleting Tags

```bash
# Delete a local tag
$ git tag -d v1.0-beta

# Delete a tag on the remote
$ git push origin --delete v1.0-beta

# Older syntax for deleting remote tag
$ git push origin :refs/tags/v1.0-beta
```

> **Warning — Think before deleting published tags**  
> If a tag is already public and other people or CI pipelines depend on it, deleting it causes confusion. It's better to publish a corrected `v1.0.1` than to delete `v1.0.0`.

---

## 6. Checking Out a Tag

```bash
# View the code at a specific tag
$ git checkout v1.0.0
Note: switching to 'v1.0.0'.
You are in 'detached HEAD' state.
```

You're now in **detached HEAD** — you can look at the code but any new commits won't belong to a branch. If you want to make changes based on a tag, create a branch:

```bash
$ git switch -c hotfix/v1.0.1 v1.0.0
```

---

## 7. Semantic Versioning

The standard format for release versions: **MAJOR.MINOR.PATCH**

| Part | When to increment | Example |
|------|-------------------|---------|
| **MAJOR** | Breaking changes — old code won't work with this version | `1.0.0` → `2.0.0` |
| **MINOR** | New features, fully backward-compatible | `1.0.0` → `1.1.0` |
| **PATCH** | Bug fixes, backward-compatible | `1.0.0` → `1.0.1` |

Pre-release labels:  
- `v1.0.0-alpha` — early testing, API may change  
- `v1.0.0-beta` — feature complete, bug fixes only  
- `v1.0.0-rc.1` — release candidate, ready unless bugs found  

Build metadata: `v1.0.0+20240115` (ignored when comparing versions)

---

## Project — Release v1.0.0 of todo-app

### Step 1 — Make sure main is clean and up to date

```bash
$ cd todo-app
$ git switch main
$ git pull
$ git log --oneline   # review what's in the release
```

### Step 2 — Update VERSION and CHANGELOG

```bash
$ echo "1.0.0" > VERSION
$ cat > CHANGELOG.md << 'EOF'
# Changelog

## [1.0.0] - 2024-01-15

### Added
- add() — create a new todo
- remove() — delete a todo (safe if not found)
- list_all() — display all todos
- count() — return number of todos
- add_priority() — add a todo with low/medium/high priority
- set_due() — assign a due date to a task
- search() — find todos by keyword
EOF

$ git add VERSION CHANGELOG.md
$ git commit -m "Prepare v1.0.0 release"
$ git push
```

### Step 3 — Create an annotated release tag

```bash
$ git tag -a v1.0.0 -m "Version 1.0.0

First stable release of todo-app.

Features:
- Add, remove, list todos
- Priority levels (low/medium/high)
- Due date assignment
- Keyword search"

$ git show v1.0.0
```

### Step 4 — Push the tag to GitHub

```bash
$ git push origin v1.0.0
To https://github.com/YOUR_USERNAME/todo-app.git
 * [new tag]         v1.0.0 -> v1.0.0
```

Visit GitHub → your repo → **Tags** — you'll see `v1.0.0` listed. Click it, then **Create release** to add release notes.

### Step 5 — Simulate a patch release

```bash
# Fix a bug
$ sed -i 's/pass   # TODO: implement/pass  # placeholder/g' todo.py
$ git commit -am "Fix: clean up placeholder comment in set_due"
$ git push

$ git tag -a v1.0.1 -m "v1.0.1 — fix placeholder comment in set_due()"
$ git push origin v1.0.1
```

```bash
$ git tag -l "v1.*"
v1.0.0
v1.0.1
```

---

## Summary

| Command | What it does |
|---------|-------------|
| `git tag` | List all tags |
| `git tag -l "v1.*"` | Filter tags by pattern |
| `git tag v1.0` | Create lightweight tag |
| `git tag -a v1.0 -m "..."` | Create annotated tag |
| `git tag -a v1.0 <hash>` | Tag a past commit |
| `git show v1.0` | Show tag details |
| `git push origin v1.0` | Push one tag |
| `git push --follow-tags` | Push annotated tags |
| `git tag -d v1.0` | Delete local tag |
| `git push origin --delete v1.0` | Delete remote tag |
| `git checkout v1.0` | View code at a tag (detached HEAD) |


---
**Previous:** [Chapter 07 — Rewriting History](ch07-rewriting-history.md)  
**Next:** [Chapter 09 — Git Internals](ch09-internals.md)

---

**Reference:** [Cheat Sheet](CHEATSHEET.md) · [Glossary](GLOSSARY.md) · [Troubleshooting](TROUBLESHOOTING.md) · [Git Config](gitconfig.md) · [Home](README.md)
