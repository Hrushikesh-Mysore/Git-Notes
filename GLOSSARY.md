# Glossary

Quick definitions for every Git term used in this handbook. Listed alphabetically.

---

### Annotated Tag
A tag that stores extra metadata: the tagger's name, email, date, and a message. Unlike a lightweight tag, it is a full Git object with its own SHA-1 hash. Use annotated tags for official releases.  
See: [Chapter 08](ch08-tags-releases.md)

---

### Bare Repository
A repository with no working directory — just the `.git/` contents at the top level. Used for remote/server-side repos where nobody works directly. GitHub stores your repos as bare repositories.

---

### Blob
One of Git's four internal object types. A blob stores the raw content of a single file version, with no filename or path attached. Two files with identical content share one blob.  
See: [Chapter 09](ch09-internals.md)

---

### Branch
A lightweight, movable pointer to a commit. When you make a new commit on a branch, the pointer advances automatically. Branches are just 41-byte text files in `.git/refs/heads/`.  
See: [Chapter 03](ch03-branching.md)

---

### Cherry-pick
Copying a single commit from anywhere in history and applying it to the current branch. The original commit is not moved — a new copy with a new hash is created on the current branch.  
See: [Chapter 07](ch07-rewriting-history.md)

---

### Clone
A full local copy of a remote repository, including the entire history. `git clone` also automatically sets up `origin` as the remote and checks out the default branch.  
See: [Chapter 02](ch02-git-basics.md)

---

### Commit
A permanent snapshot of the entire project at a point in time. Each commit stores: a pointer to the root tree, pointer(s) to parent commit(s), author, committer, timestamp, and message. Identified by a SHA-1 hash.  
See: [Chapter 02](ch02-git-basics.md)

---

### Conflict (Merge Conflict)
When two branches have changed the same lines of the same file differently, Git cannot automatically decide which version to keep. It marks the file with conflict markers and asks you to resolve it manually.  
See: [Chapter 03](ch03-branching.md)

---

### Detached HEAD
A state where HEAD points directly to a commit hash instead of a branch name. You can look at and build from any commit this way, but new commits won't belong to any branch and may be lost. Create a branch to preserve them.  
See: [Chapter 09](ch09-internals.md)

---

### Diff
A comparison between two versions of a file or set of files, showing which lines were added (`+`) and which were removed (`-`). `git diff` shows unstaged changes; `git diff --staged` shows staged changes.  
See: [Chapter 02](ch02-git-basics.md)

---

### Fast-forward
A merge that simply moves a branch pointer forward, with no new merge commit, because the merged branch is directly ahead in the history. Only possible when the target branch has not diverged.  
See: [Chapter 03](ch03-branching.md)

---

### Fetch
Downloading commits, branches, and tags from a remote repository without applying any changes to your local branches or working directory. Safe to run anytime.  
See: [Chapter 04](ch04-remotes-github.md)

---

### Fork
A server-side copy of a repository under a different user's account. Common in open-source workflows: you fork, make changes in your copy, then submit a Pull Request to the original.  
See: [Chapter 10](ch10-workflows.md)

---

### HEAD
A special pointer that tracks which branch (or commit) is currently checked out. In normal use it points to a branch name (e.g. `ref: refs/heads/main`). In detached HEAD state it points directly to a commit hash.  
See: [Chapter 09](ch09-internals.md)

---

### Hook
A script in `.git/hooks/` that Git runs automatically at a specific event — before a commit, after a merge, before a push, etc. Hooks are not version-controlled and must be set up per machine.  
See: [Chapter 09](ch09-internals.md)

---

### Index
Another name for the **staging area** — the binary file at `.git/index` that tracks which file versions are queued for the next commit. `git add` writes to the index; `git commit` creates a snapshot from it.  
See: [Chapter 01](ch01-getting-started.md)

---

### Lightweight Tag
A simple pointer to a commit — just a name with no extra metadata. Good for quick local bookmarks. Not recommended for shared releases; use an annotated tag instead.  
See: [Chapter 08](ch08-tags-releases.md)

---

### Merge
Integrating the history of one branch into another. Produces either a fast-forward (pointer move only) or a three-way merge commit with two parents.  
See: [Chapter 03](ch03-branching.md)

---

### Merge Commit
A commit with two (or more) parent commits, created when two branches with diverged histories are merged. Records that a merge happened and who the two parents are.  
See: [Chapter 03](ch03-branching.md)

---

### Object
The fundamental storage unit in Git. All content (files, directories, commits, tags) is stored as objects in `.git/objects/`, addressed by the SHA-1 hash of their content. There are four types: blob, tree, commit, tag.  
See: [Chapter 09](ch09-internals.md)

---

### Origin
The conventional default name for a remote repository. When you `git clone`, Git names the source `origin` automatically. You can rename it or have multiple remotes.  
See: [Chapter 04](ch04-remotes-github.md)

---

### Packfile
A compressed binary file in `.git/objects/pack/` that bundles many loose objects together using delta compression. Git creates packfiles automatically (via `git gc`) to save space and speed up operations.  
See: [Chapter 09](ch09-internals.md)

---

### Pull
A `git fetch` followed immediately by a `git merge` (or `git rebase` if configured). Downloads remote changes and applies them to your current branch.  
See: [Chapter 04](ch04-remotes-github.md)

---

### Pull Request (PR)
A GitHub/GitLab feature (called Merge Request on GitLab) that lets you propose merging one branch into another, with a discussion thread, reviewer assignment, and CI status. Not a core Git concept — it lives on the hosting platform.  
See: [Chapter 10](ch10-workflows.md)

---

### Push
Uploading local commits to a remote repository. Requires write access and will be rejected if the remote has commits you don't have locally (non-fast-forward).  
See: [Chapter 04](ch04-remotes-github.md)

---

### Rebase
Replaying commits from one branch on top of another, producing a linear history. Rewrites commit hashes. Never rebase commits that have been pushed to a shared branch.  
See: [Chapter 03](ch03-branching.md), [Chapter 07](ch07-rewriting-history.md)

---

### Reflog
A local log of every position HEAD has been at, stored in `.git/logs/`. Your last resort for recovering commits after an accidental hard reset or dropped stash. Entries expire after 90 days.  
See: [Chapter 05](ch05-undoing.md)

---

### Remote
A version of the repository hosted elsewhere (GitHub, GitLab, a company server). Remotes are referenced by name (`origin`, `upstream`). You interact with them via `fetch`, `pull`, and `push`.  
See: [Chapter 04](ch04-remotes-github.md)

---

### Remote-tracking Branch
A read-only local reference that records the last known state of a branch on a remote — e.g. `origin/main`. Updated automatically on `git fetch` or `git pull`. Not the same as your local `main`.  
See: [Chapter 04](ch04-remotes-github.md)

---

### Repository (Repo)
A directory tracked by Git — the project folder plus the `.git/` subdirectory that stores the full history, configuration, and objects.

---

### Reset
Moving the current branch pointer to a different commit. Three modes: `--soft` (keep changes staged), `--mixed` (unstage changes, keep files), `--hard` (discard all changes). Hard reset is permanent for uncommitted work.  
See: [Chapter 05](ch05-undoing.md)

---

### Revert
Creating a new commit that undoes the changes of a previous commit. Unlike reset, revert does not rewrite history and is safe to use on pushed/shared branches.  
See: [Chapter 05](ch05-undoing.md)

---

### SHA-1 / Hash
The 40-character hexadecimal identifier for every Git object (commits, blobs, trees, tags). Computed from the content of the object, so identical content always produces the same hash. Git 2.29+ supports SHA-256 as well.

---

### Staging Area
The middle layer between your working directory and the repository. Files you `git add` are placed here. When you `git commit`, Git snapshots everything in the staging area. Also called the **index**.  
See: [Chapter 01](ch01-getting-started.md)

---

### Stash
A temporary storage stack for uncommitted changes. `git stash` saves your working directory and staging area so you can switch context, then `git stash pop` restores them.  
See: [Chapter 06](ch06-stash-clean.md)

---

### Tag
A permanent named pointer to a specific commit — typically used to mark release versions. Two types: lightweight (just a pointer) and annotated (full object with metadata).  
See: [Chapter 08](ch08-tags-releases.md)

---

### Three-way Merge
A merge where both branches have diverged since their common ancestor. Git uses the common ancestor, the tip of each branch, and produces a new merge commit with two parents.  
See: [Chapter 03](ch03-branching.md)

---

### Tracking Branch
A local branch configured to follow a remote branch. Lets `git push` and `git pull` work without specifying a remote/branch explicitly. Set with `git push -u` or `git branch --set-upstream-to`.  
See: [Chapter 04](ch04-remotes-github.md)

---

### Tree
One of Git's four internal object types. A tree represents a directory — it stores a list of blob and tree entries, each with a name, file mode, and SHA-1 hash. The root tree of a commit represents the entire project snapshot.  
See: [Chapter 09](ch09-internals.md)

---

### Untracked File
A file in the working directory that Git has never been asked to track — not yet `git add`-ed and not in `.gitignore`. Shows up in `git status` under "Untracked files".  
See: [Chapter 02](ch02-git-basics.md)

---

### Working Directory
The actual files on disk that you edit. One of the three states a file can be in (working directory → staging area → repository). `git restore <file>` discards working directory changes.  
See: [Chapter 01](ch01-getting-started.md)

---

*Back to [README](README.md)*
