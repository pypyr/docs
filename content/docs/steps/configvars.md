---
title: pypyr.steps.configvars
linktitle: configvars
date: 2022-02-13T13:54:20Z
description: Inject variables from config into pipeline.
draft: false
card_extra_summary:
  heading: input context property
  # details:
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: configvars
seo_article_headline: Inject variables from config into pypyr pipeline.
seo_description: Use variables from pyproject.toml or pypyr config yaml file in your automation pipeline.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: configvars -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [context]
---
# pypyr.steps.configvars
Use the `configvars` step to inject variables that you set in `pyproject.toml`
or in the pypyr yaml config files into your pipeline. This allows you to
get populate your pipeline variables dynamically from project configuration
specific to your repo, or to use user or global settings in your pipeline.

`configvars` is a simple step that doesn't take any inputs - it just looks
for `vars` in the pypyr config files.

pypyr will honor the data types of the variables - so if you have a bool in your
config, pypyr will read that as a bool, an int will read as an int and so forth.

## pypyr-config.yaml
You can use properties from `pypyr-config.yaml` or `pypyr/config.yaml` in your
pipeline as variables.

The `pypyr.steps.configvars` step reads everything under the `vars` key.

So if you had this in your `./pypyr-config.yaml` file:
```yaml
vars:
  myvar: myvalue
  myint: 123
  mylist:
    - one
    - two
  mydict:
    key1: value1
    key2: value2
```

Your pipeline can import these `vars` with the `configvars` step:

```yaml
# ./mypipeline.yaml
steps:
  - pypyr.steps.configvars
  - name: pypyr.steps.echo
    comment: after configvars, can use vars from config in pipeline
    in:
      echoMe: |
        myvar: {myvar} & myint: {myint}
        reading from a list: {mylist[1]}
        reading from a dict: {mydict[key2]}
```

When you run this pipeline you'll see:
{{< app-window title="term" lang="text" >}}
$ pypyr mypipeline
myvar: myvalue & myint: 123
reading from a list: two
reading from a dict: value2
{{< /app-window >}}

## pyproject.toml
You can use properties from `pyproject.toml` in your pipeline as variables.

The `pypyr.steps.configvars` step reads from the `[tool.pypyr.vars]` sub-table.

So if you had this in your `./pyproject.toml` file:
```toml
[tool.pypyr.vars]
mydict = {key1 = "value1", key2 = "value2"}
myint = 123
mylist = ["one", "two"]
myvar = "myvalue"
```

Your pipeline can import these `vars` with the `configvars` step:
```yaml
# ./mypipeline.yaml
steps:
  - pypyr.steps.configvars
  - name: pypyr.steps.echo
    comment: after configvars, can use vars from config in pipeline
    in:
      echoMe: |
        myvar: {myvar} & myint: {myint}
        reading from a list: {mylist[1]}
        reading from a dict: {mydict[key2]}
```

When you run this pipeline you'll see:
{{< app-window title="term" lang="text" >}}
$ pypyr mypipeline
myvar: myvalue & myint: 123
reading from a list: two
reading from a dict: value2
{{< /app-window >}}

## merging config
pypyr combines the config variables from all the config sources it finds.

pypyr follows a [configuration file look-up sequence]({{< ref
"/docs/getting-started/config#config-file-locations" >}}) where pypyr will
merge the config from the various config sources in order of precedence to
produce a final effective combined config.

Notice that the example given for `pyproject.toml` and `pypyr-config.yaml`
describe the exact same structure, and the exact same pipeline can operate on
the resulting context coming from either file format in the same way.

### global config
So let's say you have your global config yaml file:
```yaml
vars:
  a: from global config
```

{{% note tip %}}
Where you global config file is depends on your O/S and how you've set it up.

See here for details on how pypyr finds the [global config location for each
platform]({{< ref "/docs/getting-started/config#config-file-locations" >}}).

The defaults are:
- Linux: `/etc/xdg/pypyr/config.yaml`
- MacOs:  `/Library/Application Support/pypyr/config.yaml`
- Windows: `C:\ProgramData\pypyr\config.yaml`
{{% /note %}}

### user config
And your user config at `~/.config/pypyr/config.yaml`:
```yaml
vars:
  b: from user config
```

{{% note tip %}}
Windows Users, `~` means your home directory - usually `C:\Users\YourUserName\`.

This gives the default user config path:
`C:\Users\YourUserName\.config\pypyr\config.yaml`.
{{% /note %}}

### project config
In your project directory you have `./pyproject.toml`:
```toml
[tool.pypyr.vars]
c = "from pypproject.toml"
```

And finally in your project directory you also have `./pypyr-config.yaml`:
```yaml
vars:
  d: from ./pypyr-config.yaml
```

### merged global, user & project config
In your pipeline you can use `pypyr.steps.configvars` to get values from the
combined config:
```yaml
# ./my-pipeline.yaml
steps:
  - pypyr.steps.configvars
  - name: pypyr.steps.echo
    in:
      echoMe: |
        a: {a}
        b: {b}
        c: {c}
        d: {d}

```

When you run this pipeline you'll see:
{{< app-window title="term" lang="text" >}}
$ pypyr mypipeline
a: from global config
b: from user config
c: from pypproject.toml
d: from ./pypyr-config.yaml
{{< /app-window >}}

Note that values from earlier in the look-up sequence will overwrite later
values. Mappings/Dicts & Lists/Arrays with the same name will overwrite, not
additively merge their content.

## set vars for a specific pipeline
The values you set in `vars` are available to all pipelines. Any given pipeline
can pull in these values with the `pypyr.steps.configvars` step, assuming the
configuration file with the `vars` is global or user, or in scope of the current
working directory.

If instead what you want to do is selectively inject specific variables only
applicable to a specific pipeline, you could create a [shortcut]({{< ref
"/docs/pipelines/shortcuts" >}}) or have your pipeline pull your variables in
via a fetch step such as [fetchyaml]({{< ref "fetchyaml" >}}), [fetchjson]({{<
ref "fetchjson" >}}) or [fetchtoml]({{< ref "fetchtoml" >}}).

All of these approaches will work - pick whichever works best for you.