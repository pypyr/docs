---
title: pypyr.steps.jsonparse
linktitle: jsonparse
date: 2020-10-26T13:12:12Z
description: Parse json string into pypyr context.
draft: false
card_extra_summary:
  heading: input context property
  details: "`jsonParse` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: jsonparse
seo_article_headline: Parse a json string into a pypyr context object.
seo_description: Parse a json string such as the output of a command, and deserialize it to the pypyr context.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: jsonparse -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [json]
---
# pypyr.steps.jsonparse
## parse json string into context object 
Parse an input json string into the pypyr context as an object. This allows you 
to work with the deserialized objects from the json string like you would 
normally work with any data structures in the pypyr context - so you can use 
all the usual [context handling functionality]({{< ref "/topics/context/">}}) 
to set, edit & manipulate context keys and values.

This step requires the `jsonParse` key in the pypyr context:

```yaml
- name: pypyr.steps.jsonparse
  comment: parse json from string
  in:
    jsonParse:
      json: '[1, 2, 3]' # required. string of json.
      key: destinationKey # optional. write result object to this context key.
```

If you do not specify `key`, json writes directly to context root.

All inputs support [substitutions]({{< ref "docs/substitutions">}}).

## parse the json output of preceding steps
Keep in mind that yaml is a superset of json. Specifically yaml 1.2, which is 
what a pypyr pipeline uses. This means that if you did want to put a literal 
bit of json anywhere in the pipeline, you can do so without bothering with 
`parsejson`.

```yaml
- name: pypyr.steps.set
  comment: put hard-coded json document into pipeline context.
           notice for both the map & the list we're directly
           using valid json syntax.
           this works because yaml is a superset of json.
  in:
    set:
      myList: [1, 2, 3]
      myMap: {"a": "b", "c": "d"}
```

This `parsejson` step only really becomes necessary when you want to parse a 
value that is a string already - likely the output of a command you executed in 
a preceding step.

You are very like to use `parsejson` using the `ff` 
[flat format directive]({{< ref "docs/substitutions/format-string#explicitly-set-flat" >}}) 
to prevent pypyr from interpreting the json's {curly braces} as pypyr formatting
expressions:

```yaml
- name: pypyr.steps.jsonparse
  in:
    jsonParse:
      json: '{myJsonString:ff}'
      key: myParsedJson
```

## example
In this pipeline we `curl` a json string from a REST api web-server and load the 
response json string into context using `jsonparse`.

```yaml
- name: pypyr.steps.cmd
  description: --> curl some json from the awesome httpbin.org
  comment: because save is True, output saves to cmdOut.
  in:
    cmd:
      run: |-
        curl -H 'Content-Type: application/json' -X GET https://httpbin.org/json
      save: True

- name: pypyr.steps.echo
  in:
    echoMe: "the response string from curl:\n{cmdOut[stdout]} "

- name: pypyr.steps.jsonparse
  description: --> parsing curl response string into context
  in:
    jsonParse:
      json: '{cmdOut[stdout]:ff}'
      key: myParsedJson

- name: pypyr.steps.echo
  in:
    echoMe: |
      the json response from the curl cmd is in context now.
      slideshow.title: {myParsedJson[slideshow][title]}
      arbitrary slide item: {myParsedJson[slideshow][slides][1][items][0]}
```

Here is a worked [example of jsonparse](https://github.com/pypyr/pypyr-example/blob/main/pipelines/jsonparse.yaml).

## json structure
pypyr will merge the json parsed from the input string into the pypyr context. 
This will overwrite existing values if the same keys already exist in context.

I.e if json string has `{'eggs' : 'boiled'}`, but context
`{'eggs': 'fried'}` already exists, after `jsonparse` the key `context['eggs']` 
will be 'boiled'.

{{% note warn %}}
If you do not specify `key`, the json must NOT be an Array `[]` or a single 
literal at the root level, but rather an Object `{}`.
{{% /note %}}