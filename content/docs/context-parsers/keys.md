---
title: pypyr.parser.keys
linktitle: keys
date: 2020-07-09T12:06:01+01:00
description: Boolean cli switches `True` if exist.
card_extra_summary:
  heading: example input
  details: "`pypyr pipelinename param1 'par am2' param3`"
categories: [context parsers]
# keywords: ""
menu:
  docs:
    parent: context-parsers
    name: keys
seo_article_headline: Custom boolean switches for the task-runner cli.
seo_description: Parse existence check for cli arg switches as boolean True to use in a pipeline.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: keys -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [args]
---
# pypyr.parser.keys
## custom boolean switches from the cli
For each input argument, initialize context with a dictionary where each 
argument becomes the key, with value set to `True`.

Escape literal spaces with single or double quotes.

Given a pipeline like this, arbitrarily saved as `./keys-parser.yaml`:
```yaml
# ./keys-parser.yaml
context_parser: pypyr.parser.keys
steps:
  - pypyr.steps.debug # prints at log level <=20
```

Running the pipeline with different inputs:

```text
$ pypyr keys-parser --log 20
{}

$ pypyr keys-parser a b c --log 20
{'a': True, 'b': True, 'c': True}

$ pypyr keys-parser "a with space" ' b' c --log 20
{' b': True, 'a with space': True, 'c': True}
```