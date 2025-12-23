---
title: Check signature params for proper documentation
pubDatetime: 2025-12-15T05:18:35
slug: check-signature-params-for-proper-documentation
tags:
  - ci
  - docsig
  - documentation
  - docstring
  - python
description: reStructuredText (`Sphinx`), `NumPy`, and `Google`
---

![docsig logo](https://raw.githubusercontent.com/jshwi/docsig/master/docs/static/docsig.svg)

`docsig` is a tool for ensuring signature parameters are correctly documented.

There is no one standard for how docstring parameters should be documented, and so with `docsig`, you can lay down a policy and stick to it, resulting in more accurate documentation.

Documenting your parameters is important, but very easy to neglect, especially when parameters belonging to a function or method change frequently.

`docsig` will help you keep your docstrings up-to-date and informative

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

```yaml file=.pre-commit-config.yaml
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
