# Chapter 01 — Getting Started

> **Time:** ~15 minutes  
> **Prerequisite:** Nothing — this is the beginning.

---

## Contents

1. [What is Git?](#1-what-is-git)
2. [Installing Git](#2-installing-git)
3. [First-Time Configuration](#3-first-time-configuration)
4. [The Three States](#4-the-three-states)
5. [Getting Help](#5-getting-help)

---

## 1. What is Git?

Git is a **version control system** — software that tracks changes to files over time so you can recall specific versions later, compare changes, and collaborate without overwriting each other's work.

Created by Linus Torvalds in 2005 for the Linux kernel. Today it's the standard tool for almost all software development.

**What Git does for you:**
- Saves snapshots of your project at any point (commits)
- Lets you go back to any previous state
- Lets multiple people work on the same codebase simultaneously
- Shows you exactly what changed, when, and who changed it

> **Note — Git vs GitHub**  
> Git is the tool on your computer. GitHub, GitLab, and Bitbucket are websites that host your repositories online. You don't need GitHub to use Git, but most teams use one.

### How Git differs from older systems

Older VCS tools (like SVN) stored *deltas* — lists of what changed in each file. Git stores a **full snapshot** of all tracked files at every commit. Unchanged files are not duplicated — Git stores a reference to the previous version. This makes branching and switching between versions very fast.

Git is also **distributed**: your local copy has the full history. You can commit, branch, and diff with no internet connection.

---

## 2. Installing Git

| OS | Command |
|----|---------|
| **macOS** | `xcode-select --install` or `brew install git` |
| **Ubuntu / Debian** | `sudo apt update && sudo apt install git` |
| **Fedora / RHEL** | `sudo dnf install git` |
| **Windows** | Download from [git-scm.com/download/win](https://git-scm.com/download/win) (includes Git Bash) |

Verify:

```bash
$ git --version
git version 2.44.0
```

---

## 3. First-Time Configuration

Before making any commit, Git needs to know who you are. This gets stamped into every commit. Do this once per machine.

```bash
$ git config --global user.name "Your Name"
$ git config --global user.email "you@example.com"
```

Set your preferred editor (used for commit messages):

```bash
# VS Code
$ git config --global core.editor "code --wait"

# Nano (simple, beginner-friendly)
$ git config --global core.editor "nano"

# Vim
$ git config --global core.editor "vim"
```

Set the default branch name to `main`:

```bash
$ git config --global init.defaultBranch main
```

Set pull to rebase by default (keeps history cleaner — explained in Ch 04):

```bash
$ git config --global pull.rebase true
```

Check all your settings:

```bash
$ git config --list
user.name=Your Name
user.email=you@example.com
core.editor=code --wait
init.defaultBranch=main
pull.rebase=true
```

> **Tip — Where configs are stored**  
> `--global` saves to `~/.gitconfig` (your home directory). You can also use `--local` (saved in `.git/config` inside a specific repo) to override settings per-project. Local always wins over global.

---

## 4. The Three States

Every file in a Git project lives in one of three states. This is the most important concept in Git — understanding it prevents most beginner confusion.

| State | Where | Meaning |
|-------|-------|---------|
| **Modified** | Working Directory | You changed the file but haven't told Git yet |
| **Staged** | Staging Area (Index) | You've marked it to go into the next commit |
| **Committed** | Git Repository (`.git/`) | The snapshot is safely stored in Git's database |

The flow is always: **modify → stage → commit**.

The **staging area** might seem like an unnecessary step. Why not just commit directly? It gives you fine control. You can modify 5 files but only stage 2 of them, so your commit is focused and clean.

> **Warning — The .git folder**  
> Every Git repo has a hidden `.git/` folder at the project root. This is where Git stores everything. **Never manually edit or delete files inside `.git/`** unless you know exactly what you're doing. If it's gone, the repository is gone.

---

## 5. Getting Help

```bash
# Full manual page for a command
$ git help commit
$ man git-commit

# Quick flag reference (much faster for day-to-day)
$ git commit -h
$ git log -h
```

---

## Summary

| Command | What it does |
|---------|-------------|
| `git config --global user.name "..."` | Set your name |
| `git config --global user.email "..."` | Set your email |
| `git config --list` | See all settings |
| `git --version` | Check Git is installed |
| `git help <command>` | Open manual for a command |

---

**Next:** [Chapter 02 — Git Basics](ch02-git-basics.md)
