---
title: pypyr.steps.stoppipeline
linktitle: stoppipeline
date: 2020-07-07T17:26:12+01:00
description: Stop current pipeline.
draft: false
card_extra_summary:
  heading: input context property
  # details: "`stoppipeline` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: stoppipeline
seo_article_headline: Stop current pipeline execution.
seo_description: Stop current pipeline immediately, but proceed with the calling parent pipeline during task-runner execution.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: stoppipeline -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [control-of-flow]
---
# pypyr.steps.stoppipeline
## stop current pipeline immediately
Stop current pipeline. Doesn't run any success or failure handlers, it
just stops the current pipeline.

This is handy if you are using [pypyr.steps.pype]({{< ref "pype" >}}) to call 
child pipelines from a parent pipeline, allowing you to stop just a child
pipeline but letting the parent pipeline continue.

You can always use `pypyr.steps.stoppipeline` as a simple step, because it 
doesn't need any input context properties.

## example
```yaml
- name: pypyr.steps.echo
  in:
    echoMe: you'll see me...
- pypyr.steps.stoppipeline
- name: pypyr.steps.echo
  in:
    echoMe: you WON'T see me...
```

See a worked [example to stop current pipeline](https://github.com/pypyr/pypyr-example/blob/master/pipelines/stop-pipeline.yaml).