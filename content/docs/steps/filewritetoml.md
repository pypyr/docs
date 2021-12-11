---
title: pypyr.steps.filewritetoml
linktitle: filewritetoml
date: 2021-12-11T11:38:51+01:00
description: Write payload to file in toml format.
card_extra_summary:
  heading: input context property
  details: "`fileWriteToml` (dict)"
categories: [steps]
menu:
  docs:
    parent: steps
    name: filewritetoml
seo_article_headline: Write dynamic payload to toml output file.
seo_description: Create a toml file from any context input object with replacement token formatting in a task-runner pipeline.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: filewritetoml -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [filesystem, toml]
---
# pypyr.steps.filewritetoml
## create toml file from any context object
Format & write a payload to a toml file on disk. This is useful for generating
toml files from your pipeline, such as when you want to create configuration 
files dynamically on the fly.

`filewritetoml` works like this:

```yaml
- name: pypyr.steps.filewritetoml
  comment: write context payload out to toml
  in:
    fileWriteToml:
      path: /path/to/output.toml # destination file
      payload: # (optional) payload to write to path
        key1: value1 # output a string
        key2: value2
        key3: 124 # output int
        key4: false # output bool
        table:
          mylist:
            - 1
            - 2
          myvalue: arb value
```

This will generate the following toml to `/path/to/output.toml`:

```toml
key1 = "value1"
key2 = "value2"
key3 = 124
key4 = false

[table]
mylist = [
    1,
    2,
]
myvalue = "arb value"
```

Note that the correct data types will serialize to the generated output toml. 
More complicated hierarchical nested structures will also just work.

If you do not specify `payload`, pypyr will write the entire context to
the output file in toml format. Be careful if you have sensitive values
like passwords or private keys!

## format toml with token replacements
All inputs support [substitutions]({{< ref "docs/substitutions">}}). This means 
that you can construct the path or payload from other context variables:

```yaml
- name: pypyr.steps.set
  comment: set some arb values in context
  in:
    set:
      arbkey: arbvalue
      writehere: /path/to/output.toml
      writeme:
        this: toml content
        will: be written to
        thepath: with substitutions like this {arbkey}.

- name: pypyr.steps.filewritetoml
  comment: write toml file from substitution expressions
  in:
    fileWriteToml:
      path: '{writehere}'
      payload: '{writeme}'
```

In the above example, the output toml file writes to `/path/to/output.toml`

Substitution processing also runs on the output payload content. This means that
substitution expressions nested inside the result of a previous substitution
expression will work. For example, the `{arbkey}` expression in `thepath` will
substitute like this:

```toml
this = "toml content"
will = "be written to"
thepath = "with substitutions like this arbvalue."
```

As always in pypyr, you can construct a value by embedding & combining
substitution expressions in other strings:

```yaml
- name: pypyr.steps.set
  comment: set some arb values in context
  in:
    set:
      arbkey: 123
      arbstr: in the middle
      filename: my-file

- name: pypyr.steps.filewritetoml
  comment: write toml file from substitution expressions
  in:
    fileWriteToml:
      path: out/{filename}.toml
      payload:
        my_table:
          my_number: '{arbkey}'
          my_string: begin {arbstr} end
```

This will to output file `./out/my-file.toml`:
```toml
[my_table]
my_number = 123
my_string = "begin in the middle end"
```