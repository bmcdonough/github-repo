# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a documentation repository that provides guidelines and reference materials for configuring GitHub repositories. It contains best practices, commands, and step-by-step instructions rather than executable code.

## Repository Structure

- `docs/github-settings.md` - Comprehensive guide for GitHub repository configuration including:
  - Branch protection rules
  - Merge strategy configuration (squash merge)
  - Auto-delete branches and other repository settings
  - GitHub CLI commands and web interface instructions
  - Verification scripts and troubleshooting

## Key GitHub CLI Commands

The repository documents these commonly used GitHub CLI patterns:

**Branch Protection (Solo Developer - Require PRs, Allow Self-Merge):**
```bash
gh api repos/OWNER/REPO/branches/main/protection \
  --method PUT \
  --input - <<'EOF'
{
  "required_status_checks": {
    "strict": false,
    "contexts": []
  },
  "required_pull_request_reviews": {
    "required_approving_review_count": 0,
    "dismiss_stale_reviews": false,
    "require_code_owner_reviews": false
  },
  "restrictions": null,
  "enforce_admins": false,
  "required_linear_history": false,
  "allow_force_pushes": false,
  "allow_deletions": false
}
EOF
```

**Note**: Use `--input -` with heredoc for proper JSON formatting. The `--field` flag doesn't handle nested JSON objects correctly and will cause HTTP 422 errors.

**Set Squash Merge as Default:**
```bash
gh repo edit OWNER/REPO \
  --enable-squash-merge \
  --enable-merge-commit=false \
  --enable-rebase-merge=false
```

**Enable Auto-Delete Branches:**
```bash
gh repo edit OWNER/REPO --delete-branch-on-merge
```

**Verify Settings:**
```bash
# Check branch protection
gh api repos/OWNER/REPO/branches/main/protection

# Check merge settings
gh repo view OWNER/REPO \
  --json mergeCommitAllowed,squashMergeAllowed,rebaseMergeAllowed
```

## Documented Workflow

The repository recommends this Git workflow:

1. Create feature branch from `main`
2. Make changes and commit to feature branch
3. Push feature branch to remote
4. Create pull request using `gh pr create`
5. Merge using squash merge: `gh pr merge --squash`
6. Branches auto-delete after merge

## Configuration Philosophy

The documented approach prioritizes:

- **Clean commit history** - Squash merge creates one commit per feature/fix on `main`
- **PR-based workflow** - All changes go through pull requests, even for solo developers
- **No force pushes** - History rewriting prevented on protected branches
- **Auto-cleanup** - Merged branches automatically deleted to prevent clutter

## Working with This Repository

When updating documentation in this repository:

- The primary content is in `docs/github-settings.md` - this is a long, detailed reference guide
- Commands shown use placeholder `OWNER/REPO` that should be replaced with actual repository names
- Examples reference `bmcdonough/example-repo` as a sample repository
- The guide provides both GitHub CLI and web interface instructions for all operations
- Verification commands are included to confirm settings are applied correctly
