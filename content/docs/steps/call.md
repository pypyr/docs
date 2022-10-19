---
title: pypyr.steps.call
linktitle: call
description: Call another step-group. Continue from the same place after the called groups complete.
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
card_extra_summary:
  heading: input context property
  details: "`call` (dict | str)"
categories: [steps]
menu:
  docs:
    parent: steps
    name: call
seo_article_headline: Call another step group in the same pipeline.
seo_description: Call another step-group in the same pipeline & continue from the invocation point after the invoked steps complete.
# social_og_description: 200 chars, if blank fall back to seo_description then description
social_og_title: Call another step group in the same pipeline.
# social_og_image_alt: max 420 chars
topics: [control-of-flow]
---
# pypyr.steps.call
## call another step in pipeline
Call (invoke) another [step-group]({{< ref "/docs/pipelines/pipeline-structure.md#step-groups" >}})
in the same pipeline. Once the called group(s) are complete, pypyr continues 
processing from the point where you initiated the call.

If you want to jump to a different step-group and ignore the rest of the
step-group you're in, use [pypyr.steps.jump]({{< ref "jump">}}) instead.

If you want to switch between calling different step-groups based on an input
expression in IF-THEN-ELSE branch style, use
[pypyr.steps.switch]({{< ref "switch">}}).

## input
`call` expects a context item *call*. It can take one of two forms:

```yaml
- name: pypyr.steps.call
  comment: simple string means just call the step-group named "callme"
  in:
    call: callme
- name: pypyr.steps.call
  comment: specify groups, success and failure.
  in:
    call:
      groups: ['callme', 'noreally'] # list. Step-groups to call.
      success: group_to_call_on_success # string. One step-group name.
      failure: group_to_call_on_failure # string. One step-group name.
```

`call.groups` can be a simple string if you're just calling a single
group - i.e you don't need to make it a list of one item.

```yaml
- name: pypyr.steps.call
  comment: specify single group
  in:
    call:
      groups: single_group # call a single group - no need for list.
      success: group_to_call_on_success # string. One step-group name.
      failure: group_to_call_on_failure # string. One step-group name.
```

`call` only runs success or failure groups if you actually specify these. You
don't _have_ to specify success or failure groups - if you don't, pypyr will
just run the groups you explicitly specified and continue when done without
trying to run any success or failure groups.

All inputs support string [substitutions]({{< ref "/docs/substitutions">}}).

## call inside a loop
`call` can be handy when you use it in conjunction with looping step
decorators like `while` or `foreach`:

```yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: this is the 1st step of steps
  - name: pypyr.steps.call
    in:
      call: arbgroup
  - name: pypyr.steps.echo
    in:
     echoMe: You'll see me AFTER arbgroup is done.
  - name: pypyr.steps.call
    foreach: ['one', 'two', 'three']
    in:
      call: repeatme
arbgroup:
    - name: pypyr.steps.echo
      in:
        echoMe: this is arb group
    - pypyr.steps.stopstepgroup
    - name: pypyr.steps.echo
      in:
        echoMe: if you see me something is WRONG.
repeatme:
    - name: pypyr.steps.echo
      in:
        echoMe: this is iteration {i}
```

This will result in:

```text
this is the 1st step of steps
this is arb group
You'll see me AFTER arbgroup is done.
this is iteration one
this is iteration two
this is iteration three
```

## worked example
See a worked [example for call](https://github.com/pypyr/pypyr-example/blob/main/pipelines/call.yaml).
