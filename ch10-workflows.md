# Chapter 10 — Git Workflows

> **Time:** ~30 minutes  
> **Prerequisite:** All previous chapters. This chapter ties everything together.

---

## Contents

1. [Why Workflows Matter](#1-why-workflows-matter)
2. [Feature Branch Workflow](#2-feature-branch-workflow)
3. [Gitflow](#3-gitflow)
4. [Trunk-Based Development](#4-trunk-based-development)
5. [Forking Workflow (Open Source)](#5-forking-workflow-open-source)
6. [Pull Requests & Code Review](#6-pull-requests-code-review)
7. [Choosing a Workflow](#7-choosing-a-workflow)
8. [Project — Full Workflow on todo-app](#project-full-workflow-on-todo-app)

---

## 1. Why Workflows Matter

Git itself doesn't enforce any process. It just tracks snapshots. A **workflow** is an agreed-upon set of rules for how your team uses branches: what they're named, when they're created, how code gets reviewed, and when it ships.

Without a workflow, teams run into: broken `main` branches, lost work, merge nightmares, and unclear ownership of code.

---

## 2. Feature Branch Workflow

The simplest useful workflow. One rule: **`main` always works. New work goes on feature branches.**

```
main:      A---B-----------F---G    (always deployable)
                \         /
feature-x:       C---D---E          (your work)
```

### The loop

```bash
# 1. Start from an up-to-date main
$ git switch main
$ git pull

# 2. Create a feature branch
$ git switch -c feature/add-export

# 3. Work, commit frequently
$ git add .
$ git commit -m "Add export to CSV function"
$ git commit -m "Handle empty todo list edge case"

# 4. Push your branch for backup / code review
$ git push -u origin feature/add-export

# 5. Open a Pull Request on GitHub (or GitLab MR)
#    Team reviews, approves, CI passes

# 6. Merge into main (via PR on GitHub, or locally)
$ git switch main
$ git pull
$ git merge --no-ff feature/add-export
$ git push

# 7. Delete the branch
$ git branch -d feature/add-export
$ git push origin --delete feature/add-export
```

**Best for:** Small to medium teams, most startups and product teams.

---

## 3. Gitflow

Gitflow uses two permanent branches and several short-lived branches. It was designed for software with scheduled releases.

### Branch structure

| Branch | Purpose | Created from | Merges into |
|--------|---------|-------------|-------------|
| `main` | Production-ready code only, tagged with versions | — | — |
| `develop` | Integration branch — latest delivered work | `main` | `main` (via release) |
| `feature/*` | One per feature | `develop` | `develop` |
| `release/*` | Prep for a release (bug fixes, version bumps only) | `develop` | `main` + `develop` |
| `hotfix/*` | Emergency production fix | `main` | `main` + `develop` |

```
main:    ──────────────────────────── v1.0 ──────────── v1.1
                                        ↑                 ↑
develop: ──── feat ──── feat ──── release/1.0    hotfix ── ...
               ↑          ↑
feature:    feat/A      feat/B
```

### Key commands

```bash
# Start a feature
$ git switch -c feature/login develop

# Finish the feature
$ git switch develop
$ git merge --no-ff feature/login
$ git branch -d feature/login

# Start a release
$ git switch -c release/1.1 develop
# Bump version, fix last bugs
$ echo "1.1.0" > VERSION
$ git commit -am "Bump version to 1.1.0"

# Finish the release — merge to both main and develop
$ git switch main
$ git merge --no-ff release/1.1
$ git tag -a v1.1.0 -m "Version 1.1.0"
$ git switch develop
$ git merge --no-ff release/1.1
$ git branch -d release/1.1
$ git push origin main develop --follow-tags

# Emergency hotfix
$ git switch -c hotfix/1.0.1 main
$ # fix the bug
$ git switch main
$ git merge --no-ff hotfix/1.0.1
$ git tag -a v1.0.1 -m "Hotfix 1.0.1"
$ git switch develop
$ git merge --no-ff hotfix/1.0.1
$ git branch -d hotfix/1.0.1
```

> **Note — Gitflow is powerful but heavy**  
> Gitflow shines for libraries, mobile apps, and software with versioned releases that must support multiple versions simultaneously. It's overkill for a web app that deploys continuously. Many teams start with Gitflow and simplify once they see the overhead.

---

## 4. Trunk-Based Development

Everyone commits to a single branch (`main` / `trunk`) frequently — at least once a day. Feature branches exist but are short-lived (hours, not weeks). The emphasis is on **keeping main green** at all times.

```
main:  A--B--C--D--E--F--G--H--I--J   (everyone here, always green)
                   ↑           ↑
                 deploy      deploy
```

### How it works

```bash
# Fetch latest before every session
$ git switch main && git pull

# Create a short branch (optional — some teams commit directly)
$ git switch -c your-name/small-feature

# Commit small, focused changes
$ git commit -m "Add CSV export (behind feature flag)"

# Push and open a PR — must be reviewed and merged same day
$ git push -u origin your-name/small-feature

# After merge, delete branch immediately
$ git branch -d your-name/small-feature
```

**Feature flags** are used to hide incomplete features from users while the code is already in `main`:

```python
FEATURE_FLAGS = {
    "csv_export": False,   # not ready for users yet
    "due_dates": True,
}

def export_csv():
    if not FEATURE_FLAGS["csv_export"]:
        return
    # ... implementation
```

> **Tip — CI is mandatory for trunk-based development**  
> With everyone on one branch, a broken commit affects the whole team. You need automated tests running on every push. No green CI = no merge.

**Best for:** Teams that deploy multiple times per day, SaaS products, mature CI/CD setups.

---

## 5. Forking Workflow (Open Source)

Used by most open-source projects on GitHub. Contributors don't have write access to the original repo — they fork it, work in their fork, then submit a Pull Request.

```bash
# 1. Fork on GitHub (click Fork button) — creates your copy

# 2. Clone YOUR fork
$ git clone https://github.com/YOU/todo-app.git
$ cd todo-app

# 3. Add the ORIGINAL as "upstream"
$ git remote add upstream https://github.com/ORIGINAL/todo-app.git
$ git remote -v
origin    https://github.com/YOU/todo-app.git (push)
upstream  https://github.com/ORIGINAL/todo-app.git (fetch)

# 4. Create a feature branch from upstream's main
$ git fetch upstream
$ git switch -c feature/my-contribution upstream/main

# 5. Make changes, commit, push to YOUR fork
$ git push -u origin feature/my-contribution

# 6. Open a Pull Request: YOUR fork → ORIGINAL repo

# 7. Keep your fork in sync with upstream
$ git fetch upstream
$ git switch main
$ git merge upstream/main
$ git push
```

---

## 6. Pull Requests & Code Review

A **Pull Request (PR)** — called a **Merge Request (MR)** on GitLab — is a request to merge one branch into another, with a discussion thread, reviewer assignment, and CI status attached.

### Good PR habits

- **Keep PRs small** — 200–400 lines of diff is ideal. Large PRs get rubber-stamped.
- **One concern per PR** — don't mix a feature and a refactor in the same PR.
- **Write a good description** — what does it do? why? any tradeoffs?
- **Link to the issue** — `Fixes #42` in the PR description automatically closes the issue on merge.
- **Respond to feedback quickly** — a PR sitting open for a week becomes a merge nightmare.

### PR lifecycle on GitHub

```bash
# After pushing your branch, GitHub shows a "Compare & pull request" button.
# Fill in: title, description, reviewers, labels, linked issue.

# Reviewer leaves comments → you push more commits to address them
# (same branch, same PR — GitHub shows the new commits automatically)
$ git add .
$ git commit -m "Address review: extract helper function"
$ git push

# Once approved and CI passes, merge (via GitHub UI or locally)
$ git switch main && git pull   # after GitHub merges it
```

---

## 7. Choosing a Workflow

| Workflow | Team size | Release cadence | Complexity |
|----------|-----------|----------------|------------|
| Feature Branch | Any | Flexible | Low |
| Gitflow | Medium–Large | Scheduled / versioned | High |
| Trunk-Based | Any | Continuous / daily | Low (but needs CI) |
| Forking | Open source | Any | Medium |

**Start simple:** Feature Branch Workflow works for 90% of teams. Add complexity only when you have a specific problem it solves.

---

## Project — Full Workflow on todo-app

This project runs the complete Feature Branch Workflow on `todo-app`, from branch creation through PR to tagged release.

### Step 1 — Sync with remote

```bash
$ cd todo-app
$ git switch main
$ git pull
$ git log --oneline -5
```

### Step 2 — Plan the feature

Feature: Add an `export_txt()` function that saves todos to a text file.

### Step 3 — Create the feature branch

```bash
$ git switch -c feature/export-txt
```

### Step 4 — Implement in small commits

```bash
# First commit — the function
$ cat >> todo.py << 'EOF'

def export_txt(filename="todos.txt"):
    """Export current todos to a plain text file."""
    with open(filename, "w") as f:
        if not todos:
            f.write("No todos.\n")
        else:
            for i, t in enumerate(todos, 1):
                f.write(f"{i}. {str(t)}\n")
    print(f"Exported {len(todos)} todos to {filename}")
EOF

$ git add todo.py
$ git commit -m "Add export_txt() function"
```

```bash
# Second commit — update CHANGELOG
$ cat >> CHANGELOG.md << 'EOF'

## [Unreleased]

### Added
- export_txt() — export all todos to a plain text file
EOF

$ git add CHANGELOG.md
$ git commit -m "Update CHANGELOG with export_txt feature"
```

### Step 5 — Push the branch

```bash
$ git push -u origin feature/export-txt
```

On GitHub, click **Compare & pull request**.  
Title: `Add export_txt() — export todos to plain text file`  
Description: `Adds a new function to export all todos to a .txt file. Handles empty list gracefully.`  
Click **Create pull request**.

### Step 6 — Simulate a review round

Someone requests a change: add the filename to the success message.

```bash
# Address the feedback
$ sed -i 's/Exported {len(todos)} todos to {filename}/Exported {len(todos)} todo(s) to "{filename}"/' todo.py
$ git add todo.py
$ git commit -m "Use quoted filename in export_txt success message"
$ git push
```

### Step 7 — Merge (after approval)

```bash
# Locally (or use GitHub's Merge button)
$ git switch main
$ git pull
$ git merge --no-ff feature/export-txt -m "Merge feature/export-txt into main"
$ git push

# Clean up
$ git branch -d feature/export-txt
$ git push origin --delete feature/export-txt
```

### Step 8 — Tag the new release

```bash
$ echo "1.1.0" > VERSION
$ git commit -am "Bump version to 1.1.0"

$ git tag -a v1.1.0 -m "v1.1.0 — add export_txt()"
$ git push --follow-tags
```

### Step 9 — Review the full project history

```bash
$ git log --oneline --graph
*   a1b2c3d (HEAD -> main, tag: v1.1.0, origin/main) Bump version to 1.1.0
*   d4e5f6g Merge feature/export-txt into main
|\
| * g7h8i9j Use quoted filename in export_txt success message
| * j0k1l2m Update CHANGELOG with export_txt feature
| * m3n4o5p Add export_txt() function
|/
*   p6q7r8s (tag: v1.0.1) Fix: clean up placeholder comment in set_due
*   s9t0u1v (tag: v1.0.0) Prepare v1.0.0 release
...
```

Clean linear-ish history, meaningful commit messages, tagged releases. This is what a well-maintained repo looks like.

---

## Summary

| Workflow | Key idea |
|----------|----------|
| **Feature Branch** | `main` always works, all new work on branches |
| **Gitflow** | Two permanent branches (`main`, `develop`), structured releases |
| **Trunk-Based** | Everyone on `main`, short branches, CI mandatory |
| **Forking** | Collaborators fork, submit PRs, never push to original |

### Common workflow commands recap

```bash
# Start work
$ git switch main && git pull
$ git switch -c feature/my-thing

# During work
$ git add . && git commit -m "..."
$ git push -u origin feature/my-thing

# Keep branch up to date with main
$ git fetch origin
$ git rebase origin/main

# Finish
$ git switch main && git pull
$ git merge --no-ff feature/my-thing
$ git push
$ git branch -d feature/my-thing
$ git push origin --delete feature/my-thing

# Release
$ git tag -a v1.2.0 -m "..."
$ git push --follow-tags
```

---

## You're Done!

You've covered:

| Ch | Topic | Key skill |
|----|-------|-----------|
| 01 | Getting Started | Install, configure |
| 02 | Git Basics | init, add, commit, log, diff |
| 03 | Branching | switch, merge, rebase |
| 04 | Remotes | push, pull, fetch, SSH |
| 05 | Undoing | amend, reset, revert, reflog |
| 06 | Stash & Clean | stash, clean |
| 07 | Rewriting History | interactive rebase, cherry-pick |
| 08 | Tags & Releases | tag, semver |
| 09 | Git Internals | objects, refs, hooks |
| 10 | Git Workflows | Feature Branch, Gitflow, Trunk-Based |

The `todo-app` repo you built now has branches, merges, tags, clean history, and a proper workflow. You know how to use Git the way professional teams do.


---
**Previous:** [Chapter 09 — Git Internals](ch09-internals.md)

---

**Reference:** [Cheat Sheet](CHEATSHEET.md) · [Glossary](GLOSSARY.md) · [Troubleshooting](TROUBLESHOOTING.md) · [Git Config](gitconfig.md) · [Home](README.md)
