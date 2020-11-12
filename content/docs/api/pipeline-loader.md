---
title: custom pype loader
# linktitle: step
description: Create a custom pipeline loader to load pipelines from elsewhere.
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
# card_extra_summary:
#   heading: input context property
#   details: "`assert` (dict)"
categories: [ pipelines ]
topics: [ custom code ]
menu:
  docs:
    parent: api
    name: pipeline loader
seo_article_headline: Customize how & from where you load task-runner pipelines.
seo_description: Create a custom pipeline loader to get pipelines from s3, consul or your own pipeline storage system.
---
# create a custom pipeline loader
## load pipeline not on the local filesystem
A pype loader is responsible for loading a pipeline.

The default pype loader is [pypyr.pypeloaders.fileloader](https://github.com/pypyr/pypyr/blob/master/pypyr/pypeloaders/fileloader.py).

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
    $ pypyr pipelinename args

    Args:
        pipeline_name (str): Name of pipeline. Passed in from the 1st 
                             positional cmd argument, or the pipeline_name
                             main() api arg.
        working_dir (Path): Passed in from the cli --dir switch or the 
                            working_dir main() api arg. If neither set, 
                            defaults to current execution dir.

    Returns:
        Dict-like object describing the pipeline's yaml, parsed from the 
        pipeline file/stream.

    Raises:
        PipelineNotFoundError: pipeline_name not found.

    """
    logger.debug("starting")

    # it's good form only to use .info and higher log levels when you must.
    # For .debug() being verbose is very much encouraged.
    logger.info("Your clever code goes here. . . ")

    # if something goes wrong you might want to raise PipelineNotFoundError
    yaml_file = get_filelike_object_from_somewhere(pipeline_name, working_dir)
    
    # pass the file-like object here to have pypyr parse the yaml for you
    # yaml_file has to be open and ready to read at this point.
    # if you prefer to use your own yaml parser, do that here instead of 
    # calling get_pipeline_yaml.
    pipeline_definition = pypyr.yaml.get_pipeline_yaml(yaml_file)

    # remember to close yaml_file here if necessary, or put it inside a with
    yaml_file.close()

    logger.debug(f"found {len(pipeline_definition)} stages in pipeline.")

    logger.debug("pipeline definition loaded")

    logger.debug("done")
    return pipeline_definition
```

## example
Here is a minimalist custom loader that just loads a pipeline from the working 
directory without doing anything special:

```python
"""Naive custom loader without any error handling."""
import pypyr.yaml


def get_pipeline_definition(pipeline_name, working_dir):
    """Simplified loader that gets pipeline_name.yaml in working_dir."""
    with open(working_dir.joinpath(f'{pipeline_name}.yaml')) as yaml_file:
        return pypyr.yaml.get_pipeline_yaml(yaml_file)
```

{{% note tip %}}
Remember that `working_dir` will contain the current execution directory if
`--dir` wasn't explicitly set from the cli or if `working_dir` was `None` when 
calling `main()` via the api.
{{% /note %}}

## use custom loader from the API
The usual python import module resolution rules apply.

```python
import pypyr.pipelinerunner

# load pipeline with get_pipeline_definition() in ./mydir/myloader.py
pypyr.pipelinerunner.main(
            pipeline_name='pipeline-name',
            pipeline_context_input=['arb', 'context', 'input'],
            loader='mydir.myloader'
        )
```

If you package your loader and you install the package into the active python 
environment, you can of course use the usual python package name instead:

```python
import pypyr.pipelinerunner

pypyr.pipelinerunner.main(
            pipeline_name='pipeline-name',
            pipeline_context_input=['arb', 'context', 'input'],
            loader='mypackage.myloader'
        )
```

This works the same way whether you use `main()` or `main_with_context()` as
entry-point. See here for full details on [how to run a pipeline from code]({{<
ref "run-pipeline">}}).

{{% note tip %}}
When you use the API as above, pypyr will automatically cache the pipeline
you fetch from your custom loader for you, you do not have to do anything 
special.
{{% /note %}}