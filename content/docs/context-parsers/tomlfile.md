---
title: pypyr.parser.tomlfile
linktitle: tomlfile
date: 2021-12-09T11:19:55+01:00
description: Load toml file into context from a cli arg path input.
card_extra_summary:
  heading: example input
  details: '`pypyr pipelinename ./path/sample.toml`'
categories: [context parsers]
# keywords: ""
menu:
  docs:
    parent: context-parsers
    name: tomlfile
seo_article_headline: Load values from toml file into a task-runner pipeline.
seo_description: Read a toml file & pass the strongly typed values to the pipeline's context to use the toml values inside pipeline.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: tomlfile -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [args, filesystem, toml]
---
# pypyr.parser.tomlfile
## read toml file into context
Take a path from the cli input argument, and read the toml file at that path
into context. Strongly typed values in the source toml will translate into the
pipeline context (in other words, integers will be integers, booleans will be
booleans etc.). This lets you initialize the pipeline context from a toml file.

The input path can be relative or absolute. Relative paths are relative to the
current working directory.

## example
Given a toml file like this, saved as `./sample-files/sample.toml`:

```toml
#this is a toml example
echoMe = "this is a value from a toml file"
key1 = "value1"
key2 = "value2"
keyBool = true
keyInt = 123

[mytable]
mytablekey = "{key1} and {key2} and {key3}"
```

And given a pipeline like this, arbitrarily saved as `./tomlfile-parser.yaml`:

```yaml
# ./tomlfile-parser.yaml
context_parser: pypyr.parser.tomlfile
steps:
  # echoMe is set in the toml input file
  - pypyr.steps.echo

  - name: pypyr.steps.set
    comment: use the int from toml as a strongly typed int
    in:
      set:
        intResult: !py keyInt * 2
        key3: value3
  
  - name: pypyr.steps.echo
    comment: print out result of int calculation
    in:
      echoMe: "strongly typed happens automatically: {intResult}"

  - name: pypyr.steps.echo
    comment: access a key in a table
             formatting expressions in toml string will work
    in:
      echoMe: "{mytable[mytablekey]}"
    
  - name: pypyr.steps.echo
    comment: only run if input bool is true
    run: '{keyBool}'
    in:
      echoMe: you'll only see me if the bool was true
```

You can run this pipeline as follows:
{{< app-window title="term" lang="text" >}}
$ pypyr tomlfile-parser ./sample-files/sample.toml
this is a value from a toml file
strongly typed happens automatically: 246
value1 and value2 and value3
you'll only see me if the bool was true
{{< /app-window >}}

Notice that `mytable.mytablekey` uses pypyr {substitution} expressions where
`key1` and `key2` are set in the toml file itself, and `key3` is set in the
pypyr pipeline with [pypyr.steps.set]({{< ref "/docs/steps/set" >}}).

If you change the `keyBool` bool to `false` in the toml input file, the last
step won't run anymore because the `run` condition is `False`.

{{< app-window title="term" lang="text" >}}
$ pypyr tomlfile-parser ./sample-files/sample.toml
this is a value from a toml file
strongly typed happens automatically: 246
value1 and value2 and value3
{{< /app-window >}}

## spaces in file path
Depending on your O/S, shell and file-system, it's up to you to deal with 
special characters in the file-name. 

Taking either zsh or fish as an example, all of the following will work:

{{< app-window title="term" lang="text" >}}
$ pypyr mypipeline ./testfiles/sample.toml
$ pypyr mypipeline ./test files/sample with space.toml
$ pypyr mypipeline "./test files/sample with space.toml"
$ pypyr mypipeline ./"test files"/"sample with space.toml"
{{< /app-window >}}