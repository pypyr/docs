---
title: pypyr.parser.argskwargs
linktitle: argskwargs
date: 2022-10-20T10:26:36+01:00
description: Pass args and/or key=value pairs from cli input to the pipeline.
card_extra_summary:
  heading: example input
  details: '`pypyr my-pipeline arg1 arg2 k1=value1 k2="value 2"`'
categories: [context parsers]
# keywords: ""
menu:
  docs:
    parent: context-parsers
    name: argskwargs
seo_article_headline: Parsing list args and key=value pairs from cli input arguments.
seo_description: Use a list of args and/or key=value pairs from cli input argument without writing any code.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: keyvaluepairs -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [args]
---
# pypyr.parser.argskwargs
## parse list of args & key=value pairs from cli
Puts input cli arguments into a list `argList`. If an argument has a `=`, will
save the key=value pair as a dictionary/mapping element.

{{< app-window title="term" lang="console" >}}
$ pypyr my-pipeline arg1 arg2 k1=value1 k2=value2

$ pypyr my-pipeline arg1 "arg 2" k1="value1" k2="value 2"
{{< /app-window >}}

The second example will result in your pipeline context looking like this:
```python
{
  'argList': ['arg1', 'arg 2'],
  'k1': 'value1',
  'k2': 'value 2'
}
```

This gives you cli input syntax very similar to makefile. Alternatively, you can
conceptualize this as a combination of [pypyr.parser.list]({{< ref "list" >}})
and [pypyr.parser.keyvaluepairs]({{< ref "keyvaluepairs" >}}).

Escape literal spaces with single or double quotes.

If you don't pass any arguments or when you pass only key=value pairs, pypyr
will initialize `argList` to an empty list `[]`.

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
$ pypyr my-pipeline "list ^ value" key1='value !@£$%^&' key_2=nothing_special
_
{{< /app-window >}}

Will parse like this:
```python
{
  'argList': ['list ^ value'],
  'key1': 'value !@£$%^&',
  'key_2': 'nothing_special'
}
```

Notice that list values for `argList` can contain whatever special characters
you want, without causing trouble for py-strings and the py step.

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
Given a pipeline like this, arbitrarily saved as `./my-pipeline.yaml`:

```yaml
# ./my-pipeline.yaml
context_parser: pypyr.parser.argskwargs
steps:
  - pypyr.steps.debug # prints at log level <=20
```

You can then pass any combination of arguments and key-value pairs from the cli
to the pipeline to initialize context.

{{< app-window title="term" lang="console" >}}
$ pypyr my-pipeline arg1 arg2 k1=value1 k2="value 2" --log 20
{'argList': ['arg1', 'arg2'], 'k1': 'value1', 'k2': 'value 2'}
{{< /app-window >}}