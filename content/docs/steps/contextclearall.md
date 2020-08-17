---
title: pypyr.steps.contextclearall
linktitle: contextclearall
date: 2020-06-30T19:04:00+01:00
description: Wipe the entire context.
card_extra_summary:
  heading: input context property
  # details: 
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: contextclearall
seo_article_headline: contextclearall to wipe the entire pipeline context
seo_description: Wipe or purge the entire task-runner pipeline context.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: contextclearall -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [context]
---
# pypyr.steps.contextclearall
Wipe the entire 
[context]({{< ref "/docs/getting-started/basic-concepts#context">}}). No input 
context arguments required.

You can always use `contextclearall` as a 
[simple step]({{< ref "/docs/getting-started/basic-concepts#simple-step">}}), 
since it does not require any input context.

Sample pipeline yaml:

```yaml
steps:
  - my.arb.step
  - pypyr.steps.contextclearall
  - another.arb.step
```