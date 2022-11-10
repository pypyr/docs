---
title: pypyr.steps.switch
linktitle: switch
date: 2022-10-19T13:54:25+01:00
description: Conditional branching for IF-THEN-ELSE control-of-flow.
card_extra_summary:
  heading: input context property
  details: "`switch` (list)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: switch
seo_article_headline: Branch pipeline execution with IF-THEN-ELSE control-of-flow.
seo_description: Switch selectively between step-groups in your pipeline by conditionally executing one rather than another.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: switch -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [control-of-flow]
---
# pypyr.steps.switch
## conditional branching
Control-of-flow statement that lets you conditionally select between execution
branches depending on whether the controlling case expression evaluates to
`True`.

You can conceptualize this as an `IF-THEN-ELSE` or `IF-ELIF-ELSE` style
conditional branch. If you're not branching between different execution paths,
but instead just want to control whether an individual step runs, you can use
the [run]({{< ref "/docs/decorators/run" >}}) or
[skip]({{< ref "/docs/decorators/skip" >}}) decorators instead.

```yaml
# ./my-pipeline.yaml
context_parser: pypyr.parser.string
steps:
  - name: pypyr.steps.switch
    comment: argString is set by the pypyr.parser.string context_parser
    in:
      switch:
        - case: !py argString == 'a'
          call: A
        - case: !py argString == 'b'
          call: B
        - case: !py argString == 'bc'
          call: [B, C] # multiple groups, will run in order
        - default: D

  - name: pypyr.steps.echo
    in:
      echoMe: done!

A:
  - name: pypyr.steps.echo
    in:
      echoMe: A

B:
  - name: pypyr.steps.echo
    in:
      echoMe: B

C:
  - name: pypyr.steps.echo
    in:
      echoMe: C

D:
  - name: pypyr.steps.echo
    in:
      echoMe: D
```

Running this pipeline will look like this:

{{< app-window title="term" lang="console" >}}
$ pypyr my-pipeline # default case, argString is empty ''
D
done!

$ pypyr my-pipeline a
A
done!

$ pypyr my-pipeline b
B
done!

$ pypyr my-pipeline bc
B
C
done!

$ pypyr my-pipeline xyz # no matches, default
D
done!
{{< /app-window >}}

When pypyr finds a matching `case`, it will call the step-groups you set in
`call`. Once the called step-group(s) complete, pypyr will complete the switch
step and move on to the next step in the pipeline after the switch.

pypyr only calls the first matching `case`, it does not fall-through to the
following case statements.

You can optionally specify `default` to execute if none of the `case`
expressions match.

If no `case` expressions match and there is no `default`, pypyr will continue
to the next step in the pipeline without executing any of the blocks in the 
switch.

All inputs support string [substitutions]({{< ref "/docs/substitutions">}}).

## call input
The input to `call` can be
- a simple string: run a single step-group
- a list of string: run a sequence of step-groups in order
- a dict for expanded syntax: set `success` and `failure` groups for the case.
  - The `groups` input can be a string for a single-step group, or a list of
    string to run a sequence of groups in order.

```yaml
- name: pypyr.steps.switch
  in:
    switch:
      - case: !py argString == 'a'
        call: A # call single group `A`

      - case: !py argString == 'bc'
        call: [B, C] # call sequence of groups in order

      - case: !py argString == 'c'
        call: # expanded syntax to set a failure handler explicitly
          groups: C # can also be a list to call a sequence of groups in order
          failure: handle_error

      - default: D
```

The expanded input for `default` is the same as for `call`: 
```yaml
switch:
  - case: !py argString == 'a'
    call:
      groups: A
      failure: handle_error
  - case: !py argString == 'b'
    call: B
  - case: !py argString == 'bc'
    call: [B, C]
  - default:
      groups: D
      failure: some_error_handling_group
```

Like [call]({{< ref "/docs/steps/call">}}), `switch` only runs `success` or
`failure` groups if you actually specify these. You don't _have_ to specify
success or failure groups - if you don't, pypyr will just run the group(s) you
explicitly specified and continue when done without trying to run any success or
failure groups. It will still fall back to the containing success or failure
group settings for the pipeline itself or the step-group the switch lives in.

### failure handler
With expanded syntax, you can add a
[failure handler]({{< ref "docs/getting-started/error-handling#failure-handlers" >}})
if you want to do some specific handling if the called step-groups for a `case`
raise an error.

Notice in the `A` step-group this pipeline deliberately raises an error to
illustrate the principle:
```yaml
context_parser: pypyr.parser.string
steps:
  - name: pypyr.steps.switch
    in:
      switch:
        - case: !py argString == 'a'
          call:
            groups: A
            failure: handle_error
        - case: !py argString == 'b'
          call: B
        - case: !py argString == 'bc'
          call: [B, C]
        - default: D

A:
  - name: pypyr.steps.echo
    in:
      echoMe: A
  
  - name: pypyr.steps.assert
    comment: arbitrarily raise an error here
    in:
      assert: False

B:
  - name: pypyr.steps.echo
    in:
      echoMe: B

C:
  - name: pypyr.steps.echo
    in:
      echoMe: C

D:
  - name: pypyr.steps.echo
    in:
      echoMe: D

handle_error:
  - name: pypyr.steps.echo
    in:
      echoMe: error handling stuff here

  # swallow error so pipeline does not quit reporting failure
  - pypyr.steps.stopstepgroup
```

Assuming you saved this pipeline as `./my-pipeline.yaml`, you can run it like
this:
{{< app-window title="term" lang="console" >}}
$ pypyr my-pipeline # default case, argString is empty ''
D
done!

$ pypyr my-pipeline a
A
Error while running step pypyr.steps.assert at pipeline yaml line: 40, col: 5
Something went wrong. Will now try to run handle_error.
error handling stuff here
{{< /app-window >}}

## combine with decorators
As with any pypyr step, you can apply any of the
[decorators]({{< ref "/docs/decorators" >}}) to the `switch` step.

Here we're running `switch` in a
[while]({{< ref "/docs/decorators/while" >}}) loop, using the while iterator
`{whileCounter}` to make decisions on each loop iteration:

```yaml
steps:
  - name: pypyr.steps.switch
    while:
      max: 100
    in:
      switch:
        - case: !py whileCounter % 15 == 0
          call: FizzBuzz

        - case: !py whileCounter % 3 == 0
          call: Fizz

        - case: !py whileCounter % 5 == 0
          call: Buzz

        - default: just_print

Fizz:
  - name: pypyr.steps.echo
    in:
      echoMe: Fizz

Buzz:
  - name: pypyr.steps.echo
    in:
      echoMe: Buzz

FizzBuzz:
  - name: pypyr.steps.echo
    in:
      echoMe: FizzBuzz

just_print:
  - name: pypyr.steps.echo
    in:
      echoMe: '{whileCounter}'
```

You could of course use a [foreach]({{< ref "/docs/decorators/foreach" >}})
loop to achieve the same effect. In this case the iterator is `i`:

```yaml
- name: pypyr.steps.switch
  foreach: !py range(0, 100)
  in:
    switch:
      - case: !py i % 15 == 0
        call: FizzBuzz

      - case: !py i % 3 == 0
        call: Fizz

      - case: !py i % 5 == 0
        call: Buzz

      - default: just_print
```

## format expressions
The examples so far have used
[py strings]({{< ref "/docs/substitutions/py-strings" >}}). You can use a
standard
[formatting expression]({{< ref "/docs/substitutions/format-string" >}}) for
truthy-style evaluation instead:

```yaml
# ./switch-truthy
steps:
  - name: pypyr.steps.set
    comment: somewhat arbitrarily set some variables in context
    in:
      set:
        A: arb string
        B: False
        D: 1 

  - name: pypyr.steps.switch
    in:
      switch:
        - case: '{A}'
          call: A
        - case: '{B}'
          call: [B, C]
        - case: '{D}'
          call: D

A:
  - name: pypyr.steps.echo
    in:
      echoMe: A

B:
  - name: pypyr.steps.echo
    in:
      echoMe: B

C:
  - name: pypyr.steps.echo
    in:
      echoMe: C

D:
  - name: pypyr.steps.echo
    in:
      echoMe: D
```

Running this pipeline will result in:
{{< app-window title="term" lang="console" >}}
$ pypyr switch-truthy
D
{{< /app-window >}}

### truthy strings
Note that pypyr
[truthy bool evaluation]({{< ref "/docs/decorators#bool-evaluation" >}})
evaluates strings somewhat atypically: case insensitive string "true", "1" &
"1.0" are `True`, while all other string values, including empty string,
evaluate to `False`. This makes it easier for you to work with string inputs
from cli arguments without having to do special casting first.

So if you changed the input in the example to this:
```yaml
- name: pypyr.steps.set
  comment: somewhat arbitrarily set some variables in context
  in:
    set:
      A: "TRUE" # notice this is a string, not a bool
      B: False
      D: 1 
```

The value of `{A}` will evaluate to boolean `True` and in the switch statement
pypyr will run the `A` group instead of `D`.
