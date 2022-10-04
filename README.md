# nbmake
[![codecov](https://codecov.io/gh/treebeardtech/nbmake/branch/main/graph/badge.svg?token=9GuDM35FuO)](https://codecov.io/gh/treebeardtech/nbmake)
[![PyPI versions](https://img.shields.io/pypi/pyversions/nbmake?logo=python&logoColor=white)](https://pypi.org/project/nbmake)
[![PyPI versions](https://img.shields.io/pypi/v/nbmake?logo=python&logoColor=white)](https://pypi.org/project/nbmake)
[![PyPI Downloads](https://img.shields.io/pypi/dm/nbmake)](https://pypi.org/project/nbmake)

**What?** Pytest plugin for testing and releasing notebook documentation

**Why?** To raise the quality of scientific material through better automation

**Who is this for?** Research/Machine Learning Software Engineers who maintain packages/teaching materials with documentation written in notebooks.

## Functionality

1. Executes notebooks using pytest and nbclient, allowing parallel notebook testing
2. Optionally writes back to the repo, allowing faster building of [nbsphinx](https://github.com/spatialaudio/nbsphinx) or [jupyter book](https://github.com/executablebooks/jupyter-book) docs
3. Optionally break down the notebook into individual tests by section (see "by cell" mode)

## Quick Start

If you have a notebook that runs interactively using an ipython kernel,
you can try testing it automatically as follows:

```sh
pip install pytest nbmake
pytest --nbmake **/*ipynb
```

Files ending with `.ipct.py` and `.notebook.py` in percent format are also supported.

## Configure Cell Timeouts

You can configure the cell timeout with the following pytest flag:

```sh
pytest --nbmake --nbmake-timeout=3000 # allows each cell 3000 seconds to finish
```

## Allow Errors For a Whole Notebook

This configuration must be placed in the notebook's **top-level metadata** (not cell-level metadata).

Your notebook should look like this:

```json
{
  "cells": [ ... ],
  "metadata": {
    "kernelspec": { ... },
    "execution": {
      "allow_errors": true,
      "timeout": 300
    }
  }
}
```

## Allow a Cell to Throw an Exception

A cell with the following metadata can throw an exception without failing the test:

```json
{
  "language": "python",
  "custom": {
    "metadata": {
      "tags": [
        "raises-exception"
      ]
    }
  }
}
```

## Ignore a Code Cell

A cell with the following metadata will not be executed by nbmake

```json
{
  "language": "python",
  "custom": {
    "metadata": {
      "tags": [
        "skip-execution"
      ]
    }
  }
}
```

## Override Notebook Kernels when Testing

Regardless of the kernel configured in the notebook JSON, you can force nbmake to use a specific kernel when testing:

```
pytest --nbmake --nbmake-kernel=mycustomkernel
```

## Add Missing Jupyter Kernel to Your CI Environment

If you are not using the flag above and are using a kernel name other than the default ‘python3’, you will see an error message when executing your notebooks in a fresh CI environment: `Error - No such kernel: 'mycustomkernel'`

Use ipykernel to install the custom kernel:

```sh
python -m ipykernel install --user --name mycustomkernel
```

If you are using another language such as c++ in your notebooks, you may have a different process for installing your kernel.

## Parallelisation

For repos containing a large number of notebooks that run slowly, you can run each notebook
in parallel using `pytest-xdist`.

```sh
pip install pytest-xdist

pytest --nbmake -n=auto
```

It is also possible to parallelise at a CI-level using strategies, see [example](https://github.com/LabForComputationalVision/plenoptic/blob/master/.github/workflows/treebeard.yml)

### Build Jupyter Books Faster

Using xdist and the `--overwrite` flag let you build a large jupyter book repo faster:

```sh
pytest --nbmake --overwrite -n=auto examples
jb build examples
```

## Mock out variables to simplify testing (experimental 🧪)

If your notebook runs a training process that takes a long time to run, you can use nbmake's
mocking feature to overwrite variables after a cell runs:

```json
{
  "cells": [
    ...,
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "nbmake": {
          "mock": {
            // these keys will override global variables after this cell runs
            "epochs": 2,
            "config": "/test/config.json",
            "args": {
              "env": "test"
            }
          }
        }
      },
      "outputs": [],
      "source": [
        "epochs = 10\n",
        "..."
      ]
    },
    ...
  ],
  ...
}
```


## Advice on Usage

nbmake is best used in a scenario where you use the ipynb files only for development. Consumption of notebooks is primarily done via a docs site, built through jupyter book, nbsphinx, or some other means. If using one of these tools, you are able to write assertion code in cells which will be [hidden from readers](https://jupyterbook.org/interactive/hiding.html).

### Pre-commit

Treating notebooks like source files lets you keep your repo minimal. Some tools, such as plotly may drop several megabytes of javascript in your output cells, as a result, stripping out notebooks on pre-commit is advisable:

```
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/kynan/nbstripout
    rev: master
    hooks:
      - id: nbstripout
```

See https://pre-commit.com/ for more...

## Disable Nbmake

Implicitly:
```
pytest
```

Explicitly:
```
pytest -p no:nbmake
```

## "By Cell" Mode

In "by cell" mode, nbmake will carve up the notebook into multiple individual
unit tests. The contents of each section, as defined by a markdown cell with a
heading (of any level), will be split out as a separate test, as long as it
qcontains at least one code cell. Sections without code cells will be ignored.
All the cells within a section will be run consecutively, in order. The title of
each heading will be used as the name for each test.

An optional initial section without a heading serves as a common initialization
block which will be run for all tests in the file.

In order to turn on this mode, use the `--nbmake-by-cell` flag, like this:

```sh
pytest --nbmake --nbmake-by-cell ...
```

For example, the following notebook structure would result in two unit tests
(example in `percent` format):

```python

# %%
"""Common initialization block."""
import re

# %% [markdown]
# # test 1: check asterisk

# %%
assert re.match("a*", "a")
assert re.match("a*", "aa")
assert not re.match("a*", "b")

# %% [markdown]
# # comments

# Note that this section happens to be ignored, because it has no codes.

# %% [markdown]
# # test 1: check plus

# %%
assert not re.match("a+", "a")
assert re.match("a+", "aa")
assert not re.match("a+", "b")
```

## See Also:

* A more in-depth [intro to nbmake](https://semaphoreci.com/blog/test-jupyter-notebooks-with-pytest-and-nbmake) running on Semaphore CI
* [nbmake action](https://github.com/treebeardtech/treebeard)
* [pytest](https://pytest.org/)
* [jupyter book](https://github.com/executablebooks/jupyter-book)
* [jupyter cache](https://github.com/executablebooks/jupyter-cache)
* [MyST-NB](https://github.com/executablebooks/MyST-NB)
