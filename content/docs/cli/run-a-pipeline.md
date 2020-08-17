---
title: run a pipeline with the pypyr cli
description: How to run a pypyr pipeline yaml from the cli
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
categories: [pipelines]
menu:
  docs:
    name: run a pipeline
    parent: cli
weight: -10
draft: false
seo_article_headline: How to run a task-runner automation pipeline from the cli.
seo_description: Run your yaml automation pipelines easily with the simple pypyr cli.
# topics: []
---
# run a pipeline from the cli
## pass arguments & command line switches to the cli
pypyr runs the pipeline specified by the name that you pass to the cli.

To make your pipelines edit easier in your favorite yaml editor, use a
.yaml extension, but to save on typing you don't need to enter the
.yaml bit at the command line.

You can use your usual directory separators if you're [running a pipeline in a
sub-directory]({{< ref "/docs/pipelines/lookup-order#pipelines-in-sub-directories" >}}), 
like `$ pypyr subdir/subsubdir/pipeline`

```bash
# run ./mypipelinename.yaml with DEBUG logging level
$ pypyr mypipelinename --loglevel 10

# run ./mypipelinename.yaml with INFO logging level.
# log is an alias for loglevel, so less typing, wooohoo!
$ pypyr mypipelinename --log 20

# If you don't specify --loglevel it defaults to 25 - NOTIFY.
$ pypyr mypipelinename

# run ./mydir/mypipelinename.yaml
# The 2nd argument is any arbitrary sequence of strings, known
# as the input context arguments.
# For this input argument to be available to your pipeline
# you need to specify a context parser in your pipeline yaml.
$ pypyr mydir/mypipelinename arbitrary string here

# run ./mypipelinename.yaml with an input context in key-value
# pair format. For this input to be available to your pipeline
# you need to specify a context_parser like
# pypyr.parser.keyvaluepairs in your pipeline yaml.
$ pypyr mypipelinename mykey=value anotherkey=anothervalue
```

## pypyr command line switches
```fish
pypyr [-h] [--groups [GROUPS [GROUPS ...]]] [--success SUCCESS_GROUP]
             [--failure FAILURE_GROUP] [--dir WORKING_DIR] [--log LOG_LEVEL]
             [--logpath LOG_PATH] [--version]
             pipeline_name [context_args [context_args ...]]
```

## get cli help
pypyr has a couple of arguments and switches you might find useful. See
them all here:

```bash
$ pypyr -h
```
