# Overview

This is a practical guide on how to manage multiple Github accounts while also using SSH. It is a [mkdocs/zensical website](https://zensical.org/docs/get-started/) using the [materials theme](https://squidfunk.github.io/mkdocs-material/creating-your-site/).

The website is [deployed here](https://fartbagxp.github.io/git-and-ssh/).

## Development

View [Contributing.md](.github/CONTRIBUTING.md) for more on development.

## Local Setup

Use [uv](https://docs.astral.sh/uv/) and run the following to host [the site locally](http://127.0.0.1:8000/git-and-ssh/):

```bash
uv run zensical build --clean
uv run zensical serve
```
