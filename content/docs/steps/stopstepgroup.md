---
title: pypyr.steps.stopstepgroup
linktitle: stopstepgroup
date: 2020-07-07T17:36:13+01:00
description: Stop current step-group.
card_extra_summary:
  heading: input context property
  # details: "`stopstepgroup` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: stopstepgroup
seo_article_headline: Stop current step-group execution.
seo_description: Stop current step-group immediately, but proceed with the calling step-groups during task-runner execution.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: stopstepgroup -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [control-of-flow]
---
# pypyr.steps.stopstepgroup
Stop current step-group. Doesn't run any success or failure handlers,
it just stops the current step-group.

This is handy if you are using [pypyr.steps.call]({{< ref "call" >}}) or 
[pypyr.steps.jump]({{< ref "jump" >}}) to run different step-groups, 
allowing you to stop just a child step-group but letting the calling parent 
step-group continue.

You can always use `pypyr.steps.stopstepgroup` as a simple step, because it 
doesn't need any input context properties.

If you use a Stop step-group instruction inside a 
[failure handler]({{< ref "/docs/getting-started/error-handling#failure-handlers" >}}) 
it will stop processing at that point AND not quit reporting failure. Do this 
when you want to handle an error condition and not raise an error to the caller.

## examples
```yaml
steps:
  - name: pypyr.steps.call
    in:
      call:
        groups: arbgroup
  - name: pypyr.steps.echo
    in:
     echoMe: You'll see me because only arbgroup was stopped.

arbgroup:
    - name: pypyr.steps.echo
      in:
        echoMe: this is arb group
    - pypyr.steps.stopstepgroup
    - name: pypyr.steps.echo
      in:
        echoMe: if you see me something is WRONG.
```

See a worked [example to stop a step-group
here](https://github.com/pypyr/pypyr-example/blob/master/pipelines/stop-stepgroup.yaml).
