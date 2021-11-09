---
title: pypyr.steps.env
linktitle: env
date: 2020-07-01T17:11:02+01:00
description: Get, set or unset $ENVs.
card_extra_summary:
  heading: input context property
  details: "`env` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: env
seo_article_headline: Get, set or unset environment variables in a task-runner pipeline.
seo_description: Get, set and unset environment variables in a pipeline workflow on a cross-platform task-runner.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: env -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [environment variables]
---
# pypyr.steps.env
## use environment variables in your pipeline
Get, set or unset environment variables.

The `env` context key must exist. `env` can contain a combination of
get, set and unset keys. You must specify at least one of `get`, `set`
and `unset`.

```yaml
env:
  get:
    contextkey1: env1
    contextkey2: env2
  set:
    env1: value1
    env2: value2
  unset:
    - env1
    - env2
```

This step will run whatever combination of Get, Set and Unset you
specify. Regardless of combination, execution order is: 
1. get
2. set
3. unset

```yaml
- name: pypyr.steps.env
  description: get, set and unset in a single step.
  in:
    env:
      get:
        key2: PWD
        key4: USER
      set:
        ARB_SET_ME1: 'arb value'
        ARB_SET_ME2: 'go go {key2} end end'
      unset:
        - ARB_SET_ME1
        - ARB_SET_ME2
```

See a worked [example for environment variables](https://github.com/pypyr/pypyr-example/tree/main/pipelines/env_variables.yaml).

## get
Get $ENVs into the pypyr context.

If the $ENV variable does not exist, this step will raise an error. If you want
to get an $ENV that might not exist without throwing an error, use
[pypyr.steps.envget]({{< ref "envget">}}) instead.

Input `context['env']['get']` must exist. It's a dictionary.

Values are the names of the $ENVs to write to the pypyr context.

Keys are the pypyr context item to which to write the $ENV values.

For example, say input context is:

```yaml
key1: value1
key2: value2
pypyrCurrentDir: value3
env:
  get:
    pypyrUser: USER
    pypyrCurrentDir: PWD
```

This will result in context:

```yaml
key1: value1
key2: value2
key3: value3
pypyrCurrentDir: <<value of $PWD here, not value3>>
pypyrUser: <<value of $USER here>>
```

## set
Set $ENVs from the pypyr context.

`context['env']['set']` must exist. It's a dictionary.

Values are strings to write to $ENV. You can use {key}
[substitutions]({{< ref "docs/substitutions">}}) to format the string from 
context. Keys are the names of the $ENV values to which to write.

For example, say input context is:

```yaml
key1: value1
key2: value2
key3: value3
env:
  set:
    MYVAR1: {key1}
    MYVAR2: before_{key3}_after
    MYVAR3: arbtexthere
```

This will result in the following $ENVs:

```yaml
$MYVAR1 == value1
$MYVAR2 == before_value3_after
$MYVAR3 == arbtexthere
```

Note that the $ENVs do not persist system-wide, they only exist for
the pypyr sub-processes, and as such for the subsequent steps during
this specific pypyr pipeline execution. If you set an $ENV here, don't expect
to see it in your system environment variables after the pipeline
finishes running.

## unset
Unset $ENVs.

Context is a dictionary or dictionary-like. context is mandatory.

`context['env']['unset']` must exist. It's a list. List items are the
names of the $ENV values to unset.

For example, say input context is:

```yaml
key1: value1
key2: value2
key3: value3
env:
  unset:
    - MYVAR1
    - MYVAR2
```

This will unset the following $ENVs:

```bash
$MYVAR1
$MYVAR2
```
