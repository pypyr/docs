---
title: pypyr.parser.dict
linktitle: dict
date: 2020-07-08T18:44:23+01:00
description: Parse cli key=value pairs into `argDict`
card_extra_summary:
  heading: example input
  details: '`pypyr my-pipeline param1=value1 param2="value 2" param3=value3`'
categories: [context parsers]
# keywords: ""
menu:
  docs:
    parent: context-parsers
    name: dict
seo_article_headline: Create a dict from cli arg input to task-runner.
seo_description: Parse key=value pairs in a string from the cli input args & create a dict from those values.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: dict -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [args]
---
# pypyr.parser.dict
## create a dict from key=value pair string
Takes a key=value pair string and returns a dictionary (aka a map) where each
pair becomes a dictionary element inside a dict with name `argDict`.

Escape literal spaces with single or double quotes.

Any arg without an `=` will parse to `key: ''`.
{{< app-window title="term" lang="console" >}}
$ pypyr my-pipeline arg0 key1=value1 key2=2 key3="value 3"
_
{{< /app-window >}}

This will parse to context like this:
```python
{
  'argDict': {
    'arg0': '',
    'key1': 'value1',
    'key2': '2',
    'key3': 'value 3'}
}
```

## example
{{< app-window title="term" lang="console" >}}
$ pypyr my-pipeline param1=value1 param2="value 2" param3=value3
{{< /app-window >}}

This will create a context dictionary like this:

```python
{'argDict': {'param1': 'value1',
             'param2': 'value 2',
             'param3': 'value3'}}
 ```                                

Or to put in in yaml terms:

```yaml
argDict:
  param1: value1
  param2: value 2
  param3: value3
 ```

 You can access the dict you create this way with 
 [formatting expressions]({{< ref "/docs/substitutions">}}):

 ```yaml
 context_parser: pypyr.parser.dict
 steps:
   - name: pypyr.steps.echo
     in:
       echoMe: param2 value is {argDict[param2]}
 ```

## special characters
Notice that you can have both keys and values with spaces by using either
single or double quotes. If you need to escape quotes, refer to your shell's
documentation for escape characters.

{{< app-window title="term" lang="console" >}}
$ pypyr my-pipeline param1=value1 param2='value 2' "param 3"=123
_
{{< /app-window >}}

It will look like this in context:
```python
{ 'argDict': {
    'param1': 'value1',
    'param2': 'value 2',
    'param 3': '123'}
}
```

Although you can have spaces and special characters such as punctuation in your
keys (such as "param 3" above), for an easy life you might want to avoid doing
so.

The reason is that when you want to reference your context keys in
[{formatting expressions}]({{< ref "/docs/substitutions/format-string" >}}) or
variables in [py string expressions]({{< ref "/docs/substitutions/py-strings" >}})
or in a [pypyr.steps.py]({{< ref "/docs/steps/py" >}}) step you're going to run
into trouble - so it's best to stick to keys that are valid Python identifiers.

Simply put, don't have punctuation other than `_` in your keys unless you're
really sure you're willing to live with the side-effects.

While you might want to keep your keys as free of special characters as you can,
the values can be whatever you want:

{{< app-window title="term" lang="console" >}}
$ pypyr my-pipeline key1='value !@£$%^&' key_2=nothing_special
_
{{< /app-window >}}

Will initialize context like this:
```python
{ 'argDict': {
    'key1': 'value !@£$%^&',
    'key_2': 'nothing_special'}
}
```

When parsing key=value pairs, pypyr only considers the first `=` as marking the
key, subsequent `=` in the same arg will be considered part of the value:

{{< app-window title="term" lang="console" >}}
$ pypyr my-pipeline k1=one+two=three k2=arb
_
{{< /app-window >}}

Will result in:
```python
{ 'argDict': {
    'k1': 'one+two=three',
    'k2': 'arb'}
}
```