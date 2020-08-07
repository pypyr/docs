---
title: sic string - literal strings
linktitle: sic string
date: 2020-06-13T21:38:57+01:00
description: Literal string values that should not have formatting expressions applied, for simplified escape sequences.
draft: false
categories: [expressions]
keywords: "literal strings, pipeline execution"
menu:
  docs:
    parent: substitutions
    id: sic-strings
    name: sic string
card_extra_summary:
    details: '`!sic literal string here`'
    heading: example
seo_article_headline: Simplify escape sequences with literal strings.
seo_description: Literal string values ignore formatting expressions in your pipeline steps & task automation sequences.
# topics: []
---
# sic strings
## literal string values
If a string is NOT to have {substitutions} run on it, it's *sic erat
scriptum*, or *sic* for short. This is handy especially when you are
dealing with json as a string, rather than an actual json object, so you
don't have to use escape sequences to double curly all the structural braces
and it simplifies the need for escape sequences.

A *sic* string looks like this:

```text
!sic <<your string literal here>>
```

For example:

```text
!sic piping {key} the valleys wild
```

Will return `piping {key} the valleys wild` without attempting to
substitute `{key}` from context. You can happily use `"`, `'` or `{}` inside a
`!sic my string` string without escaping these any further. This makes
sic strings ideal for strings containing json or code.

You can surround the Sic string with single or double quotes like this
`!sic 'my string here'` or `!sic "my string here"`. This is handy if
your string starts with a yaml structural character like square `[` or
curly `{` braces. Check example below for escape sequences if you do so.

```yaml
- name: pypyr.steps.echo
  description: >
              use a sic string not to format any {values}. Do watch the
              use of the yaml literal with block chomping indicator |- to
              prevent the last character in the string from being a LF.
  in:
    echoMe: !sic |-

            {
              "key1": "key1 value with a {curly}"
            }
- name: pypyr.steps.echo
  description: use a sic string not to format any {values} on one line. No need to escape further quotes.
  in:
    echoMe: !sic string with a {curly} with ", ' and & and double quote at end:"
- name: pypyr.steps.echo
  description: use a sic string with single quotes.
  in:
    echoMe: !sic '{string} with {curlies} inside single quotes, : colon, quote ", backslash \.'
- name: pypyr.steps.echo
  description: use a sic string with double quotes. Double up the backslashes!
  in:
    echoMe: !sic "[string] with {curlies} inside double quotes, : colon, quote ", backslash \\."
```

You can pick single or double quotes, so just go with whichever is less
annoying for your particular string.

See a worked [example for substitutions with sic strings](https://github.com/pypyr/pypyr-example/tree/master/pipelines/substitutions.yaml).