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

`foreach` takes any iterable. In your pipeline yaml, you can specify this as a
list `[]` in two ways:

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
The `foreach` input here is a standard list.

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

This example just uses the `argList` from the [pypyr.parser.list]({{< ref
"/docs/context-parsers/list" >}}) context parser, but you can use any list you
might have in your context.

```text
$ pypyr foreach-substitution-example eggs bacon ham
this time it's: eggs
this time it's: bacon
this time it's: ham
```

## loop over a mapping
You can run your `foreach` loop on any iterable, including mapping or dictionary
types. Depending on your input to `foreach`, you can either iterate only on the
keys, or on the keys and values together:

```yaml
steps:
  - name: pypyr.steps.set
    in:
      set:
        my_mapping:
          a: b
          c: d

  - name: pypyr.steps.echo
    description: --> loop over a mapping will just return the keys
    foreach: '{my_mapping}'
    in:
      echoMe: key is {i}

  - name: pypyr.steps.echo
    description: --> loop over a mapping items will return keys + values
    foreach: !py my_mapping.items()
    in:
      echoMe: key is {i[0]} & value is {i[1]}       
```

This will output:

```text
--> loop over a mapping will just return the keys
key is a
key is c
--> loop over a mapping items will return keys + values
key is a & value is b
key is c & value is d
```

## run a sequence of steps in a loop
You can loop on any step, whether it is your own or a built-in pypyr step. If 
you loop on a [call]({{< ref "/docs/steps/call">}}) step, you will run your
`foreach` loop on the entire sequence of steps inside a step-group. This lets 
you loop over multiple steps as a unit.

If you want to loop over an entire pipeline, use [pype]({{< ref
"/docs/steps/pype">}}) to invoke the pipeline and decorate the `pype` step in
the same way.

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
  - name: pypyr.steps.set
    in:
      set:
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

## nesting loops
You can nest `foreach` loops. You can either call a step with a `foreach` inside
a called group with a `foreach`, loop pipelines calling other pipelines with
loops in them, or you can flatten the loop by getting the product of lists you
want to iterate.

### nesting loops with call
When you `call` a group that contains another loop, pypyr will iterate the inner
loop for each invocation of the outer. Pay attention to the value of `i` -
because `i` will always be the iterator value of the current, innermost loop,
grab the value of the parent's iterator with `set` before the inner loop
starts.

```yaml
steps:
  - name: pypyr.steps.call
    foreach: ['A', 'B', 'C']
    in:
      call: looping_group

  - name: pypyr.steps.echo
    in:
      echoMe: done!

looping_group:
  - name: pypyr.steps.set
    in:
      set:
        current_parent_iterator: '{i}'
  - name: pypyr.steps.echo
    foreach: ['one', 'two', 'three']
    in:
      echoMe: '{current_parent_iterator}.{i}'
```

This will output:

```text
A.one
A.two
A.three
B.one
B.two
B.three
C.one
C.two
C.three
done!
```

### nesting loops with pype
When you loop when you invoke a child pipeline from a parent with [pype]({{< ref
"/docs/steps/pype">}}), the child pipeline itself can also contain loops.

So given a parent pipeline:
```yaml
steps:
  - name: pypyr.steps.pype
    foreach: ['A', 'B', 'C']
    in:
      pype:
        name: foreach-nest-pype-child

  - name: pypyr.steps.echo
    in:
      echoMe: done!
```

Calling a child pipeline that looks like this:

```yaml
# ./foreach-nest-pype-child.yaml
steps:
  - name: pypyr.steps.set
    in:
      set:
        current_parent_iterator: '{i}'
  - name: pypyr.steps.echo
    foreach: ['one', 'two', 'three']
    in:
      echoMe: '{current_parent_iterator}.{i}'
```

This will output:

```text
A.one
A.two
A.three
B.one
B.two
B.three
C.one
C.two
C.three
done!
```

As with nesting loops with `call`, pay attention to the value of `i` - because
`i` will always be the iterator value of the current, innermost loop, grab the
value of the parent's iterator with `set` before the inner loop starts.

### flatten nested loops
You can achieve the same thing by using the built-in itertools' handy
[product](https://docs.python.org/3/library/itertools.html#itertools.product)
function. This is especially handy to simplify your pipeline structure if you
are nesting several levels deep.

```yaml
- name: pypyr.steps.pyimport
  in:
    pyImport: from itertools import product

- name: pypyr.steps.set
  in:
    set:
      list_a: ['A', 'B', 'C']
      list_b: ['one', 'two', 'three']

- name: pypyr.steps.echo
  foreach: !py product(list_a, list_b)
  in:
    echoMe: '{i[0]}.{i[1]}'

- name: pypyr.steps.echo
  in:
    echoMe: done!
```

As before, this will output:
```text
A.one
A.two
A.three
B.one
B.two
B.three
C.one
C.two
C.three
done!
```

Notice in this case `i` is the tuple that `product` returns, so you can use the
tuple index to access the current values of the iterator without having to grab
a reference to the parent iterator first with `set` as in the nested
`call` or `pype` examples above.