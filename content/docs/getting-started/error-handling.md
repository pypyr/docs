---
title: error handling
description: How to handle errors in your pipelines.
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
# categories: [install]
menu:
  docs:
    parent: getting-started
seo_article_headline: Overview of error handling in a task-runner pipeline.
seo_description: Retry, swallow or ignore errors in a pipeline. Add extra diagnostic info to exceptions.
topics: [error handling]
topics_weight: -100
---
# error handling
## stop all processing on error
pypyr runs pipelines. . . and a pipeline is a sequence of steps. By default 
subsequent steps in the sequence should not run if a previous step failed. 

If your desired behavior is for pipeline processing to stop and subsequent steps 
NOT to run once an error occurs somewhere, you don't have to do anything 
special, because this is what pypyr does by default.

```yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: A
  - name: pypyr.steps.py
    comment: deliberately raise arbitrary error
    in:
      py: raise ValueError('arb')
  - name: pypyr.steps.echo
    comment: this step will never run,
             because the previous step
             always fails.
    in:
      echoMe: unreachable
```

## ignore error on a step
You can ignore an error on a specific step by setting the 
[swallow step decorator]({{< ref "/docs/decorators/swallow" >}}) to `True`.

```yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: A
  - name: pypyr.steps.py
    swallow: True
    in:
      py: raise ValueError('arb')
  - name: pypyr.steps.echo
    in:
      echoMe: You'll see me, because you told pypyr to swallow the error in the previous step.
```

## failure handlers
A failure handler is an optional step-group that pypyr jumps to when an error 
happens. In traditional programming terms, it's pretty much a `catch` block. 
Once the failure handler completes, the pipeline exits reporting failure.

Any given step-group can be a failure handler. If you don't explicitly specify 
a failure handler pypyr will look for a step group named `on_failure`. If 
`on_failure` doesn't exist it doesn't matter, pypyr will still quit the 
pipeline reporting the original error.

```yaml
# ./failure-handler-example.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: A
  - name: pypyr.steps.py
    in:
      py: raise ValueError('arb')
  - name: pypyr.steps.echo
    comment: this step will never run,
            because the previous step
            always fails.
    in:
      echoMe: unreachable

on_failure:
  - name: pypyr.steps.echo
    in:
      echoMe: B
```

Running this pipeline will result in:

```text
$ pypyr failure-handler-example
A
Error while running step pypyr.steps.py at pipeline yaml line: 6, col: 5
Something went wrong. Will now try to run on_failure.
B

ValueError: arb

$ echo $?
255
```

If a failure handler encounters another exception while processing an 
exception, then pypyr will log and report both that exception and the original 
cause's exception, although the original cause exception is what pypyr 
highlights on CLI exit and raises to API consumers.

### set your own failure handler
You can set any given step-group to be a failure handler. 

Similarly to if `on_failure` does not exist, pypyr will not raise an additional 
error if the specified custom failure-handler doesn't exist. The log output 
will tell you that it did look for it but couldn't find it.

```yaml
# ./my-pipeline.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: A
  - name: pypyr.steps.py
    in:
      py: raise ValueError('arb')
  - name: pypyr.steps.echo
    in:
      echoMe: unreachable

arb_group:
  - name: pypyr.steps.echo
    in:
      echoMe: B
```

#### from the cli
When you invoke pypyr from the cli and you don't want to use the default 
`on_failure` failure handler you can use the `--failure` input argument:

```fish
$ pypyr my-pipeline --failure arb_group 
```

#### when calling a pipeline
When you call a pipeline from within another pipeline using  
[pype]({{< ref "/docs/steps/pype">}}) you can override the default `on_failure` 
by setting your own:

```yaml
- name: pypyr.steps.pype
  in:
    pype:
      name: my-pipeline
      failure: arb_group
```

#### within a pipeline
You can specify your own failure-handler when you 
[call]({{< ref "/docs/steps/call">}}) or [jump]({{< ref "/docs/steps/jump">}}) 
to another step-group in your pipeline.

```yaml
# ./call-with-failure-group.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: A
  - name: pypyr.steps.call
    in:
      call:
        groups: call_me
        failure: arb_group
  - name: pypyr.steps.echo
    in:
      echoMe: unreachable

call_me:
  - name: pypyr.steps.echo
    in:
      echoMe: B
  - name: pypyr.steps.assert
    in:
      assert: False
  - name: pypyr.steps.echo
    in:
      echoMe: unreachable

arb_group:
  - name: pypyr.steps.echo
    in:
      echoMe: C
```

Notice that when you run this pipeline, when the `arb_group` failure handler 
completes reporting failure, pypyr still falls back to processing the actual 
pipeline's handler, if it exists. In this case, because we don't invoke the 
pipeline with a custom failure handler set, it defaults to looking for 
`on_failure`.

This is the same concept as an error bubbling up to the top of the stack.

```text
$ pypyr call-with-failure-group
A
B
Error while running step pypyr.steps.assert at pipeline yaml line: 19, col: 5
Something went wrong. Will now try to run arb_group.
C
Error while running step pypyr.steps.call at pipeline yaml line: 6, col: 5
Something went wrong. Will now try to run on_failure.

AssertionError: assert False evaluated to False.
$
```

### call or jump in failure handler
You can [call]({{< ref "/docs/steps/call">}}) or 
[jump]({{< ref "/docs/steps/jump">}}) with a failure handler. This is handy if 
you have shared or common code you want to run on different error and success 
conditions, because you can encapsulate your failure handling code in its own 
step-group. A very typical scenario for this is if you want to send a 
notification on both success and failure.

```yaml
# ./call-on-failure.yaml
steps:
  - name: pypyr.steps.assert
    in:
      assert: False
  - name: pypyr.steps.echo
    in:
      echoMe: unreachable

sg1:
  - name: pypyr.steps.echo
    in:
      echoMe: B
  - name: pypyr.steps.call
    in:
      call: sg2
  - name: pypyr.steps.echo
    in:
      echoMe: D

sg2:
  - name: pypyr.steps.echo
    in:
      echoMe: C

on_failure:
  - name: pypyr.steps.echo
    in:
      echoMe: A
  - name: pypyr.steps.call
    in:
      call: sg1
  - name: pypyr.steps.echo
    in:
      echoMe: E
```

Running this pipeline results in:

```text
$ pypyr call-on-failure
Error while running step pypyr.steps.assert at pipeline yaml line: 3, col: 5
Something went wrong. Will now try to run on_failure.
A
B
C
D
E

AssertionError: assert False evaluated to False.
```

Notice that the called group can also call another group. 

{{% note tip %}}
If you `jump` from a failure handler, you're still within the execution context 
of the failure handler. This means that once the group you jump to completes, 
pypyr will still quit reporting failure, unless you use one of the Stop 
instructions somewhere in the execution chain.
{{% /note %}}

## don't quit pipeline reporting failure
Once a pipeline's failure handler completes, by default pypyr quits the 
pipeline reporting failure. If you want to handle the error and quit the 
pipeline reporting success, you can use a Stop instruction to prevent the 
failure handler from completing and raising the error.

Depending on whether you want to stop pypyr entirely, or only the pipeline, or 
only the currently called or jumped to group, you can use any of
[stop]({{< ref "/docs/steps/stop">}}), 
[stoppipeline]({{< ref "/docs/steps/stoppipeline">}}) or 
[stopstepgroup]({{< ref "/docs/steps/stopstepgroup">}}).

```yaml
# ./stop-on-failure.yaml
steps:
  - name: pypyr.steps.assert
    in:
      assert: False
  - name: pypyr.steps.echo
    in:
      echoMe: unreachable

on_failure:
  - name: pypyr.steps.echo
    in:
      echoMe: A
  - pypyr.steps.stop
  - name: pypyr.steps.echo
    in:
      echoMe: unreachable

```

When you run this pipeline pypyr will quit reporting success because the 
failure handler considers the error condition as handled when you issue the 
stop instruction:

```text
$ pypyr stop-on-failure
Error while running step pypyr.steps.assert at pipeline yaml line: 3, col: 5
Something went wrong. Will now try to run on_failure.
A

$ echo $?
0
```

If you use `call` or `jump` inside your failure handler, you can issue the 
appropriate Stop instruction in the groups that you invoke.

## use runtime error details inside pipeline
pypyr saves all run-time errors to a list in context called `runErrors`.

```yaml
runErrors:
  - name: Error Name Here
    description: Error Description Here
    customError: # whatever you put into onError on step definition
    line: 1 # line in pipeline yaml for failing step
    col: 1 # column in pipeline yaml for failing step
    step: my.bad.step.name # failing step name
    exception: ValueError('arb') # the actual python error object
    swallowed: False # True if err was swallowed
```

The last error will be the last item in the list. The first error will be the 
first item in the list.

### using error information in subsequent steps
You can use `runErrors` in your step-group's 
[failure handler]({{< ref "#failure-handlers" >}}), 
or if you set `swallow=True` on the failing step you can use `runErrors` in 
subsequent steps to use the actual error information.

```yaml
# ./use-error-info
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: A
  - name: pypyr.steps.assert
    swallow: True
    in:
      assert: False
  - name: pypyr.steps.echo
    in:
      echoMe: there was a problem on line {runErrors[0][line]}
  - name: pypyr.steps.py
    in:
      py: raise ValueError('arb')
  - name: pypyr.steps.echo
    in:
      echoMe: unreachable

on_failure:
  - name: pypyr.steps.echo
    in:
      echoMe: B
  - name: pypyr.steps.assert
    in:
      assert:
        this: '{runErrors[0][name]}'
        equals: AssertionError
  - name: pypyr.steps.assert
    in:
      assert:
        this: '{runErrors[1][name]}'
        equals: ValueError
  - name: pypyr.steps.assert
    in:
      assert:
        this: '{runErrors[1][description]}'
        equals: arb
```

Notice this pipeline uses `runErrors` information both in the `steps` group 
after the `swallow`, and also in the `on_failure` handler. The asserts in the 
failure handler effectively tests if the errors are as expected.

```text
$ pypyr use-error-info
A
pypyr.steps.assert Ignoring error because swallow is True for this step.
AssertionError: assert False evaluated to False.
there was a problem on line 6
Error while running step pypyr.steps.py at pipeline yaml line: 14, col: 5
Something went wrong. Will now try to run on_failure.
B

ValueError: arb
```

## add extra error information
You can provide your own additional description or even your own object to add 
to the error by using the 
[onError decorator]({{< ref "/docs/decorators/onerror" >}}).

```yaml
- name: pypyr.steps.assert
  comment: deliberately raise error
           with custom error object
  in:
    assert: False
  onError:
    myerr_code: 123
    myerr_description: "my err description"
```

This information is available in the `runErrors` list in context.

## raise a custom error
You can raise your own errors in pypyr. At easiest, use the built-in assert, 
but you can also raise any Python built-in error.

### assertion error
You can use [assert]({{< ref "/docs/steps/assert">}}) to raise an error if the 
assertion condition fails. 

```yaml
steps:
  - name: pypyr.steps.assert
    in:
      assert: False
  - name: pypyr.steps.echo
    comment: this step won't ever run because pipeline always
             fails on previous step.
    in:
      echoMe: unreachable
```

{{% note tip %}}
Remember you can use formatting [substitutions]({{< ref "/docs/substitutions">}}) 
for the `assert` condition selectively to raise the `AssertionError` or not.
{{% /note %}}

### custom error
You can raise any 
[built-in Python exception](https://docs.python.org/3/library/exceptions.html) 
from an [inline python step]({{< ref "/docs/steps/py">}}).

You can also raise pypyr's built-in errors or your own custom error objects if 
you import the appropriate module as part of the py step, per usual Python 
module import referencing.

```yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin
  - name: pypyr.steps.py
    in:
      py: raise ValueError('arb error text here')
  - name: pypyr.steps.echo
    comment: this step won't ever run 
             because pipeline always
             fails on previous step.
    in:
      echoMe: unreachable
```
