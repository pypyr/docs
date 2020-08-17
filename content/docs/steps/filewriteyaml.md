---
title: pypyr.steps.filewriteyaml
linktitle: filewriteyaml
date: 2020-07-06T12:07:43+01:00
description: Write payload to file in yaml format.
card_extra_summary:
  heading: input context property
  details: "`fileWriteYaml` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: filewriteyaml
seo_article_headline: Write dynamic payload to yaml output file.
seo_description: Create a yaml file from any context input object with replacement token formatting in a task-runner pipeline.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: filewriteyaml -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [filesystem, yaml]
---
# pypyr.steps.filewriteyaml
## create yaml file from any context object
Format & write a payload to a yaml file on disk. This is useful for generating
yaml files from your pipeline such as when you want to create configuration 
files dynamically on the fly.

`filewriteyaml` works like this:

```yaml
- name: pypyr.steps.filewriteyaml
  comment: write context payload out to yaml
  in:
  fileWriteYaml:
    path: /path/to/output.yaml # destination file
    payload: # (optional) payload to write to path
      key1: value1 # output yaml will have
      key2: value2 # key1 and key2 as string.
      key3: 124 # output int
      key4: false # output bool
```

This will generate the following json to `/path/to/output.yaml`:

```yaml
key1: value1
key2: value2
key3: 123
key4: false
```

Note that the correct data types will serialize to the generated output yaml. 
More complicated hierarchical nested structures will also just work.

If you do not specify `payload`, pypyr will write the entire context to
the output file in yaml format. Be careful if you have sensitive values
like passwords or private keys!

## format yaml with token replacements
All inputs support [substitutions]({{< ref "docs/substitutions">}}). This means 
you can specify another context item to be the path and/or the payload, for
example:

```yaml
arbkey: arbvalue
writehere: /path/to/output.yaml
writeme:
  this: yaml content
  will: be written to
  thepath: with substitutions like this {arbkey}.
fileWriteYaml:
  path: '{writehere}'
  payload: '{writeme}'
```

Substitution processing runs on the output. In this example, in the output yaml 
file created at `/path/to/output.yaml`, the `{arbkey}` expression in the last 
line will substitute like this:

```yaml
this: yaml content
will: be written to
thepath: with substitutions like this arbvalue.
```

See a worked [filewriteyaml example](https://github.com/pypyr/pypyr-example/tree/master/pipelines/filewriteyaml.yaml).