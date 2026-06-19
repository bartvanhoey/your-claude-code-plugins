# BVH Claude Code Plugins

The BVH Claude Code Plugins repository lets you use Claude Skills, Agents and more in your conversations with Claude.

This guide walks through publishing your own plugin to a **private** GitHub repository and adding it to the Claude Code marketplace, so you can install and use the skills locally on your machine.

## 1. Set Up Private Repository Access

Claude Code can add a marketplace from a private GitHub repo, but it needs Git access to clone it first:

1. Make the repository private on GitHub, and make sure `.claude-plugin/marketplace.json` lives at the repo root on the default branch.
2. Install the GitHub CLI (`winget install GitHub.cli`).
3. Authenticate — when prompted, also choose to authenticate Git with your GitHub credentials, so `git clone` works for the private repo over HTTPS:

    ```markdown
    ! gh auth login
    ```

## 2. Installation

1. Add your marketplace (run inside a Claude Code session):

    ```markdown
    /plugin marketplace add bartvanhoey/bvh-claude-code-plugins
    ```

    This clones your GitHub repo and reads .claude-plugin/marketplace.json, registering the catalog —
    nothing gets installed yet. For a private repo this relies on the GitHub CLI authentication from step 1.

2. Install the plugin from it:

    ```markdown
    /plugin install bvh-claude-code-plugin@bvh-claude-code-plugins
    ```

    The @bvh-claude-code-plugins is the marketplace name (from name in marketplace.json), and
    bvh-claude-code-plugin is the plugin name (from name in plugin.json).

3. Activate it:

    ```markdown
    /reload-plugins
    ```

    Skills are namespaced by plugin name, so you'd then invoke them as:

    ```markdown
    /bvh-claude-code-plugin:code-review and /bvh-claude-code-plugin:hello.
    ```

## 3. Updating After You Push Changes

Pushing changes to the GitHub repository does not automatically refresh an already-installed plugin on your machine.

ATTENTION: Running `/plugin marketplace update bvh-claude-code-plugins` and `/reload-plugins` alone does not refresh an installed plugin.
The issue is that Claude Code only re-syncs an installed plugin's cache when the version field changes — marketplace update just refreshes the catalog of what's available, it doesn't force-refresh already-installed plugins at an unchanged version.

Fix: bump "version" in plugins/bvh-claude-code-plugin/.claude-plugin/plugin.json (e.g. 1.0.0 → 1.0.1) every time you push skill changes, then run the commands above again. That's the supported workflow for iterating on a plugin.

To do this automatically, you can use a GitHub Action to bump the version on every push to main:

```yaml
name: Bump plugin version

on:
  push:
    branches: [main]
    paths:
      - 'plugins/bvh-claude-code-plugin/**'

permissions:
  contents: write

jobs:
  bump-version:
    if: "!contains(github.event.head_commit.message, '[skip version]')"
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Bump patch version
        id: bump
        run: |
          FILE=plugins/bvh-claude-code-plugin/.claude-plugin/plugin.json
          VERSION=$(jq -r .version "$FILE")
          IFS='.' read -r MAJOR MINOR PATCH <<< "$VERSION"
          NEW_VERSION="$MAJOR.$MINOR.$((PATCH + 1))"
          jq --arg v "$NEW_VERSION" '.version = $v' "$FILE" > "$FILE.tmp" && mv "$FILE.tmp" "$FILE"
          echo "new_version=$NEW_VERSION" >> "$GITHUB_OUTPUT"

      - name: Commit and push
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add plugins/bvh-claude-code-plugin/.claude-plugin/plugin.json
          git commit -m "chore: bump plugin version to ${{ steps.bump.outputs.new_version }} [skip version]"
          git push
```

One-time setup needed on GitHub (so the bot can push):
  Repo → Settings → Actions → General → Workflow permissions → set to "Read and write permissions".

Now every time you push to main, the version in plugin.json will auto-increment. After the Action runs, refresh locally:

```markdown
/plugin marketplace update bvh-claude-code-plugins
/reload-plugins
```

## 4. Adding Other Users

Other users don't need your GitHub credentials — they authenticate with their own GitHub account, and you just grant that account access to the private repo.

1. Give each user access to the repository:
    - **A few users:** GitHub → repo → Settings → Collaborators → Add people, and invite their GitHub username or email.
    - **Many users:** Move the repo into a GitHub Organization, create a Team, and add users to it with read access to the repo.
2. Each user installs the GitHub CLI and authenticates with their own account:

    ```markdown
    ! gh auth login
    ```

3. Once they've accepted the invite and authenticated, they follow the same steps in the Installation section above — using their own GitHub account, not yours.

A shared, read-only Personal Access Token is an alternative to collaborator invites, but it's still a secret you'd need to distribute and rotate — granting access per-account is cleaner for ongoing use.
