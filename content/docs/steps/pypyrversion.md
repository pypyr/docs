---
title: pypyr.steps.pypyrversion
linktitle: pypyrversion
date: 2020-07-07T10:10:38+01:00
description: Write current pypyr version number to output.
draft: false
card_extra_summary:
  heading: input context property
  # details: 
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: pypyrversion
seo_article_headline: Get the current pypyr version number.
seo_description: Output the currently installed pypyr version number.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: pypyrversion -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
# topics: [topic1, topic2, topic3]
---
# pypyr.steps.pypyrversion
## get current installed version 
Output the same as:

```bash
pypyr --version
```

This is an actual pipeline, though, so unlike `--version`, it'll use the 
standard pypyr logging format.

Example pipeline yaml:

```yaml
steps:
  - pypyr.steps.pypyrversion
```

Since this step does not have any input context, you can always run it as a 
[simple step]({{< ref "/docs/getting-started/basic-concepts#simple-step">}}) 
by just specifying the step-name as a string.