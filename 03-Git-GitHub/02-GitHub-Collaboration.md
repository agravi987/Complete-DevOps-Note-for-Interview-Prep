# 🌎 GitHub Collaboration and Remote Workflow

## Overview
GitHub is the collaboration layer on top of Git. It stores remote repositories, powers pull requests, hosts issue tracking, and helps teams review and ship code together.

> **80/20 Summary:** Git manages history locally, GitHub manages collaboration remotely. 🤝

## Git vs GitHub

| Tool | What it does |
| --- | --- |
| Git | Tracks history on your machine. 💻 |
| GitHub | Hosts the remote repository and collaboration workflow. ☁️ |

## Why This Matters
- 👥 Teams need a shared source of truth.
- 🧾 Pull requests make review and approval visible.
- 🔒 Branch protection keeps broken changes off `main`.
- 🗂️ Issues and Projects help organize work around code.

## Core Concepts

| Concept | Meaning |
| --- | --- |
| Remote | A hosted repository such as `origin` on GitHub. 🌍 |
| Fork | A personal copy of someone else's repository. 🍴 |
| Pull request | A request to merge changes from one branch into another. 🔁 |
| Review | Feedback from teammates before merge. 👀 |
| Branch protection | Rules that prevent direct pushes to important branches. 🛡️ |
| Issues | Task, bug, or discussion trackers inside GitHub. 🐞 |
| Releases | Packaged snapshots with notes and tags. 📦 |
| GitHub CLI | The `gh` command-line tool for GitHub workflows. 🖥️ |

## Collaboration Workflow
1. Create or clone a GitHub repository. 📥
2. Create a feature branch locally. 🌱
3. Push the branch to GitHub. 🚀
4. Open a pull request. 📝
5. Review comments, updates, and checks. 🔍
6. Merge after approval. ✅
7. Sync your local `main` branch. 🔄

## Commands

Show remotes:
```bash
git remote -v
```

Add a remote:
```bash
git remote add origin https://github.com/username/repo.git
```

Push a branch and set the upstream:
```bash
git push -u origin feature-login
```

Fetch remote changes:
```bash
git fetch origin
```

Clone with GitHub CLI:
```bash
gh repo clone username/repo
```

Open a pull request:
```bash
gh pr create
```

Checkout a pull request locally:
```bash
gh pr checkout 42
```

## Example PR Template

```md
## Summary
What changed and why?

## Testing
What did you run?

## Checklist
- [ ] Code reviewed
- [ ] Tests passed
- [ ] Documentation updated
```

## Hands-on

**Step 1: Create a repo on GitHub** 🧱
1. Sign in to GitHub.
2. Create a new repository.
3. Copy the remote URL.

**Step 2: Connect your local repo** 🔗
```bash
git remote add origin https://github.com/username/repo.git
git branch -M main
git push -u origin main
```

**Step 3: Work on a feature branch** 🌿
```bash
git switch -c feature-ui
git push -u origin feature-ui
```

**Step 4: Open a pull request** 📬
1. Go to the repository on GitHub.
2. Click Compare and pull request.
3. Add a clear title and description.

**Step 5: Merge and sync** 🔄
```bash
git switch main
git pull origin main
```

## Common Errors

- `Permission denied (publickey)`
  - Your SSH key is not registered with GitHub or your agent is not loaded.
  - Fix: use HTTPS or upload the correct SSH key.
- `remote: Repository not found`
  - The URL is wrong or the repo is private and your credentials do not have access.
  - Fix: verify the remote URL and authentication.
- `non-fast-forward`
  - The remote branch has commits you do not have locally.
  - Fix: fetch first, then merge or rebase, then push again.

## Best Practices

1. Protect `main` so changes go through pull requests.
2. Keep pull requests small and easy to review.
3. Use meaningful titles, descriptions, and commit messages.
4. Automate tests so reviews focus on logic, not guesswork.

## Interview Q&A

**Q1: What is the main purpose of GitHub?**  
A1: GitHub hosts Git repositories and adds collaboration features like pull requests, issues, reviews, and Actions.

**Q2: What is a pull request?**  
A2: A pull request asks maintainers to merge one branch into another after review.

**Q3: What is the difference between a fork and a branch?**  
A3: A fork is a separate repository copy, while a branch is a line of development inside the same repository.

**Q4: Why protect the main branch?**  
A4: It prevents accidental direct pushes and enforces review and testing.

**Q5: When should you use GitHub CLI?**  
A5: Use it when you want fast terminal-based workflow for cloning, PRs, reviews, or issue management.

## Quick Revision Summary
- GitHub stores remote repos. ☁️
- Pull requests drive review and merge. 🔁
- Branch protection keeps `main` stable. 🔒
- Issues, releases, and Actions extend the workflow. 🧩

## Official Docs
- [GitHub Docs](https://docs.github.com/) 📚
- [GitHub Flow](https://docs.github.com/en/get-started/quickstart/github-flow) 🌊
- [GitHub CLI](https://cli.github.com/manual/) 💻
