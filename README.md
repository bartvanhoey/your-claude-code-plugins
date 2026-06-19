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
The issue is that /reload-plugins doesn't fetch anything from GitHub — it only re-activates plugins from whatever is already cached locally.
You need to refresh the marketplace first.

To refresh the marketplace, run:

```markdown
  /plugin marketplace update bvh-claude-code-plugins
  /reload-plugins
```

This re-clones/pulls the latest commit from your GitHub repo and refreshes the cached marketplace +
installed plugin files.

If that still doesn't pick up the change, the installed plugin's cached copy may not refresh automatically from a marketplace update — do a clean reinstall:

```markdown
  /plugin uninstall bvh-claude-code-plugin@bvh-claude-code-plugins
  /plugin install bvh-claude-code-plugin@bvh-claude-code-plugins
  /reload-plugins
```
