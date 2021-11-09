---
title: custom context parser
linktitle: context parser
description: Create your own context parser to parse cli input arguments.
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
# card_extra_summary:
#   heading: input context property
#   details: "`assert` (dict)"
# categories: [ context ]
menu:
  docs:
    parent: api
    name: context parser
seo_article_headline: Parse custom cli arguments for a task-runner pipeline.
seo_description: Customize how you parse cli input arguments when you run a task-runner pipeline from the cli.
topics: [ custom code, context ]
---
# create a custom context parser
## parse custom cli arguments
A `context_parser` parses the pypyr cli input arguments. Simply put, this is all 
the positional arguments after the pipeline-name in the cli.

```bash
$ pypyr pipelinename this is the args input
```

In this example, `['this', 'is', 'the', 'args', 'input']` will go to the 
pipeline's context parser as input.

Generally, a `context_parser` is likely to take the input args list and create
a `dict` with it somehow. pypyr will use this `dict` to initialize context for 
the pipeline run.

{{% note tip %}}
For real-life inspiration, see the actual code for pypyr's built-in 
[context parsers](https://github.com/pypyr/pypyr/tree/main/pypyr/parser).
{{% /note %}}

## parser function signature & example
```python
import logging


# getLogger will grab the parent logger context, so your loglevel and
# formatting will inherit correctly automatically from the pypyr core.
logger = logging.getLogger(__name__)


def get_parsed_context(args):
    """This is the signature for a context parser.

    Args:
      args: list of string. Passed from command-line invocation where:
            $ pypyr pipelinename this is the context_arg
            This would result in args == ['this', 'is', 'the', 'context_arg']

    Returns:
      dict. This dict will initialize the context for the pipeline run.
    """
    assert args, ("pipeline must be invoked with context arg set.")
    logger.debug("starting")

    # your clever code here. Chances are pretty good you'll be doing things
    # with the input args list to create a dictionary.

    # function signature returns a dictionary
    return {'key1': 'value1', 'key2':'value2'}
```

## use custom parser in a pipeline
The usual python import module resolution rules apply. pypyr will resolve 
modules from the 
[working directory]({{< ref "/docs/pipelines/lookup-order#the-working-directory" >}}) 
first, which you can change by using the `--dir` flag.

Assuming you saved your python with the `def get_parsed_context(args)` in a 
file like this `./dir/myparser.py`, you can use use it in your pipeline like 
this:

```yaml
context_parser: dir.myparser
steps:
    - step1
    - step2
```

If you package your parser and you install the package into the active python 
environment, you can of course use the usual python package name instead:

```yaml
context_parser: mypackage.myparser
steps:
    - step1
    - step2
```