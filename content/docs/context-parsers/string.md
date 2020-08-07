---
title: pypyr.parser.string
linktitle: string
date: 2020-07-09T16:52:56+01:00
description: Concatenate all cli input args into single string `argString`.
draft: false
card_extra_summary:
  heading: example input
  details: "`pypyr pipelinename arbitrary string here`"
categories: [context parsers]
# keywords: ""
menu:
  docs:
    parent: context-parsers
    name: string
seo_article_headline: Combine all cli input args into a single string
seo_description: Combine arbitrary cli input args into one string to use in the task-runner pipeline at run-time.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: string -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [args]
---
# pypyr.parser.string
## put all cli input args into a single string
Takes any arbitrary input from the cli input args and concatenate into a single 
string `argString` to initialize context.

If you have multiple sequential literal spaces right next to each other, escape
these with single or double quotes.

Given a pipeline like this, arbitrarily saved as `./string-parser.yaml`:

```yaml
# ./string-parser.yaml
context_parser: pypyr.parser.string
steps:
  - pypyr.steps.debug # prints at log level <=20
  - name: pypyr.steps.echo
    comment: use argString in format expression
    in:
        echoMe: "this came from the cli: {argString}"
```

You can then pass any given string, including nothing, from the cli to the 
pipeline to initialize context:

```text
$ pypyr string-parser --log 20
{'argString': None}
this came from the cli: None

$ pypyr string-parser arbitrary words here --log 20
{'argString': 'arbitrary words here'}
this came from the cli: arbitrary words here

$ pypyr string-parser a b c --log 20
{'argString': 'a b c'}
this came from the cli: a b c

$ pypyr string-parser "a " " b" 'c' --log 20
{'argString': 'a   b c'}
this came from the cli: a   b c

$ pypyr string-parser "a b  b c" --log 20
{'argString': 'a b  b c'}
this came from the cli: a b  b c
```