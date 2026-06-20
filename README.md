## Publish & Consume Claude Code plugins from a GitHub repository

## Create and clone an empty public GitHub repository

```bash
    git clone https://github.com/<your-github-account>/your-claude-code-plugins.git
    cd your-claude-code-plugins
```

## 1. Create the marketplace plugin structure

Below is an example of how to structure your repository. The `marketplace.json` file is required and should be placed in the `.claude-plugin` directory at the root of your repository.

Each plugin should have its own directory under `plugins/`, and each plugin should have its own `.claude-plugin/plugin.json` file and a `skills/` directory containing the skill markdown files.

```text
    your-claude-code-plugins/
    ├── .claude-plugin/
    │   └── marketplace.json
    └── plugins/
        └── your-claude-code-plugin/
            ├── .claude-plugin/
            │   └── plugin.json
            └── skills/
                ├── hello-world/
                │   └── SKILL.md
                └── another-one/
                    └── SKILL.md
```

This repository itself follows the structure above. You will definitely need to change the folder names, the content of the `marketplace.json`, `plugin.json` and skills to reflect your own plugin and skills.

## 2. Push your changes to GitHub

```bash
    git add .
    git commit -m "Add marketplace plugin structure"
    git push origin main
```

## 3. Install Plugin from your Marketplace

1. Add your marketplace (run inside a Claude Code session):

    ```text
        /plugin marketplace add <your-github-account>/your-claude-code-plugins
    ```

    This clones your GitHub repo and reads .claude-plugin/marketplace.json, registering the catalog —
    nothing gets installed yet.

2. Install the plugin from it:

    ```text
        /plugin install your-claude-code-plugin@your-claude-code-plugins
    ```

    The **@your-claude-code-plugins** is the marketplace name (from name in marketplace.json), and **your-claude-code-plugin** is the plugin name (from name in plugin.json).

3. Activate it:

    ```text
        /reload-plugins
    ```

    Skills are namespaced by plugin name, so you'd then invoke them as:

    ```text
        /your-claude-code-plugin:hello-world
    ```

## 4. Push your changes to GitHub

```bash
    git add .
    git commit -m "Install plugin from marketplace"
    git push origin main
```

## 5. Bump version to refresh plugin after pushing changes

Pushing changes to the GitHub repository does not automatically refresh an already-installed plugin on your machine.

ATTENTION: Running `/plugin marketplace update your-claude-code-plugins` and `/reload-plugins` alone does not refresh an installed plugin.

The issue is that Claude Code only re-syncs an installed plugin's cache when the version field changes — marketplace update just refreshes the catalog of what's available, it doesn't force-refresh already-installed plugins at an unchanged version.

Fix: bump "version" in plugins/your-claude-code-plugin/.claude-plugin/plugin.json (e.g. 1.0.0 → 1.0.1) every time you push skill changes, then run the commands above again. That's the supported workflow for iterating on a plugin.

To do this automatically, you can create a GitHub Action named **bump-version.yml** to bump the version on every push to main:

```yaml
name: Bump plugin version

on:
  push:
    branches: [main]
    paths:
      - 'plugins/your-claude-code-plugin/**'

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
          FILE=plugins/your-claude-code-plugin/.claude-plugin/plugin.json
          VERSION=$(jq -r .version "$FILE")
          IFS='.' read -r MAJOR MINOR PATCH <<< "$VERSION"
          NEW_VERSION="$MAJOR.$MINOR.$((PATCH + 1))"
          jq --arg v "$NEW_VERSION" '.version = $v' "$FILE" > "$FILE.tmp" && mv "$FILE.tmp" "$FILE"
          echo "new_version=$NEW_VERSION" >> "$GITHUB_OUTPUT"

      - name: Commit and push
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add plugins/your-claude-code-plugin/.claude-plugin/plugin.json
          git commit -m "chore: bump plugin version to ${{ steps.bump.outputs.new_version }} [skip version]"
          git push
```

One-time setup needed on GitHub (so the bot can push):
  Repo → Settings → Actions → General → Workflow permissions → set to "Read and write permissions".

Now every time a skill has changed and you push to main, the version in plugin.json will auto-increment. After the Action runs, refresh locally:

```text
/plugin marketplace update your-claude-code-plugins
/reload-plugins
```

WARNING: Be mindful that this workflow action will create a new commit on every time it runs.
Your local repository will be out of sync with the remote after this run, so you will need to pull the changes before pushing new changes:

```bash
    git pull origin main
    # resolve any merge conflicts if necessary
    git add .
    git commit -m "Your commit message"
    git push origin main
```

## 6. Set Up Private Repository Access

Claude Code can add a marketplace from a private GitHub repo, but it needs Git access to clone it first:

1. Make the repository private on GitHub, and make sure `.claude-plugin/marketplace.json` lives at the repo root on the default branch.
2. Install the GitHub CLI (`winget install GitHub.cli`).
3. Authenticate — when prompted, also choose to authenticate Git with your GitHub credentials, so `git clone` works for the private repo over HTTPS:

```text
! gh auth login
```
