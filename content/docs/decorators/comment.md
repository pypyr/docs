---
title: comment decorator
linktitle: comment
date: 2020-07-11T20:13:51+01:00
description: Annotations for pipeline authors.
draft: false
card_extra_summary:
  heading: example
  details: |
          ```yaml
          comment: arb text
          ```
card_extra_summary_is_code: True
categories: [pipeline definition]
# keywords: ""
menu:
  docs:
    parent: decorators
    name: comment
seo_article_headline: Add annotations or comments to task-runner pipeline steps.
seo_description: Pipeline authors can annotate task-runner pipeline steps with comments. These do not print to the output at runtime.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: comment -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
# topics: []
---
# comment
## comment your pipeline code
Similar to code comments, `comment` is for pipeline authors to annotate a 
pipeline step for the usual reasons. Like remembering why on earth you did
something in a certain way two weeks later.

`comment` does not output at run-time ever. If you're looking to add descriptive
text that prints to output for pipeline consumer visibility, use 
[description]({{<ref "description">}}) instead.

```yaml
- name: pypyr.steps.contextsetf
  comment: this comment is for pipeline authors.
           it is multi-line to prove a point.
           it probably describes a technical point
           about why the step is doing something.
  in:
    contextSetf:
      newKey: 'XXX {invalue} XXX'
```

 |
          ```yaml
          comment: arb text
          ```