---
title: api
description: Easily extend pypyr with the simple Python API. Create your own steps and context argument parsers. Call pypyr programmatically from your own code.
date: 2020-08-12
# categories: [developers]
menu:
  docs:
    identifier: api-overview
    name: overview
    parent: api
    weight: -100
seo_article_headline: Simple pipeline task-runner runner Python API for workflow automation.
seo_description: Easily extend pypyr with your own custom steps, cli input arguments & invoke pipelines programmatically from the API.
seo_is_carousel: true
topics: [custom code]
---
# the pypyr api
You can easily extend pypyr with your own custom steps, cli input arguments & 
you can invoke pipelines programmatically from the API.

At simplest, you can [run a pipeline]({{< ref "run-pipeline" >}}) with one
line of code.

Don't be put off & think of this as advanced functionality: it is very much the
idea of how to use pypyr effectively. Specifically making your own [custom
step]({{< ref "/docs/api/step" >}}) is likely to be a regular activity - and
happily, you can do so with a single simple function signature.
