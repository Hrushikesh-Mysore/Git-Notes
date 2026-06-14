# Git Handbook
### Based on ProGit 2nd Edition by Scott Chacon & Ben Straub

A practical, no-fluff guide to Git. Heavy on commands, light on theory.  
Works on Linux, macOS, and Windows (Git Bash).

---

## Chapters

| # | Chapter | What you'll learn |
|---|---------|-------------------|
| 01 | [Getting Started](ch01-getting-started.md) | Install Git, configure identity, understand the three states |
| 02 | [Git Basics](ch02-git-basics.md) | init, add, commit, status, log, diff — your daily commands |
| 03 | [Branching](ch03-branching.md) | Create branches, merge, handle conflicts, intro to rebase |
| 04 | [Remotes & GitHub](ch04-remotes-github.md) | push, pull, fetch, SSH keys, collaborate with others |
| 05 | [Undoing Things](ch05-undoing.md) | amend, reset, revert, restore — fix every kind of mistake |
| 06 | [Stash & Clean](ch06-stash-clean.md) | Save WIP without committing, remove untracked files |
| 07 | [Rewriting History](ch07-rewriting-history.md) | Interactive rebase, squash, cherry-pick |
| 08 | [Tags & Releases](ch08-tags-releases.md) | Mark versions, push tags, semantic versioning |
| 09 | [Git Internals](ch09-internals.md) | Blobs, trees, commits, refs — what's inside .git/ |
| 10 | [Git Workflows](ch10-workflows.md) | Feature Branch, Gitflow, Trunk-Based Development |

---

## Reference Files

| File | What it contains |
|------|-----------------|
| [CHEATSHEET.md](CHEATSHEET.md) | Every command from all 10 chapters on one page |
| [GLOSSARY.md](GLOSSARY.md) | Definitions for every Git term used in this handbook |
| [TROUBLESHOOTING.md](TROUBLESHOOTING.md) | The 15 most common Git errors with exact fix commands |
| [gitconfig.md](gitconfig.md) | Recommended `~/.gitconfig` with aliases and settings |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to suggest fixes and improvements |

---

## Built-in Project

Every chapter extends a single project — a command-line **todo app** — so you practice each concept on real code, not throwaway examples. By Chapter 10 you'll have a repo with a full history, branches, tags, and a clean workflow.

---

## Before You Begin

- Chapters 1–4 assume **zero prior Git knowledge**
- Chapters 5–8 assume you can run basic git commands
- Chapters 9–10 are for intermediate/advanced users
- You need a terminal: Bash, Zsh, or Git Bash on Windows

---

## Quick Reference Card

```bash
# Daily workflow
git status                    # what's going on?
git add <file>                # stage a file
git add .                     # stage everything
git commit -m "message"       # save a snapshot
git push                      # upload to remote

# Branches
git switch -c feature/xyz     # create + switch
git switch main               # switch branch
git merge feature/xyz         # merge into current
git branch -d feature/xyz     # delete branch

# Remotes
git pull                      # fetch + merge
git pull --rebase             # fetch + rebase (cleaner)
git push -u origin main       # first push, set tracking

# Undo
git restore <file>            # discard working dir changes
git restore --staged <file>   # unstage
git commit --amend            # fix last commit
git revert HEAD               # safe undo (new commit)
git reset --soft HEAD~1       # undo last commit, keep changes

# Inspect
git log --oneline --graph     # visual history
git diff                      # unstaged changes
git diff --staged             # staged changes
git reflog                    # full history of HEAD moves
```

---

## Source Material

This handbook is a condensed, practical rewrite of **ProGit 2nd Edition** — the official, comprehensive Git book written by **Scott Chacon** and **Ben Straub**. It is the most thorough Git reference available and is completely free.

| Resource | Link |
|----------|------|
| Read ProGit online (free) | [git-scm.com/book/en/v2](https://git-scm.com/book/en/v2) |
| Download PDF / EPUB / MOBI | [git-scm.com/book/en/v2](https://git-scm.com/book/en/v2) — click the download icon |
| ProGit source on GitHub | [github.com/progit/progit2](https://github.com/progit/progit2) |
| Git official website | [git-scm.com](https://git-scm.com) |
| Git reference manual | [git-scm.com/docs](https://git-scm.com/docs) |
| GitHub guides | [docs.github.com](https://docs.github.com) |

### About the Authors

**Scott Chacon** is a co-founder of GitHub and a core contributor to Git's documentation and tooling. He has been involved in the Git ecosystem since the early days and has given talks on Git at conferences worldwide.

**Ben Straub** is a software engineer and author who contributed extensively to the second edition of ProGit, expanding its coverage of day-to-day workflows, GitHub collaboration, and advanced Git usage.

ProGit is published and maintained by **Apress** and is freely available online under the [Creative Commons Attribution Non Commercial Share Alike 3.0 license](https://creativecommons.org/licenses/by-nc-sa/3.0/). This means you can share and adapt it freely for non-commercial purposes with proper attribution.

---

## Attribution

> This handbook is a derived work based on **ProGit, 2nd Edition**  
> by Scott Chacon and Ben Straub  
> © Apress, freely available at [git-scm.com/book](https://git-scm.com/book/en/v2)  
> Licensed under [CC BY-NC-SA 3.0](https://creativecommons.org/licenses/by-nc-sa/3.0/)  
>
> Content has been reorganised, condensed, and rewritten in simpler English  
> with a practical project thread added throughout. This is not an official  
> Apress or Git publication.
