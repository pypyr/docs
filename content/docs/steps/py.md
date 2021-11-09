---
title: pypyr.steps.py
linktitle: py
date: 2020-07-06T13:17:22+01:00
description: Execute inline python code.
card_extra_summary:
  heading: input context property
  details: "`py` or `pycode` (str)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: py
seo_article_headline: Execute inline python in pipeline.
seo_description: Run dynamic inline python inside a task-runner pipeline step.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: py -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [inline code]
---
# pypyr.steps.py
## run inline python
Executes the context value `py` as a dynamically interpreted python code block.

This is useful for adding inline Python code right in your pipeline.

You can use all the usual [Python
built-ins](https://docs.python.org/3/library/functions.html) 
like `len`, `abs` and so forth. You 
can import standard libraries or your own custom modules & objects using the 
standard Python import syntax (e.g `import x as y`, `from x import y`).

For example, this will invoke python `print`, do some arbitrary addition and
print 2:

```yaml
steps:
  - name: pypyr.steps.py
    comment: Example of arb python code. Will print 2.
    in:
      py: print(1+1)
```

## multi-line python code block
You can run multiple python statements in the same code block:
```yaml
- name: pypyr.steps.py
  comment: multi-line statement starts with |, per yaml spec
  in:
    py: |
      print(f"py step: {0+1}")
      
      arbvalue = 420
      print(arbvalue)

      # save arbvalue to context
      save('arbvalue')

- name: pypyr.steps.py
  comment: here splitting multi-line statements with ; 
           arbvalue survives between steps.
  in:
    py: print("py step 2"); arbvalue += 4
```

You can use whichever of the standard yaml block scalar style & chomping
indicators work best for you. For example, `|-` will strip newlines at the end.

{{% note tip %}}
Do remember that if your inline code block gets unwieldy, you can very easily 
run your own Python .py file from pypyr by using it as a 
[custom step]({{< ref "/docs/api/step">}}) in itself.

You don't even have to package your python code - pypyr will resolve the path 
relative to the working directory for you.
{{% /note %}}

## working with context
When you are in a `py` step, context keys exist as variables of the same name. 

```yaml
- name: pypyr.steps.set
  comment: set some arbitrary values in context
  in:
    set:
      existing_value: 123
      existing_list: ['one', 'two', 'three']
      existing_dict:
        a: a value
        b: b value

- name: pypyr.steps.py
  comment: context keys are available as variables of the same name
  in:
    py: |
      if existing_value > 100:
        print('hello!')
      else:
        print('no no no!')

      assert existing_list[1] == 'two'

      print(existing_dict['b'])
```

### save values to context
A py step scope behaves like a Python function. Which is to say any variables
you assign inside the py-step will not persist after the step completes, unless
you are altering a mutable type in-place.

If you want to persist variables or key/values to context inside a `py` code
block so that these are available to subsequent steps, use the `save` function
explicitly to specify variable names that you want to save.

```yaml
- name: pypyr.steps.py
  in:
    py: |
      a = 1
      b = 2

      # save a & and create c in context.
      save('a', c=b)

- name: pypyr.steps.echo
  comment: can use the new context values as normal in subsequent steps.
  in:
    echoMe: We set {a} and {c} in the previous step.
            There is no b in context because it wasn't saved.
```

The `save` function signature is:
```python
save(*args, **kwargs)
```

Arguments:
  - `args`
    - list of variable names to save to context.
  - `kwargs`
    - list of key=value pairs to save to context.

You can use `save` as a list of string arguments, or keywords if you want to
persist the input with a new name, or both. Here are some examples:

```python
save('a') # save contents of a to context as key 'a'
save('b', 'c') # save contents of b and c to context as b and c respectively
save(d=d) # save contents of d to context as key 'd'
save(e='arb value here') # save new key to context from a literal or expression
save(ff=f, gg=g+1) # save variables to a different name

# combine all the above & create a new variable 'l' on the fly
save('h', 'i', jj=j, kk=k, l=len('arb expression'))
```

Important, when you just specify object names to save, you have to specify the
name of the variable, not the object itself. Simply put, surround it with
quotes!

```yaml
- name: pypyr.steps.py
  comment: this is WRONG
  in:
    py: |
      my_var = 123
      another_var = 456
      save('myvar', 'another_var') # NOT save(myvar, another_var)
```

### mutable objects & scope 
Your code inside the `py` step is an enclosed scope. It behaves like a Python
function usually does - context variables pass by assignment. This means that:
- local assignments to immutable objects do not persist outside of the enclosed
  scope.
- in-place changes to mutable objects will persist to context after the py step
  completes.

This means that strictly speaking you do not need to call `save` to persist
in-place changes to mutable objects - but you won't break anything if you do, so
don't worry about it.

```yaml
- name: pypyr.steps.set
  in:
    set:
      existing_value: 123
      existing_list: ['one', 'two', 'three']
      existing_dict:
        a: a value
        b: b value

- name: pypyr.steps.py
  comment: context vars behave like args to a python function
  in:
    py: |
      # immutable type re-assignment is local to the current step
      existing_value = max(420, 69)

      # in-place mutable edits endure outside of the current scope
      existing_list[0] = 'mutable types will update'

      existing_dict['a'] = 'mutable types will update'
```

This will result in context like this:
```python
{'existing_dict': {'a': 'mutable types will update',
                   'b': 'b value'},
 'existing_list': ['mutable types will update', 'two', 'three'],
 'existing_value': 123}
```

Notice that the immutable value change did not persist to `existing_value`, but
the in-place changes to the dict and the list did persist.

### reserved keywords
{{% note warn %}}
If you have your own key in context named `save`, it will not be accessible in 
the `py` style step, because the `save()` function hides the name. pypyr won't 
touch your original `save` key + value, it's just hidden for the duration of 
the py step. Either use a different name for your key, or use the `pycode` 
style input for the py step.
{{% /note %}}

## using built-ins and imports
All of Python's built-ins and standard library modules are available to you. You
import like you usually do in Python.

```yaml
- name: pypyr.steps.py
  comment: using imports
  in:
    arb_url: http://arbhost/blah
    py: |
      # different styles of python import syntax
      from math import sqrt
      import urllib.parse
          
      # use imported code like sqrt from math
      arbvalue = sqrt(1764)
          
      # use builtin functions like int()
      print(int(arbvalue))

      host = urllib.parse.urlparse(arb_url).netloc

      # use builtin functions like len()
      print(len(host))
```

## import your own modules
You can also import your own custom modules and objects, as long as they resolve
in the current Python environment. To help with this, pypyr also looks in the
current working directory for your own custom modules, so you can import &
re-use modules in your current path without having to package & publish the code
to the current environment first.

Assume you have a python file in your working directory, saved in a `mydir`
subdirectory like this:
```python
# ./mydir/mymodule.py

def arb_function(arb_arg):
    return f'arb_function says: {arb_arg}'
```

Because pypyr will also look in your working directory for modules,
`mydir.mymodule` will resolve on `import` without you have to do anything
special. 

So you can use this from your pipeline like this:
```yaml
- name: pypyr.steps.py
  comment: import custom modules relative to your working dir
  in:
    py: |
      from mydir.mymodule import arb_function

      out = arb_function('my input')
```

## create reusable functions & classes
You can declare your own functions and classes in a py step. You can use these
in the py step itself, and you can also use these in subsequent py steps or in 
[!py strings]({{<ref "/docs/substitutions/py-strings" >}}) if you 
`save` the function or class object to context:

```yaml
- name: pypyr.steps.py
  comment: create re-usable functions & classes
  in:
    py: |
      class MyClass():
        attribute = 'my class!'

        def do_thing(self, my_arg):
          return my_arg + 1

      
      def my_function(arg1, arg2):
        return arg1 + arg2


      # call your function like you normally do
      result = my_function(420, 69)
      
      # instantiate & use your class like usual
      MyClass().do_thing(42069)

      # save to context to use class & function in subsequent steps
      save('my_function', 'MyClass')

- name: pypyr.steps.py
  comment: re-use class & function previously saved to context
  in:
    py: |
      my_instance = MyClass()
      my_instance.do_thing(123)

      for i in range(3):
        my_function(i, 3)

- name: pypyr.steps.set
  comment: re-use class & function anywhere you can use a !py string.
  run: !py my_function(1, 2) == 3
  in:
    set:
      new_key: !py MyClass().do_thing(456)
```

## the context object itself
If you want access to the entire context dictionary object itself, rather than 
just its contents, you can use the alternative `pycode` input to the 
`pypyr.steps.py` step.

When you use the `pycode` input, context exists as a dictionary-like object 
named `context`. All the usual python `dict` methods are available.

In general, you probably should prefer using the `py` style input - although 
both styles do the same thing, it's much less annoying and error-prone not to 
have to type out the dictionary accessors all the time. The `pycode` style 
input is not particularly more "advanced", or "better", but it will be somewhat
faster and also use less memory than the `py` style. By "somewhat", meaning that
the performance differential is likely to be so small that you won't notice
unless your context is exceptionally big.

In the following example, you can see how to access the context object, add or 
update values & how to import other modules:

```yaml
- name: pypyr.steps.set
  comment: set arb context to manipulate in the next step
  in:
    set:
      existing_key: existing value
      existing_dict:
        a: a value
        b: 123

- name: pypyr.steps.py
  comment: runs arb python using context from previous step
            also sets some new values in context.
  in:
    # multi-line statement starts with |, per yaml spec
    pycode: |
      import math

      print(f"py step: {0+1}")

      context['arbvalue'] = math.sqrt(36)
      print(context['arbvalue'])

      context['existing_key'] = 'updated value'
      context['existing_dict']['a'] = 'a value set in py step'
      context['existing_dict']['b'] = 456
      context['existing_dict']['new_key'] = ['zero', 'one', 2, 'three']

      print(len(context))

- name: pypyr.steps.py
  description: context['arbvalue'] survives between steps.
  in:
    # here splitting multi-line statements with ;
    pycode: print("py step 2"); context['arbvalue'] += 4

- name: pypyr.steps.echo
  in:
    echoMe: |
      arbvalue is {arbvalue} and existing_key is now {existing_key}
      you can worked with nested values too: {existing_dict[a]} - {existing_dict[b]}
```

These steps will output:

```text
py step: 1
6.0
4
context['arbvalue'] survives between steps.
py step 2
arbvalue is 10.0 and existing_key is now updated value
you can worked with nested values too: a value set in py step - 456
```

{{% note tip %}}
When you use the `pycode` style you don't need to use `save()` to persist 
values back to context, since you are working directly with the mutable 
`context` object itself. If you want to update, edit or remove a value, it's up 
to you to persist the change with `context['mykey'] = 'added/updated value'`.

`save()` is only available when you use the `py` style input.
{{% /note %}}

## example
See a worked [example of inline python using py
style](https://github.com/pypyr/pypyr-example/tree/main/pipelines/py.yaml) and
[example of inline python using py-code
style](https://github.com/pypyr/pypyr-example/tree/main/pipelines/py-code.yaml).