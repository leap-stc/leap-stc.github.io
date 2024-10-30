# LEAP Technical Documentation

This website is the home for all technical documentation related to LEAP and LEAP-Pangeo.

The website is located at <https://leap-stc.github.io/>.

## Dashboard

| Update Status                       | Contributors                          | Deployment Status                                    | Linting                                      |
| ----------------------------------- | ------------------------------------- | ---------------------------------------------------- | -------------------------------------------- |
| ![GitHub last commit][commit badge] | ![GitHub contributors][contrib badge] | [![pages-build-deployment][build badge]][build link] | [![pre-commit.ci status][pc badge]][pc link] |

## Developers Guide

- Clone this repository or your fork
- Install dependencies with `mamba env create -f environment.yml`
- Build book with `jupyter-book build book/`
- Inspect the book with e.g. `open book/_build/html/index.html`
- Or alternatively run a small server to get interactive updates:
  - `cd book/_build/html`
  - `python -m http.server`
  - Navigate to `http://localhost:8000/intro.html` in your browser.

### Run linting locally

```
pre-commit install
pre-commit run --all-files 
```

[build badge]: https://github.com/leap-stc/leap-stc.github.io/actions/workflows/pages/pages-build-deployment/badge.svg
[build link]: https://github.com/leap-stc/leap-stc.github.io/actions/workflows/pages/pages-build-deployment
[commit badge]: https://img.shields.io/github/last-commit/leap-stc/leap-stc.github.io
[contrib badge]: https://img.shields.io/github/contributors/leap-stc/leap-stc.github.io
[pc badge]: https://results.pre-commit.ci/badge/github/leap-stc/leap-stc.github.io/main.svg
[pc link]: https://results.pre-commit.ci/latest/github/leap-stc/leap-stc.github.io/main
