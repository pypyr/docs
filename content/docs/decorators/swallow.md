---
title: swallow decorator
linktitle: swallow
date: 2020-07-11T22:04:10+01:00
description: Swallow step error & continue with pipeline.
draft: false
card_extra_summary:
  heading: example
  details: |
          ```yaml
          swallow: True
          ```
card_extra_summary_is_code: True
# categories: [pipeline definition]
# keywords: ""
menu:
  docs:
    parent: decorators
    name: swallow
seo_article_headline: Ignore or swallow a task-runner pipeline error.
seo_description: Swallow or catch exception on any pipeline step. This allows the pipeline to recover from error conditions.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: swallow -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [error handling, pipeline format]
---
# swallow
## ignore pipeline step error
If `True`, catch any errors raised by the step and continue to the next step. 
pypyr logs the error both the output and to 
[runErrors]({{< ref "/docs/getting-started/error-handling" >}}) in context, so 
you'll know what happened, but processing continues. When you set `swallow` to 
`True`, pypyr will NOT go to the step-group's failure handler. 

You could think of this as `on error resume next` for your pipeline.

Defaults to `False`. This means by default pypyr will stop execution on a step
error and quit reporting failure.

```yaml
# ./swallow-decorator.yaml
steps:
  - name: pypyr.steps.assert
    comment: deliberately raise error
             step will swallow the error
             and proceed with pipeline.
    in:
      assert:
        this: False
    swallow: True
  - name: pypyr.steps.echo
    comment: this will run even though
             previous step raised exception
             because you swallowed the error.
    in:
      echoMe: You'll see me even though the previous step errored.
```

You can run this pipeline like this:

```text
$ pypyr swallow-decorator
pypyr.steps.assert Ignoring error because swallow is True for this step.
pypyr.errors.ContextError: assert False evaluated to False.
You'll see me even though the previous step errored.
```

## combining swallow & onError
You can add custom error information to a step when you `swallow` by using the
[onError]({{< ref "onerror">}}) decorator.

```yaml
# ./swallow-with-onerror.yaml
steps:
  - name: pypyr.steps.assert
    comment: deliberately raise error
             step will swallow the error
             and proceed with pipeline.
    in:
      assert:
        this: False
    onError: CUSTOM ERROR HERE
    swallow: True
  - name: pypyr.steps.echo
    comment: this will run even though
             previous step raised exception
             because you swallowed the error.
             the custom error is in runErrors.
    in:
        echoMe: "the error was: {runErrors[0][customError]}"
```

The output from this is:

```text
$ pypyr swallow-with-onerror
pypyr.steps.assert Ignoring error because swallow is True for this step.
AssertionError: assert False evaluated to False.
the error was: CUSTOM ERROR HERE
```