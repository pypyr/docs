---
title: pypyr.steps.filereplace
linktitle: filereplace
date: 2020-07-03T18:00:13+01:00
description: Find & replace any arbitrary search strings in a file.
card_extra_summary:
  heading: input context property
  details: "`fileReplace` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: filereplace
seo_article_headline: Find & replace any string in a file.
seo_description: Find & replace multiple arbitrary search strings in multiple files at the same time during a pipeline task-runner step. 
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: filereplace -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [filesystem]
---
# pypyr.steps.filereplace
## find & replace arbitrary strings in a file
Parses input text file and replaces any given search strings.

The other [fileformat]({{< ref "fileformat">}}) steps, by way of 
contradistinction, uses string formatting expressions inside {braces} to format 
values against the pypyr context. 

This step, however, lets you find any arbitrary search string and replace it 
with any replacement string. This is especially handy if you are working with a 
file where curly braces aren't helpful for a formatting expression - e.g 
inside a .js or a terraform .ts file. `filereplace` is more useful than
`fileformat` when you do not control the input file format and you have to 
replace strings as they are in the given file.

Example input context:

```yaml
fileReplace:
  in: ./infile.txt
  out: ./outfile.txt
  replacePairs:
    findmestring: replacewithme
    findanotherstring: replacewithanotherstring
    alaststring: alastreplacement
```

See a worked [example of filereplace](https://github.com/pypyr/pypyr-example/tree/main/pipelines/filereplace.yaml).

## multiple files & globs
`fileReplace` expects the following context keys:

- `fileReplace`
    - `in`
      - Mandatory path(s) to source file on disk.
      - This can be a string path to a single file, or a glob, or a list of 
        paths and/or globs. 
      - Each path can be a glob, a relative or absolute path.
    - `out` (optional)
      - Write output file to here. Will create directories in path if these do 
        not exist already.
    - `replacePairs`
        - dictionary where format is:
            - `'find string': 'replace string'`

See [file format settings]({{< ref "fileformat#file-format-settings" >}}) for 
more examples on in/out path handling - the same processing rules apply.

Here is an example with globs in a list. You can also pass a single glob as 
`in` directly as a string, not in a list.

```yaml
fileReplace:
  in:
    # ** recurses sub-dirs per usual globbing
    - ./testfiles/replace/sub/**
    - ./testfiles/replace/*.ext
  # note the dir separator at the end.
  # since >1 in files, out can only be a dir.
  out: ./out/replace/
  replacePairs:
      findmestring: replacewithme
```

If you do not specify `out`, it will over-write (i.e edit) all the files
specified by `in`.

```yaml
fileReplace:
  # in-place edit/overwrite all the files in
  in: ./infile.txt
  replacePairs:
    findmestring: replacewithme
```

The file `in` and `out` paths support 
[substitutions]({{< ref "docs/substitutions">}}).

## replace with substitution expressions
`fileReplace` also does [substitutions]({{< ref "docs/substitutions">}}) from 
context on the `replacePairs` for both the search string and the replacement 
string. pypyr does this before it search & replaces the `in` files. This allows 
you to specify dynamic search strings and dynamic replacement values.

Given an input file like this:

```text
“the beginning thy REPEATING-WORD, thy happy REPEATING-WORD,
   Sing thy songs of happy cheer!”
So I LOOK FOR ME the same again,
   <<manywords>> to arbPyExpression.
```

And a pipeline like this:

```yaml
steps:
  - name: pypyr.steps.filereplace
    comment: replace multiple string in a file.
    in:
      k1: LOOK FOR ME
      k2: wept with
      fileReplace:
        in: ./filereplace.in
        out: ./filereplace.out
        replacePairs:
          # replace many words with one word
          the beginning: Drop
          # replace the same term multiple times in same file
          REPEATING-WORD: pipe
          # dynamically set search term using formatting expression
          '{k1}': sang
          # dynamically set replacement term 
          # sometimes angled brackets make search terms easier for humans to read.
          <<manywords>>: While he {k2} joy
          # you can also use py strings - here we reverse a string with python.
          arbPyExpression: !py '"raeh"[::-1]'
```

The resulting output file is:

```text
“Drop thy pipe, thy happy pipe,
   Sing thy songs of happy cheer!”
So I sang the same again,
   While he wept with joy to hear.
```

## replacement order
Be careful of order. The last string replacement expression could well
replace a replacement that an earlier replacement made in the sequence.

```yaml
- name: pypyr.steps.filereplace
  in:
    fileReplace:
      in: ./in.txt
      out: ./out.txt
      replacePairs:
        replaceme: INTERMEDIATE
        # later replaces can affect earlier replaces
        INTERMEDIATE: final
```

Given the input file:

```text
some text replaceme end.
```

The output is:

```text
some text final end.
```

Although this example is a bit contrived, it is something to watch out for when
you are using dynamic string expressions where it might not be immediately 
obvious that a later search string is finding an earlier replacement.

{{% note tip %}}
If `replacePairs` is not an ordered collection, replacements could
evaluate in any given order. This is relevant if you are a coder.

If you are creating your `in` parameters in
the pipeline yaml, don't worry about it, it will be an ordered
dictionary already, so life is good.
{{% /note %}}

## special characters
If your search or replacement string has {curly braces}, pypyr will treat what
is inside the curly brace as a normal pypyr formatting substitution expression.

If you want to search for and replace strings with literal {curlies}, you have
to escape the literals by {{doubling}} the braces.

Given an input file like this:

```text
Piping down the {arb braces} wild
Piping manywords
```

And a pipeline like this:

```yaml
steps:
  - name: pypyr.steps.filereplace
    comment: escape literal curly braces by doubling
    in:
      arb_key: of pleasant
      fileReplace:
        in:
          - ./in/*.txt
          - ./in/sub/*
        out: ./out/replace/
        replacePairs:
            # notice escaping literal curlies by doubling
            '{{arb braces}}': valleys
            # {arb_key} will replace from content
            manywords: songs {arb_key} glee
```

The output is:

```text
Piping down the valleys wild
Piping songs of pleasant glee
```