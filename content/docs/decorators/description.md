---
title: description decorator
linktitle: description
date: 2020-07-11T20:35:07+01:00
description: Descriptive text that prints to output when step runs.
card_extra_summary:
  heading: example
  details: |
          ```yaml
          description: text to print to output
          ```
card_extra_summary_is_code: True
# categories: [pipeline definition]
# keywords: ""
menu:
  docs:
    parent: decorators
    name: description
seo_article_headline: Add text that prints to output when task-runner pipeline step runs.
seo_description: Pipeline authors can add text to print to output when a step runs. This is helpful to see pipeline progress.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: description -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [debug, print, pipeline format]
---
# description
## provide step status output
`description` is text that prints to the output when the pipeline runs. This is
useful to provide operators with visibility of pipeline progress, especially
where steps themselves do not provide any output.

The description outputs at NOTIFY - 25 level. This means you will see it by
default. If you want to suppress `description` output, run pypyr with 
`--log` higher than 25.

If you are looking to annotate your pipeline in a way that does not print to 
runtime output use [comment]({{<ref "comment">}}) instead. You can use both 
`description` and `comment` in combination too, which is handy to provide 
end-user output in addition to pipeline code annotation.

```yaml
- name: pypyr.steps.shell
  description: ->> deleting venv if venvCreate[purge]==True
  comment: using shell to allow ~ substitution
  run: !py venvCreate.get('purge', False)
  in:
    cmd: rm -rf {venvCreate[path]}
- name: pypyr.steps.shell
  description: ->> creating venv
  comment: using shell to allow ~ substitution
  in:
    cmd: python3 -m venv {venvCreate[path]}
```