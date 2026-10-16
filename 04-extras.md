# 4. Optional extras

None of this is required. These tools are most useful for packages that other people will install or build on. For a single analysis, feel free to skip this page.

I should add that I did test this a year or so ago (early 2025), but I have not tested these instructions recently. 

## Installing command-line tools with `uv tool`

Some tools (like pre-commit) are things *you* use, not packages *your project* depends on. Install them once with `uv tool`, and they're available everywhere:

```bash
uv tool install pre-commit
uv tool install ruff          # if you want Ruff on the command line too
```

To run a tool once without installing it, use `uvx` (short for `uv tool run`):

```bash
uvx ruff check .
```

`uvx` runs the tool in a temporary environment containing just that tool. If the command needs your project's packages (like `pytest` does), use `uv run` instead.

## Pre-commit hooks

Pre-commit hooks run checks (like Ruff) on your files every time you `git commit`. If a check fails, the commit stops so you can fix things first.

Hooks live in your local `.git` folder, so they're a personal helper rather than something you can require of everyone working on a project. If you're contributing to someone else's project and they have a `.pre-commit-config.yaml`, it's polite to turn it on.

### Set it up

In your project's root directory:

```bash
pre-commit install            # prints: pre-commit installed at .git/hooks/pre-commit
```

Then create `.pre-commit-config.yaml` in the project's root with:

```yaml
repos:
  - repo: https://github.com/astral-sh/uv-pre-commit
    rev: 0.5.9
    hooks:
      # Make sure uv.lock matches pyproject.toml
      - id: uv-lock

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.8.3
    hooks:
      # Run the linter and fix what it can
      - id: ruff
        args: [--fix]
      # Run the formatter
      - id: ruff-format
```

The `rev` lines pin each hook to a version, and those versions get old quickly. Update them all to the latest with:

```bash
pre-commit autoupdate
```

### Use it

- Hooks run automatically when you commit, but only on files that are staged for that commit.
- To run them on everything without committing: `pre-commit run --all-files`
- To commit without running them: `git commit -m "message" --no-verify`

### Optional: pyright

[Pyright](https://github.com/microsoft/pyright) checks your code for type errors, sort of like a spell checker for Python. To add it as a hook, add this to `.pre-commit-config.yaml`:

```yaml
  - repo: https://github.com/RobertCraigie/pyright-python
    rev: v1.1.391
    hooks:
      - id: pyright
```

And this to `pyproject.toml`, so pyright uses your project's environment:

```toml
[tool.pyright]
exclude = [".venv"]
venvPath = "."
venv = ".venv"
```

Then run `pre-commit autoupdate` to update its version too.

## Tests with pytest

Tests check that your functions do what you think they do. They're most worth it for reusable code in `src/` that other people (or your future self) will rely on. For one-off analysis code, a few `assert` lines (like checking you have the number of rows you expect, or no missing values where there shouldn't be any) catch a lot of mistakes with much less effort.

### Set it up

```bash
uv add --dev pytest
mkdir tests
```

Tell pytest where your tests are by adding this to `pyproject.toml`:

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
```

### Write a test

Say you have a function in `src/myproject/some_things.py`:

```python
def return_boo():
    return 'boo'
```

Create `tests/test_some_things.py`:

```python
from myproject.some_things import return_boo


def test_return_boo():
    assert return_boo() == 'boo'
```

Test files need to start with `test_` (or end with `_test.py`), and test functions need to start with `test_`, so pytest can find them. (Importing from `myproject` like this works in a [package](02-new-project.md#step-3-choose-a-simple-project-or-a-package).)

### Run the tests

```bash
uv run pytest
```

## Run your tests on GitHub automatically

A GitHub Action can run your tests every time you push to GitHub. Create the workflow file:

```bash
mkdir -p .github/workflows
touch .github/workflows/run_tests.yaml
```

Put this in `run_tests.yaml`:

```yaml
name: Run tests

on:
  push:
    branches:
      - main

jobs:
  uv-run-tests:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v6   # check the setup-uv README for the latest version

      - name: Install the project
        run: uv sync --locked

      - name: Run tests
        run: uv run pytest
```

`uv sync --locked` fails if `uv.lock` doesn't match `pyproject.toml`, which catches the case where someone edited one without updating the other.

Commit and push, then open the **Actions** tab of your repository on GitHub to watch it run. Shazam!

If the push fails with an error about your token not allowing workflows, your GitHub token needs the `workflow` permission. Create a new token that includes it, or re-run `gh auth login`.
