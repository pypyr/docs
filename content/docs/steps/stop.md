---
title: pypyr.steps.stop
linktitle: stop
date: 2020-07-07T17:19:20+01:00
description: Stop pypyr entirely.
draft: false
card_extra_summary:
  heading: input context property
  # details: "`stop` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: stop
seo_article_headline: Stop pypyr task-runner execution.
seo_description: Stop all pipeline processing immediately & exit task-runner.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: stop -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [control-of-flow]
---
# pypyr.steps.stop
## stop pypyr immediately
Stop all pypyr processing immediately. Doesn't run any success or
failure handlers, it just stops everything in its tracks, even when
you're nested in child pipelines or a step-group call-chain.

You can always use `pypyr.steps.stop` as a simple step, because it doesn't
need any input context properties.

## example
```yaml
- name: pypyr.steps.echo
  in:
    echoMe: you'll see me...
- pypyr.steps.stop
- name: pypyr.steps.echo
  in:
    echoMe: you WON'T see me...
```

See a worked [example to stop all processing](https://github.com/pypyr/pypyr-example/blob/master/pipelines/stop.yaml).