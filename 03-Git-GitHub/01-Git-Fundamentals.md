# Git Fundamentals

## Quick Visual Summary

![Quick Summary: Git and Version Control](../assets/topic-summaries/git-version-control.svg)

> **80/20 Summary:** commits save history, branches isolate work, pull requests review changes, and revert repairs safely.

## 1. Overview
Git is a distributed version control system. It tracks every meaningful change to a codebase, who made it, and why. In DevOps, Git is the source of truth for application code, infrastructure code, and configuration.

## 2. Why This Matters
- Collaboration: many people can work on the same codebase without overwriting each other.
- Recovery: you can safely go back to a known-good state when something breaks.
- Automation: pushes to Git can trigger CI/CD pipelines, tests, and deployments.
- Auditability: every change has a history, author, and timestamp.

## 3. Core Concepts

| Concept | Meaning |
| --- | --- |
| Repository | The project folder plus its hidden `.git` history. |
| Commit | A snapshot of the project at a specific point in time. |
| Branch | An isolated line of development. |
| Remote | A hosted copy of your repository, usually on GitHub. |
| Staging area | The selection of changes that will be included in the next commit. |
| Merge | Combine one branch into another. |
| Rebase | Reapply commits on top of a new base for a cleaner history. |

## 4. Workflow
1. Clone or initialize a repository.
2. Create a branch for the work.
3. Edit files in the working directory.
4. Stage the changes with `git add`.
5. Commit the snapshot locally.
6. Push the branch to a remote.
7. Open a pull request and review before merge.

## 5. Commands

Initialize a repository:
```bash
git init
```

Check the current state:
```bash
git status
```

See what changed:
```bash
git diff
```

Stage changes:
```bash
git add .
```

Create a commit:
```bash
git commit -m "Fix database connection timeout"
```

View history:
```bash
git log --oneline --graph --decorate
```

Create and switch to a new branch:
```bash
git switch -c feature-login
```

Restore a file:
```bash
git restore app.py
```

Push changes:
```bash
git push origin feature-login
```

## 6. Configuration / Example
Use `.gitignore` to keep noisy or sensitive files out of history.

```text
node_modules/
__pycache__/
*.pyc
.env
secret-keys.pem
.DS_Store
```

## 7. Hands-on

**Step 1: Set your identity**
```bash
git config --global user.name "John Doe"
git config --global user.email "john@example.com"
```

**Step 2: Create a practice repo**
```bash
mkdir demo-repo
cd demo-repo
git init
```

**Step 3: Track a file**
```bash
echo "print('Hello World')" > script.py
git status
```

**Step 4: Stage and commit it**
```bash
git add script.py
git commit -m "Add first script"
```

**Step 5: Try a safe experiment**
```bash
git switch -c experimental-ui
echo "print('Broken UI')" >> script.py
git commit -am "Add experimental UI"
```

**Step 6: Return to safety**
```bash
git switch main
git restore script.py
```

## 8. Common Errors

- `fatal: refusing to merge unrelated histories`
  - This happens when two separate repositories are joined for the first time.
  - Fix: use `git pull --allow-unrelated-histories` only when you are sure that merge is intended.
- `Your branch and 'origin/main' have diverged`
  - Your local branch and remote branch both have unique commits.
  - Fix: fetch first, then rebase or merge intentionally.
- Accidental secret commit
  - If a password, token, or key reaches Git history, treat it as leaked.
  - Fix: rotate the secret immediately and then clean history if needed.

## 9. Best Practices

1. Commit small, logical units of work.
2. Never commit secrets or generated artifacts.
3. Use clear commit messages that explain the change.
4. Keep `main` protected and merge through pull requests.

## 10. Interview Q&A

**Q1: What is Git?**  
A1: Git is the local version control system that stores history, branches, and commits.

**Q2: What is the difference between `git fetch` and `git pull`?**  
A2: `git fetch` downloads remote changes without changing your working tree. `git pull` fetches and then merges or rebases.

**Q3: What is a merge conflict?**  
A3: It happens when Git cannot decide how to combine overlapping changes automatically.

**Q4: How do you undo uncommitted changes?**  
A4: Use `git restore .` for working tree changes or `git reset` for staged changes.

**Q5: Why is `git rebase` powerful?**  
A5: It creates a cleaner history, but rewriting public history can confuse teammates, so use it carefully.

## 11. Quick Revision Summary
- Git stores history locally.
- Branches keep work isolated.
- Commits capture snapshots.
- `.gitignore` keeps unwanted files out of history.

## 12. Official Docs
- [Pro Git Book](https://git-scm.com/book/en/v2)
- [Git Branching Basics](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)
