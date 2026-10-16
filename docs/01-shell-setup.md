# 1. Shell setup (one time)

You only need to do this once. It tells uv which Python to use, where to keep its cache, and loads the tools uv needs every time you log in.

## Step 1: Create a cache directory

### What the cache is

Every time uv downloads (or builds) a package, it keeps a copy in its **cache**. The next time any of your projects needs that same package, uv grabs it from the cache instead of downloading or building it again. That's a big part of why uv is so fast.

The cache is not your environment. Each project's actual environment lives in its own `.venv` folder. The cache is just uv's stockpile. You can delete it at any time, and the only cost is that the next install will be slower while uv rebuilds it.

### Where to put it

By default, the cache goes in your home directory. On Sherlock, that fills up your home quota quickly, so put it on scratch instead:

```bash
mkdir -p $SCRATCH/uv_cache
```

Why scratch? In testing, installs were much faster with the cache on scratch than on Oak. Scratch purges files that haven't been used in a while, but that's fine. Your projects live elsewhere, and if the cache gets purged, uv just rebuilds it the next time you add or sync packages.

> ![Russ](./images/russ_face.png) **Poldrack lab:** If you'd rather have a cache that never gets purged, you can use `/oak/stanford/groups/russpold/uv_cache_pool/uv_cache_$USER` instead. Expect slower installs.

**Not on Sherlock?** You can skip this step. uv's default cache location is fine on a laptop.

## Step 2: Add your settings to `~/.bashrc`

Open `~/.bashrc` in a text editor and add this at the bottom:

```bash
# --- uv setup ---
module load python/3.12.1
module load gcc/12.4.0
module load uv
module load git    
module load gh
# FSL and other lab software (Poldrack lab only):
# module load contribs
# module load poldrack
# module load fsl          # uncomment if you use FSL (or add other modules you need)

umask 0006                  # new files are group-readable and group-writable (all Poldrack lab members need to use this please and thank you!)

# So uv uses the module Python instead of the ancient system /usr/bin/python3
export UV_PYTHON=$(which python3)

# Where uv keeps its cache (see Step 1)
export UV_CACHE_DIR="$SCRATCH/uv_cache"

# The cache and your projects are on different filesystems, so copy instead of hardlink
export UV_LINK_MODE=copy
```

What each part does:

- **The modules** give you a modern Python, a compiler (some packages need to be built during install on Sherlock), and uv itself. Only load the modules you actually use. FSL is in there as an example.
- **`UV_PYTHON`** tells uv to use the Python you just loaded. It has to come *after* the `module load python` line. Otherwise `which python3` finds the system Python.
- **`UV_CACHE_DIR`** points uv at the cache from Step 1.
- **`UV_LINK_MODE=copy`** stops uv from complaining that it can't hardlink between scratch and your project. If the cache and your projects are on the same filesystem, you can leave this out.

> ![Russ](./images/russ_face.png) **Poldrack lab:** `umask 0006` is required. Without it, the files you create aren't writable by other lab members, and coworkers won't be able to work in shared projects. 

**Why `.bashrc`?** Sherlock runs `~/.bashrc` when you log in, when you open new shells, and when Slurm starts a job, so your settings are there everywhere. This also matters for notebooks: VS Code on Sherlock can only find your environment if Python is already loaded when code-server starts.

Keep `.bashrc` short. Sherlock runs it a lot, and slow startup files can slow down logins and job starts. A few module loads and exports are fine.

**Not on Sherlock?** Skip the module lines and `umask`. [Install uv](https://docs.astral.sh/uv/getting-started/installation/) instead, and uv will download and manage Python for you, so you don't need `UV_PYTHON` either.

## Step 3: Check that it worked

Log out and back in (or open a fresh login shell), then run:

```bash
echo $UV_PYTHON          # should be in /share/software, not /usr/bin/python3
$UV_PYTHON --version     # Python 3.12.1
echo $UV_CACHE_DIR       # your scratch cache path
uv cache dir             # should match the line above
which uv                 # should be the module's uv (/share/software)
```

If you use VS Code on Sherlock, start a **new** code-server session after making these changes. Sessions that were already running won't pick them up.

Next: [Starting a project](02-new-project.md)
