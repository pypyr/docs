---
title: pypyr.steps.pathcheck
linktitle: pathcheck
date: 2020-07-06T13:06:55+01:00
description: Check if paths exist on filesystem.
card_extra_summary:
  heading: input context property
  details: "`pathCheck` (list | str)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: pathcheck
seo_article_headline: Check if paths exist on filesystem.
seo_description: Check and count if multiple files exist on filesystem in a task-runner pipeline. With glob support.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: pathcheck -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [filesystem, paths]
---
# pypyr.steps.pathcheck
## check if paths exist
Check if a path exists on the filesystem. Supports globbing. A path can
point to a file or a directory.

## input
The `pathCheck` context key must exist.

```yaml
- name: pypyr.steps.pathcheck
  in:
    pathCheck: ./do/i/exist.arb # single literal path
- name: pypyr.steps.pathcheck
  in:
    pathCheck: ./**/*.py # single glob expression
```

If you want to check for the existence of multiple paths, you can pass a
list instead. You can freely mix literal paths and globs.

```yaml
- name: pypyr.steps.pathcheck
  in:
    pathCheck:
      - ./file1 # literal relative path
      - ./dirname # also finds dirs
      - ./**/{arbkey}* # glob with a string formatting expression
```

All inputs support string [substitutions]({{< ref "/docs/substitutions">}}). 
This means you can specify another context item to be an individual path, or 
part of a path, or the entire path list.

## output
After `pathcheck` completes, the `pathCheckOut` context key is
available. This contains the results of the `pathcheck` operation.

```yaml
pathCheckOut:
    # the key is the ORIGINAL input, no string formatting applied.
    'inpath-is-the-key': # one of these for each pathCheck input
        exists: true # bool. True if path exists.
        count: 0 # int. Number of files found for in path.
        found: ['path1', 'path2'] # list of strings. Paths of files found.
```

Example of passing a single input and the expected output context:

```yaml
pathCheck: ./myfile # assuming ./myfile exists in $PWD
pathCheckOut:
  './myfile':
    exists: true,
    count: 1,
    found:
      - './myfile'
```

The `exists` and `count` keys can be very useful for conditional
decorators to help decide whether to run subsequent steps. You can use
these directly in string formatting expressions without any extra fuss.

```yaml
- name: pypyr.steps.pathcheck
  in:
    pathCheck: ./**/*.arb
- name: pypyr.steps.echo
  run: '{pathCheckOut[./**/*.arb][exists]}'
  in:
    echoMe: you'll only see me if ./**/*.arb found something on filesystem.
```

See a worked [example for pathcheck](https://github.com/pypyr/pypyr-example/tree/main/pipelines/pathcheck.yaml).