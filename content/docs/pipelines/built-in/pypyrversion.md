---
title: pypyrversion built-in pipeline
linktitle: pypyrversion
date: 2022-10-05T14:07:29+01:00
description: Print the pypyr & python version numbers.
# card_extra_summary:
#   heading: input context property
#   details: "`pypyrversion` (dict)"
categories: [pipelines]
# keywords: ""
menu:
  docs:
    parent: builtin-pipelines
    identifier: pipeline-pypyrversion
    name: pypyrversion
# seo_article_headline: pypyrversion max 110 chars
# seo_description: max 158. for google serp snippet. if blank falls back to description.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: pypyrversion -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
# topics: [topic1, topic2, topic3]
---
# pypyrversion
Print the pypyr & current Python version numbers to stdout.

Run me like this:

```text
$ pypyr pypyrversion
pypyr 5.6.0 python 3.10.6
```

The pipeline uses
[pypyr.steps.pypyrversion]({{< ref "/docs/steps/pypyrversion">}}) under the
hood.

Running this pipeline does the same thing as the cli `--version` switch:

```text
$ pypyr --version
```