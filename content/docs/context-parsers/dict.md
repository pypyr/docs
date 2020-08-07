---
title: pypyr.parser.dict
linktitle: dict
date: 2020-07-08T18:44:23+01:00
description: Parse cli key=value pairs into `argDict`
draft: false
card_extra_summary:
  heading: example input
  details: '`pypyr pipelinename param1=value1 param2="value 2" param3=value3`'
categories: [context parsers]
# keywords: ""
menu:
  docs:
    parent: context-parsers
    name: dict
seo_article_headline: Create a dict from cli arg input to task-runner.
seo_description: Parse key=value pairs in a string from the cli input args & create a dict from those values.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: dict -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [args]
---
# pypyr.parser.dict
## create a dict from key=value pair string
Takes a key=value pair string and returns a dictionary (aka a map) where each
pair becomes a dictionary element inside a dict with name `argDict`.

Escape literal spaces with single or double quotes.

## example
```bash
$ pypyr pipelinename param1=value1 param2="value 2" param3=value3
```

This will create a context dictionary like this:

```python
{'argDict': {'param1': 'value1',
             'param2': 'value 2',
             'param3': 'value3'}}
 ```                                

Or to put in in yaml terms:

```yaml
argDict:
  param1: value1
  param2: value 2
  param3: value3
 ```

 You can access the dict you create this way with 
 [formatting expressions]({{< ref "/docs/substitutions">}}):

 ```yaml
 context_parser: pypyr.parser.dict
 steps:
   - name: pypyr.steps.echo
     in:
       echoMe: param2 value is {argDict[param2]}
 ```
