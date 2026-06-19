# Chapter 09 — Git Internals

> **Time:** ~25 minutes  
> **Prerequisite:** Chapter 08 complete. Comfortable with commits, branches, tags.

---

## Contents

1. [Plumbing vs Porcelain](#1-plumbing-vs-porcelain)
2. [The Four Object Types](#2-the-four-object-types)
3. [Inside .git/](#3-inside-git)
4. [How Branches and HEAD Work](#4-how-branches-and-head-work)
5. [The Index (Staging Area)](#5-the-index-staging-area)
6. [Packfiles](#6-packfiles)
7. [Git Hooks](#7-git-hooks)
8. [Project — Inspect todo-app Internals](#project-inspect-todo-app-internals)

---

## 1. Plumbing vs Porcelain

Git commands come in two tiers:

- **Porcelain** — friendly, high-level commands you use every day: `commit`, `push`, `log`, `merge`
- **Plumbing** — low-level building blocks: `hash-object`, `cat-file`, `ls-tree`, `update-ref`

Porcelain commands are built on top of plumbing. This chapter uses plumbing commands to look under the hood. You won't use most of these daily, but understanding them makes every other Git concept click.

---

## 2. The Four Object Types

Everything Git stores is a **content-addressed object** — a compressed file stored in `.git/objects/`, named by the SHA-1 hash of its content. There are exactly four types:

| Type | Stores | Analogy |
|------|--------|---------|
| **blob** | Raw file content (no filename, no path) | A file's body |
| **tree** | A directory: list of blobs/trees with names and modes | A folder |
| **commit** | Tree pointer + parent commit(s) + author + message | A snapshot |
| **tag** | Commit pointer + tagger + message (annotated tags only) | A bookmark |

### Blobs

A blob stores file content only — no filename. Two files with identical content (anywhere in any repo) share the exact same blob. This is how Git deduplicates storage.

```bash
# Manually hash some content and write a blob object
$ echo "hello git" | git hash-object --stdin -w
8ab686eafeb1f44702738c8b0f24f2567c36da6d

# Read it back
$ git cat-file -p 8ab686ea
hello git

# Check the type
$ git cat-file -t 8ab686ea
blob
```

### Trees

```bash
# Inspect the tree of the latest commit
$ git cat-file -p HEAD^{tree}
100644 blob a4f3c2d...    .gitignore
100644 blob c3a9b12...    CHANGELOG.md
100644 blob f5a9b12...    README.md
100644 blob d1c2e3f...    VERSION
100644 blob e6f7a8b...    todo.py
```

The numbers (`100644`) are the Unix file mode. `040000` is a subdirectory (another tree object).

### Commits

```bash
$ git cat-file -p HEAD
tree 3b18e512d9b089b5023fa8f25a5b6d22e4d8e96f
parent a4f3c2d1b5e67890123456789012345678901234
author Your Name <you@example.com> 1705299130 +0530
committer Your Name <you@example.com> 1705299130 +0530

Prepare v1.0.0 release
```

A commit object contains: tree hash, parent hash(es), author, committer, and message. The SHA-1 of all this content is the commit hash you see in `git log`.

Because each commit hashes its parent, you cannot change any commit in the middle of history without changing all subsequent hashes. This is what makes Git history **tamper-evident**.

---

## 3. Inside .git/

```bash
$ ls .git/
HEAD         config       description  hooks/
index        info/        logs/        objects/
packed-refs  refs/
```

| Path | What it is |
|------|------------|
| `HEAD` | Text file: `ref: refs/heads/main` (or a hash when detached) |
| `config` | Per-repo settings (overrides `~/.gitconfig`) |
| `objects/` | All blobs, trees, commits, tags — the content database |
| `refs/` | Branch and tag pointers (tiny text files with commit hashes) |
| `refs/heads/` | One file per local branch |
| `refs/remotes/` | One file per remote-tracking branch |
| `refs/tags/` | One file per tag |
| `index` | Binary file — the staging area |
| `logs/` | Reflog entries (history of HEAD movements) |
| `hooks/` | Scripts Git auto-runs at certain events |
| `packed-refs` | Compact single file holding refs for large repos |

---

## 4. How Branches and HEAD Work

A branch is literally a **41-byte text file** in `.git/refs/heads/` containing the SHA-1 of the commit it points to:

```bash
$ cat .git/refs/heads/main
d1c2e3f4b5a678901234567890123456789012ab

$ cat .git/HEAD
ref: refs/heads/main
```

When you commit, Git writes the new commit hash into that file. That's how a branch "moves forward" — it's just updating a text file.

**Detached HEAD** happens when HEAD points directly to a commit hash instead of a branch name:

```bash
$ git checkout v1.0.0
$ cat .git/HEAD
d1c2e3f4b5a678901234567890123456789012ab   <- a hash, not a branch ref
```

> **Warning — Detached HEAD**  
> In detached HEAD state, any new commits you make are not on any branch. If you switch away, those commits become "unreachable" and will eventually be garbage-collected. Always create a branch before committing in this state:  
> `$ git switch -c my-branch`

---

## 5. The Index (Staging Area)

The `index` file (`.git/index`) is a binary file that represents the **staging area** — the snapshot of what will go into the next commit. When you run `git add`, Git writes the file content to the object database as a blob, then updates the index to record: filename, mode, and blob hash.

```bash
# See what's currently in the index
$ git ls-files --stage
100644 a4f3c2d... 0    .gitignore
100644 c3a9b12... 0    CHANGELOG.md
100644 f5a9b12... 0    README.md
100644 d1c2e3f... 0    VERSION
100644 e6f7a8b... 0    todo.py
```

The `0` is the "stage number" — 0 = normal, 1/2/3 = conflict stages.

---

## 6. Packfiles

Initially, Git stores each object as its own file in `.git/objects/xx/yyyyyy...`. For small repos this is fine. For large repos with thousands of commits, this becomes millions of small files — slow and wasteful.

Git periodically **packs** loose objects into a single **packfile** with delta compression: similar objects store only the difference from each other, not full copies.

```bash
# Git does this automatically; you can trigger it manually
$ git gc
Counting objects: 42, done.
Compressing objects: 100% (38/38), done.
Writing objects: 100% (42/42), done.
Total 42 (delta 12), reused 0 (delta 0)

# More aggressive compression (slower, better ratio)
$ git gc --aggressive

# See how much space objects use
$ git count-objects -vH
count: 0                  <- 0 loose objects (all packed)
size: 0 bytes
in-pack: 42
packs: 1
size-pack: 18.50 KiB

# List packfiles
$ ls .git/objects/pack/
pack-4a5b6c7d.idx    pack-4a5b6c7d.pack
```

---

## 7. Git Hooks

Hooks are scripts in `.git/hooks/` that Git automatically runs at specific events. They let you enforce rules or automate tasks.

| Hook | When it runs | Common uses |
|------|-------------|-------------|
| `pre-commit` | Before a commit is recorded | Run linter, tests, check for secrets |
| `commit-msg` | After commit message is written | Enforce message format |
| `post-commit` | After commit completes | Send notifications |
| `pre-push` | Before `git push` | Run test suite |
| `post-merge` | After a merge | Run `npm install` if package.json changed |

To activate a hook: create a file in `.git/hooks/` with the right name, make it executable.

```bash
# Example: pre-commit hook that prevents committing debug prints
$ cat > .git/hooks/pre-commit << 'EOF'
#!/bin/sh
if git diff --cached | grep -q "print(\"DEBUG"; then
  echo "Error: debug print statement found. Remove it before committing."
  exit 1
fi
EOF

$ chmod +x .git/hooks/pre-commit
```

> **Note — Hooks are not versioned**  
> The `.git/hooks/` folder is not committed to your repo. Each person cloning the repo must set up hooks themselves. For team-wide hooks, use a tool like [Husky](https://typicode.github.io/husky/) (Node.js) or [pre-commit](https://pre-commit.com/) (Python/any).

---

## Project — Inspect todo-app Internals

### Step 1 — Read HEAD and the tip commit

```bash
$ cd todo-app

$ cat .git/HEAD
ref: refs/heads/main

$ cat .git/refs/heads/main
# Some 40-char SHA

$ git cat-file -t $(cat .git/refs/heads/main)
commit

$ git cat-file -p $(cat .git/refs/heads/main)
# Shows: tree, parent, author, committer, message
```

### Step 2 — Walk the tree

```bash
# Get the tree hash from HEAD
$ TREE=$(git cat-file -p HEAD | grep "^tree" | awk '{print $2}')
$ echo $TREE

# List the tree contents
$ git cat-file -p $TREE

# Get the blob hash for todo.py
$ BLOB=$(git cat-file -p $TREE | grep "todo.py" | awk '{print $3}')

# Read the raw file content from the blob
$ git cat-file -p $BLOB
```

### Step 3 — See all objects

```bash
$ git count-objects -vH

# List all loose objects (before gc)
$ find .git/objects -type f | grep -v pack | head -20

# Run gc and see the difference
$ git gc
$ git count-objects -vH
# count should be 0, everything in pack
```

### Step 4 — Inspect the index

```bash
$ git ls-files --stage
# Shows every file in the staging area with its blob hash
```

### Step 5 — Create a test hook

```bash
$ cat > .git/hooks/pre-commit << 'EOF'
#!/bin/sh
echo "[hook] pre-commit running..."
# Check for Python syntax errors
python3 -m py_compile todo.py 2>&1
if [ $? -ne 0 ]; then
  echo "Python syntax error in todo.py — commit blocked."
  exit 1
fi
echo "[hook] OK."
exit 0
EOF

$ chmod +x .git/hooks/pre-commit

# Test it
$ echo "bad syntax !!!" >> todo.py
$ git add todo.py
$ git commit -m "test hook"
# Hook should block the commit

$ git restore todo.py  # clean up
```

---

## Summary

| Command | What it does |
|---------|-------------|
| `git cat-file -t <hash>` | Show type of an object |
| `git cat-file -p <hash>` | Print contents of an object |
| `git cat-file -p HEAD` | Show the current commit object |
| `git cat-file -p HEAD^{tree}` | Show the root tree of HEAD |
| `git ls-files --stage` | Show index contents with blob hashes |
| `git hash-object -w <file>` | Write a file as a blob object |
| `git count-objects -vH` | Show object count and storage used |
| `git gc` | Pack loose objects, prune unreachables |
| `find .git/refs -type f` | List all refs (branches, tags) |


---
**Previous:** [Chapter 08 — Tags & Releases](ch08-tags-releases.md)  
**Next:** [Chapter 10 — Git Workflows](ch10-workflows.md)

---

**Reference:** [Cheat Sheet](CHEATSHEET.md) · [Glossary](GLOSSARY.md) · [Troubleshooting](TROUBLESHOOTING.md) · [Git Config](gitconfig.md) · [Home](README.md)
