---
title: pypyr.steps.jump
linktitle: jump
date: 2020-07-06T12:43:39+01:00
description: Jump to another step-group. The rest of the current step-group doesn't run.
card_extra_summary:
  heading: input context property
  details: "`jump` (dict | str)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: jump
seo_article_headline: Jump to another step group in the same pipeline.
seo_description: Jump to another step-group in the same pipeline so that the rest of the current step-group does not run.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: jump -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [control-of-flow]
---
# pypyr.steps.jump
## jump to another step in pipeline
Jump to another step-group. This effectively stops processing on the
current step-group from which you are jumping.

If you want to return to the point of origin after the step-group you
jumped to completes, use [call]({{< ref "call">}}) instead.

## input
`jump` expects a context item `jump`. It can take one of two forms:

```yaml
- name: pypyr.steps.jump
  comment: simple string means just call the step-group named "jumphere"
  in:
    jump: jumphere
- name: pypyr.steps.jump
  comment: specify groups, success and failure.
  in:
    jump:
      groups: ['jumphere', 'andhere'] # list. Step-group sequence to jump to.
      success: group_to_call_on_success # string. Single step-group name.
      failure: group_to_call_on_failure # string. Single step-group name.
```

`jump.groups` can be a simple string if you're just jumping a single
group -i.e you don't need to make it a list of one item.

All inputs support [substitutions]({{< ref "/docs/substitutions">}}). This means 
you can dynamically specify the jump destination, success & failure handlers.

## example
`jump` is handy when you want to transfer control from a current
step-group to a different sequence of steps. So you can jump around to
your heart's content.

```yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: this is the 1st step of steps
  - name: pypyr.steps.jump
    in:
      jump: arbgroup
  - name: pypyr.steps.echo
    in:
     echoMe: You WON'T see me because we jumped.
arbgroup:
    - name: pypyr.steps.echo
      in:
        echoMe: this is arb group
    - pypyr.steps.stopstepgroup
    - name: pypyr.steps.echo
      in:
        echoMe: if you see me something is WRONG.
```

This will result in:

```text
this is the 1st step of steps
this is arb group
```

`jump` only runs success or failure groups if you actually specify these.

See a worked [example for jump](https://github.com/pypyr/pypyr-example/blob/master/pipelines/jump.yaml).