# LEAP Technical Documentation

This website is the home for all technical documentation related to LEAP and LEAP-Pangeo.

The website is located at <https://leap-stc.github.io/>.

## Dashboard

| Update Status                       | Contributors                          | Deployment Status                                    |
| ----------------------------------- | ------------------------------------- | ---------------------------------------------------- |
| ![GitHub last commit][commit badge] | ![GitHub contributors][contrib badge] | [![pages-build-deployment][build badge]][build link] |

## Developers Guide

- Clone this repository or your fork
- Install dependencies with `mamba env create -f environment.yml`
- Build book with `jupyter-book build book/`
- Inspect the book with e.g. `open book/_build/html/index.html`

### Run linting locally

```
pre-commit install
pre-commit run --all-files 
```

[build badge]: https://github.com/leap-stc/leap-stc.github.io/actions/workflows/pages/pages-build-deployment/badge.svg
[build link]: https://github.com/leap-stc/leap-stc.github.io/actions/workflows/pages/pages-build-deployment
[commit badge]: https://img.shields.io/github/last-commit/leap-stc/leap-stc.github.io
[contrib badge]: https://img.shields.io/github/contributors/leap-stc/leap-stc.github.io
