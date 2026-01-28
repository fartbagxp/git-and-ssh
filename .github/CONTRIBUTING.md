# Contributing to the project

If you want to contribute to the project, please follow the steps below:

Install the development requirements:

```bash
uv venv
source .venv/bin/activate
uv sync --all-extras --dev
```

Install the pre-commit hooks:

```bash
pre-commit install
```

Run mkdocs/zensical:

To run the [mkdocs/zensical](https://zensical.org/docs/get-started/) application,

```bash
uv run zensical build
uv run zensical serve
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

To exit out of the virtual environment (venv), run the following:

```bash
deactivate
```

To figure out which dependency may be outdated, find the dependencies via:

```bash
uv tree --depth=1 --outdated
```

To create a new dependency lock file manually (optional as it should happen automatically on uv add):

```bash
uv lock
```

To upgrade dependencies, run:

```bash
uv lock --upgrade
```
