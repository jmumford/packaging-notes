# 5. Troubleshooting

Often, with a brand new project, if something isn't working from within the code server, killing that session and starting a new one can sometimes help.  Thankfully, once things start working you're less likely to encounter problems in the future with that project.

## An install hangs or fails while building

**What you see:** `uv add` or `uv sync` sits on one package for a very long time, or fails with a wall of compiler errors (often mentioning missing header files like `jpeglib.h`).

**What's going on:** Most packages come prebuilt, so installing just means downloading and unpacking. Sherlock's older nodes run an old operating system, though, and many current packages only publish prebuilt versions for newer systems. When there's no prebuilt version that works, uv downloads the source code and tries to build the package itself. That can take a long time (a "hang" is usually a build in progress), and sometimes it fails because the system is missing pieces the build needs.

**What to do:**

1. Make sure you're in a dev session with several cores (`sdev -c 8 ...`), not on a login node, and that `gcc/12.4.0` is loaded (`module list`).
2. To see whether uv is building something, run this in another terminal:
   ```bash
   ps -u $USER | grep -E "cc1|gcc|meson|cython"
   ```
   If anything shows up, a build is in progress.
3. To find out *which* packages have no usable prebuilt version, press `Ctrl-C` and rerun the same command with `--no-build` added, for example `uv sync --no-build`. Instead of building, uv stops right away and names the package.
4. Pin that package to an older version that does have a usable prebuilt version, or to a version from the [tested list](02-new-project.md#step-5-add-packages). This works even if the package isn't one you asked for directly (one of your packages needs it):
   ```bash
   uv add "pyzmq==27.2.0"
   ```

**Example: pyzmq.** `pyzmq` gets pulled in by `ipykernel` (which you need for notebooks). If its install fails, adding it explicitly as `uv add "pyzmq==27.2.0"` installed successfully in testing.

## `uv` is using the wrong Python

**What you see:** `echo $UV_PYTHON` shows `/usr/bin/python3` (or something under `miniconda3`), or `$UV_PYTHON --version` is not 3.12.

**What to do:**

- Make sure `export UV_PYTHON=$(which python3)` comes *after* `module load python/3.12.1` in your `~/.bashrc` ([shell setup](01-shell-setup.md#step-2-add-your-settings-to-bashrc)), then log out and back in.
- If your prompt starts with `(base)`, an old conda setup is activating itself when you log in. Conda isn't recommended on Sherlock, and you don't need it with uv. To turn off the auto-activation, run `conda config --set auto_activate_base false`, or make sure the uv lines in `~/.bashrc` come after the conda block.

## VS Code doesn't find my `.venv`

**What you see:** your `.venv` doesn't appear when you click **Select Kernel**, and/or the Python extension shows an error like:

```
ModuleNotFoundError: No module named 'importlib.metadata'
```

That error means VS Code is running on a very old Python (older than 3.8), which is Sherlock's system Python.

**What's going on:** code-server gets its environment when it starts. If `python/3.12.1` wasn't loaded at that moment, VS Code only ever sees the ancient system Python, and that breaks both the Python extension and its ability to find your `.venv`.

**What to do:**

1. Make sure `module load python/3.12.1` is in your `~/.bashrc` ([shell setup](01-shell-setup.md#step-2-add-your-settings-to-bashrc)).
2. Start a **new** code-server session. Reloading the window or restarting inside the old session doesn't help.
3. Make sure you opened the **project directory itself** as the workspace, not a folder above it.
4. Make sure the `.venv` exists and `ipykernel` is installed (`uv add ipykernel`).

Things that did *not* fix it (and cost an afternoon): reloading the window, restarting the session without changing `~/.bashrc`, and manually registering a kernel with `python -m ipykernel install --user`. The kernel registered fine, and VS Code still ignored it.



## Ruff doesn't fix anything on save

Check these in order:

1. **Is the Ruff extension installed in code-server?** code-server's extensions are separate from VS Code on your laptop.
2. **Does this project have `.vscode/settings.json`?** It's ignored by git, so new projects and fresh clones won't have it. Redo [Step 2 of the Ruff setup](03-vscode.md#step-2-turn-on-fix-on-save-for-this-workspace), and make sure you used the **Workspace** tab.
3. **Is Ruff reporting errors?** Open **View → Output** and choose **Ruff** from the dropdown. If the errors mention Python or `importlib`, it's the same old-Python problem as [VS Code not finding your `.venv`](#vs-code-doesnt-find-my-venv), with the same fix.

## No `.git` or `.gitignore` after `uv init`

`uv init` normally creates both. It skips them in two situations:

1. **Your new project is inside an existing git repository** (for example, a folder above it is a repository or a DataLad dataset). uv won't create a repository inside another one. To check, run this in the folder *above* your project:
   ```bash
   git rev-parse --show-toplevel
   ```
   If it prints a path, that's the cause. Either create your project somewhere else, or keep it inside that repository on purpose.
2. **uv can't find a usable git.** Check with `which git` and `git --version`. Sherlock's system git is very old. You can add `module load git` to your `~/.bashrc` to get a newer one.

To add them yourself afterward, run `git init` in your project, and create a `.gitignore` containing what uv normally puts there:

```
# Python-generated files
__pycache__/
*.py[oc]
build/
dist/
wheels/
*.egg-info

# Virtual environments
.venv
```

## "Failed to hardlink files" warning

This is harmless. It means uv's cache and your project are on different filesystems, so uv copies files instead of linking them. Set `export UV_LINK_MODE=copy` in your `~/.bashrc` to make it go away ([shell setup](01-shell-setup.md#step-2-add-your-settings-to-bashrc)).

## My cache got purged from scratch

Nothing is broken. Your projects' `.venv` folders aren't on scratch, so they keep working. The next time you add or sync packages, uv rebuilds its cache, so that install will be slower. Make sure the cache directory exists again first (`mkdir -p $SCRATCH/uv_cache`).

## Moving a project between computers

`uv.lock` is designed to work across systems. uv works out versions that should install on Mac, Linux and Windows. What it can't promise is that every locked version has a prebuilt version that works on *every* computer. So a lock can install fine on one machine and try to build (or fail) on another.

- **Locks made on Sherlock** should usually install on newer computers, because Sherlock is the pickiest system.
- **Locks made on a newer computer** (a Mac, a recent Linux machine) may pick versions that won't install on Sherlock. If that happens, add the [tested Sherlock versions](02-new-project.md#step-5-add-packages) on Sherlock, and commit the new `uv.lock`.
- On whichever computer fails, run `uv sync --no-build` to see exactly which package is the problem.

## My `.venv` doesn't match `pyproject.toml` anymore

This usually happens after installing something with `pip` or `uv pip install` inside an activated environment. Start the environment over from the lock:

```bash
rm -rf .venv
uv sync
```

Then add the package properly with `uv add`.
