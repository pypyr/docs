---
title: pypyr.steps.fileformatyaml
linktitle: fileformatyaml
date: 2020-07-02T17:23:36+01:00
description: Find & replace substitution {tokens} in a yaml file.
card_extra_summary:
  heading: input context property
  details: "`fileFormatYaml` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: fileformatyaml
seo_article_headline: Format, find & replace tokens in yaml files.
seo_description: Find & replace tokens in multiple yaml files with correct data types. Like sed for yaml, but type safe.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: fileformatyaml -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [filesystem, yaml]
---
# pypyr.steps.fileformatyaml
## find & replace tokens in yaml file
Parses input yaml file and substitutes {tokens} from the pypyr context.

Pretty much does the same thing as
[pypyr.steps.fileformat]({{< ref "fileformat" >}}), only it makes it easier to 
work with curly braces for substitutions without tripping over the yaml's 
structural braces.

If your yaml doesn't use curly braces that aren't meant for {token} 
substitutions, you can happily use [pypyr.steps.fileformat]({{< ref "fileformat" >}}) 
instead - it's more memory efficient.

This step does not preserve comments. Use `fileformat` if you need to preserve comments on output.

Given input yaml like this:
```yaml
k1: v1
k2:
  k2.1: v2.1
  k2.2:
    - 2.2.1,
    - START {replaceMeNested} END
k3: "{replaceMeString}"
k4: "{replaceMeInt}"
k5: "{replaceMeBool}"
"{replaceMeKey}": "this will replace the key"
```

And a pipeline like this:
```yaml
steps:
  - name: pypyr.steps.fileformatyaml
    comment: read a yaml from disk, do some substitutions, write back out.
    in:
      replaceMeString: this was replaced by pypyr
      replaceMeInt: 420
      replaceMeBool: false
      replaceMeNested: doesn't matter where you are in the nesting structure
      replaceMeKey: keyfrompypyr
      fileFormatYaml:
          in: ./sample-files/sample.yaml
          out: ./out/

```

The formatted output yaml file will be:
```yaml
k1: v1
k2:
  k2.1: v2.1
  k2.2:
    - 2.2.1,
    - START doesn't matter where you are in the nesting structure END
k3: this was replaced by pypyr
k4: 420
k5: false
keyfrompypyr: this will replace the key
```

## type-safe token replacement
Notice that you can replace values in the yaml document and keep the correct 
type - so numbers are numbers, booleans are booleans and so forth.

Even though you always have to set the `"{replacement_expression}"` inside 
quotes in the source yaml to ensure valid yaml, pypyr will output the correct 
type based on the value to which the expression evaluates.

You can also use replacement expressions in the yaml document's keys.

## multiple files & globs
`fileformatyaml` expects the following context keys:

- `fileFormatYaml`
    - `in`
      - Mandatory path(s) to source file on disk.
      - This can be a string path to a single file, or a glob, or a list of 
        paths and globs. 
      - Each path can be a glob, a relative or absolute path.
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

Example with a glob input and a normal path in a list:

```yaml
fileFormatYaml:
  in: [./file1.yaml, ./testfiles/sub3/**/*.yaml]
  # note the dir separator at the end.
  # since >1 in files, out can only be a dir.
  out: ./out/replace/
```

If you do not specify `out`, it will over-write (i.e in-place edit) all the 
files specified by `in`.

The file in and out paths support 
[substitutions]({{< ref "docs/substitutions">}}), which allows you to specify
paths dynamically.

See a worked [example of fileformatyaml](https://github.com/pypyr/pypyr-example/blob/main/pipelines/fileformatyaml.yaml).

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
      myoutputfile: out/output.yaml

- name: pypyr.steps.fileformatyaml
  comment: you can set in & out entirely or partially with formatting expressions
  in:
    fileFormatYaml:
        in: testfiles/{myfilename}.yaml
        out: '{myoutputfile}'
```