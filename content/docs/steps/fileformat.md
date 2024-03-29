---
title: pypyr.steps.fileformat
linktitle: fileformat
date: 2020-07-01T20:17:37+01:00
description: Find & replace substitution {tokens} in any file.
card_extra_summary:
  heading: input context property
  details: "`fileFormat` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: fileformat
seo_article_headline: Format, find & replace text file content.
seo_description: Find & replace tokens in a text file in a pipeline task-runner step.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: fileformat -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [filesystem]
---
# pypyr.steps.fileformat
## find & replace tokens in text files
Parses input text file, substitutes {tokens} in the text of the file
from the pypyr context and writes the result to an output file.

```yaml
  - name: pypyr.steps.fileformat
    comment: read a file from disk,
             do some substitutions,
             write back to disk.
    in:
      fileFormat:
        in: ./in/arb.txt # required
        out: ./out/arb.txt # optional
```

So if you had a text file `in/arb.txt` like this:

```text
{k1} sit thee down and write
In a book that all may {k2}
```

And your pypyr context were:

```yaml
k1: pypyr
k2: read
```

You would end up with an output file `out/arb.txt` like this:

```text
pypyr sit thee down and write
In a book that all may read
```

The file `in` and `out` paths support [substitutions]({{< ref "docs/substitutions">}}).

See a worked [example of fileformat](https://github.com/pypyr/pypyr-example/blob/main/pipelines/fileformat.yaml).

## escape characters
If your file contains {curly braces} that pypyr should NOT try to substitute,
you can escape these by {{doubling}}. A doubled curly brace will output as a
single brace in the output file.

If doubling the braces gets annoying, you could use [filereplace]({{< ref
"filereplace" >}}) instead and pick your own replacement token indicators that
work better for your file.

For json, yaml and toml, where {curly braces} have structural meaning, you can
use [fileformatjson]({{< ref "fileformatjson" >}}), [fileformatyaml]({{< ref
"fileformatyaml" >}}) and [fileformattoml]({{< ref "fileformattoml" >}}).

## file format settings
`in` and `out` behave in the same way for all the format style steps. Therefore
this section's description of `in` and `out` settings applies not only to 
`fileformat`, but also 
[fileformatjson]({{< ref "fileformatjson" >}}),
[fileformatyaml]({{< ref "fileformatyaml" >}}),
[fileformattoml]({{< ref "fileformattoml" >}}) and 
[filereplace]({{< ref "filereplace" >}}).

### set input & output files
`fileformat` expects the following context keys:

- `fileFormat`
    - `in`
        - Mandatory path(s) to source file on disk.
        - This can be a string path to a single file, or a glob, or a
          list of paths and/or globs.
        - Each path can be a glob, a relative or absolute path.
    -  `out`
        - Write output file to here. Will create directories in path
          if these do not exist already.
        - `out` is optional. If not specified, will edit the `in`
          files in-place.
        - If in-path refers to >1 file (e.g it's a glob or list),
          out path can only be a directory - it doesn't make sense to
          write >1 file to the same single file output (this is not
          an appender.)
        - To ensure out path means a directory and not a file,
          be sure to have the os' path separator at the end (`/` on a sensible
          filesystem)
        - If you specify an `out` directory without a file-name, out files will
          have the same name they had in `in`.

### multiple files & globs
You can format multiple files in the same step. This works in two ways:

- `in` is glob like `./**/myfile.arb`
```yaml
fileFormat:
  # ** recurses sub-dirs per usual globbing
  in: ./testfiles/sub3/**/*.txt
  # note the dir separator at the end.
  # since >1 in files, out can only be a dir.
  out: ./out/replace/
```
- `in` is a list of paths and/or globs. 
```yaml
fileFormat:
  in:
    # ** recurses sub-dirs per usual globbing
    - ./testfiles/sub3/**/*.txt
    - ./testfiles/??b/fileformat-in.*.txt
    - ./testfiles/static.file
  # note the dir separator at the end.
  # since >1 in files, out can only be a dir.
  out: ./out/replace/
```

### in-place edit
If you do not specify `out`, pypyr will over-write (i.e edit in-place) all the 
files you specify in `in`.

```yaml
fileFormat:
  # in-place edit/overwrite all the files in. this can also be a glob, or
  # a mixed list of paths and/or globs.
  in: ./infile.txt
```

### substitutions on paths
The file in and out paths support 
[substitutions]({{< ref "docs/substitutions">}}), which allows you to specify
paths dynamically.

```yaml
- name: pypyr.steps.set
  comment: set some arb values in context
  in:
    set:
      myfilename: input-file
      myoutputfile: out/output.txt

- name: pypyr.steps.fileformat
  comment: you can set in & out entirely or partially with formatting expressions
  in:
    fileFormat:
        in: testfiles/{myfilename}.ext
        out: '{myoutputfile}'
```

## encoding
By default `in` will read and `out` will write in the platform's default
encoding. This is `utf-8` for most systems, but be aware on Windows it's still
`cp1252`.

You can use the `encoding` input explicitly to set the encoding:

```yaml
- name: pypyr.steps.fileformat
  comment: set encoding
  in:
    fileFormat:
      in: testfiles/infile.txt
      out: testfiles/outfile.txt
      encoding: utf-8
```

You can also individually set the encoding for `in` and `out`. This allows you
to convert a file from one encoding to another:

```yaml
- name: pypyr.steps.fileformat
  comment: set encoding
  in:
    fileFormat:
      in: testfiles/infile.txt
      out: testfiles/outfile.txt
      encodingIn: ascii
      encodingOut: utf-16
```

All of these are optional - if you do not explicitly over-ride the encoding for
either `in` or `out`, pypyr will just use the system default.

See here for more details on handling [text encoding in pypyr]({{< ref
"/topics/filesystem#encoding" >}}) and changing the defaults.

See here for a [list of available encoding codecs](https://docs.python.org/3/library/codecs.html#standard-encodings).