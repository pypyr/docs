---
title: pypyr.steps.python
linktitle: python
date: 2022-02-19T16:04:49Z
description: Get the absolute path of the current Python executable.
draft: false
card_extra_summary:
  heading: input context property
  # details: "`python` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: python
seo_article_headline: Get the absolute path of the current Python interpreter.
seo_description: Get the full path of the executable binary of the Python interpreter running the current pypyr session.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: python -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [execute external program]
---
# pypyr.steps.python
Get the full path of the executable binary of the Python interpreter running the
current pypyr session.

The step writes the absolute path as a string to the `python` key in context.

This means that wherever you would've invoked `python` or `python3` as a
subprocess, you can instead use `{python}`. This will replace the `{python}`
token with the absolute path of the Python executable that is currently running
pypyr.

This step doesn't take any inputs, so you can always run it as a simple step.

```yaml
steps:
  - pypyr.steps.python

  - name: pypyr.steps.cmd
    comment: build wheel + sdist 
    description: --> build wheel + sdist to dist/
    in:
      cmd: '"{python}" setup.py bdist_wheel sdist'  
```

Note the double quote around `"{python}"`. This is to make sure that if there
are any spaces in the file-path given by `{python}` it still parses correctly
when passed as a cmd to open as a sub-process.

But why? Especially on Windows, if you just invoke Python as a sub-process from
pypyr like this:

```yaml
steps:
  name: pypyr.steps.cmd
  in:
    cmd: python -m stuff.do
```

You might run into trouble because `python` will resolve to the system Python
interpreter, rather than the currently active virtual environment. Therefore
any dependencies you've installed in the virtual environment WON'T necessarily
be available in the system site packages.

This is unlikely to be the behavior that you want if you're running inside an
active venv.

To avoid this issue, use `pypyr.steps.python` and compose your command with
`"{python}"` rather than `python`:

```yaml
steps:
  - pypyr.steps.python
  - name: pypyr.steps.cmd
    comment: will run in currently active python env
    in:
      cmd: '"{python}" -m stuff.do'  
```

See the [Python docs for
Popen](https://docs.python.org/3/library/subprocess.html#popen-constructor) for
details.