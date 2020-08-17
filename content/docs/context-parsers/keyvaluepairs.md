---
title: pypyr.parser.keyvaluepairs
linktitle: keyvaluepairs
date: 2020-07-09T12:26:36+01:00
description: Pass key-value pairs from cli input args to the pipeline.
card_extra_summary:
  heading: example input
  details: '`pypyr pipelinename param1=value1 param2="value 2" "param 3"=value3`'
categories: [context parsers]
# keywords: ""
menu:
  docs:
    parent: context-parsers
    name: keyvaluepairs
seo_article_headline: Parsing key-value pairs from cli input arguments.
seo_description: Set key-value pairs as the task-runner cli input argument to use in the pipeline at run-time.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: keyvaluepairs -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [args]
---
# pypyr.parser.keyvaluepairs
## pass key value pairs from cli to pipeline
Takes `key=value` pair string from cli arg input and initialize context with a 
dictionary where each pair becomes a dictionary element.

Escape literal spaces with single or double quotes.

Given a pipeline like this, arbitrarily saved as `./keyvaluepairs-parser.yaml`:

```yaml
# ./keyvaluepairs-parser.yaml
context_parser: pypyr.parser.keyvaluepairs
steps:
  - pypyr.steps.debug # prints at log level <=20
```

You can then pass key-value pairs from the cli to the pipeline to initialize 
context. Notice that you can have both keys and values with spaces by using 
either single or double quotes.

```text
$ pypyr keyvaluepairs-parser param1=value1 param2='value 2' "param 3"=123 --log 20
{'param 3': '123', 'param1': 'value1', 'param2': 'value 2'}
```