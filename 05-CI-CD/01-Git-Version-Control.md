# 🌿 Git and Version Control


## 🖼️ Quick Visual Summary

![Quick Summary: Git and Version Control](../assets/topic-summaries/git-version-control.svg)

> **⚡ 80/20 Summary:** Commits save history • branches isolate work • pull requests review changes • revert recovers safely

## 1. 🎯 Overview
Git is a distributed version control system. It meticulously tracks every single line of code changed in your project, who changed it, and why. In a DevOps workflow, Git is the undisputed "Single Source of Truth." If infrastructure, configuration, or application code is not committed to Git, it fundamentally does not exist.

## 2. 💡 Why This Matters
- **Collaboration Guarantee:** Allows 50 developers to edit the exact same codebase simultaneously without violently overwriting each other's hard work.
- **Time Travel:** A bad bug reaches production. With Git, you can execute a single command to instantly rewind the entire codebase to the exact state it was in yesterday at 3:00 PM.
- **Triggering Pipelines (CI/CD):** The moment an engineer pushes a commit to Git, it automatically signals Jenkins or GitHub actions to start building Docker images and running tests.

## 3. 🧠 Core Concepts
- **Repository (Repo):** The project folder containing all your hidden `.git` tracking history. 
- **Commit:** A permanent snapshot of your files at a specific point in time, mathematically secured by an SHA-1 hash.
- **Branch:** An independent line of development. You branch off `main`, build a feature safely in isolation, and then merge it back.
- **Remote (e.g., GitHub, GitLab):** A copy of your repository hosted safely on a central internet server for collaboration and backup.
- **Three States of Git:**
  1. **Working Directory:** Files you are actively modifying.
  2. **Staging Area (`git add`):** Files purposefully selected to be included in your next snapshot.
  3. **History (`git commit`):** Files safely permanently stored in your local database.

## 4. 🧭 Architecture / Workflow
1. **Clone:** Developer pulls a complete copy of the remote GitHub repository down to their laptop.
2. **Branching:** The developer runs `git checkout -b feature-login` to isolate their work.
3. **Modifying:** They edit Python files in the Working Directory.
4. **Staging:** They run `git add app.py` to prepare the file for saving.
5. **Committing:** They run `git commit -m "Added login route"` to capture the snapshot locally.
6. **Pushing:** They run `git push origin feature-login` to upload their local commits to GitHub.
7. **Pull Request (PR):** Other engineers safely review the code on GitHub before formally merging it into the `main` production branch.

## 5. 🛠️ Commands & Practical Usage

Initialize a brand new, empty Git repository in a folder:
```bash
git init
```

Check the exact status of your Working Directory and Staging Area:
```bash
git status
```

Stage all modified and new files:
```bash
git add .
```

Commit your staged files locally with a descriptive message:
```bash
git commit -m "Fix database connection timeout bug"
```

Download the newest changes from your teammates on GitHub and immediately merge them:
```bash
git pull origin main
```

View the detailed history of who made which commits:
```bash
git log --oneline --graph
```

## 6. ⚙️ Configuration / YAML / Code Examples
A classic `.gitignore` file. Defining this prevents Git from tracking useless, massive, or highly sensitive files.

```text
# Ignore Node.js heavy dependencies
node_modules/

# Ignore compiled Python bytecode
__pycache__/
*.pyc

# TERRIFYING: Never commit environment variables and DB passwords!
.env
secret-keys.pem

# Ignore Mac OS system files
.DS_Store
```

## 7. 🧪 Hands-on Step-by-Step

**Step 1: Setup identity (Do this once per computer)**
```bash
git config --global user.name "John Doe"
git config --global user.email "john@example.com"
```

**Step 2: Create a workspace and initialize it**
```bash
mkdir demo-repo && cd demo-repo
git init
```

**Step 3: Create a file and track it**
```bash
echo "print('Hello World')" > script.py
git status
# Notice that script.py is red (Untracked)
```

**Step 4: Stage and Commit it**
```bash
git add script.py
# Now git status shows it green (Staged)
git commit -m "Initial commit with script"
```

**Step 5: Safely perform an isolated experiment**
```bash
git checkout -b experimental-UI
echo "print('Broken UI Code')" >> script.py
git commit -am "Added broken UI"
```

**Step 6: Abort the experiment and return to safety**
```bash
git checkout main
cat script.py
# The broken UI code is magically gone. You are back to the clean state!
```

## 8. 🚨 Common Errors & Troubleshooting

- **Error: `fatal: refusing to merge unrelated histories`**
  - **Issue:** You initiated a local repo, someone else created an empty repo on GitHub, and you both committed separately. Git legally refuses to smash two entirely different timelines together.
  - **Fix:** Force the merge by appending `--allow-unrelated-histories` to your `git pull` command.
- **Error: `Your branch and 'origin/main' have diverged.`**
  - **Issue:** You made a local commit, but concurrently, a teammate pushed a completely different commit to GitHub. Now there's a fork in the road.
  - **Fix:** Run `git fetch`, then `git rebase origin/main` to mathematically stack your commits on top of theirs cleanly.
- **Error: "I accidentally committed an AWS secret key!"**
  - **Issue:** Your API key is now permanently etched into the cryptographic history of the repo.
  - **Fix:** If you didn't push yet, `git reset HEAD~1` removes the commit locally. If you *did* push to GitHub, **revoke the key immediately in AWS**. Modifying history on GitHub (via `git rebase -i` and `git push --force`) is dangerous and the key might already be scraped by bots.

## 9. ✅ Best Practices

1. **Commit Often, Push Often:** Never work for 3 days without committing. Break work into small, logical snapshots. If your laptop dies, your code exists safely on the cloud.
2. **Never push directly to `main`:** Lock the `main` branch on GitHub so it exclusively accepts Pull Requests. This heavily enforces peer code review.
3. **Write imperative commit messages:** Instead of "fixed bug", write "Fix database connection timeout mechanism". It reads like an order executing.
4. **Use `.gitignore` relentlessly:** The moment a repo is created, add a `.gitignore`. If `node_modules` gets committed accidentally, it heavily bloats the repository size for everyone forever.

## 10. 🎤 Interview Questions & Answers

**Q1: What is the fundamental difference between Git and GitHub?**
**A1:** Git is the actual command-line software running violently on your local laptop that calculates diffs and tracks history. GitHub is merely a commercial cloud hosting provider (like GitLab or Bitbucket) that holds remote copies of your Git repositories for backup and collaboration.

**Q2: What is the mechanical difference between `git fetch` and `git pull`?**
**A2:** `git fetch` safely downloads the newest history from GitHub but does *not* touch your Working Directory files. `git pull` structurally executes a `git fetch` and then immediately forcefully executes a `git merge` into your actively opened files.

**Q3: Describe standard Merge Conflicts and how they occur.**
**A3:** A merge conflict happens when Developer A and Developer B edit the exact same line of the exact same file comprehensively. When attempting to merge the branches, Git's algorithm physically cannot automatically decide which line is correct. A human must manually open the file, delete the conflict markers, and choose the correct code.

**Q4: How do you completely discard all uncommitted changes in your working directory?**
**A4:** You execute `git restore .` (modern) or `git checkout -- .` (classic) to revert files back to their state in the last commit.

**Q5: What is a `git rebase` and why is it considered highly dangerous on public branches?**
**A5:** Rebasing rewrites cryptographic history by moving the base of your branch to a new starting point. If you mathematically rewrite history on the `main` branch that 5 other developers have already pulled, their local histories will violently conflict with the new remote history, causing massive synchronization chaos.

## 11. ⚡ Quick Revision Summary
- **Git:** The local engine. **GitHub:** The remote cloud backup.
- **Three stages:** Working Directory -> `git add` -> Staging Area -> `git commit` -> Local Repo.
- **Branching:** Creating safe sandboxes without breaking `main`.
- **Golden Rule:** If it's a password, a compiled binary, or an API key, it legally belongs in `.gitignore`.

## 12. 🔗 Official Documentation Links
- [Pro Git Book (Free Official Core Reference)](https://git-scm.com/book/en/v2)
- [GitHub Flow Best Practices](https://docs.github.com/en/get-started/quickstart/github-flow)
