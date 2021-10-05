---
title: pypyr.steps.default
linktitle: default
date: 2020-07-01T12:22:55+01:00
description: Set values if they do not exist already.
card_extra_summary:
  heading: input context property
  details: "`defaults` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: default
seo_article_headline: Initialize the task-runner pipeline context with default values
seo_description: Assign default values to the pipeline context if those keys do not already have values.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: default -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [context]
---
# pypyr.steps.default
## initialize the context with default values
Sets values in context if they do not exist already. Does not overwrite
existing values. Supports nested hierarchies.

This is especially useful for setting default values in context, for
example when using [optional arguments]({{< ref "/docs/cli/custom-args" >}}) 
from the cli.

This step sets the contents of the context key `defaults` into context
where keys in `defaults` do not exist in context already. The contents
of the `defaults` key must be a dictionary.

By comparison, the `in` step decorator, and also the steps
[contextcopy]({{< ref "contextcopy" >}}), [set]({{< ref "set" >}}) and
[contextmerge]({{< ref "contextmerge" >}}) overwrite values even if they are in
context already.

The recursive if-not-exists-then-set check happens for dictionaries, but
not for items in Lists, Sets and Tuples. You can set default values of
type List, Set or Tuple if their keys don't exist in context already,
but this step will not recurse through the List, Set or Tuple itself.

Supports [substitutions]({{< ref "docs/substitutions">}}). String interpolation 
applies to keys and values.

## example
Given a context like this:

```yaml
key1: value1
key2:
    key2.1: value2.1
key3: None
```

And `defaults` input like this:

```yaml
- name: pypyr.steps.default
  in:
    defaults:
      key1: updated value here won't overwrite since it already exists
      key2:
          key2.2: value2.2
      key3: key 3 exists so I won't overwrite
```

Will result in context:

```yaml
key1: value1
key2:
    key2.1: value2.1
    key2.2: value2.2
key3: None
```

See a worked [example for setting default values](https://github.com/pypyr/pypyr-example/blob/master/pipelines/default.yaml).
