---
title: pypyr.steps.fileformattoml
linktitle: fileformattoml
date: 2020-07-11T17:23:36+01:00
description: Find & replace substitution {tokens} in a toml file.
card_extra_summary:
  heading: input context property
  details: "`fileFormatToml` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: fileformattoml
seo_article_headline: Format, find & replace tokens in toml files.
seo_description: Find & replace tokens in multiple toml files with correct data types. Like sed for toml, but type safe.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: fileformattoml -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [filesystem, toml]
---
# pypyr.steps.fileformattoml
## find & replace tokens in toml file
Parses input toml file and creates an output toml file while substituting
{tokens} in the source toml from the pypyr context.

Pretty much does the same thing as
[pypyr.steps.fileformat]({{< ref "fileformat">}}), only it makes it easier to
work with curly braces for substitutions without tripping over any structural
curly braces in the source toml.

This step does not preserve comments or whitespace. Use `fileformat` or
[filereplace]({{< ref "filereplace" >}}) if you want to preserve
comments/whitespace.

Given an input toml file like this `./myfile.toml`:
```toml
k1 = "v1"
k3 = "{replaceMeString}"
k4 = "{replaceMeInt}"
k5 = "{replaceMeBool}"
"{replaceMeKey}" = "this will replace the key"

[table]
"k2.1" = "v2.1"

[table.sub]
mylist = [
  "2.2.1",
  "START {replaceMeNested} END",
  {first = "{replaceMeInline}", last = 456},
]
```

And a pipeline like this:
```yaml
steps:
  - name: pypyr.steps.set
    comment: set some arbitrary values in context
    in:
      replaceMeString: this was replaced by pypyr
      replaceMeInt: 420
      replaceMeBool: false
      replaceMeNested: doesn't matter where you are in the nesting structure
      replaceMeKey: keyfrompypyr
      replaceMeInline: 123

  - name: pypyr.steps.fileformattoml
    comment: read a toml from disk, do some substitutions, write back out.
    in:
      fileFormatToml:
          in: my-file.toml
          out: out/
```

The formatted output toml file at `./out/my-file.toml` looks like this:
```toml
k1 = "v1"
k3 = "this was replaced by pypyr"
k4 = 420
k5 = false
keyfrompypyr = "this will replace the key"

[table]
"k2.1" = "v2.1"

[table.sub]
mylist = [
    "2.2.1",
    "START doesn't matter where you are in the nesting structure END",
    { first = 123, last = 456 },
]
```

Notice the [inline table](https://toml.io/en/latest#inline-table) at the last
item in the `mylist` array: `{first = "{replaceMeInline}", last = 456}`. If your
toml does NOT use curly braces for inline tables, you can happily use
[pypyr.steps.fileformat]({{< ref "fileformat" >}}) instead of `fileformattoml` -
it's more memory efficient and it will preserve comments & whitespace.

## type-safe token replacement
Notice that you can replace values in the toml document and keep the correct 
type - so numbers are numbers, booleans are booleans and so forth.

Even though you always have to set the `"{replacement_expression}"` inside
quotes in the source to ensure valid toml, pypyr will output the correct type
based on the value to which the expression evaluates.

You can also use replacement expressions in the toml document's keys.

## multiple files & globs
`fileformattoml` expects the following context keys:

- `fileFormatToml`
    - `in`
      - Mandatory path(s) to source file on disk.
      - This can be a string path to a single file, or a glob, or a list of 
        paths and globs. 
      - Each path can be a glob, a relative or absolute path.
    - `out` (optional)
      - Write output file to here. Will create directories in path if these do
        not exist already.
      - `out` is optional. If not specified, will edit the `in` files in-place.
      - If in-path refers to >1 file (e.g it's a glob or list), out path can
        only be a directory - it doesn't make sense to write >1 file to the same
        single file output (this is not an appender.)
      - To ensure out path means a directory and not a file, be sure to have the
        os path separator at the end (`/` on a sensible filesystem).
      - If you specify an `out` directory without a file-name, out files will
        have the same name they had in `in`.

See [file format settings]({{< ref "fileformat#file-format-settings" >}}) for 
more examples on in/out path handling - the same processing rules apply.

Example with a glob input and a normal path in a list:

```yaml
fileFormatToml:
  in: [./file1.toml, ./testfiles/sub3/**/*.toml]
  # note the dir separator at the end.
  # since >1 in files, out can only be a dir.
  out: ./out/replace/
```

If you do not specify `out`, it will over-write (i.e in-place edit) all the 
files specified by `in`.

## substitutions on paths
The file in and out paths support 
[substitutions]({{< ref "docs/substitutions">}}), which allows you to specify
paths dynamically.

```yaml
- name: pypyr.steps.set
  comment: set some arb values in context
  in:
    set:
      myfilename: input-file
      myoutputfile: out/output.toml

- name: pypyr.steps.fileformattoml
  comment: you can set in & out entirely or partially with formatting expressions
  in:
    fileFormatToml:
        in: testfiles/{myfilename}.toml
        out: '{myoutputfile}'
```

## encoding
A TOML file must always be in `utf-8`, per the [TOML
Spec](https://toml.io/en/latest#spec).