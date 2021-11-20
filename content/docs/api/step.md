---
title: custom step
linktitle: step
description: Create your own custom step.
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
# card_extra_summary:
#   heading: input context property
#   details: "`assert` (dict)"
categories: [ steps ]
menu:
  docs:
    parent: api
    name: step
seo_article_headline: Easily code custom pipeline steps.
seo_description: Create a custom pypyr pipeline task-runner step in a few lines of Python code. Ready-made retries & loops for your code.
topics: [ custom code ]
---
# create a custom step
If you can't find a ready-made step that quite scratches your particular itch,
don't hesitate to code your own step - it's easy, and very much the philosophy
of pypyr that if you can write a quick couple of lines of python rather than
contort your pipeline with clumsy step sequences, then do so! I know some
frameworks don't really encourage you to stray outside the prescribed features,
but not so pypyr - your custom steps are first-class citizens of the pypyrverse.

You can freely mix your own custom steps and built-in steps in the same pipeline.

If you do code your own and you think it could be useful to the rest of the
community, even if it's trivial, check out the 
[contribution guide]({{< ref "/docs/contributing/contribute-to-pypyr" >}}) for 
how to submit your code and then you can bask in the glow of making open-source 
a better place.

If you don't code or that sounds like too much work, but you have an idea for a
new step that would make your life better, feel very free to
[get in touch](https://github.com/pypyr/pypyr/issues/new?title=step%20suggestion:&labels=feature,type:%20step)
with a feature request.

## step function signature
A custom step is any Python module that contains a function with this signature:
```python
run_step(context: pypyr.context.Context) -> None
```

Here is a fuller example that you can copy & paste to get started:
```python
import logging

from pypyr.context import Context

# getLogger will grab the parent logger context, so your loglevel and
# formatting automatically will inherit correctly from the pypyr core.
logger = logging.getLogger(__name__)


def run_step(context: Context) -> None:
    """Put your code in here. This shows you how to code a custom pipeline step.

    Args:
      context: dict-like. This is the entire pypyr context.
               You can mutate context in this step to make
               keys/values & data available to subsequent
               pipeline steps.

    Returns:
      None.
    """
    logger.debug("started")
    # you probably want to do some asserts here to check that the input context
    # dictionary contains the keys and values you need for your code to work.
    context.assert_key_has_value(key='mykey', caller=__name__)
    
    # do this if you want your step to support substitutions.
    # get_formatted will also iterate mykey if it's an iterable
    # and do substitutions for each item in it.
    mystep_context = context.get_formatted('mykey')
    # assuming input context
    # mykey:
    #  subkey: subkey value
    nested_value = mystep_context['subkey']

    # get a context item if you don't care about substitutions
    context_item = context['arbkey']

    # it's good form only to use .info and higher log levels when you must.
    # For .debug() being verbose is very much encouraged.
    logger.info("Your clever code goes here. . . ")

    # Add or edit context items. These are available to any pipeline steps
    # following this one.
    context['existingkey'] = 'new value overwrites old value'
    context['mynewcleverkey'] = 'new value'

    logger.debug("done")
```

## use custom step in pipeline
### module path resolution
The usual [custom module import resolution rules]({{< ref
"/docs/api/custom-module-search-path" >}}) apply.

Assuming you saved your python with the `def run_step(context)` function in a 
file like this `{pipeline dir}/mydir/mystep.py`:

```text
|- mypipelinedir/
  |- mypipe.yaml
  |- mydir/
    |- mystep.py
  |- step1.py
```

You can use use it in your `mypipe` pipeline like this:

```yaml
# {pipeline dir}/mypipe.yaml
steps:
    - step1 # run {pipeline dir}/step1.py
    - mydir.mystep # run {pipeline dir}/mydir/mystep.py
```

Because you reference the custom modules relative to the pipeline directory,
you can run this pipeline from anywhere and it'll work:

```bash
$ pypyr mypipelinedir/mypipe
```

If you package your code and you install the package into the active python
environment (i.e `$ pip install mypackage`), you can of course use the usual
python absolute package name instead:

```yaml
steps:
    - step1 # run {pipeline dir}/step1.py
    - mypackage.mystep # run mypackage.mystep
```

You can mix both packaged code and ad hoc modules in the same pipeline.

### passing context & decorators
All of the usual step decorators are available to your custom step. This makes 
it easy to use retry, looping and conditional logic on your custom step code 
without having to write any additional code.

```yaml
steps:
  - step1
  - name: mystep
    comment: run {pipeline dir}/mystep.py
             pass input context values to the step.
             run step 3 times in total for "first", "second", "third"
             retry twice if the step fails.
    foreach: [first, second, third]
    in:
      set: your own
      context: input here
      so: your step can use it
    retry:
      max: 2
  - step3

```