---
title: while loop decorator
linktitle: while
date: 2020-07-14T13:09:24+01:00
description: Repeat step until stop condition is `True` or up to the maximum iteration count.
draft: false
card_extra_summary:
  heading: example
  details: |
    ```yaml
    while:
      stop: '{keyhere}'
      max: 3
      sleep: 1.5
      errorOnMax: True
    ```
card_extra_summary_is_code: True
# categories: [pipeline definition]
# keywords: ""
menu:
  docs:
    parent: decorators
    name: while
seo_article_headline: Do a while loop in task-runner pipeline.
seo_description: Repeat steps until a stop condition is True or until you reach a maximum iteration count.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: while -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [control-of-flow, loops, pipeline format]
---
# while
## repeat step(s) in while loop
Repeat step until `stop` is `True`, or until you reach a configurable maximum 
iterations. You have to specify at least one of either `max` or `stop`.

If you specify both `max` and `stop`, the loop exits when `stop` is `True` as 
long as it's still under `max` iterations. 

`max` will exit the loop even if `stop` is still `False`. If you want to error 
and stop processing when `max` exhausts set `errorOnMax` to `True`. For example, 
maybe you are waiting for `stop` to reach `True` but want to timeout after 
`max`.

The iterator is `context['whileCounter']`. If you want to use the iterator 
value in your step with a substitution expression, you'd use `{whileCounter}`.

These are all the options to configure a `while` loop:

```yaml
while: # optional. repeat step until stop is True or max iterations reached.
  stop: '{keyhere}' # loop until this evaluates True.
  max: 1 # max loop iterations to run. integer. Defaults None (infinite).
  sleep: 0 # sleep between iterations, in seconds. Decimals allowed. Defaults 0.
  errorOnMax: False # raise error if max reached. Defaults False.
```

All the inputs support [substitutions]({{< ref "/docs/substitutions">}}), so 
you can assign values to these properties dynamically at run-time. `stop` 
evaluates on each loop iteration.

## loop until stop condition true
The `stop` condition re-evaluates on every loop iteration. This means that you
can signal from your step you want to break out of the loop reporting success.

If you use any context variables in your `stop` condition, these need to exist
before the loop starts. You can initialize these in the `in` arguments for the 
same step, or alternatively somewhere in a preceding step like 
`contextcopy` or `contextsetf`.

{{% note tip %}}
Remember that you can use 
[py strings]({{< ref "/docs/substitutions/py-strings" >}}) for the `stop` 
condition. This allows you to use more advanced python expressions for your
`stop` condition.

```yaml
stop: !py responseCode == "OK"
```
{{% /note %}}

This example uses an inline py step to execute some arbitrary python that 
changes the variable that the while `stop` condition evaluates:

```yaml
# ./while-stop.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin
  - name: pypyr.steps.py
    comment: while with a stop condition
             the step changes the value
             the stop condition checks for
             on the 3rd iteration.
    in:
      # initialize the stop boolean here
      stopWhenImTrue: False
      pycode: |
                print(f"this is while {context['whileCounter']} executing like a boss.")
                if context['whileCounter'] == 3:
                  context['stopWhenImTrue'] = True
    while:
      stop: '{stopWhenImTrue}'
  - name: pypyr.steps.echo
    in:
      echoMe: end
```

This will result in:
```text
$ pypyr while-stop
begin
this is while 1 executing like a boss.
this is while 2 executing like a boss.
this is while 3 executing like a boss.
end
```

This is a contrived example to show looping on a single step. See 
[calling multiple steps in a loop](#run-a-sequence-of-steps-in-a-loop) for a 
pipeline that does exactly the above, but with using multiple steps instead.

## loop until maximum iteration count
You can set `max` to a static positive integer, or you can use a formatting 
expression to set the loop upper bound dynamically. 

You can't change `max` mid-loop, however. `max` evaluates at the start of the 
`while` loop and becomes fixed at that point.

```yaml
# ./while-max.yaml
context_parser: pypyr.parser.string
steps:
  - name: pypyr.steps.echo
    comment: loop 5X
    while:
      max: 5    
    in:
      echoMe: Static Max -> this is step {whileCounter}
  - name: pypyr.steps.echo
    comment: get loop counter from cli arg input
             pypyr makes the string an int under
             the covers for you.
             argString comes from the cli arg.
    while:
      max: '{argString}'
    in:
      echoMe: Dynamic Max -> this is step {whileCounter}
```

This results in:

```text
$ pypyr while-max 3
Static Max -> this is step 1
Static Max -> this is step 2
Static Max -> this is step 3
Static Max -> this is step 4
Static Max -> this is step 5
Dynamic Max -> this is step 1
Dynamic Max -> this is step 2
Dynamic Max -> this is step 3

$
```

## combine max & stop condition
You can set a maximum loop iteration count in addition to a stop condition. 
This is pretty useful for situations where you want your task to repeat but 
stop after a certain number of iterations.

```yaml
#  ./while-max-stop.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin
  - name: pypyr.steps.contextsetf
    comment: arbitrary context key with value "actual".
             the loop stop condition will be looking for
             value "expected" to stop the loop.
    in:
      contextSetf:
        arbKey: actual
  - name: pypyr.steps.echo
    comment: stop evaluates on every loop iteration.
             If arbKey becomes 'expected' during 
             loop iteration, will stop loop.
             Since arbKey never becomes expected, 
             loop will exhaust on 3X.
    while: 
      max: 3
      stop: !py arbKey == 'expected'
    in:
      echoMe: counter {whileCounter} - arbKey is "{arbKey}", not "expected"
  - name: pypyr.steps.echo
    in:
      echoMe: end!
```

This will result in:

```text
$ pypyr while-max-stop
begin
counter 1 - arbKey is "actual", not "expected"
counter 2 - arbKey is "actual", not "expected"
counter 3 - arbKey is "actual", not "expected"
end!

$ echo $?
0
```

## raise error on max iteration count
If you want your loop to raise an error if it exhausts the available maximum
iterations, set `errorOnMax: True`. By default this is `False`.

This is frequently useful when you combine `max` and `stop` and you want to
stop processing after a maximum amount of iterations when the `stop` condition 
never evaluates to `True`.

Here is a simple example. In the real world, `stop` is likely to use a 
`{formatting expression}` that while evaluate dynamically on each loop 
iteration, rather than just a hard-coded `False`.

```yaml
#  ./while-max-stop-err.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin
  - name: pypyr.steps.echo
    comment: stop will never be True.
             loop will exhaust on 3X.
             And raise an error.
    while: 
      max: 3
      stop: False
      errorOnMax: True
    in:
      echoMe: counter {whileCounter}
  - name: pypyr.steps.echo
    in:
      echoMe: end!
```

```text
$ pypyr while-max-stop-err
begin
counter 1
counter 2
counter 3
ERROR:pypyr.dsl:while_loop: exhausted 3 iterations of while loop, and errorOnMax is True.
ERROR:pypyr.stepsrunner:run_step_groups: Something went wrong. Will now try to run on_failure.

LoopMaxExhaustedError: while loop reached 3.

$ echo $?
255
```

## sleep between while iterations
Use `sleep` to set the interval in seconds to wait in between step iterations.

If you don't set `sleep` it defaults to 0, which is no wait at all.

```yaml
# ./while-sleep
steps:
  - name: pypyr.steps.echo
    comment: loop 5 times with 2 seconds sleep 
             in between executions.
    in:
      echoMe: this is step {whileCounter}. ctrl-c if you're bored.
    while:
      max: 5
      sleep: 2
```

This results in:

```text
$ pypyr while-sleep
this is step 1. ctrl-c if you're bored.
this is step 2. ctrl-c if you're bored.
this is step 3. ctrl-c if you're bored.
this is step 4. ctrl-c if you're bored.
this is step 5. ctrl-c if you're bored.

$
```

## infinite loop
You can run an infinite loop by never having `stop` always evaluate to `False`.

```yaml
# ./while-infinite.yaml
steps:
  - name: pypyr.steps.echo
    description: runs infinitely
    comment: 3s sleep between iterations
    in:
      echoMe: this is a deliberate infinite loop. ctrl+c to break.
    while:
      stop: False
      sleep: 3
```

This will result in:

```text
$ pypyr while-infinite
pypyr.steps.echo: runs infinitely
this is a deliberate infinite loop. ctrl+c to break.
this is a deliberate infinite loop. ctrl+c to break.
this is a deliberate infinite loop. ctrl+c to break.
this is a deliberate infinite loop. ctrl+c to break.
this is a deliberate infinite loop. ctrl+c to break.
this is a deliberate infinite loop. ctrl+c to break.
this is a deliberate infinite loop. ctrl+c to break.
^C

$
```

## nesting foreach in while
A `foreach` on the same step will run the entire `foreach` loop once for each
`while` iteration.

```yaml
 # ./while-foreach
steps:
  - name: pypyr.steps.echo
    comment: loop 3X while.
             the entire foreach runs
             for each while iteration.
    in:
      echoMe: this is {whileCounter} - {i}
    foreach: ['eggs', 'bacon', 'spam']
    while:
      max: 3
  - name: pypyr.steps.echo
    in:
      echoMe: done!
```

This results in output:

```text
$ pypyr while-foreach
this is 1 - eggs
this is 1 - bacon
this is 1 - spam
this is 2 - eggs
this is 2 - bacon
this is 2 - spam
this is 3 - eggs
this is 3 - bacon
this is 3 - spam
done!
```

## run a sequence of steps in a loop
You can loop on any step, whether it is your own or a built-in pypyr step. If 
you loop on a [call]({{< ref "/docs/steps/call">}}) step, you will run your
`while` loop on the entire sequence of steps inside a step-group. This lets 
you loop over multiple steps as a unit.

If you want to loop over an entire pipeline, use 
[pype]({{< ref "/docs/steps/pype">}}) to invoke the pipeline and decorate the 
`pype` step in the same way.

```yaml
# ./while-call.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin
  - name: pypyr.steps.call
    comment: run entire sequence of steps
             in loop_me step-group in a loop.
    in:
      # initialize the stop boolean here
      stopWhenImTrue: False
      call: loop_me
    while:
      stop: '{stopWhenImTrue}'
  - name: pypyr.steps.echo
    in:
      echoMe: end

loop_me:
  - name: pypyr.steps.echo
    in:
      echoMe: this is while {whileCounter} executing like a boss.
  - name: pypyr.steps.contextsetf
    comment: set stopWhenImTrue to True on the 3rd iteration
             you're likely to do more meaningful work in your
             own pipelines.
    run: !py whileCounter == 3
    in:
      contextSetf:
        stopWhenImTrue: True
```

This will result in:

```text
$ pypyr while-call
begin
this is while 1 executing like a boss.
this is while 2 executing like a boss.
this is while 3 executing like a boss.
end
```