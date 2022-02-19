---
title: pypyr.steps.filewrite
linktitle: filewrite
date: 2021-12-12T11:38:51+01:00
description: Write payload to file.
card_extra_summary:
  heading: input context property
  details: "`fileWrite` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: filewrite
seo_article_headline: Write dynamic payload to output file.
seo_description: Create a text or binary file from any context input object with replacement token formatting in a task-runner pipeline.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: filewrite -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [filesystem]
---
# pypyr.steps.filewrite
## create file from any context object
Format & write a payload to a file.

`filewrite` works like this:

```yaml
- name: pypyr.steps.filewrite
  comment: write payload out to file at path.
  in:
    fileWrite:
      path: /path/to/output.ext # destination path
      payload: file content here # payload to write to path
      append: False # (optional) Default False to overwrite existing
      binary: False # (optional) Default False for text mode. True for bytes/binary.
      encoding: utf-8 # Optional. Default None (platform default).
```

If `path` is relative, it resolves relative to the current working directory.

If you set `append` to `True`, the payload will append to a file if it already
exists. The default is `append=False`, which will overwrite any existing file at
`path` entirely. In append mode, pypyr will create the file if it doesn't exist
yet.

If you set `binary` to `True` the payload will write to the file as bytes in
binary mode. The default of `False` works in text mode, writing strings in your
platform's default [encoding](#encoding).

All inputs support [substitutions]({{< ref "docs/substitutions">}}).

## newlines in yaml
A multi-line string in yaml is known as a block scalar. You use block style
indicators to control whether to fold newlines into spaces and chomping
indicators to control whether to add a newline at the end.

- Scalar Style:
  - Replace newline with space: `>`
  - Keep newlines: `|`
- Block Chomping:
  - Single newline at end: Default - no indicator necessary.
  - Strip newline at end of last line: `-`
  - Keep all newlines at end: `+`

You can combine a scalar style and block chomping indicators (`|`, `|-`, `>+`
etc.).

```yaml
- name: pypyr.steps.filewrite
  comment: Keep newlines as is.
           Add a trailing newline at end of last line.
  in:
    fileWrite:
      path: out/filewrite/filewrite-text-trailing-newline.txt
      payload: |
        this is line 1
        this is line 2
        and this is line 3

- name: pypyr.steps.filewrite
  comment: note block chomping indicator on payload
           this prevents trailing newline at end.
  in:
    fileWrite:
      path: out/filewrite/filewrite-text.txt
      payload: |-
        this is line 1
        this is line 2
        and this is line 3
```
Here's a great website to experiment with the different yaml block style &
chomping indicators: [https://yaml-multiline.info](https://yaml-multiline.info)

If `payload` is a single scalar and you want to include a newline at the end,
you have to wrap the string in quotes:

```yaml
- name: pypyr.steps.filewrite
  comment: if you want to add newline
           you HAVE to wrap a single scalar in double quotes.
  in:
    fileWrite:
      path: out/filewrite/filewrite-append-text.txt
      payload: "this is line 1\n"
```

## format output with token replacements
All inputs support [substitutions]({{< ref "docs/substitutions">}}). This means 
that you can construct the path or payload from other context variables:

```yaml
- name: pypyr.steps.set
  in:
    set:
      out_path: out/filewrite/substitutions.txt
      the_payload: I was set with a substitution expression.

- name: pypyr.steps.filewrite
  in:
    fileWrite:
      path: '{out_path}'
      payload: '{the_payload}'
```

Substitution processing also runs on the output payload content.

As always in pypyr, you can construct a value by embedding & combining
substitution expressions in other strings:

```yaml
- name: pypyr.steps.set
  comment: set some arb values in context
  in:
    set:
      arbkey: 123
      arbstr: in the middle
      filename: my-file

- name: pypyr.steps.filewrite
  comment: write file with substitution expressions in payload
  in:
    fileWrite:
      path: out/{filename}.ext
      payload: begin {arbkey} {arbstr} end
```

In the above example, in the output file created at `out/my-file.ext`, the file
contents will be a single line as follows:

```text
begin 123 in the middle end
```

## binary mode
Set `binary` to `True` to write file output as bytes without any encoding.

```yaml
- name: pypyr.steps.filewrite
  in:
    fileWrite:
      path: out/filewrite/filewrite-binary.bin
      payload: !py b'imagine this is binary data+'
      binary: True

- name: pypyr.steps.filewrite
  in:
    fileWrite:
      path: out/filewrite/filewrite-binary-yaml-oneline.bin
      payload: !!binary aW1hZ2luZSB0aGlzIGlzIGJpbmFyeQ==
      binary: True

- name: pypyr.steps.filewrite
  in:
    fileWrite:
      path: out/filewrite/arrow.gif
      payload: !!binary |
        R0lGODlhDAAMAIQAAP//9/X17unp5WZmZgAAAOfn515eXvPz7Y6OjuDg4J+fn5
        OTk6enp56enmlpaWNjY6Ojo4SEhP/++f/++f/++f/++f/++f/++f/++f/++f/+
        +f/++f/++f/++f/++f/++SH+Dk1hZGUgd2l0aCBHSU1QACwAAAAADAAMAAAFLC
        AgjoEwnuNAFOhpEMTRiggcz4BNJHrv/zCFcLiwMWYNG84BwwEeECcgggoBADs=
      binary: True
```

The yaml `!!binary` tag expects base64 strings. See the
[yaml spec on the binary type](https://yaml.org/type/binary.html).

You can set `append` to `True` to append binary data to an existing file rather
than overwriting it:

```yaml
- name: pypyr.steps.filewrite
  in:
    fileWrite:
      path: out/filewrite/filewrite-binary.bin
      payload: !py b'imaginary binary data appended to end.'
      append: True
      binary: True
```

To get a base64 string in Python, do this:

```python
import base64
base64.encodebytes(b'my string here')
```

## encoding
By default in text mode the file will write in the platform's default encoding. 
This is `utf-8` for most systems, but be aware on Windows it's still `cp1252`.

You can use the `encoding` input explicitly to set the encoding:

```yaml
- name: pypyr.steps.filewrite
  in:
    fileWrite:
      path: out/utf8-example.txt
      payload: "€ ∮ E⋅da = Q,  n → ∞, ∑ f(i) = ∏ g(i), ∀x∈ℝ: ⌈x⌉ = −⌊−x⌋, α ∧ ¬β = ¬(¬α ∨ β)"
      encoding: utf-8
```

See here for more details on handling [text encoding in pypyr]({{< ref
"/topics/filesystem#encoding" >}}) and changing the defaults.

You cannot set encoding in binary mode.

See here for a [list of available encodings](https://docs.python.org/3/library/codecs.html#standard-encodings).
