---
title: onError decorator
linktitle: onError
date: 2020-07-11T21:42:31+01:00
description: Add custom error data on step error.
draft: false
card_extra_summary:
  heading: example
  details: |
            ```yaml
            onError:
              code: 123
              description: arb text here
            ```
card_extra_summary_is_code: True
# categories: [pipeline definition]
# keywords: ""
menu:
  docs:
    parent: decorators
    name: onError
seo_article_headline: Create custom exception object on task-runner pipeline failure.
seo_description: Add extra information to error if a pipeline step raises an exception.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: onerror -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [error handling, pipeline format]
---
# onError
## add custom data to exception
Provide custom error information if the step raises an exception. This lets you
add extra information to the error itself.

If this step errors, write the contents of `onError` to 
`runErrors[n].customError` in context. Steps inside a 
[failure handler]({{< ref "/docs/pipelines/pipeline-structure#on_failure">}}) 
then can use this information. Alternatively, subsequent steps can also use this 
information, assuming you've got a [swallow]({{< ref "swallow#combining-swallow--onerror" >}}) somewhere 
in the call chain.

`onError` can be a simple string, or your your own dict, or any given object. 

You can use [substitutions]({{< ref "/docs/substitutions">}}) to assign values 
dynamically at run-time.

## simple string extra error data
You can add a simple string for extra error information:

```yaml
# ./onerror-decorator.yaml
steps:
  - name: pypyr.steps.assert
    comment: deliberately raise error
             with custom error info
    in:
      assert:
        this: False
    onError: this is a custom error
on_failure:
  - name: pypyr.steps.echo
    comment: the custom error is in runErrors
             it's the 1st and only error in
             this pipeline, hence index 0.
    in:
      echoMe: "the error was: {runErrors[0][customError]}"
```

When you run this pipeline, this is the result:

```text
$ pypyr onerror-decorator
Error while running step pypyr.steps.assert at pipeline yaml line: 3, col: 5
Something went wrong. Will now try to run on_failure.
the error was: this is a custom error

AssertionError: assert False evaluated to False.
```

## create custom exception object
`onError` will take any given type. You can add your own custom object here.

In this pipeline, add an error code and a description to the custom exception
object:

```yaml
# ./onerror-complex-obj.yaml
steps:
  - name: pypyr.steps.contextsetf
    comment: arbitrarily set a context value.
             will be using this value in error
             object.
    in:
      contextSetf: 
        arbKey: FROM SUBSTITUTION EXPRESSION
  - name: pypyr.steps.assert
    comment: deliberately raise error
             with custom error object
    in:
      assert:
        this: False
    onError:
      myerr_code: 123
      myerr_description: "my err {arbKey} description"
on_failure:
  - name: pypyr.steps.echo
    comment: the custom error is in runErrors
             notice your custom error object
             is here with substitutions applied.
    in:
      echoMe: |
               the error code: {runErrors[0][customError][myerr_code]}
               the error description: {runErrors[0][customError][myerr_description]}
```

When you run this pipeline, this is the result:

```text
$ pypyr onerror-complex-obj
Error while running step pypyr.steps.assert at pipeline yaml line: 10, col: 5
Something went wrong. Will now try to run on_failure.
the error code: 123
the error description: my err FROM SUBSTITUTION EXPRESSION description

AssertionError: assert False evaluated to False.
```