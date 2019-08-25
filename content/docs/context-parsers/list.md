---
title: pypyr.parser.list
linktitle: list
date: 2020-07-09T16:34:56+01:00
description: Put each cli input arg into a list `argList`.
draft: false
card_extra_summary:
  heading: example input
  details: "`pypyr pipelinename param1 param2 param3`"
categories: [context]
# keywords: ""
menu:
  docs:
    parent: context-parsers
    name: list
seo_article_headline: Create a list from cli args input to task-runner.
seo_description: Append each cli input argument into a list to initialize the pipeline context.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: list -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [args]
---
# pypyr.parser.list
## parse cli input args into list
Append each input argument into a list `argList` to initialize context.

Escape literal spaces with single or double quotes.

Given a pipeline like this, arbitrarily saved as `./list-parser.yaml`:

```yaml
# ./list-parser.yaml
context_parser: pypyr.parser.list
steps:
  - pypyr.steps.debug # prints at log level <=20
  - name: pypyr.steps.echo
    comment: the args passed into the pypyr with list parser 
             end up as a list in context under key argList
    in:
      echoMe: "the 2nd thing on the input list is: {argList[1]}"
```

Each argument you pass via the cli will now be in the `argList` list:


```text
$ pypyr list-parser eggs bacon ham --log 20
{'argList': ['eggs', 'bacon', 'ham']}
the 2nd thing on the input list is: bacon
```