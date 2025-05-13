# Overview

This is a practical guide on how to manage multiple Github accounts while also using SSH. It is a [mkdocs website](https://www.mkdocs.org/) using the [materials theme](https://squidfunk.github.io/mkdocs-material/creating-your-site/).

## Development

To get started with development, run the following:

```bash
uv venv
source .venv/bin/activate
uv sync --all-extras --dev
```

To run the [mkdocs](https://www.mkdocs.org/) application,

```bash
uv run mkdocs build
uv run mkdocs serve
```

and navigate to [http://127.0.0.1:8000/git-and-ssh/](http://127.0.0.1:8000/git-and-ssh/) for the latest updates.

To add a new dependency, like mkdocs-material, run this:

```bash
uv add mkdocs-material
```

To add a new dependency to a group like a lint group to segment dependencies, run this:

```bash
uv add --group lint ruff
```

To run the code from main.py, run it via uv environment.

```bash
uv run main.py data/raw
```

To exit out of the virtual environment (venv), run the following:

```bash
deactivate
```

To figure out which dependency may be outdated, find the dependencies via:

```bash
uv pip list --outdated
```

To create a new dependency lock file manually (optional as it should happen automatically on uv add):

```bash
uv lock
```

To upgrade dependencies, run:

```bash
uv lock --upgrade
```
