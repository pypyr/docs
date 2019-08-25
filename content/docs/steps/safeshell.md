---
title: pypyr.steps.safeshell
linktitle: safeshell
date: 2020-07-07T11:17:29+01:00
description: Deprecated alias for `cmd`
draft: false
card_extra_summary:
  heading: input context property
  details: "`cmd` (dict | str)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: safeshell
seo_article_headline: Deprecated alias for cmd.
seo_description: Use pypyr.steps.cmd instead.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: safeshell -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
# topics: [topic1, topic2, topic3]
---
# pypyr.steps.safeshell
Deprecated alias for [cmd]({{< ref "cmd">}}).

Example pipeline yaml:

```yaml
steps:
  - name: pypyr.steps.safeshell
    in:
      cmd: ls -a
```