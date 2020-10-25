---
title: step decorators
linktitle: decorators
description: Selectively run, skip, repeat, loop & handle errors on individual steps.
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
# categories: [pipeline definition]
menu:
  docs:
    identifier: decorators-overview
    name: overview
    parent: decorators
    weight: -100
list_fields:
  - fallback: title
    field: linktitle
    heading: title
    islink: true
  - heading: description
    fallback: summary
    field: description
  - heading: example
    field: card_extra_summary.details
# list_style: section-list/table
seo_article_headline: Apply looping, conditional & error logic to any task-runner step.
seo_description: Selectively run, skip, repeat & loop any single step in pipeline. Handle errors & automatic retries. 
seo_is_carousel: true
topics: [built-in summary tables, pipeline format]
---
# step decorators
## conditional execution, looping & error handling for any step
Complex steps have various optional step decorators that change how or
if a step is run. You _decorate_ your step's core function with these extra
behavioral attributes.

You can set any decorator on any step in your pipeline in any combination.

Don't bother specifying these unless you want to deviate from the default 
values. If you don't need any decorators for your step, you might as well save
yourself some typing and use the step in 
[simple mode]({{< ref "/docs/getting-started/basic-concepts#simple-step">}}) 
instead by just specifying the bare step name.

```yaml
steps:
  - name: my.package.another.module
    description: Optional description is for humans.
                 Any yaml-escaped text that makes your life easier.
                 Outputs to console during run-time.
    comment: Optional comments for pipeline developers. Like code comments.
             Does NOT output to console during run.
    in: # optional. Add these parameters to context for this step.
        # these key-value pairs only in scope for this step.
      parameter1: value1
      parameter2: value2
    foreach: [] # optional. Repeat the step once for each item in this list.
    onError: # optional. Custom Error Info to add to error if step fails.
      code: 111 # you can also use custom elements for your custom error.
      description: arb description here
    retry: # optional. Retry step until it doesn't raise an error.
      max: 1 # max times to retry. integer. Defaults None (infinite).
      sleep: 0 # sleep between retries, in seconds. Decimals allowed. Defaults 0.
      stopOn: ['ValueError', 'MyModule.SevereError'] # Stop retry on these errors. Defaults None (retry all).
      retryOn: ['TimeoutError'] # Only retry these errors. Defaults None (retry all).
    run: True # optional. Run this step if True, skip step if False. Defaults to True if not specified.
    skip: False # optional. Skip this step if True, run step if False. Defaults to False if not specified.
    swallow: False # optional. Swallow any errors raised by the step. Defaults to False if not specified.
    while: # optional. repeat step until stop is True or max iterations reached.
      stop: '{keyhere}' # loop until this evaluates True.
      max: 1 # max loop iterations to run. integer. Defaults None (infinite).
      sleep: 0 # sleep between iterations, in seconds. Decimals allowed. Defaults 0.
      errorOnMax: False # raise error if you reach max. Defaults False.
```

All step decorators support [substitutions]({{< ref "/docs/substitutions">}}).
This means you can dynamically use python expressions & token substitution 
strings that pypyr will interpret at runtime against changeable context values.

You can use [py strings]({{<ref "/docs/substitutions/py-strings" >}}) for 
dynamic boolean conditions like `len(key) > 0`.

If there are no looping decorators, the step will execute once by default, 
unless the conditional decorators skip the step.

If all of this sounds complicated, don't panic! If you don't bother
with any of these the step will just run once by default.

## bool evaluation
Note that for all bool values, the standard 
[Python truth value testing](https://docs.python.org/3/library/stdtypes.html#truth-value-testing)
rules apply.

Simply put, this means that `1`, `TRUE`, `True` and `true` will be `True`.

`None`/Empty, `0`,`''`, `[]`, `{}` will be `False`.

{{% note tip %}}
If pypyr finds a string where it expects a boolean, e.g on `run`, `skip` or 
`swallow`, it will interpret case insensitive string `"true"`, `"1"` & `"1.0"` 
as boolean `True`. All other string values, including empty string, evaluate to 
`False`.

This is generally not what typical programming languages do on a strict string
truthy, where any given string value other than null/empty will evaluate 
`True`, but more often than not within the context of a pipeline it saves you 
some footwork specially having to cast strings to booleans first.

If you do want the more typical string truthy evaluation, use an explicit 
py-string like this:

```yaml
myString: arbitrary string here

run: !py bool(myString)
```
{{% /note %}}

## order of precedence
Decorators can interplay, meaning that the sequence of evaluation is
important.

- `run` or `skip` controls whether a step should execute on any given loop 
  iteration, without affecting continued loop iteration.
- `run` could be `True` but if `skip` is `True` it will still skip the step.
- A step can run multiple times in a `foreach` loop for each iteration of a 
  `while` loop.
- `swallow` can evaluate dynamically inside a loop to decide whether
  to swallow an error or not on a particular iteration.
- `swallow` can swallow an error AFTER `retry` exhausted max attempts.

```yaml
in # in evals once and only once at the beginning of step
  -> while # everything below loops inside while
    -> foreach # everything below loops inside foreach
      -> run # evals dynamically on each loop iteration
       -> skip # evals dynamically on each loop iteration after run
        -> retry # repeats step execution until no error
          [>>>actual step execution here<<<]
        -> swallow # evaluates dynamically on each loop iteration
```

## decorator listing