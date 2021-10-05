---
title: in decorator
linktitle: in
date: 2020-07-11T12:56:32+01:00
description: Add arguments to context for the current step.
card_extra_summary:
  heading: example
  details: |
            ```yaml
            in:
              key: value
              anotherkey: anothervalue
            ```
card_extra_summary_is_code: True
# categories: [pipeline definition]
# keywords: ""
menu:
  docs:
    parent: decorators
    name: in
seo_article_headline: Set input arguments for a task-runner step.
seo_description: Set input arguments for an individual pipeline step with a map/dict.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: in -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [context, pipeline format]
---
# in
## add input arguments to step context
`in` sets the input arguments for a step. pypyr adds anything in `in` to the 
context so that the decorated step can use these key-value pairs.

`in` is a mapping, also known as a dict `{}`. You can use complex, nested 
structures. pypyr will honor the data types of the yaml values you set in your 
pipeline.

```yaml
# arbitrarily complex nested input args, 
# with different data types, 
# just to make a point.
in:
  key1: value 1
  key2: 
    - list item 1
    - 2.2: dict inside a list
      2.3: [1, 2, 3]
  key3: 456
  key4: True
  key5:
    nestedDictKey: nested map value
    anotherKey: another value
```

`in` evaluates once at the beginning of step execution, BEFORE the `foreach` 
and `while` decorators. It does not re-evaluate for each loop iteration.

## scope
`in` is only in scope for the duration of the step it decorates. Context 
properties that you set here in `in` do not endure after the step completes.

If you want to set context properties that exist beyond the current step, use
one of the [context]({{< ref "/topics/context">}}) steps, such as 

- [pypyr.steps.set]({{< ref "/docs/steps/set">}})
- [pypyr.steps.contextcopy]({{< ref "/docs/steps/contextcopy">}})
- [pypyr.steps.default]({{< ref "/docs/steps/default">}})

## set context parameters in preceding steps
Although `in` is handy to set input parameters for a specific step, you can
also prepare context parameters beforehand in a previous step using one of the 
context setting built-in steps. You can prepare your step's context key(s) as 
many steps previous as you like in this way, and there can be other steps in
between too, as long no step in between resets or overwrites that particular 
context key.

```yaml
# ./context-and-in.yaml
steps:
  - name: pypyr.steps.set
    comment: set echoMe for the following step here
    in:
      set:
        echoMe: arbitrary value here
  - pypyr.steps.echo # echo echoMe, set in previous step
  - name: pypyr.steps.set
    comment: newKey will stay in context after this step
    in:
      arb: arb substitution value
      set:
        newKey: 'XXX {arb} YYY'
  - name: pypyr.steps.contextcopy
    comment: copy newKey to echoMe
    in:
      contextCopy:
        echoMe: newKey
  - pypyr.steps.echo # output echoMe, set in previous step
```

This pipeline will result in:
```text
$ pypyr context-and-in
arbitrary value here
XXX arb substitution value YYY
```