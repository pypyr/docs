---
title: run your first pipeline
description: How to run your first pypyr pipeline
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
# categories: [install]
menu:
  docs:
    title: run your first pipeline
    parent: getting-started
weight: -80
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
the input string back to console output using the built-in
[echo step]({{< ref "/docs/steps/echo">}}). 

The actual pipeline looks like this:

```yaml
# To execute this pipeline, shell something like:
# pypyr echo text goes here
context_parser: pypyr.parser.string
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: '{argString}'
```

You can achieve the same thing by running a pipeline where you set the context
in the pipeline yaml rather than passing it in from the cli as the 2nd
positional argument:

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

The second step here is the [cmd step]({{< ref "/docs/steps/cmd">}}) - this
built-in step allows you to execute any command available on your system. For
the sake of hello-world simplicity, we're just using good old `echo`, but you
can substitute this with something more useful.

pypyr has a whole bunch of ready-made [built-in steps]({{< ref "/docs/steps"
>}}) that you can use for various tasks such as working with files, variables,
merging configuring, paths & custom inline code. You can also make your own
[custom step]({{< ref "/docs/api/step" >}}) without fuss.

## passing arguments to your pipeline
You can pass arguments from the cli to your pipeline easy peasy.

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

## looping over commands
You can add extra functionality to *any* step in pypyr by decorating the step
with a [decorator]({{< ref "/docs/decorators" >}}). pypyr gives you built-in
decorators to loop, retry a failure condition automatically and selectively
run a step only if a certain condition is true - all of this without you having
to code it yourself.

### static loop
Let's add a simple loop to a command using the
[foreach decorator]({{< ref "/docs/decorators/foreach" >}}):
```yaml
# ./fruit-loop.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin

  - name: pypyr.steps.cmd
    foreach: [apple, pear, banana]
    in:
      cmd: echo arb-command --arg {i}

  - name: pypyr.steps.echo
    in:
      echoMe: end
```

Notice that for the `cmd` step we're just using `echo` for the sake of example,
you can instead use whatever command available on your system.

`{i}` is a context variable that contains the current item of the iterator.

Running this pipeline looks like this:
{{< app-window title="term" lang="text" >}}
$ pypyr fruit-loop
begin
arb-command --arg apple
arb-command --arg pear
arb-command --arg banana
end

$
{{< /app-window >}}

### dynamic loop cli input args
Of course, you don't have to hard-code the list the you want to iterate. Let's
inject the values from the cli. As always, use a context parser to access
cli arguments: in this case,
[pypyr.parser.list]({{< ref "/docs/context-parsers/list" >}}) will parse all
the input arguments into a list named `{argList}` for you.

```yaml
# ./fruit-loop
context_parser: pypyr.parser.list
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin

  - name: pypyr.steps.cmd
    foreach: '{argList}'
    in:
      cmd: echo arb-command --arg {i}

  - name: pypyr.steps.echo
    in:
      echoMe: end
```

Now you can control dynamically how many to times to run your command from the
cli like this:

{{< app-window title="term" lang="text" >}}
$ pypyr fruit-loop
begin
end

$ pypyr fruit-loop apple pear
begin
arb-command --arg apple
arb-command --arg pear
end

{{< /app-window >}}

The first run, without any arguments, just prints `begin` and `end`: since the
input list is empty there is nothing over which to loop.

## what next?
You probably should check out:
- understanding [basic pipeline structure]({{< ref "basic-concepts" >}})
- How to [declare & use variables]({{< ref "variables" >}})
- How to use [loops]({{< ref "loops" >}})
- Deal with [errors]({{< ref "error-handling">}})