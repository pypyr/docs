---
title: basic concepts
description: Basic concepts of pypyr. Learn the fundamentals of how pipelines, step-groups, steps & context fit together.
date: 2019-08-21
publishdate: 2019-08-21
lastmod: 2019-08-21
# categories: [install]
menu:
  docs:
    parent: getting-started
draft: false
seo_article_headline: Task-runner pipelines, steps, step-groups & context.
seo_description: Automation by defining sequences of steps & modular step groups in a pipeline. Use context to pass values between steps.
# topics: [pipeline, tutorial, args]
---
# basic concepts
## pipeline
A pipeline is a sequence of steps. A pypyr pipeline is a simple human-readable 
and human-authored yaml file that defines your sequence of steps. pypyr 
interprets and runs the pipeline for you.

```yaml
# ./arb-example-pipeline.yaml
# optional
context_parser: my.custom.parser # how you pass cli arguments to the pipeline.

# mandatory
steps: # step-group
  - step1 # run ./step1.py
  - step2 # run ./step2.py

# optional.
on_success: # step-group
  - my.first.success.step # run ./my/first/success/step.py
  - my.second.success.step # run ./my/second/success/step.py

# optional.
on_failure: # step-group
  - my.failure.handler.step # run ./my/failure/handler/step.py
  - my.failure.handler.notifier # run ./my/failure/handler/notifier.py
```

## step-group
You organize your sequences of steps into groups. By default, pypyr looks for 
3 step-groups on a default run:

- `steps`
- `on_success`
- `on_failure`

You can make your own [custom step-groups]({{< ref "/docs/pipelines/pipeline-structure#custom-step-groups">}}) 
too, to break your pipeline into modular components.

## step
A step is the thing that actually does the work. Under the hood, a step is 
really just a bit of python that does stuff - you can 
[code your own step]({{< ref "/docs/api/step" >}}) with a simple undemanding 
function signature, or even more easily, use any of 
pypyr's many [built-in steps]({{< ref "/docs/steps" >}}) without writing your 
own code at all.

You can specify a step in the pipeline yaml in two ways:

### simple step
A simple step is just the name of the python module. pypyr will look in your 
working directory for these modules or packages.

For a package, be sure to specify the full namespace (i.e not just `mymodule`, 
but `mypackage.mymodule`.

```yaml
steps:
  - my.package.my.module # points at a python module in a package.
  - mymodule # simple step pointing at ./mymodule.py
  - another.module # simple step pointing at ./another/module.py
```

### complex step
A complex step allows you to specify a few more details for your step, but at 
heart it's the same thing as a simple step - it points at some python.

```yaml
steps:
  - name: my.package.another.module
    description: Optional Description is for humans.
                 It is any yaml-escaped text that makes your life easier.
                 Outputs to the console during runtime as NOTIFY.
    comment: Optional comments for pipeline developers.
             Does not output to console during run-time.
    in: # optional. 
      parameter1: value1
      parameter2: value2
```

These extra fields on the step are [decorators]({{< ref "/docs/decorators/" >}}). 
Decorators are powerful optional extras you can apply to your step to pass your
own variables to it, make it loop, retry or ignore errors automatically & only 
execute when certain conditions are true.

You can freely mix and match simple and complex steps in the same pipeline. 
Frankly, the only reason simple steps are there is because I'm lazy and I 
dislike redundant typing.

## context
The pypyr context is a dictionary that is in scope for the duration of the 
entire pipeline. You use the context to persist and pass values between steps 
in the pipeline.

Context is not just for simple dictionary key-value pairs. You can put any 
object into context. This is especially powerful because you can load entire 
configuration files in json or yaml into context then to use and format for 
your steps. A typical use-case is to load a configuration file for a 3rd party 
system, inject some variable values into the configuration file, and then invoke 
the 3rd party system with the newly formatted payload.

The [context_parser]({{< ref "/docs/context-parsers" >}}) can initialize the 
context from the cli. 

Any step in the pipeline can add, edit or remove items from the context 
dictionary.

Because context is so integral to doing real work in pypyr, pypyr has a lot of 
built-in steps to [set, manipulate & format context values]({{< ref "/topics/context" >}}).
