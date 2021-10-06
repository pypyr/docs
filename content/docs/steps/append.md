---
title: pypyr.steps.append
linktitle: append
date: 2021-10-05T14:42:26+01:00
description: Append item to a list.
card_extra_summary:
  heading: input context property
  details: "`append` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: append
seo_article_headline: Append item to a list in context.
seo_description: Append item to the end of a list in the pypyr task-runner context.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: append -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [context, collections]
---
# pypyr.steps.append
## append item to a list
Append item(s) to the end of a
[list](https://docs.python.org/3/library/stdtypes.html?highlight=list#lists).

The full input for the `append` step looks like this:
```yaml
- name: pypyr.steps.append
  comment: append item to list
  in:
    append:
      list: my_list # required. Name of list.
      addMe: list item # required. Append this to the list
      unpack: False # optional. Defaults False. If True, enumerate addMe & append each item individually.
```

Use the `list` argument to specify the name of the list. If the list does not
exist, pypyr will create a list of that name for you and initialize it with
the value of `addMe`.

```yaml
- name: pypyr.steps.append
  comment: instantiate list with name arblist with a single item in it.
  in:
    append:
      list: arblist
      addMe: one

- name: pypyr.steps.append
  comment: add single item to arblist
  in:
    append:
      list: arblist 
      addMe: two
```

This example will result in `arblist == ['one', 'two']`.

The `list` input argument can either be a string referring to the name of a
list, or a list instance itself. To reference a list instance, use a Py String
as described in the section further down called
[append to a nested list](#append-to-a-nested-list).

`addMe` can be any type. You can add different types to the same list. Simply
put, the same list can contain any mixture of strings, numbers, dates, booleans,
custom types and collections like dictionaries or lists.

A list allows duplicate items. If you want to work with unique values only, you
can use [add]({{< ref "add" >}}) to work with a 
[set](https://docs.python.org/3/tutorial/datastructures.html#sets) instead.
Lists are ordered, sets are unordered.

## use formatting expressions
You can use formatting expressions on any of this step's inputs.

```yaml
- name: pypyr.steps.set
  in:
    set:
      the_name: my_list
      item1: 1
      item2: 2

- name: pypyr.steps.append
  comment: use formatting expressions to specify list name & item to add.
  in:
    append:
      list: '{the_name}'
      addMe: '{item1}'
  
- name: pypyr.steps.append
  comment: use formatting expressions to specify list name & item to add.
  in:
    append:
      list: '{the_name}'
      addMe: '{item2}'
```

The will result in a list named `my_list`, containing `[1, 2]`.

## append multiple items to a list
Set `unpack` to `True` when `addMe` is enumerable and you want to enumerate
`addMe` to append each item individually to the list.

```yaml
- name: pypyr.steps.append
  comment: append multiple items from a list individually to arblist
  in:
    append:
      list: arblist
      unpack: True
      addMe:
        - three
        - four
```

This example will result in `arblist == ['three', 'four']`.

## list of lists
By default pypyr adds the contents of `addMe` to the list as a single item.

This means that if `addMe` is a list itself, you will create a list containing
a list. If you mean to enumerate the contents of `addMe` and add each
individually, set `unpack` to True.

```yaml
- name: pypyr.steps.append
  comment: instantiate list with name list_in_lists.
           the 1st item is a list itself with 3 items.
  in:
    append:
      list: lists_in_lists
      addMe:
        - one
        - two
        - three

- name: pypyr.steps.append
  comment: add single item list to arblist.
           notice that addMe is a list, not a str.
  in:
    append:
      list: lists_in_lists
      addMe:
        - four

- name: pypyr.steps.append
  comment: add a list as a list inside arblist.
  in:
    append:
      list: lists_in_lists
      addMe:
        - five
        - six
```

This example will result in:
```yaml
lists_in_lists:
  - [one, two, three]
  - [four]
  - [five, six]
```

If you set `unpack` to `True` for each step, the result would be: 
```yaml
lists_in_lists:
  - one
  - two
  - three
  - four
  - five
  - six
```
## append to a nested list
When the `list` argument is a string, it refers to the name of a list in the
root of the context. If your list is nested inside a deeper level, you can use a
[Py String]({{< ref "/docs/substitutions/py-strings">}}) formatting expression
to reference the list instance.

```yaml
steps:
  - name: pypyr.steps.set
    comment: list is nested in dict under key 'b'
    in:
      set:
        arbkey: 2
        arbvalue: 3
        mykey:
          a: value
          b:
            - 1
            - '{arbkey}'

  - name: pypyr.steps.append
    comment: use formatting expression for nested list
    in:
      append:
        list: !py mykey['b']
        addMe: '{arbvalue}'
```

This pipeline will result in `mykey['b'] == [1, 2, 3]`.

## unpack a string
If `addMe` is a single string and `unpack` is `True`, each character in the
string appends to the list individually.

```yaml
- name: pypyr.steps.append
  comment: unpacking a string
  in:
    append:
      list: strlist
      unpack: True
      addMe: one
```

The value of `strlist` is: `['o', 'n', 'e']`