# GitHub Skills (gh skill) — search/install agent skills from GitHub

Wraps the GitHub CLI's `gh skill` command family (preview feature, gh >= v2.98.0).
Use when the user wants to find, preview, or install agent skills (SKILL.md packages)
from GitHub repositories into this environment.

## Setup / prerequisites

- Binary: `/usr/local/bin/gh`. If missing, install:
  ```sh
  curl -sL https://api.github.com/repos/cli/cli/releases/latest \
    | grep browser_download_url | grep 'linux_arm64.tar.gz' | cut -d'"' -f4
  # download to /tmp, tar xzf, cp bin/gh /usr/local/bin/
  ```
  (No `gh` package in Alpine apk repos — use the release tarball.)
- Auth: token lives at `/var/minis/workspace/.secrets/github.token` (chmod 600).
  NEVER echo its value. Load it per-command:
  ```sh
  export GH_TOKEN=$(cat /var/minis/workspace/.secrets/github.token)
  gh auth status   # account: 3krbkbkrbrbg
  ```
- Code Search API requires auth — all `gh skill search` calls fail with HTTP 401 without GH_TOKEN.

## Commands

```sh
# Search public repos for skills matching a keyword
gh skill search <query> [--owner <user|org>] [--limit N] [--page N]
                [--json description,namespace,path,repo,skillName,stars]

# Install a skill (interactive selection if no skill name given)
gh skill install <owner>/<repo> [skill-name] [--agent <agent>] [--dir <path>]

# Preview a skill's SKILL.md before installing
gh skill preview <owner>/<repo> [skill-name]

# List installed skills
gh skill list

# Update installed skills
gh skill update

# Validate + publish a local skill dir to a repo
gh skill publish <path-to-skill-dir>
```

Search results rank skills whose *name* contains the query first. JSON fields:
description, namespace, path, repo, skillName, stars.

## Installing INTO Minis (this environment)

Minis reads skills from `/var/minis/skills/<name>/SKILL.md`. `gh skill install`
targets other agents' dirs by default, so for Minis either:
1. Use `--dir /var/minis/skills/<skill-name>` if supported, or
2. Fetch manually and place files:
   ```sh
   export GH_TOKEN=$(cat /var/minis/workspace/.secrets/github.token)
   gh api "repos/<owner>/<repo>/contents/<path-in-repo>" --jq .git_url | xargs -I{} gh api {} --jq .content | base64 -d > /var/minis/skills/<name>/SKILL.md
   ```
   (For multi-file skills, clone shallow: `git clone --depth 1 https://github.com/<owner>/<repo> /tmp/x`
   and copy the skill folder.)
3. Then read the new SKILL.md before using it.

## Notes

- `gh skill` is a PREVIEW feature; flags may change.
- Aliases: `gh skills` == `gh skill`.
- Examples:
  - `gh skill search terraform --limit 5`
  - `gh skill search pdf --owner anthropics`
  - `gh skill install github/awesome-copilot documentation-writer`
