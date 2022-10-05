---
title: venv-create built-in pipeline
linktitle: venv-create
date: 2022-10-05T14:16:54+01:00
description: Create venvs with extra dependencies from a config file.
# card_extra_summary:
#   heading: input context property
#   details: "`venv-create` (dict)"
categories: [pipelines]
# keywords: ""
menu:
  docs:
    parent: builtin-pipelines
    name: venv-create
seo_article_headline: Create venvs from declarative yaml or toml config files.
seo_description: Create venvs in parallel with extra dependencies from a config file without writing any code.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: venv-create -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
# topics: [topic1, topic2, topic3]
---
# venv-create
Create virtual environments from a yaml or toml config file and install extra
dependencies. To speed things up, the pipeline creates multiple venvs
concurrently in parallel.

This pipeline uses the [pypyr.steps.venv]({{< ref "/docs/steps/venv" >}}) step
under the covers, so check the step's documentation for further detail on the
config inputs.

This pipeline creates Python stdlib virtual environments
([venv](https://docs.python.org/3/library/venv.html)) from declarative config
without writing any script yourself. The underlying mechanism is pretty much
the equivalent of `$ python -m venv my-dir`.

You can run this pipeline in two ways:

## ad hoc yaml input
You can run the `venv-create` pipepeline with an arbitrary yaml file as input
like this:

```text
$ pypyr venv-create ~/my-dir/my-venvs.yaml
```

The file `~/my-dir/my-venvs.yaml` looks like this:
```yaml
venv:
  - path: ~/.venvs/python-dev
    pip: autopep8 flake8==5.0.4 pylint rstcheck pygments rope
    clear: True

  - path: # 2 venvs with the same settings
      - ~/.venvs/myproj-dev
      - /another-dir/my-venv
    pip: -e ~/git/myp-proj/my-repo[dev] tox mypy types-python-dateutil

  - path: ~/.venvs/aws-cli
    pip: awscli
    clear: True

  - path: ~/.venvs/pygments
    pip: pygments
    clear: True
```

When you run `$ pypyr venv-create ~/my-dir/my-venvs.yaml` pypyr will create
all the environments under `venv` in parallel, and once that's done run the
pip installs sequentially.

If you just want a venv with defaults and no extra options, you can simplify
the input to:

```yaml
# ~/my-dir/my-venvs.yaml
venv:
  - my-dir/venv1
  - my-dir/venv2
  - another-dir/another-venv
```

The `venv` key can co-exist with other top-level keys in your yaml config file.

When you pass a path to the `venv-create` pipeline from the cli pypyr will not
look for the `vars.venv` key in pypyr config (see next section).

## config
Alternatively, you can keep your `venv` setup settings in one of pypyr's config
files. In this case you do not need to pass a path to the `venv-create` pipeline
and just run it like this:

```text
$ pypyr venv-create
```

Here pypyr will look for a `venv` key in one of these files:
- `pyproject.toml` in current dir, under the `[tool.pypyr.vars]` table
- or under the `vars` key in any of the pypyr yaml config files:
  - `./pypyr-config.yaml`
  - `{user config dir}/pypyr/config.yaml`
  - `{global config dirs}/pypyr/config.yaml`

Remember that pypyr merges config settings from all available config sources, so
you can for example specify globally applicable venvs in the global file, user
applicable venvs in the user file and local values in `./pyproject.toml` and
`./pypyr-config.yaml`. When you run `$ pypyr venv-create` it will create all the
venvs specified by the combined effective config.

See [pypyr configuration]({{< ref "/docs/getting-started/config">}}) for
details on pypyr configuration file locations, loading order and formats.

When you run `$ pypyr venv-create` pypyr will create all the environments under
`vars.venv` in parallel, and once this is done run the specified pip installs
sequentially.

### pyproject.toml
You can create multiple venvs in parallel by specifying your config settings
in `pyproject.toml` like this:
```toml
[tool.pypyr.vars]
venv = [
  "~/.venvs/env1",
  {path = "my-dir/env2", pip = "tox pytest", quiet = true},
  {path = ["my-dir/env3-1", "another-dir/env3-2"], pip = "cowsay pypyr", quiet = true},
  "my-dir/env4",
]
```

The bare string list entries will create a default venv with no custom options.

The nested table allows you to specify extra custom options for your venv. Note
that you can create multiple venvs with the same custom options by passing a
list of paths to `path`.

If you prefer using
[toml's array of tables](https://toml.io/en/latest#array-of-tables) syntax,
that would look like this:
```toml
[tool.pypyr.vars]
[[tool.pypyr.vars.venv]]
path = "my-dir/env1"
pip = "coverage black>=1.2.3"
quiet = true

[[tool.pypyr.vars.venv]]
path = "another-dir/env2"
pip = "tox pytest"
upgrade_pip = false
```
If you just want to create one venv with no custom options, you can simplify
your input like this:
```toml
[tool.pypyr.vars]
venv = "~/my-dir/myvenv"
```

If you just want to create one venv with custom options:
```toml
[tool.pypyr.vars.venv]
path = "~/my-dir/myvenv"
pip = "cowsay pytest tox"
quiet = true
```

To create all the specified venvs from the combined effective config, from the
directory containing your `pyproject.toml` run:
```text
$ pypyr venv-create
```

### yaml config
Have a vars->venv key in any of the pypyr yaml config files:
```yaml
# ./pypyr-config.yaml
vars:
  venv:
    - path: ~/.venvs/python-dev
      pip: autopep8 flake8 pylint rstcheck pygments rope
      clear: True

    - path: ~/.venvs/pypyr-dev
      pip: -e ~/git/pypyr/pypyr[dev] tox mypy types-python-dateutil
```

## config format
You can use any of the
[custom venv creation options]({{< ref "/docs/steps/venv#custom-options" >}})
documented here. You only need to specify an optional input if you want to
over-ride the default value.

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

The toml and yaml formats both use the same inputs and are structurally
equivalent, so it's just the syntax that differs.

{{% note tip %}}
Be careful about not using an absolute path as a value for `path`. Remember that
if you use a relative path the venv will create relative to whatever directory
you're in currently.
{{% /note %}}

If you want to create multiple venvs with the same settings, you can pass a
list of paths to `path`:

```yaml
venv:
  path:
    - .venvs/venv1
    - /another-dir/another-env
  pip: cowsay pypyr
  clear: True
  quiet: True
```