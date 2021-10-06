---
title: pypyr.steps.add
linktitle: add
date: 2021-10-05T16:53:21+01:00
description: Add item to a set.
draft: false
card_extra_summary:
  heading: input context property
  details: "`add` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: add
seo_article_headline: Add item to a set in context.
seo_description: Add item to a set in the pypyr task-runner context.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: add -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [context, collections]
---
# pypyr.steps.add
## add item to a set
Add item(s) to a set.

The difference between a 
[set](https://docs.python.org/3/library/stdtypes.html?highlight=list#set-types-set-frozenset)
and a
[list](https://docs.python.org/3/library/stdtypes.html?highlight=list#lists)
is that a list allows duplicate elements, whereas every item in a set is
unique. If you're looking to work with a list, use
[append]({{< ref "append" >}}) instead. Sets are unordered, lists are ordered.

```yaml
- name: pypyr.steps.add
  comment: add item to a set
  in:
    add:
      set: my_set # required. Name of set.
      addMe: item # required. Add this to the set.
      unpack: False # optional. defaults False. If True, enumerate addMe & add each item individually.
```

Use the `set` argument to specify the name of the set. If the set does not
exist, pypyr will create a set of that name for you and initialize it with
the value of `addMe`.

```yaml
- name: pypyr.steps.add
  comment: instantiate set with name arbset with a single item in it.
  in:
    add:
      set: arbset
      addMe: one

- name: pypyr.steps.add
  comment: add single item to arbset
  in:
    add:
      set: arbset 
      addMe: two
```

This example will result in `arbset == {'one', 'two'}`.

The `set` input argument can be either a string referring to the name of a
set, or a set instance itself. To reference a set instance, use a Py String
as described in the section further down called
[add to a nested set](#add-to-a-nested-set).

`addMe` can be any [hashable](https://docs.python.org/3/glossary.html#term-hashable)
type. You cannot add a list object or dict object to a set, because these are 
not hashable - although you can add individual hashable values from a list or
dict to a set if you set `unpack` to `True`. 

You can add different types to the same set, as long as they are hashable.
Simply put, the same set can contain any mixture of strings, numbers, dates,
booleans and custom hashable types.

## use formatting expressions
You can use formatting expressions on any of this step's inputs.

```yaml
- name: pypyr.steps.set
  comment: set here means set variable, not the set() collection type.
  in:
    set:
      the_name: my_set
      item1: 1
      item2: 2

- name: pypyr.steps.add
  comment: use formatting expressions to specify set name & item to add.
  in:
    add:
      set: '{the_name}'
      addMe: '{item1}'
  
- name: pypyr.steps.add
  comment: use formatting expressions to specify set name & item to add.
  in:
    add:
      set: '{the_name}'
      addMe: '{item2}'
```

This example will result in a set named `my_set`, containing `{1, 2}`.

## add multiple items to a set
Set `unpack` to `True` when `addMe` is enumerable and you want to enumerate
`addMe` to add each item individually to the set.

If `addMe` is a list, pypyr by default assumes you mean to unpack the list
into the set. So you do not need to specify `unpack: True` explicitly.

```yaml
- name: pypyr.steps.add
  comment: add multiple items from a list individually to arbset
           you do not need to set unpack for a list.
  in:
    add:
      set: arbset
      addMe:
        - three
        - four
```

This example will result in `arbset == {'three', 'four'}`.

For other enumerable types, you do need to set `unpack` explicitly.

```yaml
- name: pypyr.steps.add
  comment: instantiate set with name dict_in_set.
  in:
    add:
      set: dict_in_set
      unpack: True
      addMe:
        k1: v1
        k2: v2

- name: pypyr.steps.add
  comment: add a dict to dict_in_set.
  in:
    add:
      set: dict_in_set
      unpack: True
      addMe:
        k3: v3
        k4: v4
```

Adding a dict to a set like this will add just the keys to the set, not the
`{key: value}` combination.

The result of this example is `dict_in_set == {'k4', 'k3', 'k1', 'k2'}`.

## duplicates
If `addMe` exists in the set already, the step will just quietly not add the
duplicate and continue without raising an error.

```yaml
- name: pypyr.steps.add
  in:
    add:
      set: example_set
      addMe: [1, 2]

- name: pypyr.steps.add
  comment: 2 exists in set already, so won't add it.
  in:
    add:
      set: example_set
      addMe: 2

- name: pypyr.steps.add
  in:
    add:
      set: example_set
      addMe: 3
```

This example will result in `example_set == {1, 2, 3}`.

## enumerables in sets
By default pypyr adds the contents of `addMe` to the set as a single item.

This means that if `addMe` is a hashable enumerable itself, you will create a
set containing the enumerable as an object rather than its items individually.
If you mean to enumerate the contents of `addMe` and add each individually,
set `unpack` to True.

In this example we add some tuples to a set:

```yaml
- name: pypyr.steps.add
  comment: instantiate set with name tuples_in_set.
           the 1st item in set is a single tuple containing 3 items.
  in:
    add:
      set: tuples_in_sets
      addMe: !py ('one', 'two', 'three')

- name: pypyr.steps.add
  comment: add a singe tuple to arbset
  in:
    add:
      set: tuples_in_sets
      addMe: !py ('four', 'five', 'six')

- name: pypyr.steps.add
  comment: unpack a tuple into arbset. each item added individually.
  in:
    add:
      set: tuples_in_sets
      unpack: True
      addMe: !py ('four', 'five', 'six')
```

This example will result in:
```python
tuples_in_sets = {
    'four',
    ('one', 'two', 'three'),
    'five',
    'six',
    ('four', 'five', 'six')}
```

If you set `unpack` to `True` for each step, the result would be: 
```python
tuples_in_sets = {'two', 'four', 'three', 'five', 'six', 'one'}
```

## add to a nested set
When the `set` argument is a string, it refers to the name of a set in the
root of the context. If your set is nested inside a deeper level, you can use a
[Py String]({{< ref "/docs/substitutions/py-strings">}}) formatting expression
to reference the set instance.

```yaml
steps:
  - name: pypyr.steps.contextsetf
    comment: set is nested in dict under key 'b'
    in:
      contextSetf:
          arbkey: 2
          arbvalue: 3
          mykey:
            a: value
            b: !py set([1, arbkey])

  - name: pypyr.steps.add
    comment: use formatting expression for nested set
    in:
      add:
        set: !py mykey['b']
        addMe: '{arbvalue}'
```

This pipeline will result in `mykey['b'] == {1, 2, 3}`.

## unpack a string
If `addMe` is a single string and `unpack` is `True`, each character in the
string adds to the set individually.

```yaml
- name: pypyr.steps.add
  comment: unpacking a string into a set named strset
  in:
    add:
      set: strset
      unpack: True
      addMe: one
```

The value of `strset` is: `{'o', 'e', 'n'}`. Remember that sets are unordered.
