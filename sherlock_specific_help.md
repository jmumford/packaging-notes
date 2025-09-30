# Setting Up Your Lab Python Project with `uv` on Sherlock (Stanford's computing cluster)

These instructions are aimed at getting a reproducible `.venv` for your project and making it easier to use in VSCode notebooks.  The two challenges I tend to run into are (1) `uv` doesn't use the version of python I'd like it to and (2) The `.venv` cannot be found by VSCode.  To solve problem (2) your best bet is to get the package started up to the point that you have a `.venv` and only then should you fire up the Code Server on Sherlock.

Of course there may be better ways to get things to work smoothly, this is just where I landed.  If you have tips/tricks, please share!

⚠️ Quick ChatGPT (or whatever LLM) warning:  In my experience (as of 9/29/2025) these resources do *not* always give great advice on using `uv`.  If the ever suggest activating the environment and making changes I DO NOT recommend this.  It will almost always result in a `pyproject.toml` that doesn't align with the `.venv`, which somewhat defeats the purpose of having a `pyproject.toml`.  In this case, go old school and look the help documentation for `uv` directly or get sassy with your LLM about giving better advice. 

---

## 1️⃣ Prepare Your Environment

Before opening VSCode, make sure you have a reasonable Python and GCC version available. Add the following to your `.bash_profile`:

```bash
module load python/3.12.1
module load gcc/12.4.0
```

Then reload your profile (note you don't have to run this again in the future it runs automatically whenever you start a new session):

```bash
source ~/.bash_profile
```

Verify the Python version:

```bash
which python3   # should point to the module version
python3 --version
```

> Note: Sherlock can be finicky with Python and conda. Using `python3` (from the module) instead of `python` seems to avoid conflicts.  

---

## 2️⃣ Initialize Your Project with `uv`

Create a new project in your desired directory:

```bash
cd /path/to/projects
uv init --package package-name --python $(which python3)
```

This creates:

- `pyproject.toml`  
- `src/package_name` for your modules  
- `.venv` will be created after adding packages  

If you don't want your code base to be a package, just delete the `src` directory or use `uv init not-package --python $(which python3)`
to initialize.
---

## 3️⃣ Add Packages

Install the packages you need for analysis (you can always add more later too!):

```bash
cd package-name
uv add pandas numpy seaborn matplotlib
```

### Troubleshooting Python Version Errors

If you get errors about Python version mismatch, force uv to use the correct interpreter:

```bash
uv add --python $(which python3) pandas numpy seaborn matplotlib
```

### Adding Packages from GitHub

You can also add packages directly from GitHub:

```bash
uv add --python $(which python3) git+https://github.com/jmumford/randomise-prep.git@main
```

> Note: Warnings about “Failed to hardlink files” are usually harmless.

---

## 4️⃣ Why `uv add` is Useful

`uv add` does two things for you:

1. Creates and updates the local `.venv`.  
2. Updates `pyproject.toml` so anyone can recreate your environment later.  

This makes your code reproducible without manually managing a virtual environment.  

---

## 5️⃣ Open VSCode and Select the `.venv`

1. Open the Sherlock code server and set the workspace to your project directory.  
2. Wait while Sherlock starts your interactive session.  
3. Once it begins, start the session and create a notebook or open an existing one.  
4. Click Select Kernel at the top and choose the `.venv/bin/python` kernel created by uv.  

> If VSCode doesn’t immediately recognize the `.venv`, restart the code server. Setting up `.venv` before opening VSCode usually fixes this.  

---

## 6️⃣ Running Jobs on Slurm with `.venv`

To run a script on SLURM using the environment defined in your `pyproject.toml`, use:

`uv run --project /path/to/your/project python path/to/my_script.py`

This ensures the job uses the `.venv` created by uv, even when running outside VSCode or Jupyter.  

---

## 7️⃣ Organizing Your Project

A suggested layout:

```
package-name/
├── src/package_name/      # reusable functions/modules
├── analysis_code/
│   ├── notebooks/         # .ipynb files
│   └── scripts/           # .py scripts for analysis
├── pyproject.toml
└── .venv/                 # virtual environment (created by uv)
```

- Keep reusable code in `src/package_name`.  
- Keep experiment scripts and notebooks separate in `analysis_code`.  

---

This keeps your project organized, reproducible, and ready for use in VSCode.  

## Running jobs on Slurm using the .venv

When running a job, if you want to use the environment specified in your `pyproject.toml` use
`uv run --project /path/to/your/project python path/to/my_script.py`