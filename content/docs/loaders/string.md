---
title: pypyr.loaders.string
linktitle: string
description: Inject pipelines directly as strings.
menu:
  docs:
    parent: loaders
    name: string
    identifier: loaders-string
weight: -10
seo_article_headline: Load pypyr pipelines from strings.
seo_description: Load pipelines directly from a string rather than have to create a file first.
topics: [lookup-order]
---
# string loader
Use the string loader to inject a pipeline directly into pypyr without needing
to save it to file first.

Specify `pypyr.loaders.string` for `loader`, and pass the pipeline body as a
string to the `pipeline_name` argument.

```python
from pypyr import pipelinerunner

pipeline = """\
steps:
- name: pypyr.steps.set
  in:
    set:
      test: 1
"""

context = pipelinerunner.run(pipeline_name=pipeline,
                             loader='pypyr.loaders.string')

assert context['test'] == 1
print(context)
```