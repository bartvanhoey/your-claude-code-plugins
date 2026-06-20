# How to Publish and Consume your own Claude Code plugins from a GitHub repository

## Create and clone a completely empty public GitHub repository

```bash
    git clone https://github.com/<your-username>/your-claude-code-plugins.git
    cd your-claude-code-plugins
```

## Create the marketplace plugin structure

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

## Push your changes to GitHub

```bash
    git add .
    git commit -m "Add marketplace plugin structure"
    git push origin main
```
