---
title: pypyr.steps.filewritejson
linktitle: filewritejson
date: 2020-07-06T11:38:51+01:00
description: Write payload to file in json format.
draft: false
card_extra_summary:
  heading: input context property
  details: "`fileWriteJson` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: filewritejson
seo_article_headline: Write dynamic payload to json output file.
seo_description: Create a json file from any context input object with replacement token formatting in a task-runner pipeline.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: filewritejson -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [filesystem, json]
---
# pypyr.steps.filewritejson
## create json file from any context object
Format & write a payload to a json file on disk. This is useful for generating
json files from your pipeline such as when you want to create configuration 
files dynamically on the fly.

`filewritejson` works like this:

```yaml
- name: pypyr.steps.filewritejson
  comment: write context payload out to json
  in:
    fileWriteJson:
      path: /path/to/output.json # destination file
      payload: # (optional) payload to write to path
        key1: value1 # output json will have
        key2: value2 # key1 and key2 as string
        key3: 124 # output int
        key4: false # output bool
```

This will generate the following json to `/path/to/output.json`:

```json
{
  "key1": "value1",
  "key2": "value2",
  "key3": 123,
  "key4": false
}
```

Note that the correct data types will serialize to the generated output json. 
More complicated hierarchical nested structures will also just work.

If you do not specify `payload`, pypyr will write the entire context to
the output file in json format. Be careful if you have sensitive values
like passwords or private keys!

## format json with token replacements
All inputs support [substitutions]({{< ref "docs/substitutions">}}). This means 
that you can specify another context item to be the path and/or the payload, for
example:

```yaml
arbkey: arbvalue
writehere: /path/to/output.json
writeme:
  this: json content
  will: be written to
  thepath: with substitutions like this {arbkey}.
fileWriteJson:
  path: '{writehere}'
  payload: '{writeme}'
```

Substitution processing also runs on the output payload content.

In the above example, in the output json file created at `/path/to/output.json`, 
the `{arbkey}` expression in the last line will substitute like this:

```json
{
    "this": "json content",
    "will": "be written to",
    "thepath": "with substitutions like this arbvalue."
}
```

See a worked [filewritejson example](https://github.com/pypyr/pypyr-example/tree/master/pipelines/filewritejson.yaml).