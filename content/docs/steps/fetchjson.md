---
title: pypyr.steps.fetchjson
linktitle: fetchjson
date: 2020-07-01T19:02:21+01:00
description: Load json file into pypyr context.
card_extra_summary:
  heading: input context property
  details: "`fetchJson` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: fetchjson
seo_article_headline: Load json into task-runner pipeline context at run-time.
seo_description: Load & parse a json file into the task-runner so that the pipeline can read, manipulate & change the data.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: fetchjson -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [filesystem, json]
---
# pypyr.steps.fetchjson
## load & parse json
Loads a json file into the pypyr context.

This step requires the following key in the pypyr context:

```yaml
- name: pypyr.steps.fetchjson
  comment: fetch json from path
  in:
    fetchJson:
      path: ./path.json # required. path to file on disk. can be relative.
      key: destinationKey # optional. write json to this context key.
```

If you do not specify `key`, json writes directly to context root.

If you do not want to specify a key, you can also use the streamlined
format:

```yaml
- name: pypyr.steps.fetchjson
  comment: simplified input with only path
  in:
    fetchJson: ./path.json # required. path to file on disk. can be relative.
```

All inputs support [substitutions]({{< ref "docs/substitutions">}}).


See some worked [examples of fetchjson](https://github.com/pypyr/pypyr-example/blob/master/pipelines/fetchjson.yaml).

## json structure
pypyr will merge the Json parsed from the file into the pypyr context. This
will overwrite existing values if the same keys already exist in context.

I.e if file json has `{'eggs' : 'boiled'}`, but context
`{'eggs': 'fried'}` already exists, after `fetchjson` the key `context['eggs']` 
will be 'boiled'.

{{% note warn %}}
If you do not specify `key`, the json must not be an Array `[]` at the
root level, but rather an Object `{}`.
{{% /note %}}