---
title: Docsig
pubDatetime: 2025-12-15T05:18:35
slug: docsig
tags:
  - docsig
description: Check signature params for proper documentation
---

![docsig logo](https://raw.githubusercontent.com/jshwi/docsig/master/docs/static/docsig.svg)

<div class="badges">
  <img src="https://img.shields.io/badge/License-MIT-yellow.svg" />
  <img src="https://img.shields.io/pypi/v/docsig" />
  <img src="https://github.com/jshwi/docsig/actions/workflows/build.yaml/badge.svg" />
  <img src="https://results.pre-commit.ci/badge/github/jshwi/docsig/master.svg" />
  <img src="https://codecov.io/gh/jshwi/docsig/branch/master/graph/badge.svg" />
  <img src="https://readthedocs.org/projects/docsig/badge/?version=latest" />
  <img src="https://img.shields.io/badge/python-3.10-blue.svg" />
  <img src="https://img.shields.io/badge/code%20style-black-000000.svg" />
  <img src="https://img.shields.io/badge/%20imports-isort-%231674b1?style=flat&labelColor=ef8336" />
  <img src="https://img.shields.io/badge/linting-pylint-yellowgreen" />
  <img src="https://img.shields.io/badge/security-bandit-yellow.svg" />
  <img src="https://snyk.io/test/github/jshwi/docsig/badge.svg" />
  <img src="https://snyk.io/advisor/python/docsig/badge.svg" />
</div>

# Check signature params for proper documentation

Supports reStructuredText (`Sphinx`), `NumPy`, and `Google`

# Contributing

If you are interested in contributing to `docsig` please read about
contributing
[here](https://docsig.io/en/latest/development/contributing.html)

# Installation

```shell
$ pip install docsig
```

# Usage

## Commandline

```
usage: docsig [-h] [-V] [-l] [-c | -C] [-D] [-m] [-N] [-o] [-p] [-P] [-i] [-a] [-k] [-T]
              [-I] [-U] [-n] [-v] [-s STR] [-d LIST] [-t LIST] [-e PATTERN]
              [-E PATH [PATH ...]]
              [path [path ...]]

Check signature params for proper documentation

positional arguments:
  path                  directories or files to check

optional arguments:
  -h, --help            show this help message and exit
  -V, --version         show program's version number and exit
  -l, --list-checks     display a list of all checks and their messages
  -c, --check-class     check class docstrings
  -C, --check-class-constructor
                        check __init__ methods. Note: mutually incompatible with -c
  -D, --check-dunders   check dunder methods
  -m, --check-protected-class-methods
                        check public methods belonging to protected classes
  -N, --check-nested    check nested functions and classes
  -o, --check-overridden
                        check overridden methods
  -p, --check-protected
                        check protected functions and classes
  -P, --check-property-returns
                        check property return values
  -i, --ignore-no-params
                        ignore docstrings where parameters are not documented
  -a, --ignore-args     ignore args prefixed with an asterisk
  -k, --ignore-kwargs   ignore kwargs prefixed with two asterisks
  -T, --ignore-typechecker
                        ignore checking return values
  -I, --include-ignored
                        check files even if they match a gitignore pattern
  -U, --enforce-capitalization
                        ensure param descriptions are capitalized
  -n, --no-ansi         disable ansi output
  -v, --verbose         increase output verbosity
  -s STR, --string STR  string to parse instead of files
  -d LIST, --disable LIST
                        comma separated list of rules to disable
  -t LIST, --target LIST
                        comma separated list of rules to target
  -e PATTERN, --exclude PATTERN
                        regular expression of files or dirs to exclude from checks
  -E PATH [PATH ...], --excludes PATH [PATH ...]
                        path glob patterns to exclude from checks
```

Options can also be configured with the pyproject.toml file

```toml file=pyproject.toml
[tool.docsig]
check-dunders = false
check-overridden = false
check-protected = false
disable = [
    "SIG101",
    "SIG102",
    "SIG402",
]
target = [
    "SIG202",
    "SIG203",
    "SIG201",
]
```

## Flake8

`docsig` can also be used as a `flake8` plugin. Install `flake8` and
ensure your installation has registered [docsig]{.title-ref}

```shell
$ flake8 --version
# 7.3.0 (docsig: 0.72.0, mccabe: 0.7.0, pycodestyle: 2.14.0, pyflakes: 3.4.0) CPython 3.10.19 on Darwin
```

And now use [flake8]{.title-ref} to lint your files

```shell
$ flake8 example.py
# example.py:1:1: SIG202 includes parameters that do not exist (params-do-not-exist) 'function'
```

With `flake8` the pyproject.toml config will still be the base config,
though the [ini
files](https://flake8.pycqa.org/en/latest/user/configuration.html#configuration-locations)
`flake8` gets it config from will override the pyproject.toml config.
For `flake8` all args and config options are prefixed with `sig` to
avoid any potential conflicts with other plugins

```ini file=.flake8
[flake8]
sig-check-dunders = True
sig-check-overridden = True
sig-check-protected = True
```

## API

```python
>>> from docsig import docsig
```

```python
>>> string = '''
... def function(param1, param2, param3) -> None:
...     """
...
...     :param param1: About param1.
...     :param param2: About param2.
...     :param param3: About param3.
...     """
... '''
>>> docsig(string=string, no_ansi=True)
0
```

```python
>>> string = '''
... def function(param1, param2) -> None:
...     """
...
...     :param param1: About param1.
...     :param param2: About param2.
...     :param param3: About param3.
...     """
... '''
>>> docsig(string=string, no_ansi=True)
2 in function
    SIG202: includes parameters that do not exist (params-do-not-exist)
1
```

A full list of checks can be found
[here](https://docsig.io/en/latest/usage/messages.html)

## Message Control

[Documentation on message
control](https://docsig.io/en/latest/usage/message-control.html)

## Classes

[Documenting
classes](https://docsig.io/en/latest/usage/configuration.html#classes)

## pre-commit

`docsig` can be used as a [pre-commit](https://pre-commit.com) hook

It can be added to your .pre-commit-config.yaml as follows:

Standalone

```yaml  file=.pre-commit-config.yaml
repos:
  - repo: https://github.com/jshwi/docsig
    rev: v0.72.0
    hooks:
      - id: docsig
        args:
          - "--check-class"
          - "--check-dunders"
          - "--check-overridden"
          - "--check-protected"
```

or integrated with `flake8`

```yaml file=.pre-commit-config.yaml
repos:
  - repo: https://github.com/PyCQA/flake8
    rev: "7.1.0"
    hooks:
      - id: flake8
        additional_dependencies:
          - docsig==0.72.0
        args:
          - "--sig-check-class"
          - "--sig-check-dunders"
          - "--sig-check-overridden"
          - "--sig-check-protected"
```
