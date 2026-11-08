# Terminal-First Git & GitHub Mastery Guide

This guide documents the complete end-to-end workflow executed directly in the terminal: global configuration, SSH authentication, CLI repo creation, feature branching, PR management with `gh`, and troubleshooting real-world command errors.

---

## 1. Global Git Configuration & Aliases

Configure your global identity and default preferences once per machine:

```bash
# Set committer identity
git config --global user.name "Darshan Heble"
git config --global user.email "darshanheble@gmail.com"

# Set default initial branch name
git config --global init.defaultBranch main

# Set pull reconciliation strategy (false = merge commits, true = rebase)
git config --global pull.rebase false

# Inspect all configurations and their source file locations
git config --list --show-origin
```

### Useful Productivity Aliases
You already have these set up in your `~/.config/git/config`:
```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
```

---

## 2. Authentication & GitHub CLI (`gh`)

### 2.1 SSH Key Verification
Verify that your local SSH key (`id_ed25519`) connects to GitHub:
```bash
ssh -T git@github.com
```
*Expected Output:*
> `Hi DarshanHeble! You've successfully authenticated, but GitHub does not provide shell access.`

### 2.2 GitHub CLI Login & Verification
Login and link `gh` with your SSH credentials:
```bash
gh auth login
```
- Host: `GitHub.com`
- Protocol: `SSH`
- Method: `Login with a web browser` (enter the 8-digit device code)

Check authentication status:
```bash
gh auth status
```

---

## 3. Initializing & Publishing a Repository

### 3.1 Initialize Local Git Repo
```bash
mkdir -p ~/Projects/terminal-git-mastery
cd ~/Projects/terminal-git-mastery

# Initialize specifically with 'main' branch
git init -b main

# Create the initial file
echo "# Terminal Git Mastery" > README.md

# Stage and commit
git add README.md
git commit -m "feat: Initial Commit"
```

### 3.2 Publish Directly to GitHub via `gh repo create`
```bash
# Push an existing local folder as a new GitHub repository
gh repo create terminal-git-mastery --public --source=. --remote=origin --push
```

Verify your remote URL:
```bash
git remote -v
```

---

## 4. Feature Branch Workflow

Never work directly on `main`. Keep `main` stable and create isolated feature branches for each task.

### 4.1 Create and Switch to a Feature Branch
```bash
# Create and switch in one command
git switch -c feature/user-auth

# Verify current branch
git branch
```

### 4.2 Make Changes and Commit
```bash
cat << 'EOF' > auth.sh
#!/usr/bin/env bash
login() {
  echo "User authenticated successfully."
}
EOF

# Stage and commit
git add auth.sh
git commit -m "feat(auth): add login functionality"
```

### 4.3 Push Branch with Upstream Tracking
```bash
# -u (lowercase) sets upstream tracking to origin
git push -u origin feature/user-auth
```

---

## 5. Pull Requests via Terminal (`gh pr`)

### 5.1 Push the Base Branch (Prerequisite)
> [!IMPORTANT]
> If `main` has not yet been pushed to GitHub, GitHub cannot compare branches. Make sure `main` is published first:
> ```bash
> git push -u origin main
> ```

### 5.2 Create the Pull Request
Explicitly provide `--base main` if GitHub has not registered `main` as the default branch yet:
```bash
gh pr create --base main --title "feat: user authentication module" --body "Adds initial login helper function."
```

### 5.3 Inspect Open Pull Requests
```bash
# List all open PRs
gh pr list

# View PR details, commit log, and diff right in the terminal
gh pr view
```

### 5.4 Merge PR & Automatically Clean Up
```bash
# Merge via merge commit and delete remote branch on GitHub
gh pr merge --merge --delete-branch
```
When this command runs, `gh`:
1. Creates the merge commit on GitHub.
2. Switches your local branch back to `main`.
3. Fast-forwards your local `main` to match `origin/main`.
4. Deletes the local `feature/user-auth` branch.
5. Deletes the remote `feature/user-auth` branch on GitHub.

### 5.5 Sync & Prune Local References
Because `gh pr merge --delete-branch` already deleted the local branch, running `git branch -d feature/user-auth` is unnecessary and outputs `error: branch not found`.

To clean up any stale remote tracking refs left in `git branch -a`:
```bash
# Prunes references to deleted remote branches
git fetch --prune origin

# Inspect clean branch list
git branch -a
```

---

## 6. CLI Gotchas & Troubleshooting Guide

| Issue / Error | Cause | Corrective Command |
| :--- | :--- | :--- |
| `error: unknown switch 'U'` | Flag is case-sensitive: `-U` is invalid. | Use lowercase `-u`: `git push -u origin <branch>` |
| `head branch is the same as base branch` | `gh` assumed target branch was current branch. | Specify base: `gh pr create --base main ...` |
| `GraphQL: Base sha can't be blank, Base ref must be a branch` | The `main` branch was not yet pushed to GitHub. | Push `main` first: `git push -u origin main` |
| `error: branch 'feature/user-auth' not found` | Attempted `git branch -d` after `gh pr merge --delete-branch` already cleaned it up. | Verify with `git branch`. Branch is already safely deleted. |
| Stale `remotes/origin/feature/...` in `git branch -a` | Local cache still holds reference to deleted remote branch. | Run `git fetch --prune origin`. |
