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

{{< app-window title="term" lang="text" >}}
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

# you can also use absolute paths. /mydir/mypipelinename.yaml 
$ pypyr /mydir/mypipelinename

# you can also use absolute paths. /Users/myusername/mydir/mypipelinename.yaml 
$ pypyr ~/mydir/mypipelinename
{{< /app-window >}}

## pypyr command line switches
```text
usage: pypyr [-h] [--groups [GROUPS ...]] [--success SUCCESS_GROUP]
             [--failure FAILURE_GROUP] [--dir PY_DIR] [--log LOG_LEVEL]
             [--logpath LOG_PATH] [--version]
             pipeline_name [context_args ...]

pypyr pipeline runner

positional arguments:
  pipeline_name         Name of pipeline to run. Don`t add the .yaml at the end.
  context_args          Initialize context with this. Parsed by the pipeline's context_parser
                        function.
                        Separate multiple args with spaces.

options:
  -h, --help            show this help message and exit
  --groups [GROUPS ...]
                        Step-Groups to run. defaults to "steps".
                        You probably want to order --groups AFTER the pipeline name and
                        context positional args. e.g
                        pypyr pipename context --groups group1 group2
                        If you prefer putting them before, use a -- to separate groups from
                        the pipeline name, e.g
                        pypyr --groups group1 group2 -- pipename context
  --success SUCCESS_GROUP
                        Step-Group to run on successful completion of pipeline.
                        Defaults to "on_success"
  --failure FAILURE_GROUP
                        Step-Group to run on error completion of pipeline.
                        Defaults to "on_failure"
  --dir PY_DIR          Load custom python modules from this directory.
                        Defaults to cwd (the current dir).
  --log LOG_LEVEL, --loglevel LOG_LEVEL
                        Integer log level. Defaults to 25 (NOTIFY).
                        10=DEBUG
                        20=INFO
                        25=NOTIFY
                        30=WARNING
                        40=ERROR
                        50=CRITICAL
                        Log Level < 10 gives full traceback on errors.
  --logpath LOG_PATH    Log-file path. Append log output to this path.
  --version             Echo version number.
```

## get cli help
pypyr has a couple of arguments and switches you might find useful. See
them all here:

```bash
$ pypyr -h
```
