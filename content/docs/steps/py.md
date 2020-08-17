---
title: pypyr.steps.py
linktitle: py
date: 2020-07-06T13:17:22+01:00
description: Execute inline python code.
card_extra_summary:
  heading: input context property
  details: "`pycode` (string)"
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
Executes the context value `pycode` as python code.

Will execute `context['pycode']` as a dynamically interpreted python code
block.

For example, this will invoke python print and print 2:

```yaml
steps:
  - name: pypyr.steps.py
    comment: Example of arb python code. Will print 2.
    in:
      pycode: print(1+1)
```

## working with context
You can access and change the context dictionary in a py step. Changes you make
to context will endure after the `py` step completes.

When you are in a `py` step, context exists as a dictionary-like object named
`context`. All the usual python `dict` methods are available.

```yaml
- name: pypyr.steps.py
  comment: set context value
  in:
    pycode: context['arbkey'] = max(420, 69)
- name: pypyr.steps.py
  comment: get context value
  in:
    pycode: print(context['arbkey'])
```

## multi-line python code block
You can run multiple python statements in the same code block:
```yaml
- name: pypyr.steps.py
  comment: multi-line statement starts with |, per yaml spec
  in:
    pycode: |
              print(f"py step: {0+1}")
              context['arbvalue'] = 420
              print(context['arbvalue'])
- name: pypyr.steps.py
  comment: here splitting multi-line statements with ; 
           context['arbvalue'] survives between steps.
  in:
    pycode: print("py step 2"); context['arbvalue'] += 4
```

{{% note tip %}}
Do remember that if your inline code block gets unwieldy, you can very easily 
run your own Python .py file from pypyr by using it as a 
[custom step]({{< ref "/docs/api/step">}}) in itself.

You don't even have to package your python code - pypyr will resolve the path 
relative to the working directory for you.

```yaml
steps:
  - my-python-file # runs ./my-python-file.py
```
{{% /note %}}

## example
See a worked [example of inline python](https://github.com/pypyr/pypyr-example/tree/master/pipelines/py.yaml).