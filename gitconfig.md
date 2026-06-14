# .gitconfig — Recommended Settings & Aliases
#
# How to use:
#   Option A: Copy the [alias] and [core] blocks into your ~/.gitconfig
#   Option B: Run the one-liner commands listed in each section below
#   Option C: Copy this entire file to ~/.gitconfig (will overwrite existing settings)
#
# Check your current config: git config --list --global

[user]
	name  = Your Name
	email = you@example.com

[core]
	# Your preferred editor for commit messages
	editor        = code --wait       # VS Code
	# editor      = nano              # Nano (simpler)
	# editor      = vim               # Vim
	autocrlf      = input             # Linux/macOS: input | Windows: true
	excludesfile  = ~/.gitignore_global

[init]
	defaultBranch = main

[pull]
	rebase = true                     # pull --rebase by default (cleaner history)

[push]
	autoSetupRemote = true            # auto set upstream on first push (Git 2.37+)

[merge]
	conflictstyle = diff3             # show base + both sides in conflict markers

[diff]
	colorMoved = zebra                # highlight moved lines differently from changed

[rerere]
	enabled = true                    # remember conflict resolutions (re-use re-resolution)

[color]
	ui = auto

# ─────────────────────────────────────────
#  ALIASES
# ─────────────────────────────────────────
[alias]

	# ── Status & Info ──────────────────────
	st     = status -s                          # short status
	s      = status

	# ── Logging ────────────────────────────
	lg     = log --oneline --graph --all --decorate
	ll     = log --oneline -20                  # last 20 commits
	last   = log -1 HEAD --stat                 # show last commit with file stats
	who    = log --oneline --author             # git who "Name"
	find   = log --oneline --all --grep         # git find "search term"

	# ── Adding & Committing ─────────────────
	a      = add
	aa     = add .
	c      = commit -m                          # git c "message"
	ca     = commit -am                         # stage tracked + commit
	amend  = commit --amend --no-edit           # add to last commit, keep message
	fix    = commit --amend                     # open editor to fix last commit

	# ── Branching ──────────────────────────
	br     = branch
	bra    = branch -a                          # all branches (local + remote)
	brv    = branch -vv                         # branches with tracking info
	sw     = switch
	new    = switch -c                          # git new feature/xyz
	gone   = "!git fetch -p && git branch -vv | grep ': gone]' | awk '{print $1}' | xargs git branch -d"
	        # delete all local branches whose remote is gone

	# ── Merging & Rebasing ──────────────────
	mg     = merge --no-ff
	rb     = rebase
	ri     = rebase -i                          # git ri HEAD~3

	# ── Remote ─────────────────────────────
	rv     = remote -v
	fp     = fetch --prune                      # fetch + remove stale remote refs
	pu     = push -u origin HEAD                # push current branch + set tracking

	# ── Undoing ────────────────────────────
	undo   = reset --soft HEAD~1                # undo last commit, keep changes staged
	unstage = restore --staged                  # git unstage file.txt
	discard = restore                           # git discard file.txt (careful!)
	nuke   = reset --hard HEAD                  # discard ALL uncommitted changes

	# ── Stash ──────────────────────────────
	save   = stash push -u -m                   # git save "description"
	pop    = stash pop
	slist  = stash list

	# ── Diff ───────────────────────────────
	d      = diff
	ds     = diff --staged
	dw     = diff --word-diff                   # show word-level changes

	# ── Tags ───────────────────────────────
	tags   = tag -l
	lasttag = describe --tags --abbrev=0        # most recent tag

	# ── Utility ────────────────────────────
	aliases = config --get-regexp alias         # list all aliases
	ignore  = "!gi() { curl -sL https://www.toptal.com/developers/gitignore/api/$@ ;}; gi"
	        # git ignore python,node  ->  fetches a .gitignore template

# ─────────────────────────────────────────
#  HOW TO ADD INDIVIDUAL ALIASES (one-liners)
# ─────────────────────────────────────────
#
# Copy-paste any of these into your terminal instead of editing the file:
#
#   git config --global alias.st "status -s"
#   git config --global alias.lg "log --oneline --graph --all --decorate"
#   git config --global alias.undo "reset --soft HEAD~1"
#   git config --global alias.new "switch -c"
#   git config --global alias.pu "push -u origin HEAD"
#   git config --global pull.rebase true
#   git config --global push.autoSetupRemote true
#   git config --global init.defaultBranch main
#   git config --global merge.conflictstyle diff3
#   git config --global rerere.enabled true
