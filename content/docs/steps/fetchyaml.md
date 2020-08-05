---
title: pypyr.steps.fetchyaml
linktitle: fetchyaml
date: 2020-07-01T20:02:35+01:00
description: Load yaml file into pypyr context.
draft: false
card_extra_summary:
  heading: input context property
  details: "`fetchYaml` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: fetchyaml
seo_article_headline: Load yaml into task-runner pipeline context at run-time.
seo_description: Load & parse a yaml file into the task-runner so that the pipeline can read, manipulate & change the data.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: fetchyaml -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [filesystem, yaml]
---
# pypyr.steps.fetchyaml
## load & parse yaml
Loads a yaml file into the pypyr context.

This step requires the following key in the pypyr context:

```yaml
 - name: pypyr.steps.fetchyaml
    description: fetch yaml from path
    in:
      fetchYaml:
        path: ./path.yaml # required. path to file on disk. can be relative.
        key: destinationKey # optional. write yaml to this context key.
```

If you do not specify `key`, yaml writes directly to context root.

If you do not want to specify a key, you can also use the streamlined
format:

```yaml
 - name: pypyr.steps.fetchyaml
    description: fetch yaml with simplified path input
    in:
      fetchYaml: ./path.yaml # required. path to file on disk. can be relative.
```

All inputs support [substitutions]({{< ref "docs/substitutions">}}).


See some worked [examples of fetchyaml](https://github.com/pypyr/pypyr-example/blob/master/pipelines/fetchyaml.yaml).

## merging logic
pypyr will parse the yaml from the file and merge it into the pypyr context. 
This will overwrite existing values if the same keys are already in there.

I.e if file yaml has

```yaml
eggs: boiled
```

but context `{'eggs': 'fried'}` already exists, after `fetchyaml` context will
be:

```yaml
eggs: fried
```

## input yaml structure
{{% note warn %}}
If you do not specify `key`, the yaml should NOT be a list at the top
level, but rather a mapping.
{{% /note %}}

So the top-level yaml should not look like this:

```yaml
- eggs
- ham
```

but rather like this:

```yaml
breakfastOfChampions:
  - eggs
  - ham
```