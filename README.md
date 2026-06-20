# How to Publish and Consume your own Claude Code plugins from a GitHub repository

## Create and clone a completely empty public GitHub repository

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

## 3. Install Plugin from your Marketplace in Claude Code

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
