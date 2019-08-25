---
title: pypyr.parser.yamlfile
linktitle: yamlfile
date: 2020-07-09T17:42:16+01:00
description: Put yaml file into context from cli.
draft: false
card_extra_summary:
  heading: example input
  details: "`pypyr pipelinename ./path/sample.yaml`"
categories: [context]
# keywords: ""
menu:
  docs:
    parent: context-parsers
    name: yamlfile
seo_article_headline: Pass values from yaml file to task-runner pipeline.
seo_description: Read a yaml file from disk & pass the strongly typed values to the pipeline's context. Use yaml file inside pipeline.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: yamlfile -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [args]
---
# pypyr.parser.yamlfile
## read yaml file into context
Opens a yaml file from the input path and use it to initialize the pypyr context dictionary. Strongly typed values in the source yaml will translate into the
pipeline context with the correct type. This lets you initialize pipeline 
context from a yaml file.

The input path can be relative or absolute. Relative paths are relative to the
current working directory.

## example
Given a yaml file like this, saved as `./sample-files/sample.yaml`:

```yaml
key1: value1
key2: value 2
key3: "value3 and {key1} and {key2}"
echoMe: "this is a value from a yaml file"
keyInt: 123
keyBool: True
```

And given a pipeline like this, arbitrarily saved as `./yamlfile-parser.yaml`:

```yaml
# ./yamlfile-parser.yaml
context_parser: pypyr.parser.yamlfile
steps:
  # echoMe is set in the yaml file
  - pypyr.steps.echo
  - name: pypyr.steps.contextsetf
    comment: use the int as a strongly typed int
    in:
        contextSetf:
            intResult: !py keyInt * 2
  - name: pypyr.steps.echo
    comment: formatting expressions in yaml file will work
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
$ pypyr yamlfile-parser ./sample-files/sample.yaml
this is a value from a yaml file
value3 and value1 and value 2
strongly typed happens automatically: 246
you'll only see me if the bool was true
```

If you change the `keyBool` bool to `False`:

```text
$ pypyr yamlfile-parser ./sample-files/sample.yaml
this is a value from a yaml file
value3 and value1 and value 2
strongly typed happens automatically: 246
```

## yaml structure
The top (or root) level yaml should describe a map, not a sequence.

Sequence (this won't work):

```yaml
- thing1
- thing2
```

Instead, do a map (aka dictionary):

```yaml
thing1: thing1value
thing2: thing2value
```

## spaces in file path
Depending on your O/S, shell and file-system, it's up to you to deal with 
special characters in the file-name. 

Taking MacOS as an example, all of the following will work:

```bash
$ pypyr yamlfilein ./testfiles/sample.yaml
$ pypyr yamlfilein ./test files/sample with space.yaml
$ pypyr yamlfilein "./test files/sample with space.yaml"
$ pypyr yamlfilein ./"test files"/"sample with space.yaml"
```