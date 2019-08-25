---
title: pipeline yaml structure
linktitle: pipeline structure
description: The format & structure of a pipeline.
date: 2019-08-21
publishdate: 2019-08-21
lastmod: 2019-08-21
categories: [pipelines]
topics: [pipeline, yaml, format, structure, definition]
menu:
  docs:
    parent: pipelines
    weight: 10
weight: 10
sections_weight: 10
draft: false
seo_article_headline: Directory lookup order for task-runner pipelines on the filesystem.
seo_description: Search for a matching pipeline first in the working directory & alternate location lookup sequence.
---
# pipeline yaml structure
## pypyr pipeline format
A pipeline is a .yaml file. pypyr uses YAML version 1.2.

Save pipelines wherever you please. To run a pipeline, execute
`pypyr pipelinename` from the directory where you saved
`pipelinename.yaml`

```yaml
# This is an example showing the anatomy of a pypyr pipeline
# A pipeline should be saved as {working dir}/mypipelinename.yaml.
# Run the pipeline from {working dir} like this: pypyr mypipelinename

# optional. set this to pass cli arguments to the pipeline.
context_parser: my.custom.parser

# mandatory.
steps: # step-group. every pipeline starts with steps unless you tell pypyr differently.
  - my.package.my.module # simple step pointing at a python module in a package
  - mymodule # simple step pointing at a python file
  - name: my.package.another.module # complex step. It contains a description and in parameters.
    description: Optional description is for humans. It's any text that makes your life easier.
    in: # optional. Set these key-value pairs in context for this step.
      parameter1: value1
      parameter2: value2
    run: True # optional. Runs this step if True, skips step if False. Defaults True if not specified.
    skip: False # optional. Skips this step if True, runs step if False. Defaults False if not specified.
    swallow: False # optional. Swallows any errors raised by the step. Defaults False if not specified.

# optional.
on_success: # step-group
  - my.first.success.step
  - my.second.success.step

# optional.
on_failure: # step-group
  - my.failure.handler.step
  - my.failure.handler.notifier
```

You don't have to specify `on_success` and/or `on_failure` if you don't feel
the need. pypyr will just skip these if you don't set them.

## step groups
### default step groups
pypyr looks for 3 different step groups on a default run:

- `steps`
- `on_success` (optional)
- `on_failure` (optional)

So:

```yaml
# the default pypyr step-groups
steps: # 'steps' is the default step-group that runs 1st
  - steps.step1 # will run ./steps/step1.py
  - arb.step2 # will run ./arb/step2.py

on_success: # on_success executes when pipeline completes successfully
  - success_step # will run ./success_step.py

on_failure: # on_failure executes whenever pipeline hits an error
  - steps.failure_step # will run ./steps/failure_step.py
```

#### steps
`steps` is a list of steps to execute in sequence. A step is simply a bit of
python that does stuff.

`steps` is the default entry-point for a pypyr pipeline. This means that if you
run a pipeline like so

{{< app-window title="term" lang="fish" >}}
$ pypyr mypipe bread circuses
{{< /app-window >}}

pypyr will looks for the `steps` group in `./mypipe.yaml` as an entry-point.
*bread* and *circuses* are the context arguments that the context-parser can use
to initialize the context.

You can use pypyr's built-in steps, or you can very easily make your own custom
steps. Don't hesitate to [roll your own step]({{< ref "/docs/api/step" >}}), 
it's really easy, no jokes!

#### on_success
`on_success` is a list of steps to execute in sequence. Runs when `steps:`
completes successfully. This is optional: if `on_success` isn't specified,
pypyr will just finish the pipeline reporting success without a problem.

You can use pypyr's [built-in steps]({{< ref "/docs/steps" >}}) or code your own
steps that you use exactly like you would for the `steps` step-group - it uses
the same function signature.

`on_success` is handy if your pipeline [jumps]({{< ref "/docs/steps/jump.md" >}})
between different step-groups but you still want to run a common sequence
at the end after everything is done.

Typical uses are to send a notification or do some clean-up when the pipeline 
sequence completes.

#### on_failure
`on_failure` is a list of steps to execute in sequence. Runs when any of the above
hits an unhandled exception.

When `on_failure` completes, pypyr will quit and report an error, with a non-zero
exit code. In other words, `on_failure` doesn't swallow the exception for you,
it just gives you the opportunity to do something about it.

This is optional: if `on_failure` isn't specified, pypyr will just finish the
pipeline reporting the error.

If `on_failure` encounters another exception while processing an exception, then
pypyr will log and report both that exception and the original cause's 
exception.

You can use built-in steps or code your own steps exactly like you would for
the `steps` step-group - it uses the same function signature.

### custom step groups
You don't have to stick to these default step-groups, though. You can specify
your own step-groups, or mix in your own step-groups with the defaults.

```yaml
# ./step-groups-example.yaml
sg1:
  - name: pypyr.steps.echo
    in:
      echoMe: sg1.1
  - name: pypyr.steps.echo
    in:
      echoMe: sg1.2
sg2:
  - name: pypyr.steps.echo
    in:
      echoMe: sg2.1
  - name: pypyr.steps.echo
    in:
      echoMe: sg2.2
sg3:
  - name: pypyr.steps.echo
    in:
      echoMe: sg3.1
  - name: pypyr.steps.echo
    in:
      echoMe: sg3.2
sg4:
  - name: pypyr.steps.echo
    in:
      echoMe: sg4.1
  - name: pypyr.steps.echo
    in:
      echoMe: sg4.2
```

You can use the `--groups` switch to specify which groups you want to run and
in what order:

{{< app-window title="term" lang="fish" >}}
$ pypyr step-groups-example --groups sg2 sg1 sg3
{{< /app-window >}}

This command will run, in order, `sg2` -> `sg1` -> `sg3`

Here's a ready cooked example of [custom step-groups in action](https://github.com/pypyr/pypyr-example/blob/master/pipelines/step-groups.yaml).

If you don't specify `--groups` pypyr will just look for the standard
`steps` group as per usual. You can still [call]({{< ref "/docs/steps/call.md" >}}) 
or [jump]({{< ref "/docs/steps/jump.md" >}}) to other step-groups from the 
default `steps` group, so you could think of `steps` a bit like the `main()` 
entry-point in traditional programming.

```yaml
# ./steps-with-custom-group.yaml
# use steps as default entry point
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin
  - name: pypyr.steps.jump
    in:
      jump: my_group

my_group:
  - name: pypyr.steps.echo
    in:
      echoMe: my_group step 1
  - name: pypyr.steps.echo
    in:
      echoMe: my_group step 2

# default on_success runs after everything complete
on_success:
  - name: pypyr.steps.echo
    in:
      echoMe: end!
```

This will result in:
{{< app-window title="term" lang="text" >}}
$ pypyr steps-with-custom-group
begin
my_group step 1
my_group step 2
end!
{{< /app-window >}}