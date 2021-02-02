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
seo_description: Retry a pipeline step up to a configurable maximum retry count with a configurable sleep interval between retries.
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

The retry iteration counter is `retryCounter`. You can use this as 
usual for any context value in a formatting string expression as 
`{retryCounter}`.

These are all the available configuration parameters for retry:

```yaml
retry: # optional. Retry step until it doesn't raise an error.
  max: 1 # max times to retry. integer. Defaults None (infinite).
  sleep: 0 # sleep between retries, in seconds. Decimals allowed. Defaults 0.
  stopOn: ['ValueError', 'MyModule.SevereError'] # Stop retry on these errors. Defaults None (retry all).
  retryOn: ['TimeoutError'] # Only retry these errors. Defaults None (retry all).
  
  # back-off configuration
  backoff: fixed # optional. Default 'fixed'.
  backoffArgs: # optional. User defined args for custom back-off. Default None.
    arg1: value 1
    arg2: value 2
  sleepMax: 123 # optional. Max sleep in seconds. Default None (infinite).
  jrc: 123.45 # optional. Jitter Range Coefficient. Default 0.
```

All inputs support [substitutions]({{< ref "/docs/substitutions">}}), so you can
[assign values to these properties
dynamically](#common--dynamic-retry-configuration) at run-time.

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
      echoMe: BEGIN
  - name: pypyr.steps.assert
    comment: assert will fail until the 3rd retry.
             will sleep 0.5s between retries.
             on 3rd retry, step succeeds, and 
             pypyr proceeds to next step.
    retry:
      max: 4
      sleep: 0.5
    in:
      assert: !py retryCounter == 3
  - name: pypyr.steps.echo
    in:
      echoMe: END. you'll see me only after previous step succeeds.
```

```text
$ pypyr retry-sleep
BEGIN
retry: ignoring error because retryCounter < max.
AssertionError: assert retryCounter == 3 evaluated to False.
retry: ignoring error because retryCounter < max.
AssertionError: assert retryCounter == 3 evaluated to False.
END. you'll see me only after previous step succeeds.
```

You can also use a list of numbers for `sleep`, where each retry will sleep for
the duration in seconds of the next value from the list:
```yaml
- name: pypyr.steps.assert
  comment: assert will fail until the 3rd retry.
           will sleep 2s on 1st retry,
           sleep 4s after 2nd retry,
           on 3rd retry, step succeeds, and 
           pypyr proceeds to next step.
  retry:
    max: 4
    sleep: [2, 4, 6]
  in:
    assert: !py retryCounter == 3
```

If you want to vary the sleep interval between retries based on a formula you
can use different [retry backoff algorithms](#backoff-algorithms). The [list
sleep input option](#fixed-list-of-sleep-intervals) is available on the default
[fixed retry backoff](#fixed) strategy.

## infinite retry
Set `max` to `0` or `null` to retry indefinitely until the step succeeds.

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
      assert: False
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
AssertionError: assert False evaluated to False.
retry: ignoring error because retryCounter < max.
AssertionError: assert False evaluated to False
^C

$ echo $?
130
```

## retry entire sequence of steps
You can retry any step, whether it is your own or a built-in pypyr step. If you
retry on a [call]({{< ref "/docs/steps/call">}}) step, you retry an entire 
sequence of steps inside a step-group. This lets you retry multiple steps as a 
unit.

Similarly, if you invoke another pipeline from a parent pipeline with 
[pype]({{< ref "/docs/steps/pype">}}), you can retry the entire child pipeline
if it raises an error.

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
        echoMe: running retry {retryCounter}
    - name: pypyr.steps.assert
      in:
        assert: False
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
AssertionError: assert False evaluated to False.
running retry 2
Error while running step pypyr.steps.assert at pipeline yaml line: 20, col: 7
retry: ignoring error because retryCounter < max
AssertionError: assert False evaluated to False.
running retry 3
Error while running step pypyr.steps.assert at pipeline yaml line: 20, col: 7
Error while running step pypyr.steps.call at pipeline yaml line: 6, col: 5
Something went wrong. Will now try to run on_failure.

AssertionError: assert False evaluated to False.
```

## backoff algorithms
You can increase (or decrease) the waiting times between retries by using
different retry back-off strategies & apply a random jitter to the calculated
sleep interval to reduce contention.

```yaml
retry:
  max: 1 # max times to retry. integer. Defaults None (infinite).
  sleep: 0 # sleep between retries, in seconds. Decimals allowed. Defaults 0.
  
  # back-off configuration
  backoff: <<backoff-algorithm-name-here>>
  backoffArgs: # optional. User defined args for custom back-off. Default None.
    arg1: value 1
    arg2: value 2
  sleepMax: 123 # optional. Max sleep in seconds. Default None (infinite).
  jrc: 123.45 # optional. Jitter Range Coefficient. Default 0.
```

pypyr has the obvious common backoff retry strategies built in, but you can also
code your own [custom retry back-off algorithm]({{< ref
"/docs/api/retry-backoff" >}}) without too much fuss.

`n` is the iteration counter.

| backoff | formula |
| ------- | ------- |
| [fixed](#fixed) | `sleep` (single or list) |
| [jitter](#jitter) | `random_between(jrc*sleep, sleep)` |
| [linear](#linear) | `n*sleep` |
| [linearjitter](#linear-jitter) | `random_between(jrc*linear(n), linear(n)` |
| [exponential](#exponential) | `(base**n)*sleep` |
| [exponentialjitter](#exponential-jitter) | `random_between(jrc*exponential(n), exponential(n))` |

For all jitter strategies, the resulting sleep value is a random number between
the calculated interval, and the product of the calculated interval and the
`jrc` (jitter range coefficient). If you do not set `jrc`, by default it is 0.
This means that by default jitter algorithms will pick a random value anywhere
between 0 and the calculated interval. If you do not want your random range to
retry too soon by picking a number too close to 0, you can use the `jrc` to
control the lower bound - for example, given a 10 second calculated sleep
interval:
- `jrc` of 0 (the default) will randomize between 0 and 10.
- `jrc` of 0.2 will randomize between 2 and 10

If you want to restrict the growth of the sleep interval, set `sleepMax` to the
maximum sleep interval you want to allow in seconds. This is handy especially
for exponential growth, which could well get too large too soon for optimal
throughput. If you do not set `sleepMax`, by default the sleep interval will
grow infinitely. (okay, strictly speaking until Python runs out of numbers, but
if you're bumping into this limit you got other probs.)

All of the builtin backoff strategies restrict the sleep interval to less than
`sleepMax`, except in the case of the jitter style backoff strategies, where
`jrc > 1` can potentially result in an interval exceeding `sleepMax`. This is
because pypyr calculates jitter AFTER it applies the `sleepMax` limit to the
calculated value. This allows you to use the jrc to jitter the retry sleep
interval over a range higher or lower than the calculated value.

### fixed
The default backoff strategy uses fixed, or static, values. pypyr sleeps for a
fixed interval between retries with no calculation to increment or decrement
that interval.

You can use two different types of input for `sleep` with the `fixed` backoff
strategy:

#### single fixed interval
In this example pypyr will retry the command 3 times, each time sleeping for
the same fixed interval.
```yaml
- name: pypyr.steps.cmd
  retry:
    max: 3
    sleep: 1.5
  in:
    cmd: curl xyz://manifestly-wrong-url
```

- `sleep` is a single float number.
- Each retry iteration will sleep for this same number in seconds.

You do not need to set `backoff` if you want to use `fixed`, because it is the
default if you don't specify anything. You can if you want to, though:

```yaml
- name: pypyr.steps.cmd
  retry:
    backoff: fixed
    max: 3
    sleep: 1.5
  in:
    cmd: curl xyz://manifestly-wrong-url
```

#### fixed list of sleep intervals
In this example pypyr will retry the command 5 times, each time sleeping for
the next value in the list:

```yaml
- name: pypyr.steps.cmd
  retry:
    max: 5
    sleep: [0, 1, 1.5, 2, 3, 5]
  in:
    cmd: curl httpz://manifestly-wrong-url
```

- `sleep` is a list of floats/numbers.
- Each retry interval will sleep for the next number from the list
- When the input list runs out of numbers, pypyr will keep on re-using the last
  number on the list for subsequent iterations.


### jitter
Jitter adds randomization on top of [fixed](#fixed).

```yaml
- name: pypyr.steps.cmd
  comment: retry 3X, sleep on a random interval between 0-1.5s each try.
  retry:
    backoff: jitter
    max: 3
    sleep: 1.5
  in:
    cmd: curl xyz://manifestly-wrong-url
```

Since `jitter` extends `fixed`, you can also use the fixed list style input for
`sleep`:

```yaml
  - name: pypyr.steps.cmd
    comment: retry 7X, sleep on a random interval between 0-n each try,
             where n is the next number from the sleep list.
             n is, in turn - 2, 4, 6, 8, 8, 8
    retry:
      backoff: jitter
      max: 7
      sleep: [2, 4, 6, 8]
    in:
      cmd: curl xyz://manifestly-wrong-url
```

Be default jitter picks a random decimal number between 0 and the value for
`sleep` for that retry iteration. If you want to narrow this randomization
range, to prevent retries happening too close to each other if random ends up
giving a number very close to 0, you can use the Jitter Range Coefficient (jrc).

Simply put: pypyr will pause for a random number between `jrc*sleep` and
`sleep`.

```yaml
- name: pypyr.steps.cmd
  comment: retry 3X, sleep randomly between 2-10s on each retry.
  retry:
    backoff: jitter
    max: 3
    sleep: 10
    jrc: 0.2
  in:
    cmd: curl xyz://manifestly-wrong-url
```

When you use `jrc` with a list style input, the `jrc*sleep` applies to the sleep
interval for the current iteration:

```yaml
- name: pypyr.steps.cmd
  comment: retry 7X, sleep randomly between 0.1*sleep and sleep,
           where sleep is the next value from the list for each retry.
  retry:
    backoff: jitter
    max: 7
    sleep: [2, 4, 6, 8]
    jrc: 0.5
  in:
    cmd: curl httpz://manifestly-wrong-url
```

In this case the sleep interval (in seconds) for each retry in turn jitters
between:
- retry #1: 1 -> 2
- retry #2: 2 -> 4
- retry #3: 3 -> 6
- retry #4: 4 -> 8
- retry #5: 4 -> 8
- retry #6: 4 -> 8

Notice that when the list runs out of numbers, pypyr re-uses the last number
in the list for the subsequent sleep intervals.

### linear
The linear backoff algorithm sleeps for `sleep * n` in between retries, where 
`n` is the iteration counter.

```yaml
- name: pypyr.steps.cmd
  comment: retry 5X, at interval (sleep * n)
           retry wait time is, in turn - 2, 4, 6, 8.
  retry:
    backoff: linear
    max: 5
    sleep: 2
  in:
    cmd: curl httpz://manifestly-wrong-url
```

#### restrict linear backoff growth
You can restrict the growth of the effective sleep interval with `sleepMax`:

```yaml
- name: pypyr.steps.cmd
  comment: retry 5X, with interval between retries = (sleep * n)
           retry wait time is, in turn - 3, 6, 8.5, 8.5
  retry:
    backoff: linear
    max: 5
    sleep: 3
    sleepMax: 8.5
  in:
    cmd: curl httpz://manifestly-wrong-url
```

### linear jitter
Jitter on top of the [linear](#linear) backoff algorithm. Each retry pauses for
a random interval between `jrc*linear(n)` and `linear(n)`, where `n` is the
iteration counter.

```yaml
- name: pypyr.steps.cmd
  comment: retry 5X, sleep randomly between 0 and (sleep * n) on each retry
  retry:
    backoff: linearjitter
    max: 5
    sleep: 4
  in:
    cmd: curl httpz://manifestly-wrong-url
```

In this case the sleep interval (in seconds) for each retry in turn jitters
between:
- retry #1: 0 -> 4
- retry #2: 0 -> 8
- retry #3: 0 -> 12
- retry #4: 0 -> 16

You can combine `linearjitter` with `sleepMax` to restrict the growth of the
effective sleep interval. Use `jrc` to change the jitter range. You can use both
`sleepMax` and `jrc`, just one or the other, or neither.

```yaml
- name: pypyr.steps.cmd
  comment: retry 6X, sleep randomly between 0 and (sleep * n) on each retry.
           do not go beyond 14s sleep between retries.
           each sleep interval jitters between (0.5*(5*n)) and (5*n).
  retry:
    backoff: linearjitter
    max: 6
    sleep: 5
    sleepMax: 14
    jrc: 0.5
  in:
    cmd: curl httpz://manifestly-wrong-url
```

In this case the sleep interval (in seconds) for each retry in turn jitters
between:
- retry #1: 2.5 -> 5
- retry #2: 5 -> 10
- retry #3: 7 -> 14
- retry #4: 7 -> 14
- retry #5: 7 -> 14

### exponential
Retry with an exponential backoff on the sleep coefficient, increasing the wait
time between retries exponentially as given by the formula `(base**n)*sleep`,
where `n` is the iteration counter. 

By default the exponential base is 2.

```yaml
- name: pypyr.steps.cmd
  comment: retry 5X, with interval between retries growing base 2 exponential
            sleep in seconds between retries, in turn, is - 2, 4, 8, 16. 
  retry:
    backoff: exponential
    max: 5
    sleep: 1
  in:
    cmd: curl httpz://manifestly-wrong-url
```

#### restrict exponential backoff growth
Use `sleepMax` to restrict the effective maximum retry sleep interval. This lets
you configure your retries to achieve optimal throughput, because often a pure
exponential will result in too long backoff intervals too soon.

```yaml
- name: pypyr.steps.cmd
  comment: retry 5X, with interval between retries (2**n)*2.5).
            sleep in seconds between retries, in turn, is - 5, 10, 20, 21.
            do not allow sleep more than 21s.
  retry:
    backoff: exponential
    max: 5
    sleep: 2.5
    sleepMax: 21
  in:
    cmd: curl httpz://manifestly-wrong-url
```

#### change exponent base
By default the exponent base is 2. You can change the base for exponentiation
using the `base` input argument:

```yaml
- name: pypyr.steps.cmd
  comment: retry 5X, with interval between retries (3**n)*sleep).
            do not allow sleep more than 41s.
            sleep in seconds between retries, in turn, is - 3, 9, 27, 41.
  retry:
    backoff: exponential
    max: 5
    sleep: 1
    sleepMax: 41
    backoffArgs:
      base: 3
  in:
    cmd: curl httpz://manifestly-wrong-url
```

### exponential jitter
Use the ready-made exponential backoff with randomized jitter on any retrying
command or step without having to write code.

This adds jitter on top of the [exponential](#exponential) backoff strategy, so
all the inputs to the underlying `exponential` such as `sleepMax` and `base` are
available to `exponentialjitter` also.

For each retry iteration, pypyr will, in order:
- calculate the `[raw exponential result]` with formula `(base**n)*sleep`, where
  `n` is the iteration counter
- proceed with the lesser of `sleepMax` or the `[raw exponential result]`. Call
  this the `[capped result]`
- jitter by picking a random interval between `jrc*[capped result]` and the
  `[capped result]`. 

```yaml
- name: pypyr.steps.cmd
  comment: retry 5X.
           sleep randomly between 0 and ((2**n)*sleep)) on each retry.
  retry:
    backoff: exponentialjitter
    max: 5
    sleep: 1
  in:
    cmd: curl httpz://manifestly-wrong-url
```

You can use the Jitter Range Coefficient `jrc` to narrow the randomization range
and prevent intervals too close to 0:

```yaml
- name: pypyr.steps.cmd
  comment: retry 5X.
            sleep randomly between 0 and (2**n)*sleep) on each retry.
  retry:
    backoff: exponentialjitter
    max: 5
    sleep: 2
    jrc: 0.5
  in:
    cmd: curl httpz://manifestly-wrong-url
```

In this case the sleep interval (in seconds) for each retry in turn jitters
between:
- retry #1: 2 -> 4
- retry #2: 4 -> 8
- retry #3: 8 -> 16
- retry #4: 16 -> 32

Use a `jrc` greater than `1` to jitter over a range higher than the capped
result.

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
    description: this step will retry infinitely until the cmd does not return 
                 an error unless an unexpected error that is NOT in retryOn 
                 occurs.
    comment: retries 1 & 2 raise ValueError, 
             retry 3 raise KeyError.
             KeyError is not in retryOn, so will quit step reporting failure.
    retry:
      # will ONLY retry these errors. All other stop processing.
      retryOn: ['KeyError', 'pypyr.errors.ContextError']
    in:
      py: |
        if retryCounter == 3:
          raise ValueError("this won't retry!")
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
# pypyr retry-stopon
steps:
  - name: pypyr.steps.py
    description: this step will retry infinitely until the cmd does not return 
                 an error or until a StopOn error happens.
    retry:
      # will stop retry on these errors. All others will carry on retry processing.
      stopOn: ['ValueError', 'pypyr.errors.ContextError']
    in:
      py: |
        if retryCounter == 3:
          raise ValueError('this will STOP processing!')
        else:
          raise KeyError('this will retry')
  - name: pypyr.steps.echo
    in:
      echoMe: this won't run because the previous step always errors.
```

This will result in:

```text
$ pypyr retry-stopon
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

## common & dynamic retry configuration
You can use the same retry configuration for multiple steps, or set retry
configuration dynamically based on run-time values, or do both at the same time.

### format expressions
Use pypyr [format substitutions]({{< ref "/docs/substitutions">}}) to assign
values to retry dynamically based on run-time parameters:

```yaml
- name: pypyr.steps.contextsetf
  in:
    contextSetf:
      my_sleep: [2, 4, 8]
      my_max: 3
      my_sleep_max: 6

- name: pypyr.steps.assert
  retry:
    sleep: '{my_sleep}'
    max: '{my_max}'
    sleepMax: '{my_sleep_max}'
  in:
    assert: !py retryCounter == 3
```

### yaml anchors & references
You can create a yaml anchor with an `&` for a retry strategy and then re-use
that in multiple steps with a yaml reference with a `*`:

In this example, both the steps will use the same retry configuration as given
in `&commonRetryWithFixedList`.

```yaml
common:
  retry1: &commonRetryWithFixedList
    sleep: [2, 4, 8]
    max: 3
    sleepMax: 6

steps:
  - name: pypyr.steps.assert
    retry: *commonRetryWithFixedList
    in:
      assert: !py retryCounter == 3

  - name: pypyr.steps.assert
    retry: *commonRetryWithFixedList
    in:
      assert: !py retryCounter == 2
```

You can create and reference multiple different retry strategies in the same
pipeline this way.

### combine yaml anchors & format expressions
You can share common retry configuration and still use format substitution
expressions to configure those values dynamically at run-time.

In this example, the first `contextsetf` step sets the values that the shared
retry configuration under `&commonRetryWithSubstitutions` will substitute at
runtime:

```yaml
common:
  retry1: &commonRetryFixedList
    sleep: [2, 4, 8]
    max: 3
    sleepMax: 6

  retry2: &commonRetryWithSubstitutions
    sleep: '{base3_exponential_retry[sleep]}'
    max: '{base3_exponential_retry[max]}'
    sleepMax: '{base3_exponential_retry[sleepMax]}'
    jrc: '{base3_exponential_retry[jrc]}'
    backoff: '{base3_exponential_retry[backoff]}'
    backoffArgs:
      base: '{base3_exponential_retry[base]}'

steps:
  - name: pypyr.steps.contextsetf
    in:
      contextSetf:
        base3_exponential_retry:
          sleep: 1
          max: 3
          sleepMax: 8.5
          jrc: 0.5
          backoff: exponentialjitter
          base: 3

  - name: pypyr.steps.assert
    retry: *commonRetryWithSubstitutions
    in:
      assert: !py retryCounter == 3

  - name: pypyr.steps.contextmerge
    comment: change sleep for next retry.
             all other retry values stay the same.
    in:
      contextMerge:
        base3_exponential_retry:
          sleep: 2

  - name: pypyr.steps.assert
    retry: *commonRetryWithSubstitutions
    in:
      assert: !py retryCounter == 2
```

If you change any of the values under `base3_exponential_retry` subsequent
retries will use the new updated values, so you can parameterize the same base
retry configuration with different values for different steps in the same
pipeline.