# 2. Starting a project

This page walks through creating a project, adding packages, sharing it with others, and running your code on Slurm.

## Step 1: Go to where your project will live

Keep projects on Oak, not in your home directory.

> ![Russ](./images/russ_face.png) **Poldrack lab:** Use your user directory (you may need to create it first):
> ```bash
> cd /oak/stanford/groups/russpold/users/$USER
> ```

## Step 2: Choose a simple project or a package

uv can set up two kinds of project. Both are fine. Pick the one that fits how you work.

| | Simple project | Package |
|---|---|---|
| Command | `uv init myproject` | `uv init --package myproject` |
| Your own code | Scripts and notebooks | Reusable functions in `src/myproject/`, plus scripts and notebooks |
| Importing your own functions | Only from files in the same folder | From anywhere in the project: `from myproject.utils import calc_values` |
| Good for | Getting started, small analyses | Analyses where you reuse your own functions across many scripts and notebooks |

I personally use packages for almost everything, because I almost always have modules with functions I'll use over and over in different analyses.

A note on names: Dashes are fine in your project name (`uv init --package my-analysis`). Python doesn’t allow dashes in import names, so uv automatically names the code folder with underscores (`src/my_analysis/`), and that’s what you use when importing (`from my_analysis.utils import calc_values`). Dashes are also common for GitHub repository names, so `my-analysis` works everywhere.


## Step 3: Create the project

For Sherlock users, the commands below automatically use the version of Python you set as `UV_PYTHON` in your `.bashrc`. I highly recommend sticking with the Sherlock-supported versions of Python.

**Not on Sherlock?** uv picks a Python on its own, which might be the newest version on your computer. To choose a version, add `--python` (e.g., `--python 3.12`) to the `uv init` command below.

Be kind to your collaborators and avoid the very newest Python. The version you choose becomes your project's minimum required version, so anyone on an older Python (including Sherlock) can't use your project. Brand-new versions also often don't have prebuilt packages yet, which makes installs slow or fail. A version that's been out for a year or so is a safe choice.

```bash
uv init --package myproject   # drop --package if you want a simple project
cd myproject
```

Check what was created:

```bash
ls -a
cat .python-version
```

You should see `pyproject.toml`, `.python-version`, `README.md`, a `.git` folder, and a `.gitignore`. A simple project also gets a `main.py`. A package gets `src/myproject/__init__.py` instead. I really like the `.gitignore` uv makes. It's a great starting point (it already ignores `.venv` and Python's junk files).

If `.git` and `.gitignore` are missing, make sure the git module is loaded (`module load git` in your `.bashrc`), then see [Troubleshooting](05-troubleshooting.md#no-git-or-gitignore-after-uv-init).

## Step 4: Add packages

The first time you add packages, uv creates the `.venv` folder. Every time you add a package, uv updates `pyproject.toml` (the packages you asked for) and `uv.lock` (the exact versions of those packages and everything they depend on). Committing these files to your repository, along with `.python-version`, is what makes your code reproducible.  Although you can manually edit the `pyproject.toml` and run `uv sync` to generate the `.venv`, etc., using `uv add` is a less prone to error.

> [!WARNING]
> **Sherlock workaround:** Many current Python packages no longer support the operating system on Sherlock's older nodes. Start from this tested list of versions, which install successfully on Sherlock. Hopefully these will meet most of your needs. `ipykernel` is a must-have if you're going to work with notebooks (`.ipynb` files).
> ```bash
> uv add "numpy==2.2.6" "scipy==1.16.3" "scikit-learn==1.7.2" \
>        "pandas==2.3.2" "pillow==12.2.0" "matplotlib==3.11.2" \
>        "nibabel>=5.4.2" "nilearn>=0.14.0,<0.15" \
>        "joblib>=1.3" "filelock" "ipykernel"
> ```
> Then add anything else you need as usual. If an install seems to hang or fails while building something, see [Troubleshooting](05-troubleshooting.md#an-install-hangs-or-fails-while-building).

### Login node or dev session?

Installing usually takes just a few minutes, and it's often fine to run `uv add` or `uv sync` on a login node. If an install seems slow (more than about 5 minutes), or you're setting up a big new project (a lot of Python packages), stop it with `Ctrl-C` and rerun it in a dev session with several cores:

```bash
sdev -c 8 -m 16GB -t 01:00:00
```

> ![Russ](./images/russ_face.png) **Poldrack lab:** Add `-p russpold` to use the lab partition.

> [!WARNING]
> **Sherlock workaround: why installs can be slow, and why cores help.** uv's speed comes from downloading and unpacking prebuilt packages (called "wheels"). On most computers, that's all an install is, so it takes seconds.
>
> Sherlock's older nodes run an old operating system, and many current packages no longer publish prebuilt versions for it. When uv can't find one that works, it downloads the source code and compiles the package with `gcc`, which is slower. That's why `gcc/12.4.0` is in your shell setup. The tested list above mostly avoids this: in testing, only `pyzmq` and `contourpy` had to be compiled. uv prints a `Built ...` line for anything it compiles, so you can see for yourself.
>
> uv also downloads, unpacks and copies files in parallel, using as many cores as it can see. A dev session only gives uv the cores you ask for (`sdev` gives you 1 by default), so ask for several. In testing, a first install of the list above took about 3 minutes with 12 cores and about 12 minutes with 4.
>
> The older hardware is being retired, and Sherlock staff have mentioned an OS upgrade with no timeline yet. Once that happens, these workarounds should no longer be needed, and uv should be as fast here as anywhere else (as it was a year ago).

### Adding packages in general
(Note, the first example won't work in current Sherlock for pandas and seaborn...)
```bash
uv add pandas seaborn             # add packages 
uv add "nilearn>=0.14,<0.15"      # add with a version range
uv remove seaborn                 # remove a package
uv add --dev pytest               # tools you only need while developing
```

Two other handy ways to add packages:

```bash
# Straight from GitHub, pinned to a specific commit so it can't change under you
uv add git+https://github.com/jmumford/randomise-prep.git@6c73254

# A local clone you're editing (e.g. a fork you're fixing for a pull request).
# Changes to that code show up immediately, without reinstalling.
uv add --editable path/to/local/clone/
```

Each `uv add` updates `pyproject.toml`, `uv.lock` and `.venv` together, so they always match.

> **The one rule:** Never activate the `.venv` and install packages with `pip` or `uv pip install`. Always use `uv add` and `uv remove`. Otherwise your `.venv` stops matching your `pyproject.toml` and `uv.lock`, and nobody else can recreate your environment.
>


## Step 5: Check that it works

```bash
uv run python -c "import numpy, scipy, sklearn, pandas, matplotlib, PIL, nibabel, nilearn; print('ok')"
```

If it prints `ok`, you're set.

## Step 6: Commit `uv.lock` and put it on GitHub

`pyproject.toml` only says which packages you asked for and which version ranges are acceptable. `uv.lock` records the *exact* version of every package that was installed, including all the packages your packages depend on. The lock is what makes your environment reproducible, so commit it along with `pyproject.toml` and `.python-version`.

```bash
git add -A
git commit -m "initial commit"
```

To put it on GitHub with the [GitHub CLI](https://cli.github.com/):

```bash
gh auth login                                          # first time only
gh repo create my-project --private --source=. --push
```

Or create an empty repository on the GitHub website and then:

```bash
git remote add origin https://github.com/YOURNAME/my-project.git
git branch -M main
git push -u origin main
```

## Step 7: Recreating the environment (collaborators, new computers)

The `.venv` folder is never committed to GitHub. Anyone who clones your project recreates it with:

```bash
git clone https://github.com/YOURNAME/my-project.git
cd my-project
uv sync
```

`uv sync` installs exactly what `uv.lock` says.

A lock created on Sherlock should usually work on newer computers too (like a Mac or a modern Linux machine), since Sherlock is the pickiest system. If `uv sync` fails on another computer, see [Troubleshooting](05-troubleshooting.md#moving-a-project-between-computers).

## Step 8: Updating packages

Day to day, `uv add` and `uv sync` won't upgrade anything you already have. To deliberately move everything to the newest allowed versions:

```bash
uv lock --upgrade
uv sync
```

> [!WARNING]
> **Sherlock workaround:** Be careful with `uv lock --upgrade` on Sherlock. Newer versions are exactly the ones most likely not to install on the older nodes. If you upgrade and something breaks, `git checkout uv.lock pyproject.toml` and `uv sync` take you back.

## Step 9: Running code on Slurm

First run `uv sync` once in an interactive session, so the `.venv` is up to date. Then, in your job script:

```bash
#!/bin/bash
#SBATCH --job-name=my_analysis
#SBATCH --time=01:00:00
#SBATCH --cpus-per-task=1
#SBATCH --mem=8G
#SBATCH --output=slurm_logs/%x_%j.out

uv run --no-sync --project /path/to/myproject python /path/to/myproject/analysis_code/my_script.py
```

- `--project` tells uv which project's `.venv` to use, so the job works no matter which directory it starts in.
- `--no-sync` tells uv to just use the existing `.venv` rather than checking it first. That's faster, and it avoids lots of jobs trying to update the same environment at the same time (which matters for job arrays).
- Your modules and uv settings come along automatically, because `~/.bashrc` runs when the job starts.
- Make the `slurm_logs` folder before submitting, or Slurm won't be able to write the log.

## Suggested project layout

**Simple project:**

```
myproject/
├── analysis_code/        # notebooks and scripts
├── slurm_logs/           # where Slurm logs go
├── pyproject.toml
├── uv.lock
├── .python-version
└── .venv/                # created by uv, never committed
```

**Package:**

```
myproject/
├── src/myproject/        # reusable functions (importable from anywhere in the project)
├── analysis_code/        # notebooks and scripts
├── slurm_logs/           # where Slurm logs go
├── pyproject.toml
├── uv.lock
├── .python-version
└── .venv/                # created by uv, never committed
```

In a package, a notebook in `analysis_code/` can use `from myproject.utils import calc_values` to import a function from `src/myproject/utils.py`.

Next: [VS Code, notebooks and Ruff](03-vscode.md)
