---
title: foreach loop decorator
linktitle: foreach
date: 2020-07-10T19:07:51+01:00
description: Repeat step for each item in list.
card_extra_summary:
  heading: example
  details:  |
          ```yaml
          foreach: ["one", "two", "three"]
          ```
card_extra_summary_is_code: True
# categories: [pipeline definition]
# keywords: ""
menu:
  docs:
    parent: decorators
    name: foreach
seo_article_headline: Repeat task-runner step for each item in list.
seo_description: Repeat (loop) any pipeline step, or your own custom code step, for each item in the input list.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: foreach -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [control-of-flow, loops, pipeline format]
---
# foreach
## repeat step for each item in list
Run the step once for each item in the list. 

The iterator is `context['i']`. If you want to use the iterator value in your 
step with a substitution expression, you'd use `{i}`.

`foreach` is a list `[]`. In your pipeline, you can specify this in two 
ways:

```yaml
foreach: [item 1, item 2, item 3]
```

or

```yaml
foreach:
  - item 1
  - item 2
  - item 3
```

## loop static input list
The `foreach` input is a list.

```yaml
# ./foreach-example.yaml
steps:
  - name: pypyr.steps.echo
    foreach:
      - one
      - two
      - three at last
    in:
      echoMe: "{i}"
```

```text
$ pypyr foreach-example
one
two
three at last
```

## loop run-time list
You can use substitutions to assign dynamic values to the list to iterate.

```yaml
# ./foreach-substitution-example.yaml
context_parser: pypyr.parser.list
steps:
  - name: pypyr.steps.echo
    foreach: '{argList}'
    in:
      echoMe: "this time it's: {i}"
```

This example just uses the `argList` from the [pypyr.parser.list]({{< ref "/docs/context-parsers/list" >}}) context 
parser, but you can use any list you might have in your context.

```text
$ pypyr foreach-substitution-example eggs bacon ham
this time it's: eggs
this time it's: bacon
this time it's: ham
```

## run a sequence of steps in a loop
You can loop on any step, whether it is your own or a built-in pypyr step. If 
you loop on a [call]({{< ref "/docs/steps/call">}}) step, you will run your
`foreach` loop on the entire sequence of steps inside a step-group. This lets 
you loop over multiple steps as a unit. If you want to loop over an entire 
pipeline, use [pype]({{< ref "/docs/steps/pype">}}) to invoke the pipeline and 
decorate the `pype` step in the same way.

```yaml
# ./foreach-call.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin
  - name: pypyr.steps.call
    comment: run entire sequence of steps
             in loop_me step-group in a loop.
    foreach:
      - item 1
      - item 2
      - item 3
    in:
      call: loop_me
  - name: pypyr.steps.echo
    in:
      echoMe: end

loop_me:
  - name: pypyr.steps.echo
    in:
      echoMe: this is foreach {i} executing like a boss.
  - name: pypyr.steps.cmd
    comment: you prob want to do something useful here
    in:
      cmd: echo yourcmd --dothing {i}
```

This results in: 

```text
$ pypyr foreach-call
begin
this is foreach item 1 executing like a boss.
yourcmd --dothing item 1
this is foreach item 2 executing like a boss.
yourcmd --dothing item 2
this is foreach item 3 executing like a boss.
yourcmd --dothing item 3
end
```

## conditional execution inside loop
The `run`, `skip` & `swallow` decorators evaluate dynamically on each iteration.
So if during an iteration the step's logic sets `run=False`, the step will not 
execute on the next iteration.

The loop will run to completion, though, so if a subsequent iteration sets 
`run=True` again, the step will execute again on the next loop iteration.

```yaml
# ./foreach-run.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin
  - name: pypyr.steps.contextsetf
    in:
      contextSetf:
        myThings:
          - name: thing 1
            isReady: True
          - name: thing 2
            isReady: False
          - name: thing 3
            isReady: True
  - name: pypyr.steps.echo
    comment: only run for thing when isReady.
    foreach: '{myThings}'
    run: '{i[isReady]}' 
    in:
      echoMe: do something with {i[name]}
  - name: pypyr.steps.echo
    in:
      echoMe: end!
```

When you run this pipeline, notice that the 2nd iteration skips execution, but
it continues with the 3rd iteration:

```text
$ pypyr foreach-run
begin
do something with thing 1
do something with thing 3
end!
```