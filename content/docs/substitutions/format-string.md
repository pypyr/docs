---
title: format string interpolation
linktitle: format string
date: 2020-06-13T21:38:57+01:00
description: String interpolation with substitution tokens are for easy value replacement inside a pipeline workflow. Works with complex types, not just strings.
# categories: [expressions]
# keywords: "token replacement, string substitutions"
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
# topics: []
---
# string interpolation
## formatting expressions with replacement tokens
You can use substitution tokens, aka string interpolation, to format strings 
where specified for context items. This substitutes anything between `{curly
braces}` with the context value for that key. This also works with nested values
where you have dictionaries/lists inside dictionaries/lists. 

For example, if your context looked like this:

```yaml
key1: down
key2: valleys
key3: value3

key4: Piping {key1} the {key2} wild
# key4 == 'Piping down the valleys wild'
```

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

Here is another example of how you access nested dictionary mappings:

```yaml
my_mapping:
  a: b
  c:
    d: e
    f: g
    h:
      - item 1
      - item 2

# nested_value is 'g'
nested_value: '{my_mapping[c][f]}'

# nested_with_list is 'item 2'
nested_with_list: '{my_mapping[c][h][1]}'
```

## replacement tokens vs structural yaml
In json & yaml, {curly braces} need to be inside quotes to make sure they parse
as string formatting tokens and not as structural indicators.

Especially watch in yaml, where `{` as the first character of a key or value 
will read as a structural mapping if it's not in quotes like this: `"{key}"`.

```yaml
invalid_value: {mytag} # INVALID! yaml reads opening curly brace as a mapping type.
valid_token: '{mytag}' # explicitly make it a string with the quotes.
also_valid: start {mytag} end # if the {curlies} are in the middle you don't need quotes.
```

## interpolate & keep the target type
If your replacement expression is a single token and nothing more, pypyr will 
keep the type of the replacement target. This is very useful when you want to 
assign or copy values in context.

```yaml
a_bool: True
an_int: 123
a_date: 2010-11-12
a_string: this is a string
a_list:
  - item 1
  - item 2
  - item 3
a_map:
  a: b
  c: d

format_me: # each new_ item is the same type as the source
  new_bool: '{a_bool}' # new_bool is also a bool
  new_int: '{an_int}' # new_int is an integer
  new_date: '{a_date}' # new_date is datetime
  new_string: '{a_string}' # new_string is string
  new_list: '{a_list}' # new_list is a list
  new_map: '{a_map}' # new_map is a dict/map
```

When a replacement expression is part of a string that contains more than just 
a single replacement token, pypyr will convert all the replacement tokens to 
their string representations. This is the case when the string contains any 
literal in addition to one or more replacement tokens. This is known as a 
compound replacement expression.

```yaml
a_bool: True
an_int: 123
a_string: this is a string

# new_string will be a string
new_string: a string with {a_bool}, {an_int} and {a_string}
# new_string is 'a string with True, 123 and this is a string'

new_string2: 0{an_int}4
# new_string2 is a string == '01234'
```

You can explicitly cast any token expression to a string like this:

```yaml
my_int: 123
my_formatted_str_from_int: '{my_int!s}'
# my_formatted_str_from_int is string "123", not number 123
```

This is useful when you have a single replacement token expression where you 
don't want to keep the target type.

## recursive vs flat format
### recursive format
A recursive format replaces tokens recursively, making token replacements in 
the result of each replacement. This is what happens by default in a single 
token expression.

```yaml
nested_key: 'nested value'
key: '{nested_key}'

format_me: '{key}'
# format_me == 'nested value'
```

### flat format
A flat format makes a token replacement once, but does not make any further 
token replacements in the result. This is what happens by default in a compound 
replacement expression.

```yaml
nested_key: arbitrary
key: contains {nested_key}

format_me: this {key} formatted flat by default
# format_me == 'this contains {nested key} formatted flat by default'
# use recursive format if you want 'this contains arbitrary flat format by default'
```

Flat Format is especially useful when your formatting expression evaluates to 
a string that contains literal curly braces that are not meant as pypyr 
replacement tokens - for example a string containing json.

### explicitly set recursive
If you want to format the result recursively in a compound token expression, 
you explicitly set `rf` (Recursive Format) to do so:

```yaml
nested_key: arbitrary result
key: contains {nested_key}

format_me: this {key:rf} formatted recursively
# format_me == 'this contains arbitrary result formatted recursively'
```

### explicitly set flat
If you want to flat format a single token expression instead, you explicitly 
set `ff` (Flat Format) to do so:

```yaml
nested_key: 'nested value'
key: '{nested_key}'

format_me: '{key:ff}'
# format_me == '{nested_key}'
```

You can explicitly stop a recursive format from further formatting nested 
values by using a `ff` directive where you want it stop:

```yaml
# recurse to the end
k3: 'the end'
k2: '{k3}'
k1: '{k2}'
k0: '{k1}'

format_me: '{k0}' # == 'the end'


# recurse until ff
k3: 'the end'
k2: '{k3}' # will NOT process {k3}, because k2:ff (Flat Format)
k1: '{k2:ff}'
k0: '{k1}'

format_me: '{k0}' # == '{k3}'
```

## iterable objects
### recursively format iterable objects
pypyr will format iterable objects like lists, sets and mappings (dict) 
recursively by default.

```yaml
k1: formatted A
k2: formatted one

my_map:
  a: '{k1}'
  c: d
  e: 
    f: g
    h: 
      - zero
      - '{k2}'
      - two

format_me: '{my_map}'
```

The resulting value of `format_me` is:

```yaml
a: formatted A
c: d
e: 
  f: g
  h: 
    - zero
    - formatted one
    - two
```

### do not recurse iterable objects
You can use the Flat Format `ff` indicator not to iterate the target object:

```yaml
k1: formatted A
k2: formatted one

my_map:
  a: '{k1}'
  c: d
  e: 
    f: g
    h: 
      - zero
      - '{k2}'
      - two

format_me: '{my_map:ff}'
```

The resulting value is of `format_me` is:

```yaml
my_map:
  a: '{k1}'
  c: d
  e: 
    f: g
    h: 
      - zero
      - '{k2}'
      - two
```

## assign variables with set
When you use the [set step]({{< ref "docs/steps/set" >}}) pypyr 
evaluates each assignment in order from the top down. Each assignment is 
atomic.

This might look like a recursive format, but it isn't - it's just that each 
line evaluates individually before pypyr moves on to the next.

```yaml
- name: pypyr.steps.set
  comment: each assignment evaluates atomically,
           in order from the top down
  in:
    set:
      k3: 'the end'
      k2: '2 {k3}' # == '2 the end'
      k1: '1 {k2}' # == '1 2 the end'
      k0: '0 {k1}' # == '0 1 2 the end'
```

## escape sequences
Escape literal curly braces with doubles: 

| special character | escape sequence |
| ----------------- | --------------- |
| `{`               | `{{`            |
| `}`               | `}}`            |


If your particular strings make using the escape sequences a nuisance, you can
avoid doubling the curly braces by using [sic strings aka literal strings]({{< ref "sic-strings">}}) 
instead.

```yaml
# two ways of escaping literal curly braces:
my_escaped_string: the doubled {{curly}} means it won't parse as a replacement token.
another_escaped_string: !sic parse {curly} as literal without doubling.
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
- name: pypyr.steps.set
  in:
    set: 
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

## format specification mini language
You can use the full expressive power of Python's format specification 
mini-language in any replacement token expression.

This allows you to 

- align left, right or center;
- left or right pad your output with spaces or custom characters;
- format numbers to add a +/- sign;
- format numbers as hex, oct, decimals
- add thousands separators to numbers;
- customize datetime output strings;
- use exponential or fixed point notation where you can set the precision.
- and more. . .

You can combine pypyr's Flat Format `ff` and Recursive Format `rf` indicators 
with any of these. Just put the `ff` or `rf` at the very beginning of the 
format specification. The subsequent formatting expression will apply to the end 
result of any recursive formatting sequence.

```yaml
- name: pypyr.steps.echo
  comment: use any python format mini-syntax
  in:
    arb_string: ABC
    arb_number: 42
    big_number: 12345678
    arb_date: 2006-01-02 15:04:05
    string_1: two
    string_2: 'one {string_1}'
    string_3: 'zero {string_2}'

    echoMe: |
      int: {arb_number:d}
      hex: {arb_number:x}
      oct: {arb_number:o}
      bin: {arb_number:b}
      %: {arb_number:.2%}
      
      left align: begin{arb_string:<6}end
      right align: begin{arb_string:>6}end
      center: begin{arb_string:^7}end

      left pad: {arb_string:+<6}
      right pad: {arb_string:+>6}
      center pad: {arb_string:+^7}

      thousands separator: {big_number:,}

      date/time formatting: {arb_date:%A %H:%M}

      rf/ff combined with other mini-format syntax
      {string_3:rf+^14}
```

This will output:

```text
int: 42
hex: 2a
oct: 52
bin: 101010
%: 4200.00%

left align: beginABC   end
right align: begin   ABCend
center: begin  ABC  end

left pad: ABC+++
right pad: +++ABC
center pad: ++ABC++

thousands separator: 12,345,678

date/time formatting: Monday 15:04

rf/ff combined with other mini-format syntax
+zero one two+
```

For full details of the Python format specification mini-language, check here:

https://docs.python.org/3/library/string.html#format-specification-mini-language

For full date time format codes, check here:

https://docs.python.org/3/library/datetime.html#strftime-and-strptime-format-codes