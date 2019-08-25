---
title: pypyr.steps.echo
linktitle: echo
date: 2019-08-21
description: Echo context value of `echoMe` to the output.
draft: false
card_extra_summary:
  heading: input context property
  details: "`echoMe` (any)"
categories: [steps]
lastmod: 2019-08-21
publishdate: 2019-08-21
menu:
  docs:
    parent: steps
    name: echo
seo_article_headline: Echo pipeline step input to the console & log.
seo_description: Print simple & complex nested types to the stdout console & log output.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: Echo pipeline step input to the console & log.
# social_og_image_alt: max 420 chars
topics: [print]
---
# pypyr.steps.echo
## write dynamic values to console output stdout
Echo (i.e print) the context value `echoMe` to the output.

For example, if you had a pipeline like this:

```yaml
# ./mypipeline.yaml
context_parser: pypyr.parser.keyvaluepairs
steps:
  - name: pypyr.steps.echo
```

You can run:

```bash
$ pypyr mypipeline "echoMe=Ceci n'est pas une pipe"
```

Alternatively, if you had a pipeline like this:

```yaml
# ./look-ma-no-params.yaml
steps:
  - name: pypyr.steps.echo
    comment: Output echoMe
    in:
      echoMe: Ceci n'est pas une pipe
```

You can run:

```bash
$ pypyr look-ma-no-params
```

Supports string [substitutions]({{< ref "docs/substitutions">}}).

## write complex objects to output
`echo` will serialize complex objects like `dict` or `list` to stdout for you.

```yaml
# ./echo-list.yaml
steps:
  - name: pypyr.steps.contextsetf
    in:
      contextSetf:
        obj:
          - item1
          - item2
          - item3
  - name: pypyr.steps.echo
    in:
      echoMe: "This is a list: {obj}"
```

When you run this pipeline, the list will serialize to human-readable form:
```bash
$ pypyr echo-list
This is a list: ['item1', 'item2', 'item3']
```

If you want to dump context to output for troubleshooting purposes, [debug]({{< ref "debug" >}})
is useful for pretty printing with more human readable indented formatting.

## suppress echo output
`echo` prints to the NOTIFY (25) log-level. This means you won't see echo
output if you set log-level higher than 25. 

For example, `pypyr mypype --log 30` will only show WARNING and ERROR log 
messages, not any `echo` output.