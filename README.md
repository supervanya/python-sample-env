# My simple Python setup

## The tools

| Tool       | What it does                                                                                                | Configured in               |
| ---------- | ----------------------------------------------------------------------------------------------------------- | --------------------------- |
| **mise**   | Installs the right **Python** and **uv** versions and switches to them when you `cd` into this folder.      | `.mise.toml`                |
| **uv**     | Manages the project's **packages** and its **virtual environment**. Replaces `pip`, `venv` and `pip-tools`. | `pyproject.toml`, `uv.lock` |
| **pytest** | Runs tests.                                                                                                 | —                           |
| **pdbp**   | A nicer drop-in replacement for Python's built-in debugger, `pdb`.                                          | —                           |
| **ruff**   | Lints and formats code.                                                                                     | —                           |

## What the files are

- **`.mise.toml`**: which Python and uv versions this project uses. It also tells mise to create and activate `.venv` automatically.
- **`pyproject.toml`**: the project's settings and the packages it needs, written by hand or by `uv add`.
- **`uv.lock`**: the exact version of every package, including dependencies of dependencies. uv generates it. Don't edit it by hand, but do commit it so everyone gets identical versions.
- **`.venv/`**: the virtual environment, a private folder holding Python and the installed packages for this project only. It's disposable: delete it and run `uv sync` to rebuild it. Don't commit it.

In short, `pyproject.toml` says what you want, `uv.lock` says exactly what you got, and `.venv` is where it's installed.

## First-time setup

1. Install mise: <https://mise.jdx.dev/getting-started.html>
2. Hook mise into your shell by adding this to `~/.zshrc`:
   ```bash
   eval "$(mise activate zsh)"
   ```
3. In this folder, run:
   ```bash
   mise trust      # allow mise to use this folder's .mise.toml (one time)
   mise install    # download the Python and uv versions it lists
   uv sync         # create .venv and install the packages from uv.lock
   ```

## Everyday use

When you `cd` into this folder, mise activates `.venv` for you, so you can run commands directly:

```bash
python buggy_partitioner.py
pytest
ruff check .
ruff format .
```

If the environment isn't active, put `uv run` in front of any of these commands. It always uses this project's environment and syncs it first if needed:

```bash
uv run pytest
```

To activate the environment manually, without mise:

```bash
source .venv/bin/activate   # turn on
deactivate                  # turn off
```

## Adding or removing packages

Use uv, not `pip install`, so `pyproject.toml` and `uv.lock` stay in sync:

```bash
uv add requests             # a package the code needs
uv add --dev pytest         # a development-only tool
uv remove requests
```

After pulling changes from someone else, run `uv sync` to update `.venv`.

## Debugging tests with pdbp

To stop in the debugger when a test fails:

```bash
pytest --pdb --pdbcls=pdbp:PdbpPdb        # every failure
pytest -x --pdb --pdbcls=pdbp:PdbpPdb     # stop at the first failure
```

To stop at a specific line, add `breakpoint()` to the code, then run:

```bash
PYTHONBREAKPOINT=pdbp.set_trace pytest -s
```

`-s` stops pytest from capturing input and output, which the debugger needs.

Useful debugger commands:

| Command   | Action                                 |
| --------- | -------------------------------------- |
| `n`       | Run the next line.                     |
| `s`       | Step into a function call.             |
| `c`       | Continue until the next breakpoint.    |
| `p x`     | Print the value of `x`.                |
| `l`       | Show the code around the current line. |
| `u` / `d` | Move up or down the call stack.        |
| `q`       | Quit.                                  |

## Troubleshooting

- **The environment seems broken or out of date:** run `rm -rf .venv && uv sync`.
- **uv creates `venv/` instead of `.venv/`, or a `uv-cache/` folder appears:** a `UV_PROJECT_ENVIRONMENT` or `UV_CACHE_DIR` variable is set in your environment. Run `env | grep UV_` to check. In VS Code, this usually means VS Code was launched from a terminal sitting in another project whose mise config sets those variables, and VS Code kept them. Quit VS Code completely with Cmd+Q and reopen it from the Dock.
- **The wrong Python is in use:** `which python` should point into `.venv/`. If it doesn't, run `mise install` and `uv sync`, or use `uv run`.
- **mise says the config isn't trusted:** run `mise trust`.
