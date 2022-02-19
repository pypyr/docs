---
title: pypyr.steps.fileformatjson
linktitle: fileformatjson
date: 2020-07-02T17:23:30+01:00
description: Find & replace substitution {tokens} in a json file.
card_extra_summary:
  heading: input context property
  details: "`fileFormatJson` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: fileformatjson
seo_article_headline: Format, find & replace tokens in json file.
seo_description: Find & replace tokens in multiple json files with correct data types. Like sed for json, but type safe.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: fileformatjson -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [filesystem, json]
---
# pypyr.steps.fileformatjson
## find & replace tokens in json
Parses input json file and substitutes {tokens} from the pypyr context.

Pretty much does the same thing as 
[pypyr.steps.fileformat]({{< ref "fileformat" >}}), only it makes it
easier to work with curly braces for substitutions without tripping over
the json's structural braces.

Given input json like this:
```json
{
  "k1": "v1",
  "k2": {
    "k2.1": "v2.1",
    "k2.2": [
      "2.2.1",
      "START {replaceMeNested} END"
    ]
  },
  "k3": "{replaceMeString}",
  "k4": "{replaceMeInt}",
  "k5": "{replaceMeBool}",
  "{replaceMeKey}": "this will replace the key"
}
```

And a pipeline like this:
```yaml
steps:
  - name: pypyr.steps.fileformatjson
    comment: read json file, do some substitutions, write back out.
    in:
      replaceMeString: this was replaced by pypyr
      replaceMeInt: 420
      replaceMeBool: false
      replaceMeNested: doesn't matter where you are in the nesting structure
      fileFormatJson:
        in: ./sample-files/sample.json
        out: ./out/
```

The formatted output json file will be:
```json
{
  "k1": "v1",
  "k2": {
    "k2.1": "v2.1",
    "k2.2": [
      "2.2.1",
      "START doesn't matter where you are in the nesting structure END"
    ]
  },
  "k3": "this was replaced by pypyr",
  "k4": 420,
  "k5": false,
  "keyfrompypyr": "this will replace the key"
}
```

## type-safe token replacement
Notice that you can replace values in the json document and keep the correct 
type - so numbers are numbers, booleans are booleans and so forth.

Even though you always have to set the `"{replacement_expression}"` inside 
quotes in the source json to ensure valid json, pypyr will output the correct 
type based on the value to which the expression evaluates.

You can also use replacement expressions in the json object's keys.

## multiple files & globs
`fileformatjson` expects the following context keys:

- `fileFormatJson`
    - `in`
      - Mandatory path(s) to source file on disk.
      - This can be a string path to a single file, or a glob, or a list of 
        paths and/or globs. 
      - Each path can be a glob, a relative or an absolute path.
    - `out` (optional)
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
        filesystem).
      - If you specify an `out` directory without a file-name, out files will
        have the same name they had in `in`.

See [file format settings]({{< ref "fileformat#file-format-settings" >}}) for 
more examples on in/out path handling - the same processing rules apply.

Example with a glob input:

```yaml
fileFormatJson:
  in: ./testfiles/sub3/**/*.json
  # note the dir separator at the end.
  # since >1 in files, out can only be a dir.
  out: ./out/replace/
```

If you do not specify `out`, it will over-write (i.e in-place edit) all the 
files specified by `in`.

The file in and out paths support 
[substitutions]({{< ref "docs/substitutions">}}), which allows you to specify
paths dynamically.

See a worked [example of fileformatjson](https://github.com/pypyr/pypyr-example/blob/main/pipelines/fileformatjson.yaml).

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
      myoutputfile: out/output.json

- name: pypyr.steps.fileformatjson
  comment: you can set in & out entirely or partially with formatting expressions
  in:
    fileFormatJson:
        in: testfiles/{myfilename}.json
        out: '{myoutputfile}'
```

## encoding
By default `in` will read and `out` will write in the platform's default
encoding. This is `utf-8` for most systems, but be aware on Windows it's still
`cp1252`.

You can use the `encoding` input explicitly to set the encoding:

```yaml
- name: pypyr.steps.fileformatjson
  comment: set encoding
  in:
    fileFormatJson:
      in: testfiles/infile.json
      out: testfiles/outfile.json
      encoding: utf-8
```

You can also individually set the encoding for `in` and `out`. This allows you
to convert a file from one encoding to another:

```yaml
- name: pypyr.steps.fileformatjson
  comment: set encoding
  in:
    fileFormatJson:
      in: testfiles/infile.json
      out: testfiles/outfile.json
      encodingIn: ascii
      encodingOut: utf-16
```

All of these are optional - if you do not explicitly over-ride the encoding for
either `in` or `out`, pypyr will just use the system default.

See here for more details on handling [text encoding in pypyr]({{< ref
"/topics/filesystem#encoding" >}}) and changing the defaults.

See here for a [list of available encoding codecs](https://docs.python.org/3/library/codecs.html#standard-encodings).