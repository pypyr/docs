---
title: format string interpolation
linktitle: format string
date: 2020-06-13T21:38:57+01:00
description: String interpolation with substitution tokens are for easy value replacement inside a pipeline workflow. Works with complex types, not just strings.
draft: false
categories: [expressions]
keywords: "token replacement, string substitutions"
topics: [substitutions]
menu:
  docs:
    parent: substitutions
    name: format string
    id: substitutions-string
card_extra_summary:
    details: "`'{token}'`"
    heading: example
seo_article_headline: Format string interpolation expressions in the pypyr task-runner.
seo_description: String interpolation with substitution tokens are for easy type-safe value replacement inside a pipeline workflow.
---
# string interpolation
## string formatting expressions with replacement tokens
You can use substitution tokens, aka string interpolation, to format strings 
where specified for context items. This substitutes anything between `{curly
braces}` with the context value for that key. This also works with nested values
where you have dictionaries/lists inside dictionaries/lists. 

For example, if your context looked like this:

```yaml
key1: down
key2: valleys
key3: value3
key4: "Piping {key1} the {key2} wild"
```

The value for `key4` will be "Piping down the valleys wild".

## escape sequences
Escape literal curly braces with doubles: 

| special character | escape sequence |
| ----------------- | --------------- |
| `{`               | `{{`            |
| `}`               | `}}`            |

In json & yaml, curlies need to be inside quotes to make sure they parse
as strings. Especially watch in .yaml, where `{` as the first character of
a key or value will throw a formatting error if it's not in quotes like
this: `"{key}"`

If your particular values make using the escape sequences a nuisance, you can
avoid doubling the curly braces by using sic strings aka [literal strings]({{< ref "sic-strings">}}) 
instead.

## nested values
You can reference keys nested deeper in the context hierarchy, in
cases where you have a dictionary that contains lists/dictionaries that
might contain other lists/dictionaries and so forth.

```yaml
root:
  - list index 0
  - key1: this is a value from a dict containing a list, which contains a dict at index 1
    key2: key 2 value
  - list index 2
```

Given the context above, you can use formatting expressions to access
nested values like this:

```text
'{root[0]}' == list index 0
'{root[1][key1]}' == this is a value from a dict containing a list, which contains a dict at index 1
'{root[1][key2]}' == key 2 value
'{root[2]}' == list index 2
```

## interpolate complex types
You can assign complex types or hierarchial, nested structures with string 
formatting expression syntax. This allows you to replace an entire key or 
value in any sub-section, or to build a new configuration section using parts
of other configuration sections. This is type-safe. 

For example, in the following example there is a complex nested dictionary 
under `key1`. 

```yaml
key1:
  k1.1: value 1.1
  k1.2: 
    - 1.2.1
    - 1.2.2
    - 1.2.3.key1: dict inside list inside dict 1
      1.2.3.key2: dict inside list inside dict 2
key2: key 2 value
```

You can use the entirety of `key1` in any other complex nested configuration 
context values you assemble in pypyr, so you effectively compose new 
configuration context structures from existing building blocks.

```yaml
- name: pypyr.steps.contextsetf
  in:
    contextSetf: 
      newkey:
        nestedkey1: nested value 1
        nestedkey2:
          - list item 0
          - '{key1}'
          - list item 2
```

In this example, nested a few levels deep under `newkey`, pypyr will
replace `{key1}` with the entirety of the complex, nested dictionary under 
`key1`. This will result in the the following context:

```yaml
key1:
  k1.1: value 1.1
  k1.2: 
    - 1.2.1
    - 1.2.2
    - 1.2.3.key1: dict inside list inside dict 1
      1.2.3.key2: dict inside list inside dict 2
key2: key 2 value
newkey:
  nestedkey1: nested value 1
  nestedkey2:
    - list item 0
    - k1.1: value 1.1
      k1.2: 
        - 1.2.1
        - 1.2.2
        - 1.2.3.key1: dict inside list inside dict 1
          1.2.3.key2: dict inside list inside dict 2
    - list item 2
```