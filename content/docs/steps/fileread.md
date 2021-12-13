---
title: pypyr.steps.fileread
linktitle: fileread
date: 2021-12-12T15:02:21+01:00
description: Read file into pypyr context.
card_extra_summary:
  heading: input context property
  details: "`fileRead` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: fileread
seo_article_headline: Read file into task-runner pipeline context at run-time.
seo_description: Load a text or binary file into the task-runner so that the pipeline can read, manipulate & change the data.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: fileread -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [filesystem]
---
# pypyr.steps.fileread
## read file into context
Load a file into the the pypyr context.

`fileread` works like this:

```yaml
- name: pypyr.steps.fileread
  comment: read file in into context
    in:
      fileRead:
        path: path/to/file.ext # path to file
        key: arb # save file contents to this context key
        binary: False # Optional. Default False. Set True to read file as bytes.
        encoding: utf-8 # Optional. Default None (platform default).
```

If `path` is relative, it resolves relative to the current working directory.

If you set `binary` to `True`, the file contents will read as `bytes` without
any decoding. When `binary` is `False` (the default), the file contents will
decode use the platform-dependant default [encoding](#encoding).

{{% note tip %}}
`fileread` does not do any parsing of the file's contents at the time
of reading. If you want to parse json, toml or yaml, check out
[fetchjson]({{< ref "fetchjson" >}}), [fetchtoml]({{< ref "fetchtoml" >}}) &
[fetchyaml]({{< ref "fetchyaml" >}}).
{{% /note %}}

## read file as text
Here is an example showing to read file contents as text in the default encoding
and then using the contents in subsequent steps in the pipeline:
```yaml
- name: pypyr.steps.fileread
  comment: read file as text and save contents to txt1
  in:
    fileRead:
      path: testfiles/my-file.txt
      key: txt1

# some arbitrary steps using the contents written to txt1
- name: pypyr.steps.echo
  in:
    echoMe: '{txt1}'

- name: pypyr.steps.echo 
  in:
    echoMe: from text {txt1} file
```

## substitution expressions
All inputs support [substitutions]({{< ref "docs/substitutions">}}). This means
that you can construct the path or key partly or entirely from other context
variables:

```yaml
  - name: pypyr.steps.set
    comment: set some arbitrary values in context
    in:
      set:
        filename: my-file
        thekey: mykey

  - name: pypyr.steps.fileread
    comment: use substitution expressions for path and key
    in:
      fileRead:
        path: testfiles/{filename}.txt
        key: '{thekey}'
```

## binary mode
Set `binary` to `True` to returns a `bytes` object from the file without any
decoding:

```yaml
- name: pypyr.steps.fileread
  comment: read file in binary mode as bytes & save contents to mykey.
  in:
    fileRead:
      path: testfiles/my-file.bin
      key: mykey
      binary: True

- name: pypyr.steps.echo
  comment: the binary data is now in mykey.
  in:
    echoMe: '{mykey}'
```

## encoding
By default in text mode the file will read in the platform's default encoding. 
This is `utf-8` for most systems, but be aware on Windows it's still `cp1252`.

You can use the `encoding` input explicitly to set the encoding:

```yaml
- name: pypyr.steps.fileread
  comment: set encoding
  in:
    fileRead:
      path: testfiles/utf8-example.txt
      key: mykey
      encoding: utf-8
```

To check your platform's default encoding, do:
```python
import locale
locale.getpreferredencoding(False)
```

You cannot set encoding in binary mode.

See here for a [list of available encodings](https://docs.python.org/3/library/codecs.html#standard-encodings).
