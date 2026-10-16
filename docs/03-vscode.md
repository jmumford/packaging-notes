# 3. VS Code, notebooks and Ruff

This page covers using your project's environment in VS Code notebooks (including code-server on Sherlock), and setting up Ruff so your code tidies itself when you save.

## Using your `.venv` in notebooks

Before you open VS Code, make sure that:

1. You've done the [shell setup](01-shell-setup.md), and started a **new** code-server session afterward.
2. Your project's `.venv` exists (you've run `uv add` or `uv sync` at least once).
3. `ipykernel` is one of your packages (`uv add ipykernel` if not).

Then:

1. Open your **project folder** as the workspace. On Sherlock, open the project directory itself when you launch code-server, not a folder above it. On a laptop, `cd` into the project and run `code ./`.
2. Open or create a notebook (`.ipynb`).
3. Click **Select Kernel** (top right) and choose the one that points to your project's `.venv/bin/python`.

If your `.venv` doesn't show up, see [Troubleshooting](05-troubleshooting.md#vs-code-doesnt-find-my-venv).

## Ruff: tidy code on save

[Ruff](https://docs.astral.sh/ruff/) is a very fast linter and formatter. With the settings below, every time you save a file (or run a notebook cell), it fixes formatting, sorts your imports, removes unused imports, and underlines problems it can't fix for you to look at.

- **Linting** catches problems that might keep your code from working (like a variable you never defined).
- **Formatting** makes your code look consistent and pretty.
  

### Step 1: Install Ruff and the VS Code Ruff extension

You only need to do this once, not for every new project.

First, install Ruff as a command-line tool:

    uv tool install ruff

This installs Ruff for you personally rather than adding it to any project. To check that it worked, run `which ruff`. It should print `~/.local/bin/ruff` (the full path to your home directory may appear instead of `~`).

Then open the Extensions panel (the four-squares icon on the left), search for **Ruff**, and install the one published by Astral.


### Step 2: Turn on fix-on-save for this workspace

Open the settings with `Cmd + ,` (Mac) or `Ctrl + ,` (Linux/Windows) and click the **Workspace** tab (not **User**). Then click the small page icon in the upper right to open the settings file:

![page_icon](./images/page_icon.png)

This creates `.vscode/settings.json` in your project. Put this in it:

```json
{
    "notebook.formatOnSave.enabled": true,
    "notebook.formatOnCellExecution": true,
    "notebook.defaultFormatter": "charliermarsh.ruff",
    "notebook.codeActionsOnSave": {
        "notebook.source.fixAll": "explicit",
        "notebook.source.organizeImports": "explicit"
    },
    "[python]": {
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "charliermarsh.ruff",
        "editor.codeActionsOnSave": {
            "source.fixAll": "explicit",
            "source.organizeImports": "explicit"
        }
    }
}
```

`fixAll` is the part that removes unused imports and applies other automatic fixes. `organizeImports` sorts your imports.

> [!WARNING]
> **Sherlock workaround:** On Sherlock, the Ruff extension can't find Ruff on its own in code-server, so tell it where Ruff lives. Add this line right after the opening `{`, replacing `YOURSUNETID` with your SUNet ID:
> ```json
> "ruff.path": ["/home/users/YOURSUNETID/.local/bin/ruff"],
> ```
> Not on Sherlock? Leave this line out. Ruff finds itself on its own elsewhere, and this path won't exist on your computer.

Why Workspace and not User? I often work on other people's code, and I don't want my preferences rewriting their files. Putting the settings in the workspace keeps them to projects where I chose to use them.

### Step 3: Keep `.vscode` out of git

Add this to your project's `.gitignore`:

```
# VS Code settings
.vscode/
```

This keeps your personal editor settings out of the shared repository. The catch: because `.vscode` isn't committed, every new project (and every fresh clone) needs Step 2 again. If Ruff suddenly isn't fixing things on save, a missing `.vscode/settings.json` is the first thing to check.

### Step 4 (optional): Choose Ruff's rules

Ruff works without any configuration. Its defaults already catch unused imports and the most common errors. If you want to customize it, add a `[tool.ruff]` section to `pyproject.toml`. These are my preferences:

```toml
[tool.ruff.format]
quote-style = "single"

[tool.ruff.lint]
extend-select = ["E501", "W", "N", "R", "I"]
```

This uses single quotes (I hate double quotes) and adds rules for line length (`E501`), warnings (`W`), naming conventions (`N`), refactoring suggestions (`R`), and import sorting (`I`). The full list of rules is [here](https://docs.astral.sh/ruff/rules/).

Unlike `.vscode`, these settings *are* committed with your project, so collaborators get the same rules. (Older guides, including an earlier version of this one, add `extend-include = ["*.ipynb"]` so Ruff checks notebooks. Current versions of Ruff check notebooks by default, so you no longer need it.)

If you're contributing to someone else's project, use their Ruff settings rather than adding yours.

### Step 5: Check that it works

Open a Python file in your project, add an import you don't use, and some sloppy spacing:

```python
import os

junk="22"
```

Save the file. The `import os` line should disappear, and `junk="22"` should become `junk = '22'` (single quotes if you used my settings). If nothing happens, see [Troubleshooting](05-troubleshooting.md#ruff-doesnt-fix-anything-on-save).

Next: [Optional extras](04-extras.md)
