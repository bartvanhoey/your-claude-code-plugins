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
