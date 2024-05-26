---
title: py string - dynamic python expressions
linktitle: py string
date: 2020-06-13T21:38:57+01:00
description: Evaluate dynamic python expressions in your pipeline steps and task automation sequences.
# categories: [expressions]
# keywords: "dynamic expressions, pipeline execution"
menu:
  docs:
    parent: substitutions
    id: py-strings
    name: py string
card_extra_summary:
    details: '`!py len(key) > 1`'
    heading: example
seo_article_headline: Evaluate dynamic python expressions in pypyr task-runner. 
seo_description: Evaluate & assign dynamic python expressions in any pypyr task-runner pipeline workflow.
topics: [inline code]
---
# py strings
## dynamic python expressions
py strings allow you to execute python expressions dynamically. This lets you 
use a python expression wherever you can use a string formatting expression.

A py string looks like this:

```text
!py <<your python expression here>>
```

Context keys exist as variables of the same name.

For example, if `context['key']` is 'abc', the following will return
True: `!py len(key) == 3"`

Notice that you can use the context keys directly as variables. Unlike
string formatting expressions, you don't surround the key name with
`{curlies}`.

```yaml
# ./pystrings.yaml
steps:
  - name: pypyr.steps.set
    comment: py strings use context keys directly, 
             without curlies around them.
             standard/normal str interpolation 
             uses {key} for replacement expression.
    in:
      set:
        arbKey: arb value
        arbInt: 123
        normal_str_format: use curly {arbKey} to format.
        py_str_format: !py "arbKey.upper() + ' : no curlies here'"
        py_int_format: !py max(arbInt, 456, 789)
  - name: pypyr.steps.debug
    in:
      debug:
        keys:
          - normal_str_format
          - py_str_format
          - py_int_format
```

The output from `debug` here is:

```python
{
  'normal_str_format': 'use curly arb value to format.',
  'py_int_format': 789,
  'py_str_format': 'ARB VALUE : no curlies here'
}
```

A py string can return any type, not just strings or bools. So if your expression
evaluates to a list, dict, tuple, or decimal, or any given type, the py string
expression will return the actual type of the expression result.

## import modules
The py string expression has the usual python builtins available to it, in
addition to everything in the Context dictionary. In other words, you can use
functions like `abs`, `len` & `max` without doing anything special. See here for
a full list of python builtins 
<https://docs.python.org/3/library/functions.html>.

You can use any Python standard library modules, external libraries in the
current environment and also your own code by using the 
[pyimport]({{< ref "/docs/steps/pyimport" >}}) step to import references.

## inline functions
If you do not want to put your own functions in a separate .py file, you can 
can 
[create your own functions or methods in a py step]({{< ref "/docs/steps/py#create-reusable-functions--classes" >}})
and reference these in any !py strings following that step in the pipeline.

## check if a variable exists
!py strings are useful selectively to run a step based upon whether certain keys
exist in context.

All the Context keys exist in the `locals()` and `globals()` namespace as
variables of the same name.

This means you can use the built-in `vars()` or `dir()` function to find
variables that exist in the local namespace. This is useful when you only want
to run a step if a certain key was previously set in context.

In this example, all the `echo` steps will only run if their `run` condition
evaluates to `True`. Checking against `vars()`, `dirs()`, `locals()` or
`globals()` is functionally equivalent here.

```yaml
- name: pypyr.steps.default
  description: set some arbitrary values in context
  in:
    defaults:
      breakfastList: ['eggs', 'spam', 'bacon']
      thisIsAnInt: 5
      thisIsABool: True
      thisIsAnotherBool: False

- name: pypyr.steps.echo
  run: !py "'breakfastList' in vars()"
  in:
    echoMe: 1. you will see me only if breakfastList exists

- name: pypyr.steps.echo
  run: !py "'thisIsAnInt' in dir()"
  in:
    echoMe: 2

- name: pypyr.steps.echo
  run: !py "'thisIsABool' in locals()"
  in:
    echoMe: 3

- name: pypyr.steps.echo
  run: !py "'thisIsAnotherBool' in globals()"
  in:
    echoMe: 4
```

This will output:

```text
set some arbitrary values in context
1. you will see me only if breakfastList exists
2
3
4
```

The !py string runs in a single global scope, so locals() and globals() both 
refer to the same namespace dictionary. 

## special characters
In pipeline yaml, if the first character of the py string is a yaml
structural character, you should put the Py string in quotes or as part
of a literal block.

Other than that, there's no particular need to wrap your py-strings in quotes 
and start doing the tap-dance of having to escape quotes-within-quotes. So 
don't.

```yaml
- name: pypyr.steps.echo
  comment: don't run this step if int > 4.
           No need to wrap the expression in extra quotes!
  run: !py thisIsAnInt < 5
  in:
    echoMe: you'll see me if context thisIsAnInt is less than 5.
- name: pypyr.steps.echo
  comment: only run this step if breakfast includes spam
           since the first char is a single quote, wrap the Py string in
           double quotes to prevent malformed yaml.
  run: !py "'spam' in ['eggs', 'spam', 'bacon']"
  in:
    echoMe: you should see me because spam is in breakfast listing!
```

## examples
See a worked [example for py strings](https://github.com/pypyr/pypyr-example/tree/main/pipelines/pystrings.yaml).