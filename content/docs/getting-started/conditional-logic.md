---
title: conditional logic
description: How to run or skip steps conditionally.
date: 2021-09-08
menu:
  docs:
    title: conditional logic
    parent: getting-started
weight: -50
seo_article_headline: How to use conditional logic in a task-runner pipeline.
seo_description: You can run or skip steps in a pypyr pipeline based on parameterized conditional statements.
topics: [control-of-flow]
---
# conditional logic
## selectively run or skip step
You can control the flow of execution in your pipeline by selectively running or
skipping a step based upon whether a conditional statement evaluates to `True`.

You use the [run]({{< ref "/docs/decorators/run">}}) or [skip]({{< ref
"/docs/decorators/skip">}}) decorators on any step to set your condition whether
to execute the step.

By default, unless you explicitly tell pypyr differently, every step will run.

```yaml
# getting-started/basic-conditional.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin

  - name: pypyr.steps.cmd
    run: False
    in:
      cmd: echo this will not run

  - name: pypyr.steps.cmd
    skip: True
    in:
      cmd: echo this will not run

  - name: pypyr.steps.echo
    in:
      echoMe: end
```

When you run this pipeline, the 2nd & 3rd steps will NOT execute:

```text
$ pypyr getting-started/basic-conditional
begin
end
```

`skip` is the inverse of `run`.

## use variables to control if step runs
You can parameterize your conditional statements with [variables]({{< ref
"variables" >}}). You can set variables in preceding steps, pass variables from
the cli, use [environment variables]({{< ref "/topics/environment-variables"
>}}) or even load your variables from a separate configuration file using
something like [fetchjson]({{< ref "/docs/steps/fetchjson" >}}) or
[fetchyaml]({{< ref "/docs/steps/fetchyaml" >}}).

### set conditional logic in pipeline
You can set the variables that control if a step runs or not in the pipeline
itself in a preceding step.

```yaml
# ./getting-started/conditional-variable-in-pipeline
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin

  - name: pypyr.steps.contextsetf
    comment: set variables for use in run/skip later
    in:
      contextSetf:
        arbVar1: False
        arbVar2: arbitrary str
        # arbVar3 will eval True
        arbVar3: !py arbVar2 == 'arbitrary str'

  - name: pypyr.steps.cmd
    run: '{arbVar1}'
    in:
      cmd: echo this will not run

  - name: pypyr.steps.cmd
    skip: '{arbVar3}'
    in:
      cmd: echo this will not run

  - name: pypyr.steps.echo
    in:
      echoMe: end
```

When you run this pipeline, the 2nd & 3rd steps will NOT execute:

```text
$ pypyr getting-started/conditional-variable-in-pipeline
begin
end
```

### use cli arguments to run step conditionally
pypyr allows you to pass your own arguments from the cli to the pipeline using a
[context parser]({{< ref "/docs/context-parsers" >}}).

You can use these variables directly in a step condition to control if the step
should run or not.

In this example, we will use the [keys]({{< ref "/docs/context-parsers/keys"
>}}) context parser because a frequent pattern to create a natural syntax for
cli applications is to pass verbs as arguments to execute select functionality.

```yaml
# ./ci.yaml
context_parser: pypyr.parser.keys
steps:
  - name: pypyr.steps.default
    comment: set default values for optional cli inputs
    in:
      defaults:
        lint: False
        build: False

  - name: pypyr.steps.echo
    in:
      echoMe: begin - always runs

  - name: pypyr.steps.cmd
    run: '{lint}'
    in:
      cmd: echo this will only run if you pass lint from cli

  - name: pypyr.steps.cmd
    run: '{build}'
    in:
      cmd: echo this will only run if you pass build from cli

  - name: pypyr.steps.echo
    in:
      echoMe: end - always runs
```

You can control which steps execute by passing the appropriate verbs as
arguments from the cli. You can also combine multiple steps by passing more than
one verb.

```text
$ pypyr ci
begin - always runs
end - always runs

$ pypyr ci lint
begin - always runs
this will only run if you pass lint from cli
end - always runs

$ pypyr ci build
begin - always runs
this will only run if you pass build from cli
end - always runs

$ pypyr ci lint build
begin - always runs
this will only run if you pass lint from cli
this will only run if you pass build from cli
end - always runs
```

### use python expressions for control flow
You can use [!py strings]({{< ref "/docs/substitutions/py-strings" >}}) to use
any valid Python expression to control if a step executes or not.

```yaml
# ./getting-started/python-expression-conditional
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin

  - name: pypyr.steps.contextsetf
    comment: set arbitrary variables to use in later step run/skip conditions
    in:
      contextSetf:
        breakfasts:
          - fish
          - bacon
          - spam

        numbersList:
          - 3
          - 4

  - name: pypyr.steps.cmd
    run: !py "'eggs' in breakfasts"
    in:
      cmd: echo this will not run

  - name: pypyr.steps.cmd
    skip: !py sum(numbersList) < 42
    in:
      cmd: echo this will not run

  - name: pypyr.steps.echo
    in:
      echoMe: end
```

Running this pipeline results in:
```text
$ pypyr getting-started/python-expression-conditional
begin
end
```

## multiple steps with the same condition
If you want to control multiple steps' conditional execution with the same
conditional statement, you can either just duplicate the same conditional
statement on each step, or you can group your steps together in a [custom
step-group]({{< ref "/docs/pipelines/pipeline-structure#custom-step-groups"
>}}) and govern the entire step-group's execution with only the one conditional
evaluation on a [call]({{< ref "/docs/steps/call">}}) step.

```yaml
# ./ci.yaml
context_parser: pypyr.parser.keys
steps:
  - name: pypyr.steps.default
    comment: set default values for optional cli inputs
    in:
      defaults:
        lint: False
        build: False

  - name: pypyr.steps.echo
    in:
      echoMe: begin - always runs

  - name: pypyr.steps.call
    run: '{lint}'
    in:
      call: lint

  - name: pypyr.steps.call
    run: '{build}'
    in:
      call: build

  - name: pypyr.steps.echo
    in:
      echoMe: end - always runs

lint:
  - name: pypyr.steps.cmd
    in:
      cmd: echo lint cmd 1 here

  - name: pypyr.steps.cmd
    in:
      cmd: echo lint cmd 2 here

build:
  - name: pypyr.steps.cmd
    in:
      cmd: echo build cmd 1 here

  - name: pypyr.steps.cmd
    in:
      cmd: echo build cmd 2 here
```

You can control this pipeline from the cli like this:

```text
$ pypyr ci
begin - always runs
end - always runs

$ pypyr ci lint
begin - always runs
lint cmd 1 here
lint cmd 2 here
end - always runs

$ pypyr ci build
begin - always runs
build cmd 1 here
build cmd 2 here
end - always runs

$ pypyr ci lint build
begin - always runs
lint cmd 1 here
lint cmd 2 here
build cmd 1 here
build cmd 2 here
end - always runs
```

{{% note tip %}} `call` invokes a step-group in the same pipeline. If you want
to invoke another pipeline instead you can use a [pype step]({{<
ref"/docs/steps/pype" >}}) and similarly set the conditional statement on that.
{{%/note %}}

## dynamic step execution
A compact pattern to create an extensible custom cli for your pipeline is to use
the [call]({{< ref "/docs/steps/call">}}) step in conjunction with the
[list]({{< ref "/docs/context-parsers/list">}}) context parser.

This way you are not hard-coding which step-groups to run - whichever group
names you pass from the cli will run in order.

```yaml
# ./ops/build.yaml
context_parser: pypyr.parser.list
steps:
  - name: pypyr.steps.call
    comment: lint & test code. this runs on every pipeline invocation.
    in:
      call: 
        - lint
        - test
  - name: pypyr.steps.call
    comment: optionally do extras like package & publish after lint & test.
    run: '{argList}'
    in:
      call: '{argList}'


lint:
  - name: pypyr.steps.cmd
    in:
      cmd: echo lint command here

test:
  - name: pypyr.steps.cmd
    in:
      cmd: echo test command here

package:
  - name: pypyr.steps.cmd
    in:
      cmd: echo package command here

publish:
  - name: pypyr.steps.cmd
    in:
      cmd: echo publish command here
```

For the sake of clarity, each step-group only has one step in it, but you can
(hopefully obviously) specify multiple steps if you want.

```text
# only run the common functionality, no extras
$ pypyr ops/build
lint command here
test command here

# run common + package
$ pypyr ops/build package
lint command here
test command here
package command here

# run common + package + publish
$ pypyr ops/build package publish
lint command here
test command here
package command here
publish command here
```