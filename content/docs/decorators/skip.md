---
title: skip decorator
linktitle: skip
date: 2020-07-14T11:38:26+01:00
description: Skip this step if condition True.
card_extra_summary:
  heading: example
  details: |
          ```yaml
            skip: True
          ```
card_extra_summary_is_code: True
# categories: [pipeline definition]
# keywords: ""
menu:
  docs:
    parent: decorators
    name: skip
seo_article_headline: Selectively skip a task-runner pipeline step.
seo_description: Don't run a pipeline step if this condition is boolean True. Control which steps in your pipeline execute.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: skip -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [control-of-flow, pipeline format]
---
# skip
## selectively skip step
Skip this step if `True`, run step if `False`. Evaluates after the `run` 
decorator. This means that if `run` is `False`, the step will never run, 
regardless of `skip`.

Default is `False`. This means by default pypyr will not skip a step.

If this looks like it's merely the inverse of `run`, that's because it is. Use 
whichever suits your pipeline better, or combine run and skip in the same 
pipeline to toggle at runtime which steps you want to execute. You can also
[combine run and skip]({{< ref "run#combine-run--skip" >}}) in the same step.

You'll almost always use `skip` with 
[substitutions]({{< ref "/docs/substitutions">}}), so you set the value at 
run-time from context.

## set skip with substitution expressions
You can use [truthy expressions]({{< ref "../decorators#bool-evaluation" >}}) 
with `run`, `skip` and `swallow`. This means you can selectively skip a step
depending on if an object is not null and evaluates truthy.

```yaml
# ./skip-decorator.yaml
steps:
  - name: pypyr.steps.echo
    comment: skip is False by default
    in:
      echoMe: begin
  - name: pypyr.steps.echo
    skip: True
    in:
      echoMe: this won't run coz skipping the step.
  - name: pypyr.steps.echo
    skip: 1
    in:
      echoMe: 1 in a boolean context means True.
  - name: pypyr.steps.set
    comment: set some arb values to use in 
             next steps.
    in:
      set:
        isAnotherThing: 0 # int 0 evals False.
        isThing: False
        arbObj: ["one", "two", "three"] # list
  - name: pypyr.steps.echo
    skip: '{arbObj}'
    in:
      echoMe: you won't see me because truthy list is True.
  - name: pypyr.steps.echo
    skip: '{isAnotherThing}'
    in:
      echoMe: you'll see me.
  - name: pypyr.steps.echo
    in:
      echoMe: end
```

This pipeline will run like this:
```text
pypyr skip-decorator
begin
you'll see me.
end
```

## skip step if variable exists
You can use a [py string expression]({{< ref
"/docs/substitutions/py-strings">}}) to skip a step based on whether a key
exists in context.

```yaml
- name: pypyr.steps.echo
  skip: !py "'myvar' in locals()" 
  in:
    echoMe: skip this step if myvar exists in context.

- name: pypyr.steps.echo
  skip: !py "'myvar' not in locals()" 
  in:
    echoMe: do not skip this step if myvar exists in context.
```