# Contributing

Thanks for taking the time to improve this handbook. Contributions are welcome — whether it's fixing a typo, improving an explanation, adding a missing command, or flagging something that's wrong.

---

## What You Can Contribute

- **Typo or grammar fixes** — just open a PR directly
- **Incorrect commands** — please include what the correct output should be
- **Missing commands or flags** — especially things that are commonly used but not covered
- **Clearer explanations** — if a section confused you, it'll confuse others too
- **Troubleshooting entries** — new error messages with their fixes
- **Glossary terms** — any term used in the chapters that isn't defined yet

---

## What This Handbook Is Not

Before contributing, keep in mind the goals of this project:

- **Simple English** — explanations should be readable by beginners
- **Practical first** — commands and real usage over theory
- **Concise** — avoid adding theory that doesn't directly help someone use Git
- **Not a replacement for ProGit** — for deep dives, we link to ProGit instead of duplicating it

---

## How to Contribute

### 1. Fork and clone

```bash
$ git clone https://github.com/YOUR_USERNAME/git-handbook.git
$ cd git-handbook
```

### 2. Create a branch

Use a clear branch name:

```bash
$ git switch -c fix/typo-in-ch03
$ git switch -c add/glossary-term-rebase
$ git switch -c improve/troubleshooting-ssh
```

### 3. Make your changes

- Keep one concern per PR — don't mix a typo fix with a new section
- Test any commands you add or change in a real terminal
- Follow the existing formatting style (headings, code blocks, callout style)

### 4. Commit with a clear message

```bash
$ git commit -m "Fix: correct git reset --soft example in ch05"
$ git commit -m "Add: cherry-pick range syntax to ch07"
$ git commit -m "Improve: reword detached HEAD explanation in glossary"
```

### 5. Push and open a Pull Request

```bash
$ git push -u origin fix/typo-in-ch03
```

Open a PR on GitHub. In the description, briefly explain:
- What you changed and why
- Which chapter or file it affects
- If it fixes something wrong, what the correct behavior is

---

## Style Guide

### Callout format

Use `>` blockquotes with a bold label for notes, warnings, and tips:

```markdown
> **Note — Title here**  
> Body text explaining the note.

> **Warning — Title here**  
> Body text explaining the warning.

> **Tip — Title here**  
> Body text explaining the tip.
```

### Code blocks

Always specify the language for syntax highlighting:

````markdown
```bash
$ git status
```
````

Include `$` before commands to show they're terminal input. Include expected output as plain lines below:

````markdown
```bash
$ git log --oneline
a4f3c2d Add README
7e3a1d9 Initial commit
```
````

### Internal links

Link to chapters using their filename:

```markdown
See: [Chapter 03](ch03-branching.md)
```

Link to a specific section using the GitHub anchor format (lowercase, spaces to hyphens, special characters stripped):

```markdown
See: [Merge Conflicts](ch03-branching.md#4-merge-conflicts)
```

### Commands

- Always use `git switch` over `git checkout` for branch operations (modern syntax)
- Always use `git restore` over `git checkout --` for file operations
- Include the most common flags, not every possible flag

---

## Reporting Issues

If you spot something wrong but don't want to fix it yourself, open a GitHub Issue with:

- Which file and section the problem is in
- What it currently says
- What it should say (or what's confusing about it)

---

## Questions

If you're unsure whether something is a good contribution, open an Issue first and ask. Better to discuss before spending time writing something that won't fit.

---

*Back to [README](README.md)*
