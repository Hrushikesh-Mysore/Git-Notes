# Cheat Sheet

Every command from all 10 chapters in one place, grouped by topic.

---

## Setup & Config

```bash
git --version                                      # check Git is installed
git config --global user.name "Your Name"          # set name
git config --global user.email "you@example.com"   # set email
git config --global core.editor "code --wait"      # set editor (VS Code)
git config --global init.defaultBranch main        # default branch name
git config --global pull.rebase true               # pull = fetch + rebase
git config --global push.autoSetupRemote true      # auto-track on first push
git config --list                                  # view all settings
git config --list --show-origin                    # view settings + which file
```

---

## Starting a Repository

```bash
git init                          # init repo in current folder
git init my-project               # init repo in new folder
git clone <url>                   # clone remote repo
git clone <url> my-folder         # clone into specific folder name
git clone git@github.com:u/r.git  # clone via SSH
```

---

## Daily Workflow

```bash
git status                        # what's going on?
git status -s                     # short status

git add <file>                    # stage a file
git add .                         # stage everything
git add *.py                      # stage by pattern
git add -p <file>                 # stage parts of a file interactively

git commit -m "message"           # commit with inline message
git commit                        # commit (opens editor)
git commit -am "message"          # stage tracked files + commit
git commit --amend -m "message"   # fix last commit message
git commit --amend --no-edit      # add to last commit, keep message
```

---

## Inspect & Compare

```bash
git log                           # full history
git log --oneline                 # compact one-line format
git log --oneline --graph --all   # visual branch graph
git log -5 --oneline              # last 5 commits
git log --author="Name"           # commits by author
git log --grep="keyword"          # search commit messages
git log --oneline -- file.py      # commits touching a file
git log --after="2024-01-01"      # commits after date

git diff                          # unstaged changes
git diff --staged                 # staged changes
git diff abc123 def456            # between two commits
git diff main..feature -- file    # file across two branches
git diff --stat                   # summary of changes (no full diff)
git diff --word-diff              # word-level diff

git show <hash>                   # show a commit's changes
git show v1.0.0                   # show a tag
```

---

## Branches

```bash
git branch                        # list local branches
git branch -a                     # list all (local + remote)
git branch -vv                    # show tracking info

git branch <name>                 # create a branch
git switch <name>                 # switch to a branch
git switch -c <name>              # create + switch
git checkout -b <name>            # create + switch (older syntax)

git branch -d <name>              # delete (safe — refuses if unmerged)
git branch -D <name>              # force delete
git branch -m <old> <new>         # rename a branch
```

---

## Merging

```bash
git merge <branch>                # merge into current branch
git merge --no-ff <branch>        # force a merge commit
git merge --abort                 # cancel an in-progress merge
git diff main..feature            # preview before merging
```

---

## Rebasing

```bash
git rebase <branch>               # replay commits on top of branch
git rebase --continue             # continue after resolving conflict
git rebase --skip                 # skip current conflicting commit
git rebase --abort                # cancel rebase completely

git rebase -i HEAD~n              # interactive rebase of last n commits
# In the editor:
#   pick   = keep as-is
#   reword = edit message
#   edit   = pause to amend
#   squash = merge + combine messages
#   fixup  = merge + discard message
#   drop   = delete commit
```

---

## Remote Repositories

```bash
git remote -v                             # list remotes
git remote add <name> <url>               # add a remote
git remote rename <old> <new>             # rename
git remote remove <name>                  # remove
git remote show origin                    # detailed info

git fetch origin                          # download, don't apply
git fetch --all                           # fetch all remotes
git fetch --prune                         # fetch + remove stale refs

git pull                                  # fetch + merge/rebase
git pull --rebase                         # fetch + rebase
git pull origin main                      # pull specific branch

git push -u origin main                   # push + set tracking
git push                                  # push (after tracking set)
git push origin --delete <branch>         # delete remote branch
git push origin --tags                    # push all tags
git push --follow-tags                    # push annotated tags only

git branch --set-upstream-to=origin/main  # set tracking on existing branch
```

---

## Undoing Things

```bash
# Unstage
git restore --staged <file>       # unstage a file
git restore --staged .            # unstage everything

# Discard working directory changes (permanent for unstaged files)
git restore <file>                # discard changes in one file
git restore .                     # discard all working dir changes

# Reset (moves branch pointer)
git reset --soft HEAD~1           # undo commit, keep changes staged
git reset HEAD~1                  # undo commit, unstage changes
git reset --hard HEAD~1           # undo commit, discard all changes
git reset --hard <hash>           # reset to a specific commit

# Revert (safe for shared branches — creates a new undo commit)
git revert HEAD                   # revert last commit
git revert <hash>                 # revert specific commit
git revert --no-edit HEAD         # revert without opening editor

# Recover
git reflog                        # history of HEAD movements
git reset --hard <hash>           # restore to a reflog entry
git branch <name> <hash>          # create branch at a lost commit
```

---

## Stash

```bash
git stash                         # save WIP to stash
git stash push -m "description"   # stash with a message
git stash -u                      # include untracked files
git stash -a                      # include untracked + ignored files
git stash --keep-index            # stash but keep staging area

git stash list                    # show all stashes
git stash show -p stash@{0}       # see what's in a stash

git stash pop                     # apply + remove latest stash
git stash apply stash@{1}         # apply specific stash (keep in list)
git stash drop stash@{0}          # delete a stash
git stash clear                   # delete all stashes

git stash branch <name>           # new branch from stash
```

---

## Cleaning

```bash
git clean -n                      # dry run — see what would be deleted
git clean -f                      # delete untracked files
git clean -fd                     # delete untracked files + dirs
git clean -fdx                    # delete everything not tracked
git clean -i                      # interactive mode
```

---

## Tags

```bash
git tag                           # list all tags
git tag -l "v1.*"                 # filter tags by pattern

git tag v1.0                      # lightweight tag on current commit
git tag -a v1.0 -m "message"      # annotated tag (recommended)
git tag -a v1.0 <hash> -m "..."   # tag a past commit

git show v1.0                     # show tag details
git push origin v1.0              # push one tag
git push --follow-tags            # push all annotated tags
git tag -d v1.0                   # delete local tag
git push origin --delete v1.0     # delete remote tag

git checkout v1.0                 # view code at a tag (detached HEAD)
git switch -c <branch> v1.0       # new branch from a tag
```

---

## Git Internals (Plumbing)

```bash
git cat-file -t <hash>            # show type of object
git cat-file -p <hash>            # print object contents
git cat-file -p HEAD              # show current commit
git cat-file -p HEAD^{tree}       # show root tree of HEAD

git ls-files --stage              # show index (staging area) contents
git hash-object -w <file>         # write a file as a blob object
git count-objects -vH             # object count + storage size

git gc                            # pack loose objects, prune unreachables
git gc --aggressive               # more thorough gc (slower)

cat .git/HEAD                     # see what HEAD points to
cat .git/refs/heads/main          # see what main points to
find .git/refs -type f            # list all refs
```

---

## Useful One-liners

```bash
# See which branch each remote commit came from
git log --oneline --graph --all

# Find which commit introduced a bug (binary search)
git bisect start
git bisect bad                    # current commit is broken
git bisect good <hash>            # this commit was fine
# Git checks out middle commits — mark each as good/bad
git bisect reset                  # finish bisect

# Show all files changed in a commit
git show --name-only <hash>

# Find which commit deleted a file
git log --all --full-history -- path/to/deleted-file.py

# Show the content of a file at a specific commit
git show <hash>:path/to/file.py

# Count commits per author
git shortlog -sn

# List all tracked files
git ls-files

# Remove a file from Git but keep it on disk
git rm --cached <file>

# Rename a file through Git
git mv old-name.py new-name.py

# Check remote URL
git remote get-url origin
```

---

## Exit / Abort Quick Reference

| You're stuck in… | Command to escape |
|-----------------|-------------------|
| Vim editor | `:wq` to save, `:q!` to quit without saving |
| Merge conflict | `git merge --abort` |
| Rebase | `git rebase --abort` |
| Cherry-pick | `git cherry-pick --abort` |
| Revert | `git revert --abort` |
| Bisect | `git bisect reset` |
| Detached HEAD | `git switch main` |

---

*Back to [README](README.md)*
