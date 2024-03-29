---
title: pypyr.parser.keyvaluepairs
linktitle: keyvaluepairs
date: 2020-07-09T12:26:36+01:00
description: Pass key-value pairs from cli input args to the pipeline.
card_extra_summary:
  heading: example input
  details: '`pypyr my-pipeline param1=value1 param2="value 2" "param 3"=value3`'
categories: [context parsers]
# keywords: ""
menu:
  docs:
    parent: context-parsers
    name: keyvaluepairs
seo_article_headline: Parsing key-value pairs from cli input arguments.
seo_description: Set key-value pairs as the task-runner cli input argument to use in the pipeline at run-time.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: keyvaluepairs -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [args]
---
# pypyr.parser.keyvaluepairs
## pass key-value pairs from cli to pipeline
Takes `key=value` pairs from cli arg input and initialize context with a 
dictionary where each pair becomes a dictionary element.

{{< app-window title="term" lang="console" >}}
$ pypyr my-pipeline key1=value1 key2=2 key3="value 3"
_
{{< /app-window >}}

This will parse to context like this:
```python
{
  'key1': 'value1',
  'key2': '2',
  'key3': 'value 3'
  }
```

Escape literal spaces with single or double quotes.

Any arg without an `=` will parse to `key: ''`.
{{< app-window title="term" lang="console" >}}
$ pypyr my-pipeline arg0 key1=value1 key2=2 key3="value 3"
_
{{< /app-window >}}

This will parse to context like this:
```python
{
  'arg0': '',
  'key1': 'value1',
  'key2': '2',
  'key3': 'value 3'
  }
```

## special characters
Notice that you can have both keys and values with spaces by using either
single or double quotes. If you need to escape quotes, refer to your shell's
documentation for escape characters.

{{< app-window title="term" lang="console" >}}
$ pypyr my-pipeline param1=value1 param2='value 2' "param 3"=123
{'param1': 'value1', 'param2': 'value 2', 'param 3': '123'}
{{< /app-window >}}

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
{
  'key1': 'value !@£$%^&',
  'key_2': 'nothing_special'
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
{
  'k1': 'one+two=three',
  'k2': 'arb'
}
```

## example
Given a pipeline like this, arbitrarily saved as `./keyvaluepairs-parser.yaml`:

```yaml
# ./keyvaluepairs-parser.yaml
context_parser: pypyr.parser.keyvaluepairs
steps:
  - pypyr.steps.debug # prints at log level <=20
```

You can then pass key-value pairs from the cli to the pipeline to initialize 
context. Notice that you can have both keys and values with spaces by using 
either single or double quotes.

{{< app-window title="term" lang="console" >}}
$ pypyr keyvaluepairs-parser param1=value1 param2='value 2' "param 3"=123 --log 20
{'param1': 'value1', 'param2': 'value 2', 'param 3': '123'}
{{< /app-window >}}
