# Python and Jupyter setup

This repository keeps its Python project configuration and virtual environment
at the repository root. Notebooks and analysis-specific documentation live in
`analysis/`.

## Prerequisites

- macOS, Windows, or Linux
- Python 3.12
- [uv](https://docs.astral.sh/uv/)
- VS Code with the Python and Jupyter extensions, or a browser for JupyterLab

Install `uv` using its official installer:

```sh
curl -LsSf https://astral.sh/uv/install.sh | sh
```

On macOS, if `uv` is not found after installation, restart the terminal or load
the installer-managed PATH configuration for the current shell:

```sh
source "$HOME/.local/bin/env"
```

## Initialize a new project

From the directory that should contain the project configuration and `.venv`,
run:

```sh
uv init --bare --python 3.12
uv python pin 3.12
uv add pandas scipy cartopy matplotlib seaborn ipykernel jupyterlab pip
```

The first command creates `pyproject.toml`; the second records the Python
minor version in `.python-version`; and `uv add` records dependencies and
creates `uv.lock` while syncing the environment. Keep the notebook in a
subfolder such as `analysis/` if you want project configuration and `.venv/`
at the repository root.

Add a root `.gitignore` so generated environments and notebook state are not
committed:

```gitignore
.venv/
__pycache__/
*.py[cod]
.ipynb_checkpoints/
```

## Create or restore the environment

Run these commands from the repository root. The Python version is recorded in
`.python-version`; project requirements and dependencies are in
`pyproject.toml`, and exact resolved versions are recorded in `uv.lock`.

```sh
uv sync
```

This creates `.venv/` if needed and installs the locked project dependencies.
The virtual environment is local to this checkout and is excluded from Git.

To create a notebook in VS Code, use **Create: New Jupyter Notebook**, save it
under `analysis/`, and select the repository's `.venv` using **Select Kernel**.

The project includes:

- pandas, NumPy, and SciPy for data analysis
- Matplotlib and Seaborn for plotting
- Cartopy for map projections and geographic plotting
- JupyterLab and IPython kernel support
- pip for compatibility with pip-based instructions

Prefer `uv add package-name` to add a project dependency. This updates
`pyproject.toml` and `uv.lock`; commit both files so other checkouts can
reproduce the environment. Then run `uv sync` to apply the locked environment.
For a one-off package that should not become a project dependency, use
`uv pip install --python .venv/bin/python package-name` (on Windows, the
interpreter path is `.venv\\Scripts\\python.exe`).

When pip compatibility is needed, use it through the project interpreter:

```sh
uv run python -m pip install package-name
```

Packages installed this way are not recorded in `pyproject.toml` or `uv.lock`.
For dependencies that should be reproducible, prefer `uv add`.

## Connect Jupyter to the environment

The project includes `ipykernel`, which lets Jupyter run notebook cells using
`.venv`. For the simplest setup, open the notebook in VS Code, choose **Select
Kernel**, and select the Python environment at the repository's `.venv`. The
VS Code Python and Jupyter extensions are required. This does not require
installing a kernel into your user-level Jupyter configuration.

To use a named kernel in other Jupyter clients, register it explicitly from
the repository root:

```sh
uv run python -m ipykernel install --user \
  --name vcc-energy-assistance \
  --display-name "VCC Energy Assistance"
```

This creates a kernelspec in your user Jupyter configuration that points to
this checkout's `.venv`. If the repository is moved, run the registration
command again from its new location. To remove an obsolete registration, use
`jupyter kernelspec uninstall vcc-energy-assistance`.

For a browser-based JupyterLab session, start the server from the repository
root:

```sh
uv run jupyter lab
```

Select the registered **VCC Energy Assistance** kernel in JupyterLab. If it
has not been registered, use the registration command above first.

## Verify the installation

Run this from the repository root to check the interpreter and core imports:

```sh
uv run python -c "import sys, pandas, scipy, cartopy, matplotlib, seaborn; print(sys.executable); print('Imports successful')"
```

In a notebook, verify that the selected kernel is the project environment:

```python
import sys

print(sys.executable)
```

The printed path should point to this checkout's `.venv`.

## Cartopy notes

Cartopy depends on geospatial libraries such as PROJ and GEOS. `uv` normally
installs compatible prebuilt wheels where they are available, as it did for
this macOS setup. If resolution or installation fails on another platform,
first check the Python version and platform wheel availability in the Cartopy
installation documentation. Avoid installing system geospatial libraries
unless the package manager reports that a source build requires them.
