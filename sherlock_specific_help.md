# Setting Up Your Lab Python Project with `uv` on Sherlock

These instructions will help you create a reproducible Python environment (`.venv`) for your project and make it usable in VSCode notebooks.  

Two common pitfalls when using `uv` on Sherlock:  

1. `uv` might not use your preferred Python version.  
2. VSCode sometimes cannot detect the `.venv` automatically.  

The workflow below avoids both issues by preparing your `.venv` **before** opening VSCode.  

> ⚠️ Warning: Advice from LLMs or online resources about modifying a `uv` environment **after activation** can be misleading. Changing the environment manually may break the alignment between `.venv` and `pyproject.toml`. When in doubt, check `uv --help`.

---

## ⚡ Quick Start

1. Source your project setup (described below):
```bash
source /path/to/project/setup_uv_sherlock.sh
```

2. Initialize the project (if new):
```bash
cd /path/to/projects
uv init --package my_project
```

3. Install dependencies:
```bash
cd my_project
uv add pandas numpy matplotlib
```

4. Run your script on SLURM:
```bash
uv run --project /path/to/my_project python analysis_code/my_script.py
```

5. Open VSCode and select `.venv/bin/python` as kernel.

---
## Full Setup
---


### 1️⃣ Prepare Your Environment via `setup_uv_sherlock.sh`

Instead of modifying your `.bash_profile`, create a project-specific shell script:

```bash
#!/bin/bash
# setup_uv_sherlock.sh - load modules and configure uv

# Set default permissions: files and directories are group-readable and group-writable
# IMPORTANT: POLDRACK LAB MEMBERS should set this in their .bash_profile!
# Ensures shared projects and the central UV_CACHE are accessible to all lab members
# A coworker won't be able update a uv_cache in a shared project if you do not do this
umask 0006

# Load required modules for your work (varies by project)
module load fsl
module load python/3.12.1
module load gcc/12.4.0
module load uv

# Tell uv which Python to use (this will correspond to module loaded version)
export UV_PYTHON=$(which python3)

# Use a central uv cache for reproducibility
export UV_CACHE_DIR="/oak/stanford/groups/russpold/uv_cache_pool/uv_cache_jmumford"
```

**⚠️![Russ](./images/russ_face.png) Attention – Poldrack Lab Members:** Please set `umask 0006` in your `.bash_profile`.

**⚠️![Russ](./images/russ_face.png) Attention – Poldrack Lab Members:** You may use `/oak/stanford/groups/russpold/uv_cache_pool/uv_cache_USERNAME` as your `uv_cache` directory.  Otherwise it default to your home directory and can eat up a lot of space.  

Use it in your shell session:

```bash
source setup_uv_sherlock.sh
```

Check your Python version:

```bash
python3 --version
```

Where UV will default to python version and cache location
```bash
echo $UV_PYTHON
echo $UV_CACHE_DIR
```

---

### 2️⃣ Initialize Your Project

```bash
cd /path/to/projects
uv init --package analysis_directory_name 
```

This creates:  

- `pyproject.toml`  
- `src/analysis_directory_name` for reusable modules  
- `.venv` (after adding packages)  

> If your code is not a package, you can remove `src` or run `uv init analysis_directory_name`.

---

### 3️⃣ Install Dependencies

```bash
cd analysis_directory_name
uv add pandas numpy seaborn matplotlib
```

For GitHub packages:

```bash
uv add git+https://github.com/jmumford/randomise-prep.git@main
```

> If you see “Failed to hardlink files,” it’s usually harmless.

---

### 4️⃣ Why `uv add` is Useful

- Updates the local `.venv` automatically  
- Updates `pyproject.toml` for reproducibility  

No manual virtual environment management required.

---

### 5️⃣ Open VSCode and Select the `.venv`

1. Launch the Sherlock code server after `.venv` was make and open your project directory.  
2. Open a notebook.  
3. Select the kernel corresponding to your `.venv/bin/python`.  

> If VSCode doesn’t detect the `.venv`, restart the code server.

---

### 6️⃣ Running Scripts on SLURM

```bash
source /path/to/project/setup_uv_sherlock.sh
uv run --project /path/to/project python path/to/project/analysis/my_script.py
```

This ensures SLURM jobs use the same `.venv` as your notebooks and sourcing the `setup_uv_sherlock.sh` loads necessary modules and sets `$UV_PYTHON$` and `$UV_CACHE_DIR`.

---

### 7️⃣ Suggested Project Layout

```
package_name/
├── src/package_name/       # reusable modules
├── analysis_code/          # notebooks and scripts
├── slurm_logs              # A place to dump your slurm logs
├── pyproject.toml
├── setup_uv_sherlock.sh    # environment setup script
└── .venv/                  # virtual environment created by uv
```

- Keep reusable code in `src/`  
- Keep experiment scripts and notebooks in `analysis_code/`  
- Configure modules per project to avoid conflicts across projects  

---

