# BVH Claude Code Plugins

The Bvh Claude Code Plugins allows you to use Claude Skills, Agents and more in your conversations with Claude.

## Installation

1. Add your marketplace (run inside a Claude Code session):

    ```markdown
    /plugin marketplace add bartvanhoey/bvh-claude-code-plugins
    ```

    This clones your GitHub repo and reads .claude-plugin/marketplace.json, registering the catalog —
    nothing gets installed yet.

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

## Update Repository

To update the repository, simply push your changes to the GitHub repository does not automatically refresh the plugin on your machine.

ATTENTION: Running the commands /plugin marketplace update bvh-claude-code-plugins and /reload-plugins do not work yet.
The issue is that Claude Code only re-syncs an installed plugin's cache when the version field changes — marketplace update just refreshes the catalog of what's available, it doesn't force-refresh already-installed plugins at an unchanged version.

Fix: bump "version" in plugins/bvh-claude-code-plugin/.claude-plugin/plugin.json (e.g. 1.0.0 →   1.0.1) every time you push skill changes, then run /plugin marketplace update again. That's the supported workflow for iterating on a plugin.

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

Now every time you push to main, the version in plugin.json will auto-increment, and you can then run /plugin marketplace update to refresh the marketplace and pick up the new version.

To refresh the marketplace, run:

```markdown
    /plugin marketplace update bvh-claude-code-plugins  
    /reload-plugins
```
