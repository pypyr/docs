---
title: pypyr.parser.json
linktitle: json
date: 2020-07-08T19:18:15+01:00
description: Put json input string from cli arg into context.
card_extra_summary:
  heading: example input
  details: '`pypyr my-pipeline {"key1": "value 1", "key2": 123}`'
categories: [context parsers]
# keywords: ""
menu:
  docs:
    parent: context-parsers
    name: json
seo_article_headline: Parse json string from the cli arg input to task-runner.
seo_description: Parse json string from the cli input argument & create a strongly typed dict object from those values.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: json -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [args, json]
---
# pypyr.parser.json
## parse json input string from cli
Takes a json string from the cli argument input, parses it into a type-safe 
json object, and puts the object into the pypyr context.

pypyr honors the input json data types. In other words, bools are bools, numbers
are numbers, nulls are nulls.

## example
Given a pipeline like this, arbitrarily saved as `./json-parser.yaml`:
```yaml
# ./json-parser.yaml
context_parser: pypyr.parser.json
steps:
  # echoMe will be set in the input json str
  - pypyr.steps.echo
  - name: pypyr.steps.set
    comment: use the int as a strongly typed int
    in:
      set:
        intResult: !py keyInt * 2

  - name: pypyr.steps.echo
    comment: print out result of int calculation
    in:
      echoMe: "strongly typed happens automatically: {intResult}"

  - name: pypyr.steps.echo
    comment: only run if input bool is true
    run: '{keyBool}'
    in:
      echoMe: you'll only see me if the bool was true
```

You can run this pipeline as follows:
```text
$ pypyr json-parser '{"echoMe": "hello from json", "keyInt": 123, "keyBool": true}'
hello from json
strongly typed happens automatically: 246
you'll only see me if the bool was true
```

If you change the bool to `false`:
```text
$ pypyr json-parser '{"echoMe": "hello from json", "keyInt": 123, "keyBool": false}'
hello from json
strongly typed happens automatically: 246
```