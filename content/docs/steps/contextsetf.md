---
title: pypyr.steps.contextsetf
linktitle: contextsetf
date: 2020-06-30T20:28:46+01:00
description: Set & format context keys.
card_extra_summary:
  heading: input context property
  details: "`contextSetf` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: contextsetf
seo_article_headline: Set task-runner pipeline context values.
seo_description: Set pipeline context to static or dynamic values, read interactive user input from stdin & use formatting expressions.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: contextsetf -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [context]
---
# pypyr.steps.contextsetf
## set context values with formatting & dynamic expressions
Set context keys with arbitrary values of different types. You can 
also use formatting expressions for assigning dynamic run-time values, using 
[substitutions]({{< ref "docs/substitutions">}}).

This is roughly the equivalent of instantiating and assigning a variable in 
traditional programming.

Requires the `contextSetf` key in context. `contextSetf` is a dictionary of
items to set in context. For example, here is how you can set arbitrary values
with different types to arbitrary keys:

```yaml
format_expression: arb value # pre-existing context key+value
key: middle # pre-existing context key+value
contextSetf:
  newkey_formatting_expression: '{format_expression}'
  newkey_string_concat: 'start {key} end'
  newkey_string: arbitrary string here
  newkey_int: 8
  newkey_decimal: 1.23
  newkey_bool: False
  newkey_list: 
    - list item 1
    - list item 2
  newkey_dict:
    key1: value 1
    key2: value 2
    nested: 
      nested_1: nested value 1
      nested_2: nested value 2
```

Running `contextsetf` with the above input will result in the following context:

```yaml
format_expression: arb value
key: middle
newkey_formatting_expression: arb value
newkey_string_concat: start middle end
newkey_string: arbitrary string here
newkey_int: 8
newkey_decimal: 1.23
newkey_bool: False
newkey_list: 
  - list item 1
  - list item 2
newkey_dict:
  key1: value 1
  key2: value 2
  nested: 
    nested_1: nested value 1
    nested_2: nested value 2
```

`contextsetf` applies any {formatting expressions} immediately during the 
`contextsetf` step execution. This means that the value you assign will
reflect the context at that point in time, and not any later changes to the 
source values.

[contextcopy]({{< ref "contextcopy" >}}) and `contextsetf` overwrite existing 
keys. If you want to merge new values into an existing destination hierarchy,
use [contextmerge]({{< ref "contextmerge" >}}) instead.

## examples
For example, say your context looks like this:

```yaml
key1: value1
key2: value2
answer: 42
```

and your pipeline yaml looks like this:

```yaml
steps:
  - name: pypyr.steps.contextsetf
    in:
      contextSetf:
        key2: any old value without a substitution - it will be a string now.
        key4: 'What do you get when you multiply six by nine? {answer}'
```

This will result in context like this:

```yaml
key1: value1
key2: any old value without a substitution - it will be a string now.
answer: 42
key4: 'What do you get when you multiply six by nine? 42'
```

See a worked [example for contextsetf](https://github.com/pypyr/pypyr-example/tree/master/pipelines/contextset.yaml).

## conditional assignment & ternary expressions
You can use `contextsetf` in conjunction with [py strings]({{< ref "docs/substitutions/py-strings">}})
for conditional assignment of context items or ternary expressions.

This is useful for creating boolean values in context based upon existence 
checks, null checks or if an item in a list or dict exists.

```yaml
arb1: null
arb2: ''
arb4: [1,1,2,3,5,8]
contextSetf:
  arb3: eggy # you can now use arb3 below in this same contextSetf step
  fromArb: '{arb3}'
  isNull: !py arb1 is None # make a bool based on None
  isEmpty: !py bool(arb2) # use truthy, empty strings are false
  ternaryResult: !py "'eggs' if arb3 == 'eggy' else 'ham'"
  isIn: !py 10 in arb4 # bool true if thing in list
```

## read user input from the console
### read from stdin into pipeline context
You can read user input from the console with `contextsetf` using a py string.

Pipeline execution will pause and wait for interactive user input on each 
`read()` before proceeding with the rest of the pipeline.

```yaml
contextSetf:
  input_with_prompt: !py input("enter something here:\n")
  input_sans_prompt: !py input()
```

### don't wait for stdin on server or unattended execution
Do remember that if you plan to run your pipeline on a server or in the cloud
somewhere, you probably don't want your pipeline to stop and wait for stdin
input that will never come. You can selectively control when to wait for 
interactive user input like this by using the conditional decorators 
[run]({{< ref "/docs/decorators/run" >}}) or 
[skip]({{< ref "/docs/decorators/skip" >}}):

```yaml
# ./stdin-example.yaml
context_parser: pypyr.parser.keyvaluepairs
steps:
  - name: pypyr.steps.default
    in:
      defaults:
          arb_key: sensible default
          is_server: False
  - name: pypyr.steps.contextsetf
    skip: '{is_server}'
    in:
      contextSetf:
        arb_key: !py input("enter client side value here:\n")
  - name: pypyr.steps.echo
    in:
      echoMe: 'arbkey is {arb_key} and is_server is {is_server}'
```

With this pipeline, you can switch from the cli whether you want to wait for 
user input or not like this:

{{< app-window title="term" lang="text" >}}
$ pypyr stdin-example
enter client side value here:
user input here
arbkey is user input here and is_server is False

$ pypyr stdin-example is_server=True
arbkey is sensible default and is_server is True

$ pypyr stdin-example is_server=True arb_key="from the cli arg"
arbkey is from the cli arg and is_server is True
{{< /app-window >}}
