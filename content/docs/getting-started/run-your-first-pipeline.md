---
title: run your first pipeline
description: How to run your first pypyr pipeline
date: 2019-08-21
publishdate: 2019-08-21
lastmod: 2019-08-21
categories: [install]
menu:
  docs:
    title: run your first pipeline
    parent: getting-started
draft: false
weight: -5
seo_article_headline: Tutorial showing you how to run your first task-runner pipeline.
seo_description: An introduction to how to write & run your first pipeline with the pypyr task-runner to automate your own tasks.
# topics: [pipeline, tutorial, args]
---
# run your first pipeline
## run a built-in pipeline
Run one of the built-in pipelines to get a feel for it:

```bash
$ pypyr echo "Ceci n'est pas une pipe"
```

`echo` is the name of a built-in pypyr pipeline. The pipeline simply echoes
the input string back to console output using the built-in [echo step]({{< ref "/docs/steps/echo">}}). 

The actual pipeline looks like this:

```yaml
# To execute this pipeline, shell something like:
# pypyr echo text goes here
context_parser: pypyr.parser.string
steps:
  - name: pypyr.steps.contextset
    comment: assign input arg to echoMe so echo step can echo it
    in:
      contextSet:
        echoMe: argString
  - pypyr.steps.echo
```

You can achieve the same thing by running a pipeline where you set the context 
in the pipeline yaml rather than passing in as the 2nd positional
argument:

```bash
$ pypyr magritte
```

```yaml
# To execute this pipeline, shell something like:
# pypyr magritte
steps:
  - name: pypyr.steps.echo
    comment: output echoMe
    in:
      echoMe: Ceci n'est pas une pipe
```

## write your first pipeline
A pipeline is simply a yaml file.

The simplest pipeline just needs a `steps` list. This is the default entry
point. `steps` is a list that has a bunch of individual steps in it that will
run sequentially one after the other.

```yaml
# ./my-first-pipeline.yaml
# To execute this pipeline, shell something like:
# pypyr my-first-pipeline
steps:
  - name: pypyr.steps.echo
    comment: output echoMe
    in:
      echoMe: this is step 1
  - name: pypyr.steps.cmd
    comment: actually invokes any program on your system.
             the program must be available in current path.
             run any program you fancy.
             we're just running echo here for an easy demo.
    in:
      cmd: echo this is step 2
```

Save this file as `./my-first-pipeline.yaml`.

Now you can run your pipeline like this:

{{< app-window title="term" lang="text" >}}
$ pypyr my-first-pipeline
this is step 1
this is step 2

$
{{< /app-window >}}

## passing arguments to your pipeline
You can pass arguments from the command line to your pipeline easy peasy.

All you need to do is add a [context_parser]({{< ref "/docs/context-parsers" >}})

In this example we'll be using a key-value pair parser, which will let you specify
your own key-value pairs so you can access the values you need by key. There
are other types of context parser that can parse the cmd input differently for
you.

The context parser will add these values into the pypyr context for you. Your
pipeline steps can then use these values as they please with 
[{formatting expressions}]({{< ref "/docs/substitutions">}}). 

```yaml
# ./pipe-with-args.yaml
# To execute this pipeline, shell something like:
# pypyr pipe-with-args akey=avalue anotherkey="another arbitrary value"
context_parser: pypyr.parser.keyvaluepairs
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: this is akey's value {akey}. It came from the cli args.
  - name: pypyr.steps.cmd
    in:
      cmd: echo we can inject {anotherkey} into your cmd
```

Save this file as `./pipe-with-args.yaml`.

Now you can run this pipeline like this:

{{< app-window title="term" lang="text" >}}
$ pypyr pipe-with-args akey=avalue anotherkey="another value"
this is akey's value avalue. It came from the cli args.
we can inject another value into your cmd

$
{{< /app-window >}}
