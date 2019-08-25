---
title: error handling
description: How to handle errors in your pipelines.
date: 2019-08-21
publishdate: 2019-08-21
lastmod: 2019-08-21
# categories: [install]
menu:
  docs:
    parent: getting-started
draft: false
seo_article_headline: Overview of error handling in a task-runner pipeline.
seo_description: Retry, swallow or ignore errors in a pipeline. Add extra diagnostic info to exceptions.
topics: [error handling]
---
# error handling
pypyr runs pipelines. . . and a pipeline is a sequence of steps. If 
your desired behavior is for pipeline processing to stop and subsequent steps 
NOT to run once an error occurs somewhere, you don't have to do anything 
special, because this is pypyr's default. 

pypyr assumes that any error is a hard stop, unless you explicitly tell pypyr 
differently by setting the [swallow step decorator]({{< ref "/docs/decorators/swallow" >}}) 
to `True`.

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

You can use `runErrors` in your step-group's failure handler, or if you set 
`swallow=True` on the failing step you can use `runErrors` in subsequent
steps to provide exact error information.