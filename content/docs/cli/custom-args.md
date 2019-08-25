---
title: custom args
description: Passing custom args from the cli to your pipeline.
date: 2019-08-21
publishdate: 2019-08-21
lastmod: 2019-08-21
categories: [cli]
menu:
  docs:
    name: custom args
    parent: cli
draft: false
seo_article_headline: Passing custom args with the pipeline task-runner cli.
seo_description: Pass your own custom optional arguments from the cli to the task-runner pipeline & set your own default values.
topics: [args]
---
# pass custom optional arguments from the cli
## pass custom arguments to a pipeline from the cli
You can easily pass your own arguments to your pipelines from the cli. You can 
decide per pipeline what style of arguments work better for that specific
pipeline, for example key-value pairs, or a single string, or boolean switches.

See [context parsers]({{< ref "/docs/context-parsers/">}}) for more information
on how to pass your own arguments to a pipeline and using the runtime values in 
the pipeline.

## boolean switches
```yaml
# ./defaultarg.yaml
#
# To execute this pipeline, shell something like:
# pypyr defaultarg arb
#
# And to run with isRetry
# pypyr defaultarg isRetry
#
# And to run with isCI:
# pypyr defaultarg isCI
#
# And to run with both Retry and CI:
# pypyr defaultarg isCI isRetry
context_parser: pypyr.parser.keys
steps:
  - name: pypyr.steps.default
    in:
      defaults:
        isCI: False
        isRetry: False
  - name: pypyr.steps.echo
    run: '{isRetry}'
    in:
      echoMe: you'll only see me if IsRetry is True
  - name: pypyr.steps.echo
    run: '{isCI}'
    in:
      echoMe: this only runs in the CI environment, not on dev
  - name: pypyr.steps.echo
    in:
      echoMe: done!
```