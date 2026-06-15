# Chapter 02 — Git Basics

> **Time:** ~30 minutes  
> **Prerequisite:** Chapter 01 complete, Git installed and configured.

---

## Contents

1. [git init & git clone](#1-git-init-git-clone)
2. [Checking Status](#2-checking-status)
3. [Staging Files — git add](#3-staging-files-git-add)
4. [Committing](#4-committing)
5. [Viewing History — git log](#5-viewing-history-git-log)
6. [Seeing Changes — git diff](#6-seeing-changes-git-diff)
7. [The .gitignore File](#7-the-gitignore-file)
8. [Project — Your First Repository](#project-your-first-repository)

---

## 1. git init & git clone

Two ways to get a Git repository:

### Start from scratch — git init

```bash
$ mkdir my-project
$ cd my-project
$ git init
Initialized empty Git repository in /home/user/my-project/.git/
```

This creates a `.git/` folder. Your directory is now a Git repo. No files are tracked yet.

### Get an existing repo — git clone

```bash
# Clone from GitHub (HTTPS)
$ git clone https://github.com/user/repo.git

# Clone into a specific folder name
$ git clone https://github.com/user/repo.git my-folder

# Clone using SSH (requires SSH key — see Ch 04)
$ git clone git@github.com:user/repo.git
```

Clone downloads the full history, not just the latest files. You can go back to any commit.

---

## 2. Checking Status

`git status` is the most important command for daily work. Run it constantly. It shows what's going on in your working directory and staging area.

```bash
$ git status
On branch main
No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        README.md

nothing added to commit but untracked files present
```

**Short format** — compact and useful once you know the symbols:

```bash
$ git status -s
?? README.md       # untracked
A  newfile.py      # staged new file
 M todo.py         # modified, not staged
M  config.py       # modified, staged
```

Left column = staging area. Right column = working tree.

---

## 3. Staging Files — git add

```bash
# Stage a single file
$ git add README.md

# Stage everything in the current directory (recursively)
$ git add .

# Stage all .py files
$ git add *.py

# Stage interactively — choose which parts of a file to stage
$ git add -p todo.py
```

> [!Warning]
> — Staging is a snapshot of that moment**  
> If you `git add` a file and then edit it again before committing, only the version at the time of `git add` is staged. Run `git add` again after your second edit to include the newer changes. `git status` will show the file in both "staged" and "not staged" sections if this happens.

---

## 4. Committing

A commit is a permanent snapshot with a message explaining what you did.

```bash
# Opens your configured editor for the message
$ git commit

# Inline message (most common)
$ git commit -m "Add README with project description"

# Stage all tracked (already committed) files and commit in one step
# Does NOT stage brand-new untracked files
$ git commit -am "Fix typo in index.html"
```

> **Note — Writing good commit messages**  
> Use the imperative mood: "Fix bug" not "Fixed bug". Keep the first line under 72 characters. A good message completes: *"This commit will..."*  
> Good: `Add user authentication`, `Remove deprecated API calls`, `Fix crash on empty input`  
> Bad: `stuff`, `wip`, `fix`, `asdfgh`

---

## 5. Viewing History — git log

```bash
# Full log with author, date, message
$ git log

# Compact one-line format (use this daily)
$ git log --oneline
a4f3c2d Add remove function
c3a9b12 Add todo.py and .gitignore
7e3a1d9 Initial commit: add README

# Visual branch graph — very useful
$ git log --oneline --graph --all

# Last 5 commits
$ git log -5 --oneline

# Commits by a specific author
$ git log --author="Your Name" --oneline

# Commits that changed a specific file
$ git log --oneline -- todo.py

# Commits in a date range
$ git log --oneline --after="2024-01-01" --before="2024-06-01"

# Search commit messages
$ git log --oneline --grep="fix"
```

---

## 6. Seeing Changes — git diff

```bash
# Unstaged changes (working dir vs last commit)
$ git diff

# Staged changes (staging area vs last commit)
$ git diff --staged
$ git diff --cached          # same thing, older alias

# Compare two specific commits
$ git diff abc123 def456

# Compare a file between two branches
$ git diff main..feature -- app.py

# Just show which files changed, not the full diff
$ git diff --name-only
$ git diff --stat
```

The output shows `+` for added lines and `-` for removed lines. The `@@` header shows where in the file the change is.

---

## 7. The .gitignore File

Create a `.gitignore` file at the project root. Git will never track files matching its patterns.

```gitignore
# Python
__pycache__/
*.pyc
*.pyo
.venv/
*.egg-info/

# Node
node_modules/
dist/
build/

# Environment & secrets — never commit these
.env
.env.local
.env.*.local
*.pem
secrets.json

# OS files
.DS_Store
Thumbs.db

# Editor files
.vscode/
.idea/
*.swp
*~
```

> **Warning — .gitignore only affects untracked files**  
> If a file is already committed and you add it to `.gitignore`, Git keeps tracking it. You must first remove it from tracking:
> ```bash
> $ git rm --cached filename.txt
> $ git commit -m "Stop tracking filename.txt"
> ```
> Now `.gitignore` will work for it going forward.

> **Tip — Global gitignore**  
> Set a global ignore file for things like `.DS_Store` and editor files so you don't add them to every project:
> ```bash
> $ git config --global core.excludesfile ~/.gitignore_global
> ```

---

## Project — Your First Repository

**Goal:** Build a Python todo app from scratch using Git — from `init` to a 3-commit history.

### Step 1 — Create and enter the project folder

```bash
$ mkdir todo-app && cd todo-app
```

### Step 2 — Initialize Git

```bash
$ git init
Initialized empty Git repository in .../todo-app/.git/
```

### Step 3 — Create a README

```bash
$ echo "# Todo App" > README.md
$ echo "A simple command-line todo manager." >> README.md
```

### Step 4 — Check what Git sees

```bash
$ git status
On branch main
No commits yet

Untracked files:
        README.md
```

### Step 5 — Stage and commit the README

```bash
$ git add README.md
$ git commit -m "Initial commit: add README"
[main (root-commit) 7e3a1d9] Initial commit: add README
 1 file changed, 2 insertions(+)
 create mode 100644 README.md
```

### Step 6 — Create the app and a .gitignore

```bash
$ cat > todo.py << 'EOF'
todos = []

def add(task):
    todos.append(task)
    print(f"Added: {task}")

def remove(task):
    if task in todos:
        todos.remove(task)
        print(f"Removed: {task}")
    else:
        print(f"Not found: {task}")

def list_all():
    if not todos:
        print("No todos yet.")
    for i, t in enumerate(todos, 1):
        print(f"{i}. {t}")
EOF

$ echo "__pycache__/" > .gitignore
$ echo "*.pyc" >> .gitignore
```

### Step 7 — Stage both and commit

```bash
$ git add todo.py .gitignore
$ git status
Changes to be committed:
        new file:   .gitignore
        new file:   todo.py

$ git commit -m "Add todo.py with add/remove/list and .gitignore"
[main f4d8e01] Add todo.py with add/remove/list and .gitignore
 2 files changed, 13 insertions(+)
```

### Step 8 — Edit, diff, commit

```bash
# Add a count function
$ cat >> todo.py << 'EOF'

def count():
    return len(todos)
EOF

# See what changed before staging
$ git diff
+
+def count():
+    return len(todos)

# Stage and commit
$ git add todo.py
$ git commit -m "Add count() function to todo.py"
```

### Step 9 — Review your history

```bash
$ git log --oneline
c3a9b12 Add count() function to todo.py
f4d8e01 Add todo.py with add/remove/list and .gitignore
7e3a1d9 Initial commit: add README
```

You now have a 3-commit Git history. Keep this repo — you'll use it in every chapter.

---

## Summary

| Command | What it does |
|---------|-------------|
| `git init` | Create a new repo |
| `git clone <url>` | Copy an existing repo |
| `git status` | Show current state |
| `git status -s` | Short status |
| `git add <file>` | Stage a file |
| `git add .` | Stage everything |
| `git add -p` | Stage parts of a file |
| `git commit -m "msg"` | Commit with message |
| `git commit -am "msg"` | Stage tracked files + commit |
| `git log --oneline` | Compact history |
| `git log --oneline --graph --all` | Visual branch history |
| `git diff` | Unstaged changes |
| `git diff --staged` | Staged changes |


---
**Previous:** [Chapter 01 — Getting Started](ch01-getting-started.md)  
**Next:** [Chapter 03 — Branching](ch03-branching.md)

---

**Reference:** [Cheat Sheet](CHEATSHEET.md) · [Glossary](GLOSSARY.md) · [Troubleshooting](TROUBLESHOOTING.md) · [Git Config](gitconfig.md) · [Home](README.md)
