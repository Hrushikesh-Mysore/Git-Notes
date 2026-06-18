# Chapter 04 — Remotes & GitHub

> **Time:** ~30 minutes  
> **Prerequisite:** Chapter 03 complete, `todo-app` repo exists locally.

---

## Contents

1. [What is a Remote?](#1-what-is-a-remote)
2. [Managing Remotes](#2-managing-remotes)
3. [fetch, pull, push](#3-fetch-pull-push)
4. [Tracking Branches](#4-tracking-branches)
5. [HTTPS vs SSH Authentication](#5-https-vs-ssh-authentication)
6. [Project — Push todo-app to GitHub](#project-push-todo-app-to-github)

---

## 1. What is a Remote?

A **remote** is a version of your repository hosted somewhere else — on GitHub, GitLab, your company's server, etc. It lets you back up your work, share it with others, and collaborate.

By convention, the default remote is named **`origin`**. When you `git clone` a repo, Git automatically sets `origin` to where you cloned from.

---

## 2. Managing Remotes

```bash
# List remotes with their URLs
$ git remote -v
origin  https://github.com/you/todo-app.git (fetch)
origin  https://github.com/you/todo-app.git (push)

# Add a remote
$ git remote add origin https://github.com/you/todo-app.git

# Add a second remote (e.g. a fork's upstream)
$ git remote add upstream https://github.com/original/todo-app.git

# Rename a remote
$ git remote rename origin old-origin

# Remove a remote
$ git remote remove upstream

# See detailed info: branches, tracking, fetch/push URLs
$ git remote show origin
```

---

## 3. fetch, pull, push

### git fetch

Downloads changes from the remote but does **not** touch your working directory or local branches. Always safe to run.

```bash
# Fetch everything from origin
$ git fetch origin

# Fetch from all remotes
$ git fetch --all

# After fetching, see what's new on origin/main that you don't have
$ git log HEAD..origin/main --oneline
```

### git pull

`git pull` = `git fetch` + `git merge` (or rebase, depending on config).

```bash
# Pull and merge remote changes into current branch
$ git pull origin main

# Pull using rebase instead of merge (cleaner history)
$ git pull --rebase origin main

# If tracking is configured, just run:
$ git pull
```

> **Tip — Set pull to rebase globally**  
> `git config --global pull.rebase true`  
> This avoids "Merge branch 'main' of github.com/..." noise commits in your history. Rebase replays your local commits on top of the fetched ones.

### git push

```bash
# Push local main to origin
$ git push origin main

# First push — also set upstream tracking so future pushes just need `git push`
$ git push -u origin main

# After tracking is set
$ git push

# Push a new local branch to remote
$ git push -u origin feature/priority

# Delete a branch on the remote
$ git push origin --delete feature/old-branch

# Push all local tags (tags are NOT pushed automatically)
$ git push origin --tags
```

> **Warning — Rejected push**  
> If your push is rejected with "rejected — non-fast-forward", someone else pushed to that branch first. Run `git pull --rebase`, resolve any conflicts, then push again. **Never force-push to a shared branch** without team agreement — it rewrites history for everyone.

---

## 4. Tracking Branches

A **tracking branch** links a local branch to a remote branch. Once set up, `git push` and `git pull` (with no arguments) know where to go.

```bash
# See tracking info for all local branches
$ git branch -vv
* main  a4f3c2d [origin/main] Add count() function
  dev   9b1e4f2 [origin/dev: ahead 2] WIP feature

# "ahead 2" = 2 local commits not yet pushed
# "behind 3" = 3 remote commits not yet pulled

# Set tracking on an existing branch
$ git branch --set-upstream-to=origin/main main

# Check out a remote branch and auto-track it
$ git switch --track origin/feature-xyz
# Git shorthand — if the branch name matches a remote branch:
$ git switch feature-xyz
```

> **Note — origin/main vs main**  
> `origin/main` is a **remote-tracking branch** — a local read-only snapshot of what `origin/main` looked like the last time you fetched. It updates on `git fetch`/`git pull`. Your local `main` is separate and can be ahead or behind.

---

## 5. HTTPS vs SSH Authentication

| Method | Credentials | Best for |
|--------|-------------|----------|
| **HTTPS** | Username + Personal Access Token (PAT) | Quick start, CI/CD |
| **SSH** | SSH key pair | Daily dev — no password prompts |

### Setting up SSH (recommended for day-to-day use)

```bash
# 1. Generate a key (use ed25519, it's modern and secure)
$ ssh-keygen -t ed25519 -C "you@example.com"
# Press Enter to accept default path (~/.ssh/id_ed25519)
# Optionally set a passphrase

# 2. Copy the PUBLIC key
$ cat ~/.ssh/id_ed25519.pub
# Copy the output

# 3. Add to GitHub:
#    GitHub → Settings → SSH and GPG keys → New SSH key → paste it

# 4. Test it
$ ssh -T git@github.com
Hi yourusername! You've successfully authenticated.
```

### Switch an existing repo from HTTPS to SSH

```bash
$ git remote set-url origin git@github.com:you/todo-app.git
$ git remote -v   # confirm the change
```

### Creating a Personal Access Token (for HTTPS)

GitHub → Settings → Developer settings → Personal access tokens → Generate new token.  
Give it `repo` scope. Use the token as your password when Git prompts.

> **Warning — Never commit credentials**  
> Never put passwords, tokens, or API keys in your code or commit them to Git. Use `.env` files (in `.gitignore`) or a secrets manager. If you accidentally commit a secret, rotate it immediately — even if you delete the commit, it may have already been logged.

---

## Project — Push todo-app to GitHub

**Goal:** Get your local `todo-app` onto GitHub and practice the push/pull loop.

### Step 1 — Create a new repo on GitHub

1. Go to [github.com](https://github.com) → **New repository**
2. Name it `todo-app`
3. **Do not** tick "Add a README" — your local repo already has one
4. Click **Create repository**

### Step 2 — Connect and push

```bash
$ cd todo-app

$ git remote add origin https://github.com/YOUR_USERNAME/todo-app.git

# Push and set tracking in one go
$ git push -u origin main
Enumerating objects: 12, done.
Counting objects: 100% (12/12), done.
...
Branch 'main' set up to track remote branch 'main' from 'origin'.
```

Visit your GitHub repo — you'll see all your commits and files.

### Step 3 — Make a change and push it

```bash
$ echo "1.0.0" > VERSION
$ git add VERSION
$ git commit -m "Add VERSION file"

# Tracking is set, so just:
$ git push
To https://github.com/YOUR_USERNAME/todo-app.git
   d1c2e3f..f5a9b12  main -> main
```

### Step 4 — Simulate a collaborator pulling

```bash
# In a different folder, clone your own repo
$ cd /tmp
$ git clone https://github.com/YOUR_USERNAME/todo-app.git todo-app-clone
$ cd todo-app-clone

$ git log --oneline
f5a9b12 Add VERSION file
d1c2e3f Merge branch 'feature/priority'
...
```

The full history is there in the fresh clone.

### Step 5 — Make a change in the clone and pull it back

```bash
# In the clone
$ echo "Cloned and edited" >> README.md
$ git commit -am "Edit README from clone"
$ git push

# Back in the original
$ cd ~/todo-app
$ git pull
Updating f5a9b12..a3c4d5e
Fast-forward
 README.md | 1 +
```

---

## Summary

| Command | What it does |
|---------|-------------|
| `git remote -v` | List remotes |
| `git remote add <name> <url>` | Add a remote |
| `git fetch origin` | Download changes, don't apply |
| `git pull` | Fetch + merge/rebase |
| `git pull --rebase` | Fetch + rebase |
| `git push -u origin main` | Push + set tracking |
| `git push` | Push (once tracking set) |
| `git push origin --delete <branch>` | Delete remote branch |
| `git branch -vv` | Show tracking info |


---
**Previous:** [Chapter 03 — Branching](ch03-branching.md)  
**Next:** [Chapter 05 — Undoing Things](ch05-undoing.md)

---

**Reference:** [Cheat Sheet](CHEATSHEET.md) · [Glossary](GLOSSARY.md) · [Troubleshooting](TROUBLESHOOTING.md) · [Git Config](gitconfig.md) · [Home](README.md)
