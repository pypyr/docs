---
title: pypyr.steps.fetchtoml
linktitle: fetchtoml
date: 2021-12-09T19:02:21+01:00
description: Load toml file into pypyr context.
card_extra_summary:
  heading: input context property
  details: "`fetchToml` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: fetchtoml
seo_article_headline: Load toml into task-runner pipeline context at run-time.
seo_description: Load & parse a toml file into the task-runner so that the pipeline can read, manipulate & change the data.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: fetchtoml -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [filesystem, toml]
---
# pypyr.steps.fetchtoml
## load & parse toml
Parse a toml fie and load it into the pypyr context.

This step requires the following key in the pypyr context:

```yaml
- name: pypyr.steps.fetchtoml
  comment: fetch toml from path and store result in key.
  in:
    fetchToml:
      path: ./path.toml # required. path to file on disk. can be relative.
      key: destinationKey # optional. write toml to this context key.
```

If you do not specify `key`, the toml structure writes directly to context root.

If you do not want to specify a key, you can instead use the streamlined input
format to save yourself some typing:

```yaml
- name: pypyr.steps.fetchtoml
  comment: simplified input with only path
  in:
    fetchToml: ./path.toml # required. path to file on disk. can be relative.
```

All inputs support [substitutions]({{< ref "docs/substitutions">}}).


## example
Given a toml file like this, saved as `./sample-files/sample.toml`:

```toml
#this is a toml example
echoMe = "this is a value from a toml file"
key1 = "value1"
key2 = "value2"
keyBool = true
keyInt = 123

[mytable]
mytablekey = "{key1} and {key2} and {key3}"
```

And given a pipeline like this, arbitrarily saved as `./fetch-toml.yaml`:

```yaml
# ./fetch-toml.yaml
steps:
  - name: pypyr.steps.fetchtoml
    comment: fetch toml from path
    in:
      fetchToml:
        path: ./testfiles/sample.toml

  # echoMe is set in the toml input file
  - pypyr.steps.echo

  - name: pypyr.steps.set
    comment: use the int from toml as a strongly typed int
    in:
      set:
        intResult: !py keyInt * 2
        key3: value3
  
  - name: pypyr.steps.echo
    comment: print out result of int calculation
    in:
      echoMe: "strongly typed happens automatically: {intResult}"

  - name: pypyr.steps.echo
    comment: access a key in a table
             formatting expressions in toml string will work
    in:
      echoMe: "{mytable[mytablekey]}"
    
  - name: pypyr.steps.echo
    comment: only run if input bool is true
    run: '{keyBool}'
    in:
      echoMe: you'll only see me if the bool was true
```

You can run this pipeline as follows:
{{< app-window title="term" lang="text" >}}
$ pypyr fetch-toml
this is a value from a toml file
strongly typed happens automatically: 246
value1 and value2 and value3
you'll only see me if the bool was true
{{< /app-window >}}

Notice that `mytable.mytablekey` uses pypyr {substitution} expressions where
`key1` and `key2` are set in the toml file itself, and `key3` is set in the
pypyr pipeline with [pypyr.steps.set]({{< ref "/docs/steps/set" >}}).

If you change the `keyBool` bool to `false` in the toml input file, the last
step won't run anymore because the `run` condition is `False`.

{{< app-window title="term" lang="text" >}}
$ pypyr fetch-toml
this is a value from a toml file
value3 and value1 and value2
strongly typed happens automatically: 246
{{< /app-window >}}

## substitutions
You can use substitutions on all the step's inputs:

```yaml
- name: pypyr.steps.set
  comment: use set to set arbitrary values for path & key referenced
           by the fetch toml step. 
  in:
    set:
      filename: 'sample'
      keyname: 'savemehere'

- name: pypyr.steps.fetchtoml
  description: fetch toml into key with path substitutions
  in:
    fetchToml:
      path: ./testfiles/{filename}.toml
      key: '{keyname}'
```

## toml structure merging
pypyr will merge the toml parsed from the file into the pypyr context. This
will overwrite existing values if the same keys already exist in context.

I.e if file toml has 
```toml
eggs = "boiled"
```

And in context the following already exists:
```json
{
  "eggs": "raw"
}
```

After `fetchtoml` the value of key 'eggs' will be 'boiled'.

## encoding
A TOML file must always be in `utf-8`, per the [TOML
Spec](https://toml.io/en/latest#spec).