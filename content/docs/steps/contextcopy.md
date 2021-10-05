---
title: pypyr.steps.contextcopy
linktitle: contextcopy
date: 2020-08-15T20:28:42+01:00
description: Copy entire context keys.
card_extra_summary:
  heading: input context property
  details: "`contextCopy` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: contextcopy
seo_article_headline: Copy context key values in a task-runner pipeline.
seo_description: Copy context keys with their entire values from other context keys in a task-runner pipeline context.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: contextcopy -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [context]
---
# pypyr.steps.contextcopy
## copy values & structures from one part of context to another
Copies context values from already existing context values.

This is handy if you need to prepare certain keys in context where a
next step might need a specific key. If you already have the value in
context, you can create a new key (or update existing key) with that
value.

`contextcopy` and [set]({{< ref "set" >}}) overwrite existing keys. If you
want to merge new values into an existing destination hierarchy, use
[contextmerge]({{< ref "contextmerge" >}}) instead.

So let's say you already have context['currentKey'] = ['eggs']. 
If you set `newKey: currentKey`, you'll end up with:

```yaml
currentKey: eggs
newKey: eggs
```

## examples
For example, say your context looks like this,

```yaml
key1: value1
key2: value2
key3: value3
```

and your pipeline yaml looks like this:

```yaml
steps:
  - name: pypyr.steps.contextcopy
    in:
      contextCopy:
        key2: key1
        key4: key3
```

This will result in context like this:

```yaml
key1: value1
key2: value1
key3: value3
key4: value3
```

See a worked [example for contextcopy](https://github.com/pypyr/pypyr-example/tree/master/pipelines/contextcopy.yaml).