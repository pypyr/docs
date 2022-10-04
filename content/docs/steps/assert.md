---
title: pypyr.steps.assert
linktitle: assert
date: 2020-08-12
description: Stop pipeline if item in context is not as expected.
publishdate: 2020-08-13
card_extra_summary:
  heading: input context property
  details: "`assert` (dict)"
categories: [ steps ]
menu:
  docs:
    parent: steps
    name: assert
seo_article_headline: Stop pipeline execution & raise an error if assert evaluates false.
seo_description: Stop pipeline execution & raise an error when a dynamic assert condition evaluates to False.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: Assert dynamic value as expected in pipeline.
topics: [ control-of-flow, debug ]
---
# pypyr.steps.assert
## stop pipeline execution if condition false
Assert that something is `True` or equal to something else. The step raises an 
exception of type `AssertionError` if the assertion fails.

You can express an assert in three different ways:

```yaml
  - name: pypyr.steps.assert
    comment: evaluates `assert` as truthy
    in:
      assert: '{evaluateMe}'
  - name: pypyr.steps.assert
    comment: evaluate `this` as truthy
    in:
      assert:
        this: '{evaluateMe}'
  - name: pypyr.steps.assert
    comment: assert that two things are equal
    in:
      assert:
        this: '{complexThing1}'
        equals: '{complexThing2}'
```

The first two mostly do the same thing, so use whichever pleases your eye more. 
The only difference is in how pypyr 
[processes mappings for truthy]({{<ref "#asserting-mappings-as-truthy" >}}).

Uses these context keys:

- `assert` - evaluate this as truthy when `this` or `equals` do not exist. 
  - `this` (optional)
    - If `equals` not specified, evaluates as a boolean truthy.
  - `equals` (optional)
    - If specified, compares `this` to `equals`
  - `msg`(optional)
    - If specified, use this value as a custom error message if the assert is
      False.

If `this` evaluates to `False` raises error.

If you also specify `equals`, raises error if `this != equals`.

When you do specify an `equals` condition, you must also have a `this` 
condition set to which to compare it.

All inputs support string [substitutions]({{< ref "/docs/substitutions">}}).

## set custom error message
Use the `msg` input to set a custom error message if you want to output more
information about the failure condition.

```yaml
- name: pypyr.steps.assert
  comment: evaluate `this` as truthy, raise msg if it's false.
  in:
    assert:
      this: '{evaluateMe}'
      msg: informative error message here

- name: pypyr.steps.assert
  comment: Compare `this` to `equals`, raise msg if it's false.
  in:
    assert:
      this: 'known value'
      equals: '{arb_key}'
      msg: informative error message here, {arb_key} value was wrong.
```

Notice you can also use {substitutions} in your custom message.

## truthy bool evaluation
The standard 
[Python truth value testing](https://docs.python.org/3/library/stdtypes.html#truth-value-testing)
rules apply.

Simply put, this means that `1`, `TRUE`, `True` and `true` will be `True`.

`None` or empty, `0`,`''`, `[]`, `{}` will be `False`.

{{% note tip %}}
pypyr will interpret case insensitive string `"true"`, `"1"` & `"1.0"` 
as boolean `True`. All other string values, including empty string, evaluate to 
`False`.

This is generally not what typical programming languages do on a strict string
truthy, where any given string value other than null/empty will evaluate 
`True`, but more often than not within the context of a pipeline where you are 
processing text-based flags it saves you some footwork specially having to 
cast strings to booleans first.

If you do want the more typical string truthy evaluation, use an explicit 
py-string like this:

```yaml
myString: arbitrary string here

assert: !py bool(myString)
```
{{% /note %}}

## asserting mappings as truthy
If you happen to be asserting a mapping (aka dictionary) that contains a 
"this" or "equals" key that is *not* meant as a pypyr instruction, use the 2nd 
form where you set your mapping as the `this` condition.

If your mapping/dict doesn't contain a non-pypyr this/equals, feel free to save 
yourself some typing and use the simplified `assert: '{mydict}'` form.

```yaml
- name: pypyr.steps.assert
  comment: setting assert to an arbitrary mapping  
           evaluates the whole mapping as truthy.
  in:
    arb_dict:
      mykey: my value
      anotherkey: another value
    assert: '{arb_dict}'

- name: pypyr.steps.assert
  comment: if your mapping has a "this" or "equals" that 
           is not meant for pypyr, you can prevent pypyr 
           from interpreting it as a processing instruction 
           by setting the mapping under "this".
  in:
    arb_dict_with_this:
      mykey: my value
      this: this "this" is not meant as a pypyr instruction
      equals: this "equals" is not meant as a pypyr instruction
    assert:
      this: '{arb_dict_with_this}'
```

## examples
### boolean & truthy
```yaml
assert: False # stop pipeline

assert: True # continue pipeline

assert: '{myObj}' # evaluate myObj as truthy
```

### substitutions

```yaml
interestingValue: True

assert: '{interestingValue}' # continue with pipeline
```

### non-0 numbers evaluate to True

```yaml
assert: 1 # non-0 numbers assert to True. continue with pipeline
```

### string equality

```yaml
assert:
  this: 'up the valleys wild'
  equals: 'down the valleys wild' # strings not equal. stop pipeline.
```

### string equality with substitutions

```yaml
k1: 'down'
k2: 'down'

assert:
  this: '{k1} the valleys wild'
  equals: '{k2} the valleys wild' # substituted strings equal. continue pipeline.
```

### number equality

```yaml
assert:
  this: 123.45
  equals: 0123.450 # numbers equal. continue with pipeline.
```

### number equality with substitutions

```yaml
numberOne: 123.45
numberTwo: 678.9

assert:
  this: '{numberOne}'
  equals: '{numberTwo}' # substituted numbers not equal. Stop pipeline.
```

### existence check
You can check if a key exists in context using a [py string expression]({{< ref
"/docs/substitutions/py-strings">}}):

```yaml
myvar: 123

# myvar exists in context. continue with pipeline.
assert: !py "'myvar' in locals()"

# only continue if myvar does NOT exist in context.
assert: !py "'myvar' not in locals()"
```

### complex types

```yaml
complexOne:
  - thing1
  - k1: value1
    k2: value2
    k3:
      - sub list 1
      - sub list 2

complexTwo:
  - thing1
  - k1: value1
    k2: value2
    k3:
      - sub list 1
      - sub list 2

assert:
  this: '{complexOne}'
  equals: '{complexTwo}' # substituted types equal. Continue pipeline.
```

### worked example
See a worked [example for assert](https://github.com/pypyr/pypyr-example/tree/main/pipelines/assert.yaml).
