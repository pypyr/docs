---
title: pypyr.steps.glob
linktitle: glob
date: 2020-07-06T12:26:50+01:00
description: Get paths from glob expression.
card_extra_summary:
  heading: input context property
  details: "`glob` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: glob
seo_article_headline: Get all directories & files that match a glob.
seo_description: Get all existing paths that match a glob in a task-runner pipeline.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: glob -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [filesystem, paths]
---
# pypyr.steps.glob
## get all matching existing paths from glob
Resolve a glob and get all the paths that exist on the filesystem for the input 
glob.

Any given path can point to a file or a directory.

The `glob` context key must exist in input context:

```yaml
- name: pypyr.steps.glob
  in:
    glob: ./**/*.py # single glob
```

If you want to resolve multiple globs simultaneously and combine the
results, you can pass a list instead. You can freely mix literal paths
and globs.

```yaml
- name: pypyr.steps.glob
  in:
    glob:
      - ./file1 # literal relative path
      - ./dirname # also finds dirs
      - ./**/{arbkey}* # glob with a string formatting expression
```

After `glob` completes, the `globOut` context key is available. This
contains the results of the `glob` operation.

```yaml
globOut: # list of strings. Paths of all files found.
    ['file1', 'dir1', 'blah/arb']
```

You can use `globOut` as the list to enumerate in a `foreach` decorator
step, to run a step for each file found. In this example you run a pipeline on 
every file found by the glob.

```yaml
- name: pypyr.steps.glob
  in:
   glob: ./get-files/**/*
- name: pypyr.steps.pype
  foreach: '{globOut}'
  in:
    pype:
      name: pipeline-does-something-with-single-file
```

All inputs support [substitutions]({{< ref "docs/substitutions">}}). This means 
you can specify another context item to be an individual path, or part of a
path, or the entire path list.

See a worked [example for glob](https://github.com/pypyr/pypyr-example/tree/main/pipelines/glob.yaml).