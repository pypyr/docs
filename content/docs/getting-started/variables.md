---
title: variables
description: How to use variables & arguments in a pipeline.
date: 2021-09-06
# categories: [install]
menu:
  docs:
    title: variables
    parent: getting-started
weight: -60
seo_article_headline: How to use variables in a task-runner pipeline.
seo_description: Use local & global variables in a pypyr pipeline to parameterize your pipelines.
topics: [context]
---
# how to use variables in a pypyr pipeline
You can use variables to parameterize your pypyr pipelines and pass values
between steps. pypyr lets you pass values seamlessly between the pipeline yaml,
the cli (or api) and the steps in the pipeline.

pypyr stores variables in the [context]({{< ref
"/docs/getting-started/basic-concepts#context">}}). The context is a dictionary
that stays in scope for the duration of the entire pipeline.

Variables can be simple types like string or int, and they can also contain
complex nested structures like dictionaries or lists. So for example, a variable
could be a dictionary mapping, which in turn contains lists/arrays, and those
lists in turn contain other dicts or lists.

## global variables
After you set a global variable it is available to all the subsequent steps in
the pipeline.

You can instantiate & set your variables in different ways:

### inject arguments from the cli
You can inject cli arguments into a pipeline by using a [context parser]({{< ref
"/docs/context-parsers" >}}) in the pipeline.

```fish
$ pypyr mypipeline these are all context input arguments
$ pypyr mypipeline var1=value1 var2="value 2"
```

These values stay in the scope for the duration of the pipeline's execution,
unless you explicitly unset with [clear]({{< ref "/docs/steps/contextclear">}})
or explicitly overwrite them in a step.

### set variable in a step
You can set variables using the following steps:
- [set]({{< ref "/docs/steps/set" >}})
- [default]({{< ref "/docs/steps/default" >}})

These variables are available to all subsequent steps.

The difference between `default` and `set` is that `default` _only_ sets
a variable if it does not exist already. It will not overwrite existing
variables. This makes `default` especially handy if you want to provide default
values for optional arguments coming from the cli.

By comparison, `set` will always set the variable, overwriting a variable even
if it exists already.

```yaml
steps:
  - name: pypyr.steps.set
    comment: set some arbitrary variables
    in:
      set:
        var1: my value
        var2: 2
        var3: True

  - name: pypyr.steps.echo
    comment: use the variables in subsequent steps
    run: '{var3}'
    in:
      echoMe: |
              var1 is '{var1}'
              var2 is {var2}
```

### environment variables
You can set a pypyr variable from an [environment variable]({{< ref
"/topics/environment-variables" >}}).

```yaml
steps:
- name: pypyr.steps.envget
  comment: get env $MACAVITY
           assign it to pypyr variable theHiddenPaw
           if env $MACAVITY doesn't exist, use the default value
  in:
    envGet:
      env: MACAVITY
      key: theHiddenPaw
      default: but macavity wasn't there!

- name: pypyr.steps.echo
  comment: theHiddenPaw was set from env $MACAVITY
  in:
    echoMe: the master criminal {theHiddenPaw}
```

### load variables from a file
You can load & initialize variables from a bespoke json, yaml or toml file.

If you want to set your custom file path as a cli argument, you can use the
following context parsers:
- [initialize variables from a json file]({{< ref "/docs/context-parsers/jsonfile" >}})
- [initialize variables from a toml file]({{< ref "/docs/context-parsers/tomlfile" >}}) 
- [initialize variables from a yaml file]({{< ref "/docs/context-parsers/yamlfile" >}}) 

With these context parsers, you invoke your pipeline and pass the path of your
custom input file as an argument:
```fish
$ pypyr mypipeline ./myvars.json
$ pypyr mypipeline ./myvars.toml
$ pypyr mypipeline ./myvars.yaml
```

You can also use a step explicitly to load your variables from your own custom
file at a specific point in the pipeline's execution:
- [load variables from json]({{< ref "/docs/steps/fetchjson" >}})
- [load variables from toml]({{< ref "/docs/steps/fetchtoml" >}})
- [load variables from yaml]({{< ref "/docs/steps/fetchyaml" >}})

### set variables from config
You can initialize your global variables using [vars]({{< ref
"/docs/getting-started/config#vars" >}}) in `pyproject.toml` or in the pypyr
yaml config file.

Use [pypyr.steps.configvars]({{< ref "/docs/steps/configvars" >}}) to inject
your config values into the pipeline.

## local variables
You can set a variable that is only in scope for the current step using the [in
decorator]({{< ref "/docs/decorators/in" >}}).

Unlike setting a "global" variable, arguments that you pass to a step using `in`
are not available to subsequent steps.

```yaml
steps:
  - name: mystep
    comment: variables A, B and C only in scope for this step.
    in:
      A: value 1
      B: 2
      C: true
  - name: pypyr.steps.echo
    in:
      echoMe: done! A, B & C are NOT available in this step.
```

## unset variables
You can unset or delete variables using the following steps:
- [contextclear]({{< ref "/docs/steps/contextclear">}})
- [contextclearall]({{< ref "/docs/steps/contextclearall">}})

## read variables
You can read, format and combine static values & variables using
[substitutions]({{< ref "docs/substitutions/format-string">}}).

In this example, we set 2 local variables `A` and `B`, and then use a
substitution expression to pass these to the [echo step]({{< ref
"/docs/steps/echo" >}}) `echoMe` input argument:

```yaml
- name: pypyr.steps.echo
  comment: concat static string and 2 vars.
           will output "piping down the valleys wild".
  in:
    A: down the
    B: valleys
    echoMe: piping {A} {B} wild
```

Instead of assigning `A` and `B` here in the step's `in`, you could also set `A`
and `B` in any previous step as "global" variables using [set]({{< ref
"/docs/steps/set" >}}) or [default]({{< ref "/docs/steps/default" >}}):

```yaml
steps:
  - name: pypyr.steps.set
    in:
      set:
        A: down the
        B: valleys

  - name: pypyr.steps.echo
    comment: concat static string and 2 vars.
             will output "piping down the valleys wild".
    in:
      echoMe: piping {A} {B} wild       
```