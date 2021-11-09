---
title: pypyr.steps.envget
linktitle: envget
date: 2020-07-01T17:19:44+01:00
description: Get $ENVs & use a default fallback if they don't exist.
card_extra_summary:
  heading: input context property
  details: "`envget` (list)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: envget
seo_article_headline: Safely get environment variable into pipeline context. 
seo_description: Get environment variable with safe existence check that doesn't throw an error into a task-runner pipeline context.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: envget -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [environment variables]
---
# pypyr.steps.envget
## get environment variables into context
Get environment variables, and assign an optional default value to context if 
the sought environment variables do not exist.

The difference between `pypyr.steps.envget` and [pypyr.steps.env]({{< ref "env" >}})
, is that `pypyr.steps.envget` won't raise an error if the $ENV doesn't exist.

The `envget` context key must exist.

All inputs support [substitutions]({{< ref "docs/substitutions">}}).

See a worked [example for getting environment variables with defaults
here](https://github.com/pypyr/pypyr-example/tree/main/pipelines/envget.yaml).

## get an environment variable with a default fallback
```yaml
- name: pypyr.steps.envget
  description: if env MACAVITY is not there, set context theHiddenPaw to default.
  in:
    envGet:
      env: MACAVITY
      key: theHiddenPaw
      default: but macavity wasn't there!
```

-   `env`: Mandatory. This is the environment variable name. This is the
    bare environment variable name, do not put the $ in front of it.
-   `key`: Mandatory. The pypyr context key destination to which to copy
    the $ENV value.
-   `default` Optional. Assign this value to `key` if the $ENV
    specified by `env` doesn't exist.
    -   If you want to create a key in the pypyr context with an empty
        value, specify `null`.
    -   If you do NOT want to create a key in the pypyr context, do not
        have a default input.

## get environment variable with no default
```yaml
# save ENV_NAME to key. If ENV_NAME doesn't exist, do NOT set saveMeHere.
envGet:
  - env: ENV_NAME
    key: saveMeHere # saveMeHere won't be in context if ENV_NAME not there.
    # this is because the default keyword is not specified.
```

## get multiple environment variables in one step
If you need to get more than one $ENV, you can pass a list to `envget`.

```yaml
envGet:
  # get >1 $ENVs by passing them in as list items
  - env: ENV_NAME1 # mandatory
    key: saveMeHere1 # mandatory
    default: null # optional
  - env: ENV_NAME2
    key: saveMeHere2
    default: 'use-me-if-env-not-there' # optional
```
