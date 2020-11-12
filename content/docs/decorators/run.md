---
title: run decorator
linktitle: run
date: 2020-07-14T11:02:07+01:00
description: Run this step only if condition True.
card_extra_summary:
  heading: example
  details:  |
          ```yaml
          run: False
          ```
card_extra_summary_is_code: True
# categories: [pipeline definition]
# keywords: ""
menu:
  docs:
    parent: decorators
    name: run
seo_article_headline: Selectively run a task-runner pipeline step.
seo_description: Only run a pipeline step if this condition is boolean True. Control which steps in your pipeline execute.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: run -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [control-of-flow, pipeline format]
---
# run
## selectively run step
Runs this step if `True`, skips step if `False`.

Default is `True`. The means by default pypyr will run a step unless you tell it
otherwise.

You'll almost always use `run` with 
[substitutions]({{< ref "/docs/substitutions">}}), so you set the value at 
run-time from context.

## set run with substitution expressions
You can use [truthy expressions]({{< ref "../decorators#bool-evaluation" >}}) 
with `run`, `skip` and `swallow`. This means you can selectively run a step
depending on if an object is not null and evaluates truthy.

```yaml
# ./run-decorator.yaml
steps:
  - name: pypyr.steps.echo
    comment: run is True by default
    in:
      echoMe: begin
  - name: pypyr.steps.echo
    run: False
    in:
      echoMe: this won't run coz run is False.
  - name: pypyr.steps.echo
    run: 0
    in:
      echoMe: 0 in a boolean context means False.
  - name: pypyr.steps.contextsetf
    comment: set some arb values to use in 
             next steps.
    in:
      contextSetf:
        isAnotherThing: 1 # int 1 evals true.
        isThing: False
        arbObj: [] # empty list
  - name: pypyr.steps.echo
    run: '{arbObj}'
    in:
      echoMe: won't run because truthy empty list is False
  - name: pypyr.steps.echo
    run: '{isAnotherThing}'
    in:
      echoMe: you'll see me.
  - name: pypyr.steps.echo
    in:
      echoMe: end
```

This pipeline will run like this:

```text
$ pypyr run-decorator
begin
you'll see me.
end
```

## only run step if variable exists
You can use a [py string expression]({{< ref
"/docs/substitutions/py-strings">}}) only to run a step if a key exists in 
context.

```yaml
- name: pypyr.steps.echo
  run: !py "'myvar' in locals()" 
  in:
    echoMe: You'll only see me if myvar exists in context.

- name: pypyr.steps.echo
  run: !py "'myvar' not in locals()" 
  in:
    echoMe: You'll only see me if myvar does NOT exist in context.
```

## combine run & skip
You can combine `run` and `skip` on the same step, only to run the step in
certain conditions and to skip it on other conditions. Remember that `skip`
supersedes run where `run=True`. If `run=False`, `skip` won't have any impact, 
the step will never run.

```yaml
# ./run-and-skip.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin
  - name: pypyr.steps.echo
    run: True
    skip: True
    in:
      echoMe: this won't run, because skip supersedes run.
  - name: pypyr.steps.echo
    comment: run evaluates first. 
             if run is False, skip won't have any influence,
             the step will never run.
    run: False
    skip: False
    in:
      echoMe: this won't run, because run evaluates before skip.
  - name: pypyr.steps.echo
    run: True
    skip: False
    in:
      echoMe: you'll see me.
  - name: pypyr.steps.echo
    in:
      echoMe: end
```

This will result in:

```text
$ pypyr run-and-skip
begin
you'll see me.
end
```