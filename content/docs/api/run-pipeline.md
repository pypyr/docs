---
title: run pipeline api
# linktitle: 
description: Run pipeline programmatically from the API in your own code.
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
# categories: [ ]
menu:
  docs:
    parent: api
    name: run pipeline
seo_article_headline: Run pipeline from API in your own code.
seo_description: Use the Python API to run task-runner pipeline workflows from your own code in a few simple lines.
topics: [ custom code ]
---
# run a pipeline from the api
## main entrypoint api
The main entrypoint for running your own pipeline is `main` in the 
`pypyr.pipelinerunner` module.

```python
import pypyr.pipelinerunner

pypyr.pipelinerunner.main(pipeline_name='arb pipe',
                          pipeline_context_input='arb context input',
                          working_dir='arb/dir',
                          groups=['group'],
                          success_group='success_group',
                          failure_group='failure_group')
```

Call this once per pypyr pipeline. Call me if you want to run a pypyr pipeline
from your own code. This function does some one-off 1st time initialization
before running the actual pipeline.

If you're invoking pypyr from your own application via the API,
it's your responsibility to set up and configure logging. If you just want
to replicate the log handlers & formatters that the pypyr cli uses, you can
call `pypyr.log.logger.set_root_logger()` before invoking 
`pipelinerunner.main()`.

Be aware that if you invoke this method, pypyr adds a `NOTIFY` - `25` custom
log-level and `notify()` function to logging.

`{pipeline_name}.yaml` is relative to the the working_dir directory.

### input args
- `pipeline_name`: string.
    - Name of pipeline, sans .yaml at end.
- `pipeline_context_input`: string. 
    - Initialize the pypyr context with this string.
- `working_dir`: path-like. 
    - Look for pipelines and modules in this directory.
- `groups`: list of string. Optional. 
    - Step-group names to run in pipeline. 
    - Default to `steps` if not specified.
- `success_group`: string. Optional. 
    - Step-group name to run on success completion. 
    - Default to `on_success` if not specified.
- `failure_group`: string. Optional. 
    - Step-group name to run on pipeline failure. 
    - Default to `on_failure` if not specified.

### returns
None

## invoke pipeline from api example
```python
from pathlib import Path
import pypyr.pipelinerunner

# as API consumer, it is your responsibility to configure logging.
# if you just want to use pypyr's default logging, do this:
pypyr.log.logger.set_root_logger(log_level=25,
                                 log_path=None)

# run pipeline exactly like the cli would do
pypyr.pipelinerunner.main(pipeline_name='arb pipe',
                          pipeline_context_input='arb context input',
                          working_dir=Path.cwd())
```