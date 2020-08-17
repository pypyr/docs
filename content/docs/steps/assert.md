---
title: pypyr.steps.assert
linktitle: assert
date: 2020-08-12
description: Stop pipeline if item in context is not as expected.
publishdate: 2020-08-13
lastmod: 2020-08-16
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
Assert that something is `True` or equal to something else.

```yaml
  - name: pypyr.steps.assert
    comment: evaluate as truthy
    in:
      assert:
        this: '{evaluateMe}'
  - name: pypyr.steps.assert
    comment: compare two complex types - list with nested dictionaries.
    in:
      assert:
        this: '{complexThing1}'
        equals: '{complexThing2}'
```

Uses these context keys:

- `assert`
  - `this`
    - mandatory
      - If `equals` not specified, evaluates as a boolean.
  - `equals`
    - optional
      - If specified, compares `this` to `equals`

If `assert['this']` evaluates to `False` raises error.

If you also specify `assert['equals']`, raises error if `assert['this'] != assert['equals']`.

The step raises an exception of type `AssertionError` if the assertion fails.

All inputs support string [substitutions]({{< ref "/docs/substitutions">}}).

## examples
### boolean
```yaml
assert: # continue pipeline
  this: True
assert: # stop pipeline
  this: False
```

### substitutions

```yaml
interestingValue: True
assert:
  this: '{interestingValue}' # continue with pipeline
```

### non-0 numbers evaluate to True

```yaml
assert:
  this: 1 # non-0 numbers assert to True. continue with pipeline
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
See a worked [example for assert](https://github.com/pypyr/pypyr-example/tree/master/pipelines/assert.yaml).
