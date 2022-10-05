---
title: pypyr.steps.venv
linktitle: venv
date: 2022-10-04T15:27:43+01:00
description: Create a virtual environment (venv).
card_extra_summary:
  heading: input context property
  details: "`venv` (str | dict | list)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: venv
seo_article_headline: Create virtual environments (venv) from config.
seo_description: Automate venv creation from config with no code! Create multiple venv fast in parallel with concurrent creation.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: venv -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [execute external program]
---
# pypyr.steps.venv
Create Python stdlib virtual environments
([venv](https://docs.python.org/3/library/venv.html)) from config without
writing any script yourself. This is the equivalent of running
`$ python -m venv my-dir`.

This step lets you create venvs from within your automation pipeline.

If you just want to create a bunch of venvs but you don't want to write your own
pipeline to do so, check out the
[built-in venv-create pipeline]({{< ref "/docs/pipelines/built-in/venv-create" >}}).

This steps creates multiple venvs in parallel concurrently, making for faster
processing on what is usually a pretty slow process.

## default options
### single venv
Create a single venv with the default options:
```yaml
- name: pypyr.steps.venv
  in:
    venv: mydir/my-venv
```

This will create a venv on disk at `mydir/my-venv`, relative to the current
directory. You can also specify an absolute path, including using `~`
substitution for the user's home directory.

### multiple venvs
You can create multiple venvs in parallel concurrently like this:
```yaml
- name: pypyr.steps.venv
  in:
    venv:
      - .venvs/venv1
      - .venvs/venv2
      - ~/arb/another-venv
```

Notice that the first two venvs are relative to the current directory, whereas
the last uses an absolute path in the user's home directory.

## custom options
Here are all the available options:
```yaml
venv:
  path: mydir/my-venv
  pip: cowsay pypyr
  quiet: False
  system_site_packages: False
  clear: False
  symlinks: True # default True on posix; False on windows
  upgrade: False
  with_pip: True
  upgrade_pip: True
  prompt: None
```

Only `path` is mandatory. The other optional values you only need to specify
if you want to override their default values.

### path
- Mandatory.
- Directory in which to create the venv.
- Absolute path or relative to the current working directory.
  - Supports `~` expansion.
- Strictly speaking any valid filesystem characters are allowed, but do
  yourself a favor and avoid spaces & other special chars.
- Can be a list of paths to create multiple venvs with the same settings.

### pip
- Optional. Default none/empty.
- Specify which packages to install into the virtual environment.
- The value is a string that pypyr passes to the pip tool like this:
  - `$ pip install {your-string-here}`
- Some examples of typical inputs:
  - `cowsay tox pytest coverage[toml]`
  - `SomeProject>=1,<2 AnotherProject==1.0.4`
  - `-e /Users/CaptainHook/git/myproject/myrepo AnotherProjectOnPypi`
  - `-e ~/git/pypyr/pypyr[dev] tox mypy types-python-dateutil`
  - `--index-url https://test.pypi.org/simple/ MyProject`
- If you prefer, instead of a string where the args are separated by spaces,
  you can pass a list of the args:
  - `['cowsay', 'tox', 'arb-project==1.2.3']`

### quiet
- Optional. Default `False`.
- Reduce logging output. Especially pip produces a lot of output, which you
  might not want cluttering your logs.

### system_site_packages
- Optional. Default `False`.
- Make the system Python site-packages available to the environment.

### clear
- Optional. Default `False`.
- Delete the contents of any existing target path before creating the venv.

### symlinks
- Optional. Default `True` on posix, `False` on windows.
- Attempt to symlink the Python binary rather than copying it.

### upgrade
- Optional. Default `False`.
- Upgrade an existing environment with the currently running Python.
- You can use this when the current Python was upgraded in-place.
- You can't set `upgrade` to `True` if `clear` is also `True`.

### with_pip
- Optional. Default `True`.
- Install the `pip`  tool into the environment.

### upgrade_pip
- Optional. Default `True`.
- Upgrade `pip` and `setuptool` to latest versions.
- Only relevant when `with_pip` is `True`.

### prompt
- Optional. Default none/empty.
- Friendly name for venv to display in terminal prompt.
- If empty uses the directory name by default.
- Special value "." means the basename of the current directory.

## custom options examples
### single venv
You can set custom options to control how to create your venv:

```yaml
- name: pypyr.steps.venv
  in:
    venv:
      path: mydir/my-venv
      pip: cowsay pypyr arb-project==1.2.3 coverage[toml]
      quiet: True
```

This will install `cowsay`, `pypyr`, `arb-project` version 1.2.3 and `coverage`
with the `toml` extensions into the `mydir/my-venv` venv.

### multiple venvs
If you want to apply the same custom options while creating multiple venvs:

```yaml
- name: pypyr.steps.venv
  in:
    venv:
      path:
          - .venvs/venv1
          - /another-dir/another-env
      pip: cowsay pypyr
      clear: True
      quiet: True
```

## mix default and custom options
You can create multiple environments in the same step, mixing environments
with default options and custom:

```yaml
- name: pypyr.steps.venv
  in:
    venv:
      - mydir/env1
      
      - path: mydir/env2
        pip: cowsay pypyr
        quiet: True
        prompt: arb-name
      
      - path:
          - mydir/env3-1
          - mydir/env3-2
        pip: cowsay pypyr
        clear: True
        quiet: True
      
      - mydir/env4
```

Note that in this example:
  - `env3-1` and `env3-2` are both created with the same custom options
  - `env2` has its own distinct custom options.
  - `env1` and `env4` create with the defaults.

All the venvs listed here will create in parallel - this is not
particularly important to you as an end-user, other than just to point out that
it'll be faster than what you're used to with common sequential venv scripts.

## pip install extra dependencies
pypyr passes the `pip` input str to the `pip` command inside the venv. This
means you can specify all the usual arguments that you would when you run
`$ pip install` yourself from the terminal.

You can pass a string to `pip`, exactly like you would at the terminal.

```yaml
- name: pypyr.steps.venv
  in:
    venv:
      path: .venvs/my-venv
      pip: -e ~/git/my-proj/my-repo[extrasname] tox mypy types-python-dateutil

- name: pypyr.steps.venv
  in:
    venv:
      path: .venvs/another-venv
      pip: --index-url https://test.pypi.org/simple/ MyProject AnotherProject==1.0.4
```

Alternatively, you can pass a list of string to `pip`, where each individual
argument separated by a space at the terminal becomes an individual list item. 
This might be helpful if you're generating a list of extras to install
programmatically.

```yaml
- name: pypyr.steps.venv
  in:
    venv:
      path: .venvs/my-venv
      pip:
        - -e
        - ~/git/my-proj/my-repo[extrasname]
        - tox
        - mypy
        - types-python-dateutil

- name: pypyr.steps.venv
  in:
    venv:
      path: .venvs/another-venv
      pip:
        - --index-url
        - https://test.pypi.org/simple/
        - MyProject
        - AnotherProject==1.0.4
```

Note that while the venv creation happens in parallel, the pip install
thereafter is sequential. This is because the pip tool does not support
concurrent execution - it can lead to trouble if multiple packages are
downloading the same dependencies at the same time.