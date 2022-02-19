---
title: filesystem
description: Read & write files and work with paths on the local filesystem.
date: 2020-08-06T21:07:28+01:00
seo_article_headline: Work with files & paths in a task-runner automation pipeline.
seo_description: Read, write and format files as part of your automation pipeline tasks. 
seo_is_carousel: true
---
# filesystem
pypyr has many ready-made functions to read, write & format files on the
filesystem.

You can convert between text, json, yaml & toml formats without you having to
write code. This is especially handy when you're generating configuration or
template files on-the-fly that you want to inject into other tools that you are
automating with pypyr.

## encoding
pypyr will use the platform's default encoding to deal with text-based files.
This is `utf-8` for most sensible systems, but be aware on Windows it's still
`cp1252`.

### steps
All steps that work with files have an `encoding` input where you can override
the default and explicitly set the encoding:

```yaml
- name: pypyr.steps.filewrite
  in:
    fileWrite:
      path: out/utf8-example.txt
      payload: "€ ∮ E⋅da = Q,  n → ∞, ∑ f(i) = ∏ g(i), ∀x∈ℝ: ⌈x⌉ = −⌊−x⌋, α ∧ ¬β = ¬(¬α ∨ β)"
      encoding: utf-8
```

See here for a [list of available encodings](https://docs.python.org/3/library/codecs.html#standard-encodings).

You cannot set encoding in binary mode.

#### in-out steps
All steps with an `in` and `out` input allow you to override the default
encoding with three possible inputs:
- `encoding` - set the encoding for both in + out.
- `encodingIn` - set the encoding for the in file.
- `encodingOut` - set the encoding for the out file.

These are all the "format" and "replace" steps.

You can convert files between different encodings by setting a different
encoding for the input than the output.

If you don't specify `encodingIn` pypyr will default to the value of `encoding`
for the input file.

If you don't specify `encodingOut` pypyr will default to the value of `encoding`
for the output file.

If you did not specify `encoding` either, pypyr will use the [default
encoding](#default-encoding).

### context parsers
The file-based context parsers always use the default encoding when reading
input files. This is the case when you:
- [initialize variables from a json file]({{< ref "/docs/context-parsers/jsonfile" >}})
- [initialize variables from a yaml file]({{< ref "/docs/context-parsers/yamlfile" >}}) 

A [toml file]({{< ref "/docs/context-parsers/tomlfile" >}}) must always be in
`utf-8`, per the [TOML Spec](https://toml.io/en/latest#spec).

### default encoding
The platform default is `utf-8` for most systems, but be aware on Windows it's
still `cp1252`. To check your platform's default encoding, do:
```python
import locale
locale.getpreferredencoding(False)
```

If you're getting annoyed with having to change the encoding on each step, you
can change the default if you either:

- Set pypyr's [default encoding in config]({{< ref "/docs/getting-started/config#default_encoding" >}}).
    - This changes the encoding just for pypyr.
- Set Python to run in `utf-8` mode.
    - See [PEP 0540](https://www.python.org/dev/peps/pep-0540/).
    - This sets the default encoding to `utf-8` for your entire Python environment.
- Set Python encoding to something other than `utf-8`.
    - Maybe don't do this.
    - Check the Python docs if you want to go this route.

If you explicitly set `encoding` on any [filesystem step]({{< ref
"#filesystem-operations" >}}) it will override the default for that particular
step.

{{% note tip %}}
Windows users, it _might_ be an idea to set your over-all
Python runtime to use `utf-8` like all other modern platforms do. Then you
wouldn't need to worry about any of this.

See [PEP 0540](https://www.python.org/dev/peps/pep-0540/).

TLDR; Set the environment variable `PYTHONUTF8` to `1`.
 
{{% /note %}}


## filesystem operations