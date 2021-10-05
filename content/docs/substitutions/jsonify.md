---
title: jsonify
# linktitle: jsonify
date: 2020-06-13T21:38:57+01:00
description: Serialize context object to json string
# categories: [expressions]
keywords: "json, serialize, convert"
menu:
  docs:
    parent: substitutions
    name: jsonify
card_extra_summary:
    details: '`!jsonify [0, 1, 2]`'
    heading: example
seo_article_headline: Serialize a context object to json in pypyr task-runner. 
seo_description: Convert object structures like lists & dictionaries from yaml to a string of json.
topics: [json]
---
# jsonify
## convert object to json string
Use `jsonify` to serialize a pypyr context object to a json string.

```yaml
- name: pypyr.steps.set
  in:
    set:
        myJsonDict: !jsonify
            k1: v1
            k2: 123
            k3: False
            k4:
            - 1
            - 2
            - a: b
                c: d
        myJsonList: !jsonify
            - zero
            - one
            - two
        myJsonNull: !jsonify null
        myJsonNumber: !jsonify 99
        myJsonQuotedString: !jsonify "0123"
        myJsonBareString: !jsonify arbitrary string
- name: pypyr.steps.echo
  in:
    echoMe: |
        myJsonDict: {myJsonDict}
        myJsonList: {myJsonList}
        myJsonNull: {myJsonNull}
        myJsonNumber: {myJsonNumber}
        myJsonQuotedString: {myJsonQuotedString}
        myJsonBareString: {myJsonBareString}
```

This gives output:

```text
myJsonDict: {"k1": "v1", "k2": 123, "k3": false, "k4": [1, 2, {"a": "b", "c": "d"}]}
myJsonList: ["zero", "one", "two"]
myJsonNull: null
myJsonNumber: 99
myJsonQuotedString: "0123"
myJsonBareString: "arbitrary string"
```

You could think of this as converting yaml to json, however, in actuality under 
the hood pypyr will serialize any given object in context to json with 
`jsonify` so it's not necessarily just yaml as the input. It just so happens 
that yaml from pipeline is the likeliest input, but `jsonify` will work 
equally well if you manipulate context directly via the api.

## use with substitutions
A `jsonify` structure can contain pypyr formatting expressions for 
[substitutions]({{< ref "docs/substitutions/format-string">}}), including 
[py strings for inline python]({{< relref "py-strings" >}}) and 
[sic strings to avoid processing curly braces]({{< relref "sic-strings" >}}).

```yaml
- name: pypyr.steps.set
  comment: combine jsonify with other formatting expressions
  in:
    a_string: hello there
    replace_me: 123
    
    set:
      arb_payload: !jsonify
        k1: value {a_string}
        k2: !py hex(255)
        k3: !sic a string with {literal} curly braces
        k4: '{replace_me}'
        k5: False

- name: pypyr.steps.cmd
  comment: post the json payload string to a web-server via curl
  in:
    cmd: |
      curl -H 'Content-Type: application/json' -d '{arb_payload}' -X POST 'https://httpbin.org/post'
```

The json payload posted to the web-server looks like this:

```json
{
  "k1": "value hello there",
  "k2": "0xff",
  "k3": "a string with {literal} curly braces",
  "k4": 123,
  "k5": false
}
```

## use with flat format
If you want to assign a jsonified string to another context key using a single 
replacement token expression, use the `ff` 
[flat format directive]({{< ref "docs/substitutions/format-string#explicitly-set-flat" >}}) 
to prevent pypyr from interpreting the json string's {curly braces} as pypyr formatting
expressions:

```yaml
- name: pypyr.steps.set
  comment: combine jsonify with other formatting expressions
  in:
    a_string: hello there
    replace_me: 123
    
    set:
      arb_payload: !jsonify
        k1: value {a_string}
        k2: !py hex(255)
        k3: !sic a string with {literal} curly braces
        k4: '{replace_me}'
        k5: False
    
- name: pypyr.steps.echo
  comment: arb_payload contains a string with json curly braces
           since pypyr recurses single token expressions by default,
           use Flat Format (ff).
  in:
    echoMe: '{arb_payload:ff}'
```