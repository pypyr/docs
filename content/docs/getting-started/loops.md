---
title: loops
description: How to loop or iterate over a step.
date: 2021-09-09
menu:
  docs:
    title: loops
    parent: getting-started
weight: -45
seo_article_headline: How to use loops (iterations) in a task-runner pipeline.
seo_description: You can repeatedly run a step by looping in a pypyr pipeline without writing code.
topics: [loops, control-of-flow]
---
# loops
You can loop (or iterate) over any given step in a pypyr pipeline.

This means you can repeatedly run or loop over your own custom commands without
writing any code.

Looping happens on the step-level, using the following step decorators:

- [foreach]({{< ref "/docs/decorators/foreach">}})
- [while]({{< ref "/docs/decorators/while">}})

The difference is that `foreach` iterates over every element in an iterable
(such as a `list`), whereas `while` keeps on looping until a stop condition
evaluates `True`.

{{% note tip %}}
If you want to retry a step automatically until it succeeds, you can use
[retry]({{< ref "/docs/decorators/retry">}}) instead of coding your own loop
with your own error handling.
{{% /note %}}

You can specify a `foreach` or `while` loop on any given step. You can also nest
`foreach` and `while` loops on the same step.

## foreach
Here is a simple `foreach` loop in action. For the sake of an easy example,
we're just calling `echo` here, but you can run any command you'd like.

The iterator is `{i}`. This is the current item on each iteration of the
`foreach` iterable.

```yaml
# ./foreach-example.yaml
steps:
  - name: pypyr.steps.cmd
    comment: loop over your own command
    foreach: ['apple', 'pear', 'banana']
    in:
      cmd: echo mycommand --arg={i}
```

Running this will result in:

```text
$ pypyr foreach-example
mycommand --arg=apple
mycommand --arg=pear
mycommand --arg=banana

$
```

## while
A `while` loop iterates until a stop condition is `True`, such as any given
boolean expression evaluating `True`, or if it reaches a certain number of
iterations.

The current iteration count is `{whileCounter}`.

### while with max count
Here is a simple example of a `while` loop that continues until it reaches a
maximum iteration count:

```yaml
# ./while-max-example
steps:
  - name: pypyr.steps.cmd
    while:
      max: 3
    in:
      cmd: echo mycommand --arg={whileCounter}
```

Running this will result in:
```text
$ pypyr while-max-example
mycommand --arg=1
mycommand --arg=2
mycommand --arg=3

$
```

### while with stop condition
Here is a simple example of a `while` loop that continues until a stop condition
is `True`. In this example we're using a [py]({{< ref "/docs/steps/py">}}) step
to run some custom inline python for every loop iteration, but as ever, you can
use any step you'd like (including running your own external command or script
via [cmd]({{< ref "/docs/steps/cmd">}})).

```yaml
# ./while-stop-example
steps:
  - name: pypyr.steps.py
    while:
      stop: !py arb_value > 5
    in:
      py: |
        arb_value = whileCounter * 2
        print(f'{arb_value=}')
        save('arb_value')
```

Running this will result in:
```text
$ pypyr while-stop-example
arb_value=2
arb_value=4
arb_value=6

$
```

## retry
If you specifically want to loop over a step until it succeeds, you don't have
to construct your own `while` loop, you can use pypyr's [retry]({{< ref
"/docs/decorators/retry">}}) decorator instead.

The `retry` decorator supports adding a sleep interval between retries with
various back-off strategies.

This example will automatically retry a command up to 3 times and only continue
if the command succeeds within the max limit:

```yaml
# ./retry-example
steps:
  - name: pypyr.steps.cmd
    retry:
      max: 3
    in:
      cmd: mycommand --arg=myvalue
```

If you want to retry infinitely, set `max: 0`.

{{% note tip %}}
See the Getting Started guide for [pypyr error handling]({{<ref "error-handling" >}})
for full details on error handling concepts.
{{% /note %}}

## using variables to loop
You can parameterize your looping statements with [variables]({{< ref
"variables" >}}). You can set variables in preceding steps, pass variables from
the cli, use [environment variables]({{< ref "/topics/environment-variables"
>}}) or even load your variables from a separate configuration file using
something like [fetchjson]({{< ref "/docs/steps/fetchjson" >}}) or
[fetchyaml]({{< ref "/docs/steps/fetchyaml" >}}).

### foreach
In this example we're going to loop over every argument given to the cli and
inject it into a custom command:

```yaml
# ./foreach-variable-example.yaml
context_parser: pypyr.parser.list
steps:
  - name: pypyr.steps.set
    comment: set defaults for optional cli args
    run: !py len(argList) == 0
    in:
      set:
        argList: ['one', 'two']

  - name: pypyr.steps.cmd
    foreach: '{argList}'
    in:
      cmd: echo mycommand --arg={i}
```

This example just uses the `argList` from the [pypyr.parser.list]({{< ref
"/docs/context-parsers/list" >}}) context parser, but you can use any list you
might have in your context.

```text
$ pypyr foreach-variable-example
mycommand --arg=one
mycommand --arg=two

$ pypyr foreach-variable-example four five
mycommand --arg=four
mycommand --arg=five
```

### while
You can use variables with any looping decorator. Here is an example using a
`while` loop where an environment variable controls the maximum iteration limit. 

```yaml
# ./while-variable-example.yaml
steps:
  - name: pypyr.steps.envget
    comment: get $ARB environment variable
    in:
      envGet:
        env: ARB
        key: my_counter
        default: 3

  - name: pypyr.steps.cmd
    while:
      max: '{my_counter}'
    in:
      cmd: echo mycommand --arg={whileCounter}
```

If the `$ARB` environment variable does not exist, we use a default value of `3`
instead.

```text
$ pypyr while-variable-example
mycommand --arg=1
mycommand --arg=2
mycommand --arg=3

$ ARB=2 pypyr while-variable-example
mycommand --arg=1
mycommand --arg=2
```

## looping over multiple steps
If you want to loop over multiple steps as a unit, group these into a [custom
step-group]({{< ref "/docs/pipelines/pipeline-structure#custom-step-groups"
>}}) and then use [call]({{< ref "/docs/steps/call">}}).

Alternatively, you can use [pype]({{< ref "/docs/steps/pype">}}) to invoke
another pipeline repeatedly in a loop.

Either way, you can use `foreach` and/or `while` as you choose on these steps.

```yaml
# ./foreach-call-example
steps:
  - name: pypyr.steps.call
    foreach: ['apple', 'pear', 'banana']
    in:
      call: my_step_group

my_step_group:
  - name: pypyr.steps.echo
    in:
      echoMe: processing '{i}'

  - name: pypyr.steps.cmd
    in:
      cmd: echo mycommand --arg={i}
```

This example just has an `echo` step and a `cmd` step in the step-group to
illustrate the point - you can have as many useful steps in a custom step-group
as you want. This is what it looks like at runtime:

```text
$ pypyr foreach-call-example
processing 'apple'
mycommand --arg=apple
processing 'pear'
mycommand --arg=pear
processing 'banana'
mycommand --arg=banana
```