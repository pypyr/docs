---
title: custom pype loader
# linktitle: step
description: Create a custom pipeline loader to load pipelines from a different place.
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

The default pype loader is [pypyr.loaders.file](https://github.com/pypyr/pypyr/blob/main/pypyr/loaders/file.py).

This default loader loads pipelines from the local file-system, following the
usual [pypyr pipeline look-up sequence]({{< ref "/docs/pipelines/lookup-order">}}).

If you want to load pipelines from somewhere else, like maybe a shared pipeline
library, or implement your own caching, or maybe if you want to load a pipeline 
from something like s3 or consul, you can roll your own pype loader.

{{% note tip %}}
pypyr internally automatically caches a pipeline by name for each loader on 1st
load, so that subsequent calls in the same session to run that pipeline will
use the cached pipeline rather than invoke the loader each time.

This is the case both when you invoke pipelines from the api/cli, and when you
are calling a pipeline from within a pipeline with
[pypyr.steps.pype]({{< ref "/docs/steps/pype" >}}).
{{% /note %}}

## custom loader function signature
A loader is any Python module with a function like this:
```python
get_pipeline_definition(name: str, parent: Any) -> pypyr.pipedef.PipelineDefinition | Mapping
```

Here it is in more detail - all you have to do is replace 
`get_filelike_obj_from_somewhere` with your own code that, well, gets a
file-like object from somewhere:
```python
import logging
from pypyr.errors import PipelineNotFoundError
import pypyr.yaml

logger = logging.getLogger(__name__)

def get_pipeline_definition(pipeline_name, parent):
    """Open and parse the pipeline yaml.

    Parse the pipeline yaml and return dictionary representing the pipeline.

    pipeline_name is whatever is passed in from the shell like:
    $ pypyr pipelinename args

    Args:
        pipeline_name (str): Name of pipeline. Passed in from the 1st 
                             positional cmd argument, or the pipeline_name
                             run() api arg.
        parent (any): Set by the parent's loader when this call is to get the
                      definition for a child pipeline called by that parent
                      pipeline with pype.
        
    Returns:
        pypyr.pipedef.PipelineDefinition, or Dict-like object describing the 
        pipeline's yaml, parsed from the pipeline file/stream.

    Raises:
        PipelineNotFoundError: pipeline_name not found.

    """
    logger.debug("starting")

    # it's good form only to use .info and higher log levels when you must.
    # For .debug() being verbose is very much encouraged.
    logger.info("Your clever code goes here. . . ")

    # if something goes wrong you might want to raise PipelineNotFoundError
    with get_filelike_obj_from_somewhere(pipeline_name) as yaml_file:
    # pass the file-like object here to have pypyr parse the yaml for you
    # yaml_file has to be open and ready to read at this point.
    # if you prefer to use your own yaml parser, do that here instead of 
    # calling get_pipeline_yaml.
        pipeline_yaml = pypyr.yaml.get_pipeline_yaml(yaml_file)

    # remember to close yaml_file here if necessary if not using "with"

    # some arbitrary log output
    logger.debug(f"found {len(pipeline_yaml)} stages in pipeline.")

    logger.debug("done")
    return pipeline_yaml
```

{{% note tip %}}
The `parent` argument only has a value when loading a child pipeline via a
`pypyr.steps.pype` step.
{{% /note %}}

## example
Here is a minimalist custom loader that just loads a pipeline from the current
working directory without doing anything special.

Create a file `./mydir/myloader.py` as follows (the file-name is not important,
what is important is that the .py file has a `def get_pipeline_definition` in
it):

```python
"""Naive custom loader without any error handling."""
from pathlib import Path
import pypyr.yaml

CWD = Path.cwd()

def get_pipeline_definition(pipeline_name, parent):
    """Simplified loader that gets pipeline_name.yaml in working dir."""
    with open(CWD.joinpath(f'{pipeline_name}.yaml')) as yaml_file:
        return pypyr.yaml.get_pipeline_yaml(yaml_file)
```

{{% note tip %}}
pypyr will automatically cache the return value of the
`get_pipeline_definition` function of your custom loader for you, you do not
have to do anything special.
{{% /note %}}

## use custom loader from the API
The usual [custom module import resolution rules]({{< ref
"/docs/api/custom-module-search-path" >}}) apply. If your loader path is not in
`sys.path` already, pypyr can only look for ad hoc loader modules relative to
the `py_dir` directory or in packages installed in the current Python
environment.

```python
from pathlib import Path
import pypyr.pipelinerunner

CWD = Path.cwd()

# load pipeline with get_pipeline_definition() in ./mydir/myloader.py
pypyr.pipelinerunner.run(
            pipeline_name='pipeline-name',
            args_in=['arb', 'context', 'input'],
            loader='mydir.myloader',
            py_dir=CWD
        )
```

If you package your loader and you install the package into the active python
environment (i.e you did a `$ pip install mypackage`), you can of course use the
usual python package name instead:

```python
import pypyr.pipelinerunner

# mypackage.myloader is installed in the current Python env
pypyr.pipelinerunner.run(
            pipeline_name='pipeline-name',
            args_in=['arb', 'context', 'input'],
            loader='mypackage.myloader'
        )
```

Since in this case the loader exists in the Python environment, you do NOT need
to pass `py_dir` to `run()`.

See here for full details on
[how to run a pipeline from code]({{<ref "run-pipeline">}}).

## use custom loader with pype
When you call a child pipeline from a parent pipeline with 
[pypyr.steps.pype]({{< ref "/docs/steps/pype" >}}) you can explicitly set the
loader when you invoke the child:

```yaml
steps:
  - name: pypyr.steps.pype
    comment: loader ./mydir/myloader.py will load pipeline subdir/child-pipeline
    in:
      pype:
        name: subdir/child-pipeline
        loader: mydir.myloader
```

{{% note tip %}}
By default, pypyr will try to load a child pipeline with the same loader that
handled the parent, so you do NOT explicitly need to set `loader` on the `pype`
step if you just want the child to use the same loader.

If you do NOT want the child to inherit the parent's loader setting by default
for your custom loader, see the next section for how to set 
`is_loader_cascading` on the `PipelineInfo` object.
{{% /note %}}

## custom pipeline metadata
Your custom `get_pipeline_definition` function can return either just the bare
dict-like yaml payload of the pipeline itself, or a
`pypyr.pipedef.PipelineDefinition` object. Internally, the pypyr core will
wrap the pipeline body in a default `PipelineDefinition` instance if you did
not do so yourself.

If you want to add custom metadata from your loader to your pipeline you can do
so by setting a `pypyr.pipedef.PipelineInfo` object on your
`PipelineDefinition`.

```python
# namespace: pypyr.pipedef

class PipelineDefinition():
    pipeline: Mapping # the dict-like pipeline body
    info: PipelineInfo # pipeline metadata

class PipelineInfo():
    pipeline_name: str # name of pipeline
    loader: str # absolute module name of loader (string)
    parent: Any = None # relative path hint for child pipelines
    is_parent_cascading: bool = True
    is_loader_cascading: bool = True
```

If you want to add extra custom metadata properties to your pipeline, derive
your own class from `PipelineInfo` and add such properties as you want.

This is entirely optional - you don't get any particular advantages by
explicitly setting & returning a `PipelineDefinition` from your custom
`get_pipeline_definition` function, since the pypyr core wraps the return value
from `get_pipeline_definition` in a `PipelineDefinition` for you if you did not
do so yourself.

You can control whether pipelines loaded by your custom loader will cascade
their `parent` and `loader` properties down to child pipelines when using
`pype` by setting the `is_parent_cascading` and `is_loader_cascading` boolean
flags.

### example
Here is a silly example where we expand on the naive loader from earlier with
an extra meta-data property called `arb_property`, by creating a custom
metadata class `ArbPipeInfo` for this loader.

```python
from pathlib import Path

from pypyr.pipedef import PipelineDefinition, PipelineInfo
import pypyr.yaml

CWD = Path.cwd()

class ArbPipeInfo(PipelineInfo):
    """Custom pipeline meta-data."""
    # you don't HAVE to use slots, but it's a tiny bit more efficient
    __slots__ = ['arb_property']

    def __init__(self, pipeline_name, loader, parent, arb):
        # is_parent_cascading & is_loader_cascading default True if not specified
        super().__init__(pipeline_name=pipeline_name,
                         loader=loader,
                         parent=parent)
        self.arb_property = arb


def get_pipeline_definition(pipeline_name, parent):
    """Simple loader gets pipeline_name.yaml in working dir + custom metadata."""
    # get & parse the pipeline yaml payload
    with open(CWD.joinpath(f'{pipeline_name}.yaml')) as yaml_file:
        pipeline_yaml = pypyr.yaml.get_pipeline_yaml(yaml_file)
    
    # custom meta-data to associate with pipeline
    info = ArbPipeInfo(pipeline_name=pipeline_name,
                       parent=parent,
                       loader=__name__,
                       arb_property='some useful value')

    # wrap pipeline body in a PipelineDefinition alongside its metadata
    return PipelineDefinition(pipeline=pipeline_yaml, info=info)
```

If you want to access your custom property from the pipeline at run-time, you
can get it from the `current_pipeline` on the `Context` object:

```yaml
steps:
  - name: pypyr.steps.py
    comment: arb_property of the current pipeline, as set by the custom pipeline loader.
    in:
      pycode: print(context.current_pipeline.pipeline_definition.info.arb_property)
  
  - name: pypyr.steps.py
    comment: arb_property on the 1st pipeline in the pipeline call-stack.
             this is useful when pypyr.steps.pype calls pipelines within pipelines.
    in:
      pycode: print(context.get_root_pipeline().pipeline_definition.info.arb_property)
```

You can access your custom metadata properties in exactly the same way on the
`Context` object in a [custom step]({{< ref "/docs/api/step" >}}).

### non-cascading loader
On [pypyr.steps.pype]({{< ref "/docs/steps/pype" >}}), if you do not want to
inherit which loader to use from the parent you can set `is_loader_cascading` to
`False` on the `PipelineInfo` object.

If you do not cascade your loader setting to child pipelines, your pipeline
authors will _explictly_ have to set `loader` on each `pype` step, even if it
is the same as the parent.

```python
from pypyr.pipedef import PipelineDefinition, PipelineInfo
import pypyr.yaml

class ArbPipeInfo(PipelineInfo):
    """Non-cascading custom pipeline meta-data."""
    # you don't HAVE to use slots, but it's a tiny bit more efficient
    __slots__ = ['arb_property']

    def __init__(self, pipeline_name, loader, parent, arb):
        # This is a non-cascading loader.
        super().__init__(pipeline_name=pipeline_name,
                         loader=loader,
                         parent=parent,
                         is_loader_cascading=False)
        self.arb_property = arb


def get_pipeline_definition(pipeline_name, parent):
    """Non-cascading because PipelineInfo has is_loader_cascading False."""
    # get & parse the pipeline yaml payload from somewhere
    with get_filelike_obj_from_somewhere(pipeline_name) as yaml_file:
        pipeline_yaml = pypyr.yaml.get_pipeline_yaml(yaml_file)
        actual_name = yaml_file.some_api_property
        actual_parent = yaml_file.another_api_property
    
    # custom meta-data to associate with pipeline
    info = ArbPipeInfo(pipeline_name=actual_name,
                       parent=actual_parent,
                       loader=__name__,
                       arb_property='some useful value')

    # wrap pipeline body in a PipelineDefinition alongside its metadata
    return PipelineDefinition(pipeline=pipeline_yaml, info=info)
```

## load child pipeline relative to parent
If wherever you store your pipelines has a hierarchical structure, you can load
child pipelines invoked by [pypyr.steps.pype]({{< ref "/docs/steps/pype" >}})
so that the child `pipeline_name` resolves relative to the parent pipeline's
location. Use the `parent` argument on
`def get_pipeline_definition(pipeline_name, parent)` to do so.

The `parent` property of the calling pipeline will cascade down to the child
by default if `is_parent_cascading` on the `PipelineInfo` instance is `True` - 
which it is by default.

```python
from pathlib import Path

from pypyr.pipedef import PipelineDefinition, PipelineInfo
import pypyr.yaml

CWD = Path.cwd()

def get_pipeline_definition(pipeline_name, parent):
    """Simple loader that gets pipeline_name.yaml in parent or working dir."""
    # pipeline_name could be "subdir/mydir/subdir/mypipe"
    pipeline_path = (parent.joinpath(f'{pipeline_name}.yaml')
                     if parent else CWD.joinpath(f'{pipeline_name}.yaml'))

    with open() as yaml_file:
        pipeline_yaml = pypyr.yaml.get_pipeline_yaml(pipeline_path)
    
    # set parent property so child pipelines can resolve relative to it
    info = PipelineInfo(pipeline_name=pipeline_name,
                        parent=pipeline_path.parent,
                        loader=__name__)

    # wrap pipeline body in a PipelineDefinition alongside its metadata
    return PipelineDefinition(pipeline=pipeline_yaml, info=info)
```

The root pipeline will always have `parent=None`. When this root pipeline calls
child pipelines with `pypyr.steps.pype`, the `pype` step will pass the calling
pipeline's `info.parent` property to the loader when retrieving the child
pipeline.

In this simple example the `parent` argument is of type `pathlib.Path`, but it
can be any type that works for you.

{{% note warn %}}
Although it us up to your loader to interpret the `parent` argument as it sees
fit, and it therefore could be any type, whatever type of object `parent` is
should have a sensible `__str__` representation.

This is a bit tough to explain with a typing hint. 😬

The reason is that pypyr caches child pipelines by the key 
`"{parent}+{pipeline_name}"`, so if the string representation of `parent` is
not unique enough, you could end up overwriting existing pipelines in cache
when you load a child pipeline with the same name but different parents.
{{% /note %}}

If your pipeline store is flat and/or you identify pipelines solely by name,
there is no need to use the `parent` property. The `parent` property is there
to deal with the complexities that arise from the file system where it is
convenient to address a child pipeline relative to the parent's location.
If wherever you store your pipelines does not allow for this, there is no point
in implementing anything special for the `parent` property.

Pipeline authors can use the `parent` input on the `pype` step to over-ride the
value of loader's `info.parent` property cascading to the child pipeline. This
will only change the value of `parent` for the child pipeline load, it does not
touch the calling pipeline's `info.parent` property. (`info.parent` refers to
the `PipelineInfo` object assigned to `.info` on the `PipelineDefinition`.)

```yaml
steps:
- name: pypyr.steps.pype
  comment: call child pipeline from here
  in:
    pype:
      name: pipeline-name # mandatory. string.
      parent: myparent
```

### multi-address pipelines
If different combinations of `parent` + `pipeline_name` could refer to the same
underlying pipeline, it is up to your loader to implement a loader specific
cache if you do not want to parse & load that pipeline >1 in a session.

For example:
- parent `/dir` + pipeline name `sub/pipe`
- parent `/dir/sub` + pipeline name `pipe`

On the filesystem, both of these is the same - both refer to `/dir/sub/pipe`.

If this is a concern for you, you can use pypyr's built-in cache class to make
your life a bit easier:

```python
from pypyr.cache.cache import Cache
from pypyr.errors import PipelineNotFoundError
from pypyr.pipedef import PipelineDefinition, PipelineInfo
import pypyr.yaml

_pipedef_cache = Cache() 

def get_pipeline_definition(pipeline_name, parent):
    """Loader with its own cache for combinations of parent+name."""
    # loader has to find UNIQUE canonical path for pipeline here somehow
    canonical_path = get_canonical_path_somehow(parent, pipeline_name)

    # if canonical_path in cache, return it. if not, put it in cache with lambda.
    pipeline_definition = _pipedef_cache.get(
        canonical_path,
        lambda: load_pipeline_from_somewhere(canonical_path))

    return pipeline_definition

def load_pipeline_from_somewhere(canonical_path):
    """Use whatever API you gotta use to get the pipeline here."""
    # if something goes wrong you might want to raise PipelineNotFoundError
    with get_filelike_obj_from_somewhere(canonical_path) as yaml_file:
        actual_name = yaml_file.some_api_property
        actual_parent = yaml_file.another_api_property
        
        # if you prefer to use your own yaml parser, do that here instead of 
        # calling get_pipeline_yaml.
        pipeline_yaml = pypyr.yaml.get_pipeline_yaml(yaml_file)

    # set parent property so child pipelines can resolve relative to it
    info = PipelineInfo(pipeline_name=actual_name,
                        parent=actual_parent,
                        loader=__name__)

    # wrap pipeline body in a PipelineDefinition alongside its metadata
    return PipelineDefinition(pipeline=pipeline_yaml, info=info)    
```

You do NOT need to worry about this additional caching complexity if a pipeline
is only addressable by one and only one combination of `parent` and
`pipeline_name` for your loader, because the pypyr core already caches the
reference returned from `get_pipeline_definition` for you. If CPU & memory are
not particularly of concern for you, then you also don't need to worry about
this. The yaml loading & parsing is the most resource intensive operation in
pypyr, so this extra cache is only really helpful if you're seeking to
optimize this by preventing >1 parsing of the same underlying pipeline in the
edge case where a different parent+name combination refers to an already loaded
pipeline.