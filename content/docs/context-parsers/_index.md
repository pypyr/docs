---
title: context parsers
# linktitle: pypyr context parser overview
description: pypyr context parsers initialize the context from your own custom cli input arguments.
date: 2020-08-12
draft: false
publishdate: 2020-08-13
lastmod: 2020-08-16
# categories: [context]
menu:
  docs:
    identifier: context-parser-overview
    name: overview
    parent: context-parsers
    weight: -100
list_fields:
  - fallback: title
    field: linktitle
    heading: title
    islink: true
  # - heading: description
  #   fallback: summary
  #   field: description
  - heading: example cli input
    field: card_extra_summary.details
list_style: section-list/table
seo_is_carousel: true
seo_article_headline: Built-in pypyr context parsers for cli args.
seo_description: Parse task-runner pipeline custom cli input arguments as key-value pairs, comma delimited values, simple strings & more.
topics: [built-in summary tables, context]
---
# context parsers
pypyr has a whole bunch of ready-made built-in context parsers to make your 
life easier. A context parser allows you to customize how you want to pass cli 
arguments to your pipeline.

Making your own [custom context-parser]({{< ref "/docs/api/context-parser" >}}) 
is super easy too.

```fish
# everything after mypipeline goes to the context_parser
$ pypyr mypipeline these are all context input arguments
$ pypyr mypipeline key1=value2 key2="value 2"
```

A `context_parser` parses the pypyr command's context input arguments.
This is all the positional arguments after the pipeline-name from the
command line.

The chances are pretty good that the `context_parser` will take the
context command arguments and put in into the pypyr context.

The [pypyr context]({{< ref "/docs/getting-started/basic-concepts#context">}}) 
is a dictionary that is in scope for the duration of the entire pipeline. The 
`context_parser` can initialize the context. Any step in the pipeline can add, 
edit or remove items from the context dictionary.

## built-in context parsers