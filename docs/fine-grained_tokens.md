# Summary
- For creating or cloning repos, I need a GitHub Personal Access Token (PAT). A fine-grained PAT is preferred over a classic one.

## Minimum scopes for a fine-grained PAT:
- `Contents` — read/write (clone, push, pull)
- `Metadata` — read (required baseline)
- `Pull requests` — read/write (if you want PR workflows)
- `Administration` — read/write (only if creating new repos)

Store it in a password vault under a key like GITHUB_TOKEN
