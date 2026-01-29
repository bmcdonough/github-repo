# GitHub Repository Settings

Configuration guide for repository protection and merge strategies.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Quick Setup (TL;DR)](#quick-setup-tldr)
4. [Branch Protection Rules](#branch-protection-rules)
   - [Using GitHub CLI](#using-github-cli-branch-protection)
   - [Using Web Interface](#using-web-interface-branch-protection)
   - [Verifying Protection Settings](#verifying-protection-settings)
   - [Testing Branch Protection](#testing-branch-protection)
5. [Default Merge Strategy](#default-merge-strategy)
   - [Using GitHub CLI](#using-github-cli-merge-strategy)
   - [Using Web Interface](#using-web-interface-merge-strategy)
   - [Verifying Merge Settings](#verifying-merge-settings)
6. [Additional Recommended Settings](#additional-recommended-settings)
   - [Auto-Delete Merged Branches](#auto-delete-merged-branches)
   - [Allow Pull Request Branch Updates](#allow-pull-request-branch-updates)
   - [Enable Auto-Merge](#enable-auto-merge)
7. [Complete Verification](#complete-verification)
8. [Troubleshooting](#troubleshooting)
9. [References](#references)

## Overview

This guide explains how to configure GitHub repository settings for the `example-repo` project to enforce best practices for code review and commit history management.

### Configured Settings

- **Branch Protection**: Prevent direct commits to `main`, require pull requests
- **Merge Strategy**: Set squash merge as default to maintain clean commit history
- **Additional Settings**: Auto-delete branches, auto-update PRs, etc.

### Why These Settings Matter

**Branch Protection:**
- Prevents accidental direct commits bypassing code review
- Enforces pull request workflow for all changes to `main`
- Prevents force pushes that rewrite history
- Prevents accidental deletion of the main branch
- Ensures code quality through review process

**Squash Merge:**
- Maintains clean, linear commit history on `main`
- One commit per feature/fix (easier to understand git log)
- Easier rollback (revert entire PR with single commit)
- Better changelog generation and release notes
- Consistent commit message format across the project

**Auto-Delete Branches:**
- Prevents branch clutter after merging
- Keeps repository clean and organized
- Reduces confusion about active vs. merged work

## Prerequisites

Before configuring repository settings, ensure you have:

- **GitHub Repository Access**: Admin or maintain role on the repository
- **GitHub CLI** (optional, for CLI method): `gh` installed and authenticated
  ```bash
  # Check if gh is installed
  gh --version

  # Authenticate if needed
  gh auth login
  ```
- **Web Browser**: For web interface method

## Quick Setup (TL;DR)

### Using GitHub CLI

For solo developers who want to require PRs but allow self-merge:

```bash
# 1. Enable branch protection on main (require PRs, allow self-merge)
gh api repos/bmcdonough/example-repo/branches/main/protection \
  --method PUT \
  --field required_pull_request_reviews='{"required_approving_review_count":0}' \
  --field restrictions=null \
  --field enforce_admins=false \
  --field allow_force_pushes=false \
  --field allow_deletions=false

# 2. Set squash as default merge method
gh repo edit bmcdonough/example-repo \
  --enable-squash-merge \
  --enable-merge-commit=false \
  --enable-rebase-merge=false

# 3. Enable auto-delete branches (optional but recommended)
gh repo edit bmcdonough/example-repo --delete-branch-on-merge

# 4. Verify settings
gh api repos/bmcdonough/example-repo/branches/main/protection
gh repo view bmcdonough/example-repo \
  --json allowMergeCommit,allowSquashMerge,allowRebaseMerge,deleteBranchOnMerge
```

### Using Web Interface

1. Go to **Settings** → **Branches** → **Add rule**
2. Branch name pattern: `main`
3. Check: **"Require a pull request before merging"**
4. Set **"Required approvals"** to **0** (solo) or **1+** (team)
5. Uncheck: **"Allow force pushes"** and **"Allow deletions"**
6. Click **"Create"**
7. Go to **Settings** → **General** → scroll to **"Pull Requests"**
8. Uncheck: **"Allow merge commits"** and **"Allow rebase merging"**
9. Check: **"Allow squash merging"**
10. Check: **"Automatically delete head branches"**

## Branch Protection Rules

Branch protection prevents unauthorized or accidental changes to important branches. For this project, we protect the `main` branch to ensure all changes go through pull requests.

### Using GitHub CLI (Branch Protection)

#### Basic Protection (Solo Developer)

Require pull requests but allow self-approval (0 required approvals):

```bash
gh api repos/bmcdonough/example-repo/branches/main/protection \
  --method PUT \
  --field required_pull_request_reviews='{"required_approving_review_count":0}' \
  --field restrictions=null \
  --field enforce_admins=false \
  --field allow_force_pushes=false \
  --field allow_deletions=false
```

**What this does:**
- Requires all changes to go through pull requests
- Allows you to merge your own PRs without waiting for approval
- Prevents force pushes and branch deletion
- Does not enforce rules on repository admins

#### Team Protection (Require Approvals)

Require at least 1 approval before merging:

```bash
gh api repos/bmcdonough/example-repo/branches/main/protection \
  --method PUT \
  --field required_pull_request_reviews='{
    "required_approving_review_count":1,
    "dismiss_stale_reviews":true,
    "require_code_owner_reviews":false
  }' \
  --field restrictions=null \
  --field enforce_admins=false \
  --field allow_force_pushes=false \
  --field allow_deletions=false
```

**What this does:**
- Requires all changes to go through pull requests
- Requires at least 1 approval from another contributor
- Dismisses stale reviews when new commits are pushed
- Prevents force pushes and branch deletion

#### Advanced Protection (With CI/CD Status Checks)

Require CI/CD tests to pass before merging:

```bash
gh api repos/bmcdonough/example-repo/branches/main/protection \
  --method PUT \
  --field required_status_checks='{
    "strict":true,
    "contexts":["Lint / lint","Test / test (3.9)","Test / test (3.10)","Test / test (3.11)","Test / test (3.12)"]
  }' \
  --field required_pull_request_reviews='{
    "required_approving_review_count":1,
    "dismiss_stale_reviews":true
  }' \
  --field restrictions=null \
  --field enforce_admins=false \
  --field allow_force_pushes=false \
  --field allow_deletions=false
```

**What this does:**
- Requires all status checks to pass before merging
- Requires branch to be up to date with base branch (strict mode)
- Requires at least 1 approval
- Status check names match the GitHub Actions workflows in this repository

**Note**: The `contexts` array should match your actual GitHub Actions workflow job names. Check your `.github/workflows/` files to find the correct job names.

### Using Web Interface (Branch Protection)

#### Step-by-Step Instructions

1. **Navigate to Branch Protection Settings**
   - Go to: `https://github.com/bmcdonough/example-repo/settings/branches`
   - Or: Repository → **Settings** (tab) → **Branches** (left sidebar)

2. **Add Branch Protection Rule**
   - Click **"Add branch protection rule"** or **"Add rule"**
   - In **"Branch name pattern"** field, enter: `main`

3. **Configure Protection Rules**

   **Essential Settings (Required):**

   - ✅ Check: **"Require a pull request before merging"**
     - For **solo development**: Set **"Required number of approvals before merging"** to **0**
     - For **team development**: Set **"Required number of approvals before merging"** to **1** or more

   **Recommended Settings:**

   - ✅ Check: **"Require conversation resolution before merging"** (ensures all PR comments are addressed)
   - ✅ Check: **"Require linear history"** (optional, enforces clean history)
   - ❌ Uncheck: **"Allow force pushes"** (prevents history rewriting)
   - ❌ Uncheck: **"Allow deletions"** (prevents accidental branch deletion)

   **Optional Settings (if using CI/CD):**

   - ✅ Check: **"Require status checks to pass before merging"**
   - ✅ Check: **"Require branches to be up to date before merging"**
   - In the search box, add your status check names (e.g., `Lint / lint`, `Test / test (3.9)`, etc.)

   **For Team Development:**

   - ✅ Check: **"Dismiss stale pull request approvals when new commits are pushed"**
   - ✅ Consider: **"Require review from Code Owners"** (if using CODEOWNERS file)
   - ✅ Check: **"Do not allow bypassing the above settings"** (enforces rules on admins)

4. **Save Protection Rule**
   - Scroll to bottom of page
   - Click **"Create"** or **"Save changes"**
   - You should see a success message

### Verifying Protection Settings

Check if branch protection is properly configured:

```bash
# Check if branch protection is enabled
gh api repos/bmcdonough/example-repo/branches/main/protection

# Pretty-printed summary of key settings
gh api repos/bmcdonough/example-repo/branches/main/protection \
  --jq '{
    pull_requests_required: (.required_pull_request_reviews != null),
    required_approvals: .required_pull_request_reviews.required_approving_review_count,
    status_checks_required: (.required_status_checks != null),
    required_status_checks: .required_status_checks.contexts,
    force_pushes_allowed: .allow_force_pushes.enabled,
    deletions_allowed: .allow_deletions.enabled,
    enforce_admins: .enforce_admins.enabled
  }'
```

**Expected Output (Solo Developer):**
```json
{
  "pull_requests_required": true,
  "required_approvals": 0,
  "status_checks_required": false,
  "required_status_checks": [],
  "force_pushes_allowed": false,
  "deletions_allowed": false,
  "enforce_admins": false
}
```

**Expected Output (Team with CI/CD):**
```json
{
  "pull_requests_required": true,
  "required_approvals": 1,
  "status_checks_required": true,
  "required_status_checks": ["Lint / lint", "Test / test (3.9)", ...],
  "force_pushes_allowed": false,
  "deletions_allowed": false,
  "enforce_admins": false
}
```

### Testing Branch Protection

Verify that branch protection is working by attempting a direct push to `main`:

```bash
# Ensure you're on main branch
git checkout main

# Try to create and push a direct commit (this should fail)
git commit --allow-empty -m "Test direct commit"
git push origin main
```

**Expected Result (Protection Working):**
```
remote: error: GH006: Protected branch update failed for refs/heads/main.
remote: error: At least 1 pull request review is required.
To github.com:bmcdonough/example-repo.git
 ! [remote rejected] main -> main (protected branch hook declined)
error: failed to push some refs to 'github.com:bmcdonough/example-repo.git'
```

This error confirms that branch protection is working correctly.

**Correct Workflow (Using Pull Request):**
```bash
# Create a feature branch
git checkout -b feature/my-change

# Make changes and commit
git add .
git commit -m "Add my feature"

# Push feature branch
git push origin feature/my-change

# Create pull request
gh pr create --base main --head feature/my-change \
  --title "Add my feature" \
  --body "Description of changes"

# After review/approval, merge PR via GitHub web interface or:
gh pr merge feature/my-change --squash
```

## Default Merge Strategy

GitHub supports three merge strategies: merge commits, squash merging, and rebase merging. For this project, we use **squash merge** exclusively to maintain a clean, linear commit history.

### Why Squash Merge?

**Benefits:**
- **Clean history**: One commit per feature/fix instead of many small commits
- **Easier rollback**: Revert entire PR with a single `git revert`
- **Better changelog**: Each commit on `main` represents a complete feature or fix
- **Consistent format**: Standardized commit messages following project conventions
- **Easier bisect**: When debugging, each commit is a complete, working change

**Comparison:**
- **Merge Commit**: Creates a merge commit, preserves all PR commits (cluttered history)
- **Squash Merge**: Combines all PR commits into one (recommended for this project)
- **Rebase Merge**: Replays all PR commits onto base branch (can make history confusing)

### Using GitHub CLI (Merge Strategy)

Enable only squash merge, disable other merge methods:

```bash
gh repo edit bmcdonough/example-repo \
  --enable-squash-merge \
  --enable-merge-commit=false \
  --enable-rebase-merge=false
```

**What this does:**
- Enables the "Squash and merge" button in PRs
- Disables "Create a merge commit" button
- Disables "Rebase and merge" button
- Makes squash merge the only available merge method

### Using Web Interface (Merge Strategy)

#### Step-by-Step Instructions

1. **Navigate to Repository Settings**
   - Go to: `https://github.com/bmcdonough/example-repo/settings`
   - Or: Repository → **Settings** (tab) → **General** (left sidebar)

2. **Scroll to "Pull Requests" Section**
   - Located in the middle of the settings page
   - Look for the heading **"Pull Requests"**

3. **Configure Merge Methods**

   Under **"Allow merge commits"**:
   - ❌ **Uncheck** this option

   Under **"Allow squash merging"**:
   - ✅ **Check** this option (ensure it's enabled)
   - This will be the only merge method available

   Under **"Allow rebase merging"**:
   - ❌ **Uncheck** this option

4. **Additional Recommended Settings**

   - ✅ **Check**: **"Always suggest updating pull request branches"**
   - ✅ **Check**: **"Automatically delete head branches"** (see Additional Settings section)
   - ✅ **Check**: **"Allow auto-merge"** (optional, see Additional Settings section)

5. **Save Changes**
   - Changes are usually auto-saved
   - Look for a success message at the top of the page
   - If there's a **"Save changes"** button at the bottom, click it

### Verifying Merge Settings

Check current merge strategy configuration:

```bash
# Check merge settings
gh repo view bmcdonough/example-repo \
  --json allowMergeCommit,allowSquashMerge,allowRebaseMerge

# Pretty-printed
gh repo view bmcdonough/example-repo \
  --json allowMergeCommit,allowSquashMerge,allowRebaseMerge \
  --jq '{
    merge_commit: .allowMergeCommit,
    squash_merge: .allowSquashMerge,
    rebase_merge: .allowRebaseMerge
  }'
```

**Expected Output:**
```json
{
  "merge_commit": false,
  "squash_merge": true,
  "rebase_merge": false
}
```

This confirms that only squash merge is enabled.

## Additional Recommended Settings

Beyond branch protection and merge strategy, these additional settings improve repository management.

### Auto-Delete Merged Branches

Automatically delete branch after a pull request is merged. This keeps the repository clean and prevents confusion about which branches are active.

**Using GitHub CLI:**
```bash
gh repo edit bmcdonough/example-repo --delete-branch-on-merge
```

**Using Web Interface:**
1. Go to **Settings** → **General**
2. Scroll to **"Pull Requests"** section
3. Check: **"Automatically delete head branches"**
4. Changes auto-save

**What this does:**
- After merging a PR, the source branch is automatically deleted
- Only deletes the head (source) branch, not the base (target) branch
- Keeps branch list clean and focused on active work
- Can still restore deleted branches if needed

**Verify:**
```bash
gh repo view bmcdonough/example-repo \
  --json deleteBranchOnMerge \
  --jq '{auto_delete_branches: .deleteBranchOnMerge}'
```

### Allow Pull Request Branch Updates

Allow updating PR branch when the base branch advances. This makes it easier to keep PRs up to date without force pushing.

**Using GitHub CLI:**
```bash
gh repo edit bmcdonough/example-repo --allow-update-branch
```

**Using Web Interface:**
1. Go to **Settings** → **General**
2. Scroll to **"Pull Requests"** section
3. Check: **"Always suggest updating pull request branches"**
4. Changes auto-save

**What this does:**
- Adds an "Update branch" button to PRs when they're behind the base branch
- Allows contributors to easily sync their PR with the latest changes
- Maintains PR branch without requiring local force push
- Useful when branch protection requires "up to date" branches

### Enable Auto-Merge

Allow pull requests to automatically merge when all requirements are met. This is useful for solo developers or when you want PRs to merge after CI passes.

**Using GitHub CLI:**
```bash
gh repo edit bmcdonough/example-repo --enable-auto-merge
```

**Using Web Interface:**
1. Go to **Settings** → **General**
2. Scroll to **"Pull Requests"** section
3. Check: **"Allow auto-merge"**
4. Changes auto-save

**What this does:**
- Adds an "Enable auto-merge" option to PRs
- PR will automatically merge once all requirements are met (approvals, CI checks, etc.)
- Useful for automated dependency updates (Dependabot PRs)
- Can be enabled per-PR basis

**Note**: Auto-merge requires branch protection rules to be effective. Without protection rules, PRs would auto-merge immediately.

**How to use (once enabled):**
```bash
# Enable auto-merge on a PR
gh pr merge <PR-number> --auto --squash

# Or via web interface: Click "Enable auto-merge" dropdown on PR page
```

## Complete Verification

Use this comprehensive script to verify all GitHub repository settings are configured correctly.

### Verification Script

Save this as `verify-github-settings.sh`:

```bash
#!/bin/bash
# verify-github-settings.sh
# Verifies GitHub repository settings for example-repo

REPO="bmcdonough/example-repo"

echo "=========================================="
echo "GitHub Repository Settings Verification"
echo "Repository: $REPO"
echo "=========================================="
echo ""

echo "=== Branch Protection Status ==="
gh api repos/$REPO/branches/main/protection \
  --jq '{
    pull_requests_required: (.required_pull_request_reviews != null),
    required_approvals: .required_pull_request_reviews.required_approving_review_count,
    status_checks_required: (.required_status_checks != null),
    force_pushes_allowed: .allow_force_pushes.enabled,
    deletions_allowed: .allow_deletions.enabled,
    enforce_admins: .enforce_admins.enabled
  }' 2>/dev/null

if [ $? -ne 0 ]; then
  echo "⚠️  Branch protection not configured or not accessible"
fi

echo ""
echo "=== Merge Strategy Settings ==="
gh repo view $REPO \
  --json allowMergeCommit,allowSquashMerge,allowRebaseMerge \
  --jq '{
    merge_commit_allowed: .allowMergeCommit,
    squash_merge_allowed: .allowSquashMerge,
    rebase_merge_allowed: .allowRebaseMerge
  }'

echo ""
echo "=== Additional Settings ==="
gh repo view $REPO \
  --json deleteBranchOnMerge \
  --jq '{
    auto_delete_branches: .deleteBranchOnMerge
  }'

echo ""
echo "=== Expected Configuration ==="
echo "Branch Protection:"
echo "  ✓ Pull requests required: true"
echo "  ✓ Force pushes allowed: false"
echo "  ✓ Deletions allowed: false"
echo ""
echo "Merge Strategy:"
echo "  ✓ Merge commit: false"
echo "  ✓ Squash merge: true (only enabled method)"
echo "  ✓ Rebase merge: false"
echo ""
echo "Additional:"
echo "  ✓ Auto-delete branches: true"
echo ""
echo "=========================================="
```

Make it executable and run:
```bash
chmod +x verify-github-settings.sh
./verify-github-settings.sh
```

### Manual Verification Checklist

Use this checklist to manually verify settings:

- [ ] **Direct push to main fails**: Try `git push origin main` (should error with "protected branch")
- [ ] **PR can be created**: Try `gh pr create --base main` (should succeed)
- [ ] **Only squash merge available**: Open any PR, check that only "Squash and merge" button is visible
- [ ] **Branches auto-delete after merge**: Merge a test PR, verify branch is deleted automatically
- [ ] **Status checks block merge** (if configured): Create PR with failing tests, verify "Merge" button is disabled
- [ ] **Update branch button available** (if configured): Create PR from outdated branch, verify "Update branch" button appears

### Quick Verification Commands

```bash
# One-liner to check all critical settings
gh api repos/bmcdonough/example-repo/branches/main/protection \
  --jq '.required_pull_request_reviews' && \
gh repo view bmcdonough/example-repo \
  --json allowSquashMerge,allowMergeCommit,allowRebaseMerge \
  --jq 'select(.allowSquashMerge == true and .allowMergeCommit == false and .allowRebaseMerge == false)'
```

If this returns data, your core settings are configured correctly.

## Troubleshooting

### "Resource not accessible by integration" Error

**Problem**: GitHub CLI lacks permissions to modify branch protection.

**Error Message**:
```
gh: Resource not accessible by personal access token (HTTP 403)
```

**Solution**:
```bash
# Re-authenticate with additional scopes
gh auth refresh -s admin:org,repo

# Or re-login completely
gh auth logout
gh auth login
# Select scopes: repo (full control)
```

**Alternative**: Use the web interface instead of CLI for initial setup.

### Branch Protection Not Taking Effect

**Problem**: Direct pushes to `main` still work after enabling protection.

**Possible Causes**:

1. **Branch name mismatch**: Protection rule name doesn't match actual branch
   ```bash
   # Verify exact branch name
   git branch -r | grep main

   # Check protection rule
   gh api repos/bmcdonough/example-repo/branches/main/protection
   ```

2. **Admin bypass**: You're a repository admin and "enforce admins" is disabled
   - Solution: Enable "enforce admins" in protection rules or test with non-admin account

3. **Propagation delay**: Changes not yet propagated (rare)
   - Solution: Wait 1-2 minutes and try again

4. **Wrong remote**: Pushing to a fork instead of the main repository
   ```bash
   # Check remote URL
   git remote -v
   ```

**Solution**:
```bash
# Verify protection is actually set
gh api repos/bmcdonough/example-repo/branches/main/protection

# If not set, reapply using commands from Quick Setup section
```

### Squash Merge Option Not Available in PR

**Problem**: "Squash and merge" button missing or disabled in pull request.

**Possible Causes**:

1. **Setting not enabled**: Squash merge not enabled in repository settings
   ```bash
   # Check if squash merge is enabled
   gh repo view bmcdonough/example-repo \
     --json allowSquashMerge \
     --jq '{squash_merge_enabled: .allowSquashMerge}'
   ```

   If `false`, enable it:
   ```bash
   gh repo edit bmcdonough/example-repo --enable-squash-merge
   ```

2. **Browser cache**: Stale page cached in browser
   - Solution: Hard refresh (Ctrl+Shift+R on Linux/Windows, Cmd+Shift+R on Mac)

3. **Insufficient permissions**: Need write access to merge PRs
   - Solution: Check repository access level (must be Write or Admin)

4. **PR requirements not met**: Status checks failing or approvals missing
   - Check PR status and ensure all requirements are satisfied

### Status Checks Not Found

**Problem**: When configuring required status checks, the check names don't appear in the search box.

**Possible Causes**:

1. **No recent workflow runs**: Status checks only appear after they've run at least once
   - Solution: Create a test PR to trigger workflows, then configure protection

2. **Wrong check name**: Check name doesn't match actual workflow job name
   ```bash
   # View recent check runs
   gh api repos/bmcdonough/example-repo/commits/main/check-runs \
     --jq '.check_runs[] | {name: .name, status: .status}'
   ```

   Use the exact names from this output in your protection rule.

3. **Workflows not on default branch**: Workflows must exist on the branch you're protecting
   - Solution: Ensure `.github/workflows/` files exist on `main` branch

### Can't Remove Branch Protection

**Problem**: Unable to delete or modify protection rules, or getting errors.

**Solution**:

Remove branch protection entirely:
```bash
# Delete branch protection
gh api repos/bmcdonough/example-repo/branches/main/protection \
  --method DELETE

# Or via web: Settings → Branches → Click "Delete" button next to the rule
```

Then recreate with correct settings.

### Auto-Merge Not Working

**Problem**: Enabled auto-merge on a PR, but it's not merging automatically.

**Checklist**:
- [ ] Auto-merge feature enabled in repository settings
- [ ] All required status checks passed
- [ ] Required approvals obtained
- [ ] No merge conflicts
- [ ] Branch is up to date with base branch (if required)

**Debug**:
```bash
# Check PR merge status
gh pr view <PR-number> --json mergeable,mergeStateStatus

# Check what's blocking the merge
gh pr checks <PR-number>
```

### Permission Denied Errors

**Problem**: "You don't have permission to modify branch protection rules"

**Solution**:
1. Verify you have Admin or Maintain role:
   ```bash
   gh api repos/bmcdonough/example-repo/collaborators/$(gh api user --jq .login)/permission \
     --jq '.permission'
   ```

   Must be `admin` or `maintain`.

2. If you're the owner but still getting errors, check organization settings (if applicable)

3. For personal repositories, ensure you're authenticated as the owner:
   ```bash
   gh auth status
   ```

## References

### Official GitHub Documentation

- [About Protected Branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [Managing a Branch Protection Rule](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule)
- [About Pull Request Merges](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/about-pull-request-merges)
- [About Merge Methods](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/about-merge-methods-on-github)
- [GitHub CLI Manual](https://cli.github.com/manual/)
- [GitHub REST API - Branch Protection](https://docs.github.com/en/rest/branches/branch-protection)

### Related Project Documentation

- [Versioning and Release Process](docker/versioning-and-releases.md) - Mentions branch protection in context of release workflow
- [Development Guide](development.md) - General development workflow and coding standards
- [Docker Quick Start](docker/quickstart.md) - Quick start guide for using the project
- [Troubleshooting](docker/troubleshooting.md) - Common issues and solutions

### GitHub CLI Resources

- [GitHub CLI Installation](https://github.com/cli/cli#installation)
- [GitHub CLI Authentication](https://cli.github.com/manual/gh_auth_login)
- [GitHub CLI Repository Commands](https://cli.github.com/manual/gh_repo)
- [GitHub CLI API Commands](https://cli.github.com/manual/gh_api)

### Best Practices

- [Git Branching Strategy](https://nvie.com/posts/a-successful-git-branching-model/)
- [Conventional Commits](https://www.conventionalcommits.org/) - Commit message format
- [Keep a Changelog](https://keepachangelog.com/) - Changelog format (used by this project)
- [Semantic Versioning](https://semver.org/) - Version numbering (this project uses CalVer instead)

---

**Document Version**: 1.0
**Last Updated**: 2026-01-29
**Maintainer**: bmcdonough
**Repository**: [example-repo](https://github.com/bmcdonough/example-repo)
