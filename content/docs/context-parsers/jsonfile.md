---
title: pypyr.parser.jsonfile
linktitle: jsonfile
date: 2020-07-09T11:19:55+01:00
description: Put json file into context from cli.
draft: false
card_extra_summary:
  heading: example input
  details: '`pypyr pipelinename ./path/sample.json`'
categories: [context]
# keywords: ""
menu:
  docs:
    parent: context-parsers
    name: jsonfile
seo_article_headline: Pass values from json file to task-runner pipeline.
seo_description: Read a json file from disk & pass the strongly typed values to the pipeline's context. Use json file inside pipeline.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: jsonfile -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [args]
---
# pypyr.parser.jsonfile
## read json file into context
Takes a path from the cli input argument, and read the json file at that path
into context. Strongly typed values in the source json will translate into the
pipeline context. This lets you initialize pipeline context from a json file.

The input path can be relative or absolute. Relative paths are relative to the
current working directory.

## example
Given a json file like this, saved as `./sample-files/sample.json`:

```json
{
    "key1": "value1",
    "key2": "value2",
    "key3": "value3 and {key1} and {key2}",
    "echoMe": "this is a value from a json file",
    "keyInt": 123,
    "keyBool": true
}
```

And given a pipeline like this, arbitrarily saved as `./jsonfile-parser.yaml`:

```yaml
# ./jsonfile-parser.yaml
context_parser: pypyr.parser.jsonfile
steps:
  # echoMe is set in the json input file
  - pypyr.steps.echo
  - name: pypyr.steps.contextsetf
    comment: use the int as a strongly typed int
    in:
        contextSetf:
            intResult: !py keyInt * 2
  - name: pypyr.steps.echo
    comment: formatting expressions in json file will work
    in:
        echoMe: '{key3}'
  - name: pypyr.steps.echo
    comment: print out result of int calculation
    in:
        echoMe: "strongly typed happens automatically: {intResult}"
  - name: pypyr.steps.echo
    comment: only run if input bool is true
    run: '{keyBool}'
    in:
        echoMe: you'll only see me if the bool was true
```

You can run this pipeline as follows:
```text
$ pypyr jsonfile-parser ./sample-files/sample.json
this is a value from a json file
value3 and value1 and value2
strongly typed happens automatically: 246
you'll only see me if the bool was true
```

If you change the `keyBool` bool to `false`:

```text
$ pypyr jsonfile-parser ./sample-files/sample.json
this is a value from a json file
value3 and value1 and value2
strongly typed happens automatically: 246
```

## json structure
The top root level json should not be an Array `[]`, but rather an Object `{}`.

Array (this won't work):

```json
[
  "eggs",
  "ham"
]
```

Instead, do an Object:

```json
{
  "breakfastOfChampions": [
    "eggs",
    "ham"
  ]
}
```

## spaces in file path
Depending on your O/S, shell and file-system, it's up to you to deal with 
special characters in the file-name. 

Taking MacOS as an example, all of the following will work:

```bash
$ pypyr jsonfilein ./testfiles/sample.json
$ pypyr jsonfilein ./test files/sample with space.json
$ pypyr jsonfilein "./test files/sample with space.json"
$ pypyr jsonfilein ./"test files"/"sample with space.json"
```