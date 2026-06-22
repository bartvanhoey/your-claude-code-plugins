# Claude Code Plugin Marketplace on GitHub

A step-by-step guide to publishing Claude Code plugins from your own GitHub repository and installing them via a self-hosted marketplace — including versioning, private repos, and multi-user access.

## Create and Clone an Empty Public GitHub Repository

```bash
    git clone https://github.com/<your-github-account>/your-claude-code-plugins.git
    cd your-claude-code-plugins
```

## 1. Create the Marketplace Plugin Structure

Below is an example of how to structure your repository. The [`marketplace.json`](.claude-plugin/marketplace.json) file is required and should be placed in the `.claude-plugin` directory at the root of your repository.

Each plugin should have its own directory under `plugins/`, and each plugin should have its own [`.claude-plugin/plugin.json`](plugins/your-claude-code-plugin/.claude-plugin/plugin.json) file and a `skills/` directory containing the skill markdown files.

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

## 2. Push Your Changes to GitHub

```bash
    git add .
    git commit -m "Add marketplace plugin structure"
    git push origin main
```

## 3. Install Plugin from Your Marketplace

1. Add your marketplace (run inside a Claude Code session):

    ```text
        /plugin marketplace add <your-github-account>/your-claude-code-plugins
    ```

    This clones your GitHub repo and reads .claude-plugin/marketplace.json, registering the catalog — nothing gets installed yet.

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

## 4. Bump Version to Refresh Plugin after Pushing Changes

Pushing changes to the GitHub repository does not automatically refresh an already-installed plugin on your machine.

> [!WARNING]
> Running `/plugin marketplace update your-claude-code-plugins` and `/reload-plugins` alone does not refresh an installed plugin.

The issue is that Claude Code only re-syncs an installed plugin's cache when the version field changes — marketplace update just refreshes the catalog of what's available, it doesn't force-refresh already-installed plugins at an unchanged version.

Fix: bump "version" in plugins/your-claude-code-plugin/.claude-plugin/plugin.json (e.g. 1.0.0 → 1.0.1) every time you push skill changes, then push again (Step 2) and run `/plugin marketplace update` + `/reload-plugins` to refresh. That's the supported workflow for iterating on a plugin.

To do this automatically, you can create a GitHub Action named [**bump-version.yml**](.github/workflows/bump-version.yml) to bump the version on every push to main:

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

> [!WARNING]
> Be mindful that this workflow action will create a new commit on every time it runs.
> Your local repository will be out of sync with the remote after this run, so you will need to pull the changes before pushing new changes:

```bash
    git pull origin main
    # resolve any merge conflicts if necessary
    git add .
    git commit -m "Your commit message"
    git push origin main
```

## 5. Set Up Private Repository Access

Claude Code can add a marketplace from a private GitHub repo, but it needs Git access to clone it first:

1. Make the repository private on GitHub, and make sure `.claude-plugin/marketplace.json` lives at the repo root on the default branch.
2. Open a terminal and install the GitHub CLI (`winget install GitHub.cli`).
3. Authenticate — when prompted, also choose to authenticate Git with your GitHub credentials, so `git clone` works for the private repo over HTTPS:

    ```bash
        gh auth login
    ```

4. Now you can update the marketplace in Claude Code as before, using the same command but with your private repo

    ```text
        /plugin marketplace update your-claude-code-plugins
        /reload-plugins
    ```

## 6. Add Other Users

Other users don't need your GitHub credentials — they authenticate with their own GitHub account, and you just grant that account access to the private repo.

1. Give each user access to the repository:
    - **A few users:** GitHub → repo → Settings → Collaborators → Add people, and invite their GitHub username or email.
    - **Many users:** Move the repo into a GitHub Organization, create a Team, and add users to it with read access to the repo.
2. Each user installs the GitHub CLI and authenticates with their own account:

    ```bash
        gh auth login
    ```

3. Once they've accepted the invite and authenticated, they follow the same steps in the "Install Plugin from Your Marketplace" section above — using their own GitHub account, not yours.

A shared, read-only Personal Access Token is an alternative to collaborator invites, but it's still a secret you'd need to distribute and rotate — granting access per-account is cleaner for ongoing use.

## 7. Set Up Pull-Before-Push Safety Hooks

This repo ships two layers of protection that pull the latest changes before any `git push`, so you never push against a stale branch:

1. **Git-native hook** (`.githooks/pre-push`) — runs for *any* push, from any tool or terminal. It runs `git pull`; if that fails (conflicts) or brings in new commits, it aborts the push so you can resolve and re-run it against the now up-to-date branch. (A clean merge can't be pushed in the same invocation - git resolves what to push *before* the hook runs, so it always needs a re-run.)
2. **Claude Code hook** (`.claude/settings.json` → `PreToolUse`) — fires *before* Claude's `Bash`/`PowerShell` `git push` command even starts, so it can pull/merge first and then let that fresh `git push` go out already up to date - it only blocks if the pull itself fails (conflicts).

### Activating the git-native hook

Git only runs hooks from `.githooks/` once it's told where to look. A `SessionStart` hook (`.claude/hooks/configure-git-hooks-path.sh`) does this automatically the first time you open this repo in Claude Code:

```bash
    git config core.hooksPath .githooks
```

If you're using plain `git` without Claude Code, run that command once after cloning.

> [!WARNING]
> On Windows, `core.fileMode` typically defaults to `false`, so `chmod +x .githooks/pre-push` won't be reflected in the committed file mode. Make sure the hook is tracked as executable, otherwise git silently skips it on Linux/macOS clones:
>
> ```bash
>     git update-index --chmod=+x .githooks/pre-push
> ```

### Files involved

```text
    .githooks/
    └── pre-push                          # git-native pre-push hook
    .claude/
    ├── settings.json                     # wires up the hooks below
    └── hooks/
        ├── git-push-pull-check.sh        # PreToolUse: pull before Claude's `git push`
        └── configure-git-hooks-path.sh   # SessionStart: sets core.hooksPath automatically
```

`git-push-pull-check.sh` needs `bash` and `jq` on `PATH`. Both ship with Git for Windows; on Linux/macOS install `jq` separately if it's missing (`apt install jq` / `brew install jq`) — without it the check just fails open (silently skips the pull, push proceeds normally).

## 8. Remind Consuming Apps About Marketplace Updates

Pushing new commits to this marketplace repo doesn't notify anyone who already has it installed — they only find out by remembering to run `/plugin marketplace update your-claude-code-plugins` themselves. A `SessionStart` hook in the **consuming app** closes that gap: every time someone opens that project in Claude Code, it quietly checks whether the cached marketplace is behind `origin`, and only speaks up when there's actually something new.

### Add the hook to the consuming app

Add to the consuming app's `.claude/settings.json` — check it in so every teammate who clones the app gets the reminder automatically, or put the same snippet in `~/.claude/settings.json` instead if you want it to apply to every project on just your own machine:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/check-marketplace-update.sh"
          }
        ]
      }
    ]
  }
}
```

`.claude/hooks/check-marketplace-update.sh`:

```bash
    #!/bin/sh
    # SessionStart hook: reminds you to update the your-claude-code-plugins
    # marketplace if the cached copy is behind origin. Stays silent when current.

    MP_DIR="$HOME/.claude/plugins/marketplaces/your-claude-code-plugins"
    [ -d "$MP_DIR/.git" ] || exit 0

    git -C "$MP_DIR" fetch --quiet origin 2>/dev/null

    branch=$(git -C "$MP_DIR" symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null | sed 's#^origin/##')
    [ -n "$branch" ] || exit 0

    behind=$(git -C "$MP_DIR" rev-list --count "HEAD..origin/$branch" 2>/dev/null)
    [ -n "$behind" ] && [ "$behind" -gt 0 ] || exit 0

    jq -n --arg n "$behind" \
        '{systemMessage: ("⚡ your-claude-code-plugins marketplace has " + $n + " new commit(s) available. Run: /plugin marketplace update your-claude-code-plugins then /reload-plugins")}'
```

Make it executable so the mode is respected on Linux/macOS clones too (same `core.fileMode` caveat as Section 7):

```bash
    git update-index --chmod=+x .claude/hooks/check-marketplace-update.sh
```

### Why a SessionStart hook instead of a git hook

It's tempting to wire this up as a `post-merge` hook in `.git/hooks/` of the consuming app instead, so it fires right after `git pull`. Two problems rule that out:

- `.git/hooks/` isn't version-controlled — it only exists on whoever's machine set it up, and won't follow the repo to a fresh clone or a teammate's machine.
- It's the wrong signal — pulling the *consuming app's* own commits has nothing to do with whether the marketplace repo changed upstream. It would nag on every pull regardless of whether the marketplace actually changed, and it stays silent forever if the team uses `git pull --rebase` (no merge commit, so `post-merge` never fires).

A `SessionStart` hook checks the real thing — the marketplace's own git history — at the moment it actually matters (before you start using its plugins), and when committed to `.claude/settings.json`, it propagates to every clone the normal way: through version control.

### Files involved

```text
    .claude/
    ├── settings.json                       # wires up the hook below
    └── hooks/
        └── check-marketplace-update.sh     # SessionStart: warns if the marketplace is behind origin
```

`check-marketplace-update.sh` needs `git` and `jq` on `PATH` — both ship with Git for Windows; on Linux/macOS install `jq` separately if it's missing (`apt install jq` / `brew install jq`).
