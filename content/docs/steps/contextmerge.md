---
title: pypyr.steps.contextmerge
linktitle: contextmerge
date: 2020-06-30T19:51:26+01:00
description: Merges values into context, preserving the existing context hierarchy.
card_extra_summary:
  heading: input context property
  details: "`contextMerge` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: contextmerge
seo_article_headline: Merge context & config properties in task-runner pipeline.
seo_description: Merge simple and complex types like lists and nested dictionaries in a task-runner pipeline.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: contextmerge -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [context]
---
# pypyr.steps.contextmerge
## merging context values
Merges values into context, preserving the existing hierarchy while only
updating the differing values as specified in the `contextMerge` input.

By comparison, [contextcopy]({{< ref "contextcopy" >}}) and 
[set]({{< ref "set" >}}) overwrite the destination hierarchy 
that is in context already.

This step merges the contents of the context key `contextMerge` into
context. The contents of the `contextMerge` key must be a dictionary.

## examples
For example, say input context is:

```yaml
key1: value1
key2: value2
key3:
    k31: value31
    k32: value32
contextMerge:
    key2: 'aaa_{key1}_zzz'
    key3:
        k33: value33_{key1}
    key4: 'bbb_{key2}_yyy'
```

This will result in return context:

```yaml
key1: value1
key2: aaa_value1_zzz
key3:
    k31: value31
    k32: value32
    k33: value33_value1
key4: bbb_aaa_value1_zzz_yyy
```

See a worked [example for contextmerge](https://github.com/pypyr/pypyr-example/blob/main/pipelines/contextmerge.yaml).

## merging collections like lists, dicts, sets & tuples
List, Set and Tuple merging is purely additive, with no checks for
uniqueness or already existing list items. E.g context [0,1,2] with 
`contextMerge` [2,3,4] will result in [0,1,2,2,3,4].

Keep this in mind especially where complex types like dicts nest inside
a list - a merge will always add a new dict list item, not merge it into
whatever dicts might exist on the list already.