# Jeanette's guide to Python projects with `uv` (with extra help for Sherlock)

Last tested: October 2026 · `uv` 0.10.8 (Sherlock `uv` module) · `python/3.12.1`

These are my notes on setting up reproducible Python projects with [uv](https://docs.astral.sh/uv/), written for the Poldrack lab and anyone else at Stanford using Sherlock. Most of it applies to any computer. The Sherlock-specific parts are clearly marked. Feedback is welcome!

## What is uv, and why use it?

`uv` is a very fast tool for managing Python projects. Each project gets its own environment (a `.venv` folder) that lives right inside the project directory, and two files describe exactly what's in it:

- `pyproject.toml` lists the packages your project needs.
- `uv.lock` records the exact version of every package that was installed, including the packages your packages depend on.

Commit both to GitHub and anyone (including future you) can recreate the same environment with one command. You never need to "activate" anything: `uv run` uses the right environment automatically, no matter where you launch it from.

Another great reason for Sherlock users to use `uv` is the kind maintainers of Sherlock [recommend avoiding conda.](https://www.sherlock.stanford.edu/docs/software/using/anaconda/) 

> ⚠️ **A warning about LLM advice:** ChatGPT, Claude and friends don't always give great advice about `uv`. If one suggests activating the environment and installing things with `pip` (or `uv pip install`), don't. You'll end up with a `.venv` that no longer matches your `pyproject.toml`, which defeats the whole point. Look at `uv --help` or the [uv docs](https://docs.astral.sh/uv/) instead, or get sassy with your LLM about giving better advice.
>

## Coming from conda?

If you've used conda, uv will feel familiar. Both give each project its own isolated environment so projects don't interfere with each other. The big difference is *how* that environment gets built. A conda environment is a complete setup that conda builds itself, with its own copy of Python and often its own versions of system libraries. uv instead creates a lightweight *virtual environment* (the `.venv` folder) that uses Sherlock's module Python and adds only your Python packages. That's exactly the approach Sherlock recommends.

Other differences you'll notice:

- **Where environments live.** Conda keeps environments in one central place (like `~/miniconda3/envs`), separate from your code. uv puts each project's environment right in the project folder (`.venv`), so the code and its environment travel together.
- **Reproducibility.** uv automatically writes a lock file (`uv.lock`) with the exact version of every package. Plain conda doesn't. You have to remember to export an `environment.yml` yourself, which usually isn't as exact (add-on tools like conda-lock can fill the gap). `pyproject.toml` itself is a Python standard, not a uv invention, so it works with other tools too.
- **What they install.** Conda can install non-Python software too (R, compilers, CUDA, command-line tools). uv only installs Python packages. On Sherlock, use modules for non-Python software instead.
- **No activating.** Instead of activating an environment and then running commands, you put `uv run` in front of each command.

| What you want to do | The old way (conda) | The uv way |
|---|---|---|
| Create an environment | `conda create -n myenv python=3.12` ⚠️ | `uv init myproject` (the `.venv` appears with your first `uv add`) |
| Turn it on | `conda activate myenv` ⚠️ | Nothing to turn on. `uv run` uses the project's `.venv` automatically |
| Install a package | `pip install pandas` (or `conda install pandas` ⚠️) | `uv add pandas` |
| Install a local repo in editable mode | `pip install -e .` | `uv add --editable path/to/repo` |
| Remove a package | `pip uninstall pandas` | `uv remove pandas` |
| Run a script | `python my_script.py` (after activating) | `uv run python my_script.py` |
| Run tests | `pytest` (after activating) | `uv run pytest` |
| Save the environment for others | `conda env export > environment.yml` (if you remember) | Automatic: commit `pyproject.toml` and `uv.lock` |
| Recreate it somewhere else | `conda env create -f environment.yml` ⚠️ | `uv sync` |

> [!WARNING]
> **What Sherlock discourages (the ⚠️ steps).** Sherlock [recommends avoiding conda](https://www.sherlock.stanford.edu/docs/software/using/anaconda/). The problem isn't environments or `pip`. It's how conda builds things:
> - **`conda create` and `conda install`** install conda's own copies of Python and libraries instead of Sherlock's versions, which are optimized for Sherlock's hardware. Conda can even pull in its own CUDA or R, which run worse than (or conflict with) Sherlock's modules.
> - **Conda environments fill your home directory.** They write huge numbers of files to `$HOME`, and a single conda install can use up most of your quota.
> - **`conda activate` requires `conda init`,** which edits your `~/.bashrc`. Sherlock lists that as a source of conflicts and slow logins.
>
> The `pip` commands in the old column are fine by Sherlock's standards. Sherlock's recommended alternative is a virtual environment with `pip`, which is what uv does for you, with a lock file on top.

## Reading order

1. [Shell setup](01-shell-setup.md): one-time setup of your shell and uv's cache.
2. [Starting a project](02-new-project.md): creating a project, adding packages, sharing it, and running it on Slurm.
3. [VS Code, notebooks and Ruff](03-vscode.md): using your environment in notebooks and getting tidy code on save.
4. [Optional extras](04-extras.md): pre-commit hooks, tests, and GitHub Actions.
5. [Troubleshooting](05-troubleshooting.md): when things go sideways.

## How to read the callouts

> [!WARNING]
> **Sherlock workaround:** Boxes like this one describe workarounds for Sherlock's older hardware, which runs an old operating system that many current Python packages no longer support. These are temporary. When the older nodes are retired or upgraded, these steps should become unnecessary and the plain instructions will be enough.

> ![Russ](./images/russ_face.png) **Poldrack lab:** Boxes like this one are lab-specific conventions. If you're not in the lab, you can skip them.

## What about containers?

Apptainer containers are another way to get a modern Python stack on Sherlock, since a container brings its own operating system. They also make notebooks and day-to-day editing more cumbersome. Instructions are coming later.
