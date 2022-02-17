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

Here's a silly pipeline:
```yaml
# ./pipeline-dir/my-pipe.yaml
context_parser: pypyr.parser.keyvaluepairs
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: piper {arbkey} that {anotherkey} again
  - name: pypyr.steps.set
    in:
      set:
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

You can run this same pipeline programmatically with the same inputs like this:
```python
# Optional. If you want to see pypyr stdout output, log level should be <= 25
# import logging
# logging.basicConfig(level=25)

from pypyr import pipelinerunner

# You can run a pipeline via the API like this:
context = pipelinerunner.run(pipeline_name='pipeline-dir/my-pipe',
                             args_in=['arbkey=pipe', 'anotherkey=song'])

# Or like this:
context = pipelinerunner.run(pipeline_name='pipeline-dir/my-pipe',
                             dict_in={'arbkey': 'pipe',
                                      'anotherkey': 'song'})

# context behaves like a dict
print(context['myoutput'])  # I was set in the pipeline!
print(context['input_values'][0])  # pipe
print(context['some_nesting']['down_level']['arb_number'])  # 123
```

Use the `run()` function in the `pypyr.pipelinerunner` module to run a pipeline.
For the exact same behavior as the CLI, you also have to [initialize
config](#initialize-config) and [logging](#logging).

This example shows two different ways of initializing the pipeline's context:
both are equivalent and results in the same values being passed to the pipeline -
the pipeline step sequences run identically. For this example pipeline, the
output `context` for both `run()` calls is identical. See [initialize
context](#passing-values-to-the-pipeline) for details on `args_in` vs `dict_in`.

`run()` returns the pypyr `Context` object to you as it is after the pipeline
completes, giving you access to any mutations the pipeline made to it.

## run() entry-point
This is the full `pypyr.pipelinerunner.run()` function signature:

```python
run(
  pipeline_name: str,
  args_in: list[str] | None = None,
  parse_args: bool | None = None,
  dict_in: dict | None = None,
  groups: list[str] | None = None,
  success_group: str | None = None,
  failure_group: str | None = None,
  loader: str | None = None,
  py_dir: str | bytes | PathLike | None = None
) -> pypyr.context.Context:
```

### input args
```python
import pypyr.pipelinerunner

out = pypyr.pipelinerunner.run(pipeline_name='arb-pipe',
                               args_in=['arb', 'context input'],
                               parse_args=True,
                               dict_in={'key': 'value', 'arbkey': 'arb value'},
                               groups=['group1', 'group2'],
                               success_group='success_group',
                               failure_group='failure_group',
                               loader='mypackage.myloader'
                               py_dir='arb/dir')
```

- `pipeline_name`: string. Required.
  - Name of pipeline, sans .yaml at end.
  - `{pipeline_name}.yaml` is relative to the current working directory.
  - You can also specify an absolute path here (again, just leave out the 
    .yaml at the end).
- `args_in`: list of string. Optional.
  - Initialize the pypyr context with this list of strings.
  - Use the python shlex split function on a string to get a parsed list.
  - If not specified pypyr will create an empty `Context` object for you, 
    depending on how the pipeline's `context_parser` handles `None` input.
- `parse_args`: Boolean. Optional.
  - Explicitly set whether to run the `context_parser` on the pipeline.
  - If you set `args_in`, `parse_args` defaults to `True`.
  - If you set `dict_in` and NOT `args_in`, pypyr assumes you don't want to run
    the pipeline's context parser to initiate context and defaults `parse_args`
    to `False`.
- `dict_in`: dict. Optional.
  - Initialize the pypyr `Context` object with this dict.
  - If not specified pypyr will create an empty `Context` for you.
  - If you set both `dict_in` AND `args_in`, pypyr will initialize Context with
    `dict_in` and then merge the results of `args_in` processed by the
    pipeline's `context_parser` into that before running the pipeline with the
    resulting combined context. This is probably what you want by default, since
    the point of using `dict_in` is to bypass the inefficient string parsing of
    the context_parser.
- `groups`: list of string. Optional. 
  - Step-group names to run in pipeline. 
  - Defaults to `['steps']` if not specified.
- `success_group`: string. Optional. 
  - Step-group name to run on success completion. 
  - Defaults to `on_success` if not specified.
- `failure_group`: string. Optional. 
  - Step-group name to run on pipeline failure. 
  - Defaults to `on_failure` if not specified.
- `loader`: string. Optional.
  - Absolute name of pipeline loader module.
  - If not specified will use `pypyr.loaders.file` - the standard builtin pypyr
    pipeline loader.
- `py_dir`: Path-like. Optional.
  - Look for custom modules in this directory.
  - Under the hood, pypyr adds this directory to `sys.path`.
  - This is useful if your pipeline uses _ad hoc_ .py files that are NOT
    installed in the current Python environment.
  - Be aware that if you use the standard default file loader, pypyr will add
    the pipeline's parent directory automatically for you after it finds the
    pipeline. You therefore do NOT need to set `py_dir` to the pipeline
    directory when using the default loader.
  - If you are using a custom loader that is not installed in the current Python
    environment, you have to set `py_dir` to allow pypyr to find it.
  - If you have installed (typically with `$ pip install`) all the custom
    modules your pipeline uses into the current Python environment you do NOT
    need to set `py_dir`.
  - If your `sys.path` already contains the necessary paths to discover the
    custom modules that your pipeline uses, you do NOT need to set this.
  - If your pipeline does NOT use any custom Python modules, you do NOT need to
    set `py_dir`.

### returns
The pypyr context as it is after the pipeline completes. This is of type
`pypyr.context.Context()`. Each pipeline invocation uses its own fresh context -
a context is unique to a single root pipeline run.

The `Context` object behaves pretty much like a standard `dict`.


## passing values to the pipeline
If you want to inject values into the pipeline you are running, you do so by
initializing the context for the pipeline run with the data you need.

### args_in vs dict_in
```python
# Initialize context with args_in:
context_out_1 = pipelinerunner.run(pipeline_name='pipeline-dir/my-pipe',
                                   args_in=['arbkey=pipe', 'anotherkey=song'])

# Or with dict_in:
context_out_2 = pipelinerunner.run(pipeline_name='pipeline-dir/my-pipe',
                                   dict_in={'arbkey': 'pipe',
                                            'anotherkey': 'song'})

# Both results in the pipeline running the same, with the same output
assert context_out_1 == context_out_2
```

You can initialize context in 2 different ways:
- `args_in`
    - Use this if you want to pass context arguments to the pipeline's
      `context_parser` in exactly the same way as the cli does - as a list of
      strings.
    - This makes it the pipeline's `context_parser`'s responsibility to
      interpret & parse those strings.
- `dict_in`
    - Use this if you have your own dict-like structure you want to pass to 
      your pipeline.
    - This will bypass the pipeline's `context_parser` and use
      your input dict directly to initialize the pypyr Context.
    - This way you can directly control the context structure and the types of 
      the values you put in it.

You can also set both `dict_in` and `args_in` - in which case pypyr will
initialize context with `dict_in`, run the `context_parser` with `args_in` and
then merge the results of both into a single context before running the
pipeline with the combined context.

You can use whichever you prefer when you invoke your pipelines
programmatically. Either way is identical for actual pipeline control-of-flow
execution - the only difference is how you initialize the context that the
pipeline will use.

{{% note tip %}}
Now, having said that, parsing input strings and inferring types with the
`context_parser` (like you have to when the cli is passing the values) is
inherently inefficient. Since you are in structured Python code already when
using the API, you might as well prefer `dict_in` to `args_in`.

What works well is to set a `context_parser` in your pipeline yaml, which allows
you to use it from the cli with custom arguments, and then you can just bypass
the `context_parser` when you use the API by using `dict_in` explicitly to 
initialize the Context exactly how you want it. This allows you to call the same
pipeline from both the cli and the api - and if you only pass `dict_in` to the
api rather than `args_in`, pypyr will automatically bypass the `context_parser`
for you unless you explicitly set `parse_args` to `True`.
{{% /note %}}

### args_in list
`args_in` takes a list of strings. This is a POSIX style argument split on the
input arguments following the pipeline name as passed from the cli.

```text
$ pypyr mypipeline arg1 "arg 2" arg3=4
```

Equates to `args_in` of:
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
# shlex result: ['arbkey=pipe', 'anotherkey=song with a space ']

pipelinerunner.run(pipeline_name='pipeline-dir/my-pipe',
                   args_in=in_args)
```

### invoke pipeline from api with no input context
Setting an input context is optional. It's only relevant when you have to pass
values into your pipeline.

```yaml
# ./no-input-context.yaml
steps:
  - name: pypyr.steps.py
    in:
      py: print('hello hello!')
  - name: pypyr.steps.set
    in:
      set:
        arbkey: I was set in the pipeline!
```

You can run this pipeline like this:
```python
# For a pipeline ./no-input-context.yaml
# that you'd run from the cli like this:
# $ pypyr no-input-context

from pypyr import pipelinerunner

out = pipelinerunner.run('no-input-context')
print(out['arbkey']) # I was set in the pipeline!
```

## initialize config
By default, when you call `run()` pypyr will use default configuration values
and "just work" without you having to do anything special.

If you want to follow the [configuration look-up sequences]({{< ref
"/docs/getting-started/config" >}}) where pypyr looks for yaml/toml config
sources on disk, you explicitly need to tell pypyr to do so using
`config.init()`.

Because this is a relatively expensive operation you probably only want to run
this once per Python session, regardless of how many pipelines you execute with
`pipelinerunner.run()`. That said, if you really wanted to you could run it
multiple times to reconfigure pypyr on the fly.

```python
from pypyr import pipelinerunner
from pypyr.config import config

# initialize config once
config.init()

# run multiple pipelines
out1 = pipelinerunner.run('my-pipeline-1')
out2 = pipelinerunner.run('my-pipeline-2')
```

If your code invokes `config.init()`, you can still bypass the heavy
initialization sequence on-the-fly by using the [PYPYR_SKIP_INIT]({{< ref
"docs/getting-started/config#pypyr_skip_init" >}}) environment variable.

## logging
By default python runs with a log level of 30 (WARNING). This means you won't 
see pypyr NOTIFY output like [pypyr.steps.echo]({{< ref "docs/steps/echo" >}}) 
or [description output]({{< ref "docs/decorators/description">}}) when you 
invoke the pypyr api from your own code without setting the log-level. This is 
because, as an API, pypyr shouldn't clutter your stdout unless you explicitly 
tell it to do so.

If you're invoking pypyr via the API from your own application, it's your
responsibility to set up and configure logging. If you just want the same log
handlers & formatters that the pypyr cli uses, you can call
[pypyr.log.logger.set_root_logger()](#set_root_logger-input-args) before
invoking `pipelinerunner.run()`.

```python
from pypyr import pipelinerunner
from pypyr.config import config
import pypyr.log.logger

# optional - one-time loading of config from files
config.init()

# initialize logging once
# use the same log format & level defaults as the cli
pypyr.log.logger.set_root_logger()

# For a pipeline that you'd run from the cli like this:
# $ pypyr pipeline-dir/my-pipe arbkey=pipe anotherkey=song

context_out = pipelinerunner.run('pipeline-dir/my-pipe',
                                 dict_in={'arbkey': 'pipe',
                                          'anotherkey': 'song'})
```

If you are calling both `config.init()` and `set_root_logger()`, be sure to call
`config.init()` FIRST, because this will allow the logger to get its
configuration from [config]({{< ref "/docs/getting-started/config" >}}). You can
customize the logging output format with the config properties prefixed with
`log_`.

Be aware that pypyr adds a `NOTIFY` - `25` custom log-level and a `notify()`
function to `logging` in all cases, even when you don't call
`set_root_logger()`.

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

If you do call `set_root_logger` do so once and only once at program
initialization.

Call `set_root_logger` only _after_ `config.init()` if you want to override
logging defaults from yaml/toml configuration files. This is optional though,
since if you do not call `config.init()` at all `set_root_logger` will just use
the standard out-of-box defaults.

```python
import pypyr.log.logger

pypyr.log.logger.set_root_logger(log_level=25,
                                 log_path='./dir/my-file.log')
```

- `log_level`: int. Optional.
    - Defaults to `25` if not specified.
- `log_path`: Path-like. Optional.
    - If specified, append pypyr output to this file.
    - Defaults to `None` if not specified - which means output only goes to the
      console.

