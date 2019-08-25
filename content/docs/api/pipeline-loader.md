---
title: custom pype loader
# linktitle: step
description: Create a custom pipeline loader to load pipelines from elsewhere.
date: 2019-08-21
publishdate: 2019-08-21
lastmod: 2019-08-21
# card_extra_summary:
#   heading: input context property
#   details: "`assert` (dict)"
categories: [ pipelines ]
topics: [ custom code ]
menu:
  docs:
    parent: api
    name: pipeline loader
draft: false
seo_article_headline: Customize how & from where you load task-runner pipelines.
seo_description: Create a custom pipeline loader to get pipelines from s3, consul or your own pipeline storage system.
---
# create a custom pipeline loader
## load pipeline not on the local filesystem
A pype loader is responsible for loading a pipeline.

The default pype loader is [pypyr.pypeloaders.fileloader](https://github.com/pypyr/pypyr-cli/blob/master/pypyr/pypeloaders/fileloader.py).

This default loader loads pipelines from the local file-system, following the
usual [pypyr pipeline look-up sequence]({{< ref "/docs/pipelines/lookup-order">}}).

If you want to load pipelines from somewhere else, like maybe a shared pipeline
library, or implement your own caching, or maybe if you want to load a pipeline 
from something like s3 or consul, you can roll your own pype loader.

{{% note tip %}}
pypyr API consumers be aware that pypyr internally already automatically caches 
a pipeline by name on 1st load, so that subsequent calls to the pypyr API to 
run that pipeline will use the cached pipeline rather than invoke the loader 
each time.

From the cli the loader has to load the pipeline from the actual loader each
time.
{{% /note %}}

## custom loader function signature
```python
import logging
from pypyr.errors import PipelineNotFoundError
import pypyr.yaml

# use pypyr logger to ensure loglevel is set correctly
logger = logging.getLogger(__name__)

def get_pipeline_definition(pipeline_name, working_dir):
    """Open and parse the pipeline definition yaml.

    Parses pipeline yaml and returns dictionary representing the pipeline.

    pipeline_name is whatever is passed in from the shell like:
    pypyr pipelinename args

    Args:
        pipeline_name: string. Name of pipeline.
                       Passed in from the 1st positional cmd argument.
        working_dir: path. passed in from the --dir switch.

    Returns:
        dict describing the pipeline's yaml, parsed from the pipeline yaml.

    Raises:
        PipelineNotFoundError: pipeline_name not found.

    """
    logger.debug("starting")

    # it's good form only to use .info and higher log levels when you must.
    # For .debug() being verbose is very much encouraged.
    logger.info("Your clever code goes here. . . ")

    # if something goes wrong you might want to raise PipelineNotFoundError
    yaml_file = get_filelike_object_from_somewhere(pipeline_name, 
                                                      working_dir)
    
    # pass the file-like object here to have pypyr parse the yaml for you
    # yaml_file has to be open and ready to read at this point
    pipeline_definition = pypyr.yaml.get_pipeline_yaml(yaml_file)

    # remember to close yaml_file here if necessary

    logger.debug(
        f"found {len(pipeline_definition)} stages in pipeline.")

    logger.debug("pipeline definition loaded")

    logger.debug("done")
    return pipeline_definition
```

## use custom loader from the API
The usual python import module resolution rules apply.

Assuming you saved your python with the 
`def get_pipeline_definition(pipeline_name, working_dir)` in a file like this 
`./dir/myloader.py`

```python
import pypyr.pipelinerunner

pypyr.pipelinerunner.load_and_run_pipeline(
            pipeline_name='pipeline-name',
            pipeline_context_input='arb context input',
            loader='dir.myloader'
        )
```

If you package your loader and you install the package into the active python 
environment, you can of course use the usual python package name instead:

```python
import pypyr.pipelinerunner

pypyr.pipelinerunner.load_and_run_pipeline(
            pipeline_name='pipeline-name',
            pipeline_context_input='arb context input',
            loader='mypackage.myloader'
        )
```

{{% note tip %}}
When you use the API as above, pypyr will automatically cache the pipeline
you fetch from your custom loader for you, you do not have to do anything 
special.
{{% /note %}}

### set working directory
If you want pypyr to load python modules and steps in the current directory
automatically, like it does from the cli, include `set_working_directory` 
beforehand:

```python
import pypyr.moduleloader
import pypyr.pipelinerunner

# pipelines specify steps in python modules that load dynamically.
# make it easy for the operator so that the cwd is automatically included
# without needing to pip install a package 1st.
pypyr.moduleloader.set_working_directory('path/to/dir/here')

pypyr.pipelinerunner.load_and_run_pipeline(
            pipeline_name='pipeline-name',
            pipeline_context_input='arb context input',
            loader='mypackage.myloader'
        )

```

### handling stop & notify
Normally, `main` takes care of handling `Stop` instructions. Because you skip
`main` in order to run `load_and_run_pipeline` directly, it is up to you to 
handle `Stop`, which is manifest as an exception.

pypyr `echo` and step `comment` output uses the NOTIFY - 25 log-level. If you 
want this to work as per usual, you can easily set up the level with one line
of code: `pypyr.log.logger.set_up_notify_log_level()`. 

```python
import logging
from pypyr.errors import Stop
import pypyr.log.logger
import pypyr.moduleloader
import pypyr.pipelinerunner

logger = logging.getLogger(__name__)

pypyr.log.logger.set_up_notify_log_level()

logger.debug("starting pypyr")

# pipelines specify steps in python modules that load dynamically.
# make it easy for the operator so that the cwd is automatically included
# without needing to pip install a package 1st.
pypyr.moduleloader.set_working_directory('path/to/dir/here')

try:
    load_and_run_pipeline(pipeline_name='pipeline-name',
                          pipeline_context_input='arb context input',
                          loader='mypackage.myloader',
                          groups=['group'],
                          success_group='success_group',
                          failure_group='failure_group')
except Stop:
    logger.debug("Stop: stopped pypyr")

logger.debug("pypyr done")
```

### function signature
The full function signature is:
```python
def load_and_run_pipeline(pipeline_name,
                          pipeline_context_input=None,
                          context=None,
                          parse_input=True,
                          loader=None,
                          groups=None,
                          success_group=None,
                          failure_group=None):
```

In general, prefer to run `main` as described in [run pipeline]({{< ref "run-pipeline">}}).
Use `load_and_run_pipeline` only when you want to use a custom loader.

#### args
- pipeline_name (str)
    - Name of pipeline, sans .yaml at end.
- pipeline_context_input (str). Optional. Default `None`.
    - Initialize the pypyr context with this string.
- context (`pypyr.context.Context`). Optional. Default `None`. 
    - Use if you already have a Context object, such as if you are running a 
      pipeline from within a pipeline and you want to re-use the same context
      object for the child pipeline. Any mutations of the context by the 
      pipeline will be against this instance of it.
- parse_input (bool). Optional. Default `True`.
    - Run context_parser in pipeline if `True`.
- loader (str). Optional.
    - Absolute name of pipeline loader module. If not specified will use 
      `pypyr.pypeloaders.fileloader`.
- groups (list of str). Optional.
    - step-group names to run in pipeline.
    - Default to `steps` if not specified.
- success_group (str). Optional.
    - step-group name to run on success completion.
    - Default to `on_success` if not specified.
- failure_group: (str). Optional.
    - step-group name to run on pipeline failure.
    - Default to `on_failure` if not specified.

#### returns
None