---
title: run pipeline api
# linktitle: 
description: Run pipeline programmatically from the API in your own code.
date: 2020-08-12
publishdate: 2020-08-13
# categories: [ ]
menu:
  docs:
    parent: api
    name: run pipeline
seo_article_headline: Run an automation pipeline programmatically in your own code.
seo_description: Use the Python API to run task-runner pipeline workflows from your own code in a few simple lines.
topics: [ custom code ]
---
# run a pipeline from the api
You can run a pypyr automation pipeline programmatically from your own code 
using the python api.

```yaml
# ./pipeline-dir/my-pipe.yaml
context_parser: pypyr.parser.keyvaluepairs
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: piper {arbkey} that {anotherkey} again
  - name: pypyr.steps.contextsetf
    in:
      contextSetf:
        myoutput: I was set in the pipeline!
        input_values:
          - '{arbkey}'
          - '{anotherkey}'
        some_nesting:
          down_level:
            arb_number: 123
            arb_bool: False
```

Run this pipeline from the cli like this:

{{< app-window title="term" lang="text" >}}
$ pypyr pipeline-dir/my-pipe arbkey=pipe anotherkey=song
piper pipe that song again

$
{{< /app-window >}}

You can run this same pipeline programmatically like this:
```python
# Optional. If you want to see pypyr stdout output, log level should be <= 25
# import logging
# logging.basicConfig(level=25)

from pypyr import pipelinerunner

# You can run via API like this:
pipelinerunner.main(pipeline_name='pipeline-dir/my-pipe',
                    pipeline_context_input=['arbkey=pipe', 'anotherkey=song'])

# Or like this:
context_out = pipelinerunner.main_with_context(
    pipeline_name='pipeline-dir/my-pipe',
    dict_in={'arbkey': 'pipe',
             'anotherkey': 'song'})

# context_out behaves like a dict
print(context_out['myoutput'])  # I was set in the pipeline!
print(context_out['input_values'][0])  # pipe
print(context_out['some_nesting']['down_level']['arb_number'])  # 123
```

Use the `pypyr.pipelinerunner` module. There are two sensible entry-points for 
most developers:
- [main()](#main-entry-point)
    - Use this if you want to pass context arguments to the pipeline's
      `context_parser` in exactly the same way as the cli does.
- [main_with_context()](#main-with-context-entry-point)
    - Use this if you have your own dict-like structure you want to pass to 
      your pipeline.
    - This will bypass the pipeline's `context_parser` and use
      your input dict directly.
    - This function also returns the pypyr `Context` object to you after the
      pipeline completes.

Call whichever you prefer once per pypyr pipeline invocation. Both entry-points
are identical for actual pipeline control-of-flow execution - the only
difference is how you initialize the context that the pipeline will use.

## main entry-point
Use `main()` if you want to run a pipeline exactly like the cli does, by using 
the pipeline's `context_parser` to initialize context with a list of string
arguments.

`pipeline_context_input` takes a list of strings. This is a POSIX style argument
split on the input arguments following the pipeline name as passed from the cli.

```fish
$ pypyr mypipeline arg1 "arg 2" arg3=4
```

Equates to a `pipeline_context_input` of:
```python
['arg1', 'arg 2', 'arg3=4']
```

If you have a single string representing all the input arguments, a convenient
way to split it into a list while honoring escape sequences & quotes is to use
the built-in python [shlex
split](https://docs.python.org/3/library/shlex.html#shlex.split) function.

```python
import shlex
from pypyr import pipelinerunner

# For a pipeline that you'd run from the cli like this:
# $ pypyr pipeline-dir/my-pipe arbkey=pipe anotherkey="song with a space"

in_args = shlex.split('arbkey=pipe anotherkey="song with a space "')
# in_args == ['arbkey=pipe', 'anotherkey=song with a space ']

pipelinerunner.main(pipeline_name='pipeline-dir/my-pipe',
                    pipeline_context_input=in_args)
```

### input args
```python
import pypyr.pipelinerunner

pypyr.pipelinerunner.main(pipeline_name='arb-pipe',
                          pipeline_context_input=['arb', 'context input'],
                          working_dir='arb/dir',
                          groups=['group1', 'group2'],
                          success_group='success_group',
                          failure_group='failure_group',
                          loader='mypackage.myloader')
```

- `pipeline_name`: string. Required.
    - Name of pipeline, sans .yaml at end.
    - `{pipeline_name}.yaml` is relative to the `working_dir`.
- `pipeline_context_input`: list of string. Optional.
    - Initialize the pypyr context with this list of strings.
    - Use the python shlex split function on a string to get a parsed list.
    - If not specified pypyr will create an empty `Context` object for you, 
      depending on how the pipeline's `context_parser` handles `None` input.
- `working_dir`: Path-like. Optional.
    - Look for pipelines and modules in this directory.
    - Default to the current directory if not specified.
- `groups`: list of string. Optional. 
    - Step-group names to run in pipeline. 
    - Default to `steps` if not specified.
- `success_group`: string. Optional. 
    - Step-group name to run on success completion. 
    - Default to `on_success` if not specified.
- `failure_group`: string. Optional. 
    - Step-group name to run on pipeline failure. 
    - Default to `on_failure` if not specified.
- `loader`: string. Optional.
    - Absolute name of pipeline loader module.
    - If not specified will use `pypyr.pypeloaders.fileloader` - the standard
      pypyr pipeline loader.

### returns
None. If you want to interact with the `Context` object after the pipeline 
run completes, call `main_with_context` instead.

## main with context entry-point
Use `main_with_context` instead of `main` if you have a a dict-like object you
want to use to initialize the context, rather than using the context parser with
a string input that pypyr needs to parse first.

`main_with_context` also returns the context after the pipeline completes,
giving you access to the values the pipeline stored to context during its run.

If you do want to run the pipeline's `context_parser`, use `main()` instead.

### input args
```python
import pypyr.pipelinerunner

context_out = pypyr.pipelinerunner.main_with_context(
    pipeline_name='mypath/arb-pipe',
    dict_in={'key': 'value', 'arbkey': 'arb value'},
    working_dir='arb/dir',
    groups=['group1', 'group2'],
    success_group='success_group',
    failure_group='failure_group',
    loader='mypackage.myloader')
```

The input arguments are the same as for `main()`, with the only difference being 
that instead of passing a list of string in `pipeline_context_in` you pass a 
dict-like object in `dict_in`.

- `pipeline_name`: string. Required.
    - Name of pipeline, sans .yaml at end.
    - `{pipeline_name}.yaml` is relative to the `working_dir`.
- `dict_in`: dict. Optional.
    - Initialize the pypyr `Context` object with this dict.
    - If not specified pypyr will create an empty `Context` for you.
    - The function returns the created `Context` object with any mutations
      made by the pipeline.
- `working_dir`: Path-like. Optional.
    - Look for pipelines and modules in this directory.
    - Default to the current directory if not specified.
- `groups`: list of string. Optional. 
    - Step-group names to run in pipeline. 
    - Default to `steps` if not specified.
- `success_group`: string. Optional. 
    - Step-group name to run on success completion. 
    - Default to `on_success` if not specified.
- `failure_group`: string. Optional. 
    - Step-group name to run on pipeline failure. 
    - Default to `on_failure` if not specified.
- `loader`: string. Optional.
    - Absolute name of pipeline loader module.
    - If not specified will use `pypyr.pypeloaders.fileloader` - the standard
      pypyr pipeline loader.

### returns
The pypyr context as it is after the pipeline completes. This is of type
`pypyr.context.Context()`. The `Context` object behaves pretty much like a
standard `dict`.

## invoke pipeline from api with no input context
Setting an input context is optional.

```yaml
# ./no-input-context.yaml
steps:
  - name: pypyr.steps.py
    in:
      pycode: print('hello hello!')
  - name: pypyr.steps.contextsetf
    in:
      contextSetf:
        arbkey: I was set in the pipeline!
```

You can run this pipeline like this:
```python
# For a pipeline ./no-input-context.yaml
# that you'd run from the cli like this:
# $ pypyr no-input-context

from pypyr import pipelinerunner

pipelinerunner.main(pipeline_name='no-input-context')

# or if you want to work with return context from pipeline
out = pipelinerunner.main_with_context(pipeline_name='no-input-context')
print(out['arbkey']) # I was set in the pipeline!
```

## logging
By default python runs with a log level of 30 (WARNING). This means you won't 
see pypyr NOTIFY output like [pypyr.steps.echo]({{< ref "docs/steps/echo" >}}) 
or [description output]({{< ref "docs/decorators/description">}}) when you 
invoke the pypyr api from your own code without setting the log-level. This is 
because as an API pypyr shouldn't clutter your stdout unless you explicitly 
tell it to do so.

If you're invoking pypyr via the API from your own application, it's your
responsibility to set up and configure logging. If you just want the same log
handlers & formatters that the pypyr cli uses, you can call
[pypyr.log.logger.set_root_logger()](#set_root_logger-input-args) before
invoking `pipelinerunner.main()` or `pipelinerunner.main_with_context()`.

```python
import pypyr.log.logger
from pypyr import pipelinerunner

# use the same log format & level defaults as the cli
pypyr.log.logger.set_root_logger()

# For a pipeline that you'd run from the cli like this:
# $ pypyr pipeline-dir/my-pipe arbkey=pipe anotherkey=song

context_out = pipelinerunner.main_with_context(
    pipeline_name='pipeline-dir/my-pipe',
    dict_in={'arbkey': 'pipe',
             'anotherkey': 'song'})
```

Be aware that pypyr adds a `NOTIFY` - `25` custom log-level and a `notify()` 
function to `logging`.

### log levels
Log level enumeration:
- < 10 gives full traceback on errors.
- 10=DEBUG
- 20=INFO
- 25=NOTIFY (default)
- 30=WARNING
- 40=ERROR
- 50=CRITICAL

### set_root_logger input args
Use `set_root_logger` when you want to use the same logging format as the cli.

This is optional. Your application is free to define its own log level and 
handlers - in which case, don't bother with `set_root_logger()`.

```python
import pypyr.log.logger

pypyr.log.logger.set_root_logger(log_level=25,
                                 log_path='./dir/my-file.log')
```

- `log_level`: int. Optional.
    - Defaults to `25` if not specified.
- `log_path`: Path-like. Optional.
    - If specified, append pypyr output to this file.
    - Defaults to `None` if not specified.

