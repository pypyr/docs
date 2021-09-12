---
title: control-of-flow
description: Flow control for your pipelines. Conditional execution, loops, jump between steps & stop processing.
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
seo_article_headline: Control-of-Flow for a task-runner pipeline.
seo_description: Conditional execution, loops, jump between steps & stop processing instructions in a pipeline.
seo_is_carousel: true
---
# control-of-flow
You can control the flow of pypyr pipeline execution between step-groups
with the following handy steps:

- [pypyr.steps.call]({{< ref "/docs/steps/call">}})
- [pypyr.steps.jump]({{< ref "/docs/steps/jump">}})
- [pypyr.steps.stopstepgroup]({{< ref "/docs/steps/stopstepgroup">}})
- [pypyr.steps.stoppipeline]({{< ref "/docs/steps/stoppipeline">}})
- [pypyr.steps.stop]({{< ref "/docs/steps/stop">}})

You can call other pipelines from within a pipeline with:

- [pypyr.steps.pype]({{< ref "/docs/steps/pype">}})

On top of this, you can control which individual steps should run or not
using the conditional step decorators:

- [run]({{< ref "/docs/decorators/run">}})
- [skip]({{< ref "/docs/decorators/skip">}})

Looping happens on the step-level, using the following step decorators:

- [while]({{< ref "/docs/decorators/while">}})
- [foreach]({{< ref "/docs/decorators/foreach">}})

You can set a `while` or `foreach` loop on any given step, including on
a [call]({{< ref "/docs/steps/call">}}) step or a 
[pype]({{< ref "/docs/steps/pype">}}) step, which lets you call another
step-group or an entire pipeline repeatedly in a loop.

## control-of-flow instructions