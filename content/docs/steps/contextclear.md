---
title: pypyr.steps.contextclear
linktitle: contextclear
date: 2020-06-30T19:03:21+01:00
description: Remove specified items from context.
card_extra_summary:
  heading: input context property
  details: "`contextClear` (list)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: contextclear
seo_article_headline: Remove or delete items from the task-runner pipeline context.
seo_description: Remove or delete specified items from the task-runner pipeline context dictionary.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: Remove items from the pipeline taskcontext.
# social_og_image_alt: max 420 chars
topics: [context]
---
# pypyr.steps.contextclear
Remove the specified items from the 
[context]({{< ref "/docs/getting-started/basic-concepts#context">}}).

Will iterate `contextClear` and remove those keys from context.

```yaml
steps:
  - name: pypyr.steps.contextclear
    description: delete these 2 context keys
    in:
      contextClear:
        - removeMe
        - removeMeToo
```

For example, say input context is:

```yaml
key1: value1
key2: value2
key3: value3
key4: value4
contextClear:
    - key2
    - key4
    - contextClear
```

This will result in return context:

```yaml
key1: value1
key3: value3
```

Notice how `contextClear` also cleared itself in this example.