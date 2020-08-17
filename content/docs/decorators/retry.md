---
title: retry decorator
linktitle: retry
date: 2020-07-13T13:37:06+01:00
description: Retry step until it succeeds.
card_extra_summary:
  heading: example
  details: |
            ```yaml
            retry: 
              max: 1 
              sleep: 2
              stopOn: ['ValueError', 'MyModule.SevereError']
              retryOn: ['TimeoutError']
            ```
card_extra_summary_is_code: True
# categories: [pipeline definition]
# keywords: ""
menu:
  docs:
    parent: decorators
    name: retry
seo_article_headline: Automatically retry a task-runner pipeline step.
seo_description: Retry a pipeline step to a configurable maximum retry count with a configurable sleep interval between retries.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: retry -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [error handling, loops, pipeline format]
---
# retry
## automatic retries
Retries the step until it succeeds. If you do not set `retry`, pypyr will not
retry the step automatically. When you do set `retry`, pypyr will retry whatever
step it is without you having to do anything else.

The retry iteration counter is `context['retryCounter']`. You can use this as 
usual for any context value in a formatting string expression as 
`{retryCounter}`.

These are all the available configuration parameters for retry:

```yaml
retry: # optional. Retry step until it doesn't raise an error.
  max: 1 # max times to retry. integer. Defaults None (infinite).
  sleep: 0 # sleep between retries, in seconds. Decimals allowed. Defaults 0.
  stopOn: ['ValueError', 'MyModule.SevereError'] # Stop retry on these errors. Defaults None (retry all).
  retryOn: ['TimeoutError'] # Only retry these errors. Defaults None (retry all).
```

All inputs support [substitutions]({{< ref "/docs/substitutions">}}), so you 
can assign values to these properties dynamically at run-time.

If you reach `max` retry count while the step still errors, pypyr will raise 
the last error and stop further pipeline processing.

If you set [swallow]({{< ref "swallow" >}}) to `True`, pypyr will swallow the 
error after max retries exhausts.

## retry with sleep
Use `sleep` to pause in between retries. Sleep is a decimal with the unit 
seconds. If you do not set `sleep`, the default wait between retries is 0s.


```yaml
# ./retry-sleep.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin
  - name: pypyr.steps.assert
    comment: assert will fail until the 3rd retry.
             will sleep 0.5s between retries.
             on 3rd retry, step succeeds, and 
             pypyr proceeds to next step.
    retry:
      max: 4
      sleep: 0.5
    in:
      assert:
        this: !py retryCounter == 3
  - name: pypyr.steps.echo
    in:
      echoMe: end. you'll see me only after previous step succeeds.
```

```text
$ pypyr retry-sleep
begin
retry: ignoring error because retryCounter < max.
ContextError: assert retryCounter == 3 evaluated to False.
retry: ignoring error because retryCounter < max.
ContextError: assert retryCounter == 3 evaluated to False.
end. you'll see me only after previous step succeeds.
```

## infinite retry
Set `max` to `0` or `null` for to retry indefinitely until the step succeeds.

It's up to you to interrupt an infinite loop. If you're using the cli, you can 
hit CTRL+C to interrupt and exit pypyr.

```yaml
# ./retry-infinite.yaml
steps:
  - name: pypyr.steps.assert
    description: CTRL+C to exit
    comment: 0 or null means infinite retry.
             since sleep not set, will retry
             immediately with no delay between
             retries.
    in:
      assert:
          this: False
    retry:
      max: 0 # 0 or null means indefinitely
  - name: pypyr.steps.echo
    in:
      echoMe: this won't run because the previous step always errors.
```

```text
$ pypyr retry-infinite
pypyr.steps.assert: CTRL+C to exit
retry: ignoring error because retryCounter < max.
ContextError: assert False evaluated to False.
retry: ignoring error because retryCounter < max.
ContextError: assert False evaluated to False
^C

$ echo $?
130
```

## retry entire sequence of steps
You can retry any step, whether it is your own or a built-in pypyr step. If you
retry on a [call]({{< ref "/docs/steps/call">}}) step, you retry an entire 
sequence of steps inside a step-group. This lets you retry multiple steps as a 
unit.

```yaml
# ./retry-call
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin
  - name: pypyr.steps.call
    comment: all of retry_me will retry 3X if it returns an error.
    retry:
      max: 3
    in:
      call:
        groups: retry_me
  - name: pypyr.steps.echo
    in:
      echoMe: you won't see me, because the previous step always errs.

retry_me:
    - name: pypyr.steps.echo
      in:
        echoMe: "running retry {retryCounter}"
    - name: pypyr.steps.assert
      in:
        assert:
          this: False
    - name: pypyr.steps.echo
      in:
        echoMe: you won't see me, because the previous step always errs.

```

This will result in:

```text
$ pypyr retry-call
begin
running retry 1
Error while running step pypyr.steps.assert at pipeline yaml line: 20, col: 7
retry: ignoring error because retryCounter < max
ContextError: assert False evaluated to False.
running retry 2
Error while running step pypyr.steps.assert at pipeline yaml line: 20, col: 7
retry: ignoring error because retryCounter < max
ContextError: assert False evaluated to False.
running retry 3
Error while running step pypyr.steps.assert at pipeline yaml line: 20, col: 7
Error while running step pypyr.steps.call at pipeline yaml line: 6, col: 5
Something went wrong. Will now try to run on_failure.

ContextError: assert False evaluated to False.
```

## only retry specific error types
When you set neither `stopOn` nor `retryOn`, all types of errors will retry. 
This is the default behavior.

If you specify `stopOn`, errors listed in `stopOn` will stop retry processing 
and raise an error. Errors not listed in `stopOn` will retry.

If you specify `retryOn`, ONLY errors listed in `retryOn` will retry.

`max` evaluates before `stopOn` and `retryOn`. 

`stopOn` supersedes `retryOn`.

For builtin python errors, specify the bare error name for `stopOn` and 
`retryOn`, e.g `ValueError`, `KeyError`.

For all other errors, use `module.errorname`, e.g `mypackage.mymodule.myerror`

### only retry on specific errors
```yaml
# ./retry-retryon.yaml
steps:
  - name: pypyr.steps.py
    description: this step will retry infinitely until the cmd does not return an error.
    comment: retries 1 & 2 raise ValueError, 
              retry 3 raise KeyError.
              KeyError is not in retryOn, so will quit step reporting failure.
    retry:
      # will ONLY retry these errors. All other stop processing.
      retryOn: ['KeyError', 'pypyr.errors.ContextError']
    in:
      pycode: |
              if context['retryCounter'] == 3:
                raise ValueError('this won't retry!')
              else:
                raise KeyError('this will retry')
  - name: pypyr.steps.echo
    in:
      echoMe: this won't run because the previous step always errors.
```

This will result in:

```text
 $ pypyr retry-retryon
this step will retry infinitely until the cmd does not return an error.
retry: ignoring error because retryCounter < max.
KeyError: 'this will retry'
retry: ignoring error because retryCounter < max.
KeyError: 'this will retry'
ValueError not in retryOn. Raising error and exiting retry.
Error while running step pypyr.steps.py at pipeline yaml line: 4, col: 7
Something went wrong. Will now try to run on_failure.

ValueError: Booom!

$ echo $?
255
```

### stop processing on specific error type
```yaml
# ./retry-retrystopon
steps:
  - name: pypyr.steps.py
    description: this step will retry infinitely until the cmd does not return an error.
    retry:
      # will stop retry on these errors. All others will carry on retry processing.
      stopOn: ['ValueError', 'pypyr.errors.ContextError']
    in:
      pycode: |
              if context['retryCounter'] == 3:
                raise ValueError('this will STOP processing!')
              else:
                raise KeyError('this will retry')
  - name: pypyr.steps.echo
    in:
      echoMe: this won't run because the previous step always errors.
```

This will result in:

```text
$ pypyr retry-retrystopon
this step will retry infinitely until the cmd does not return an error.
retry: ignoring error because retryCounter < max.
KeyError: 'this will retry'
retry: ignoring error because retryCounter < max.
KeyError: 'this will retry'
ValueError in stopOn. Raising error and exiting retry.
Error while running step pypyr.steps.py at pipeline yaml line: 4, col: 7
Something went wrong. Will now try to run on_failure.

ValueError: this will STOP processing!
```