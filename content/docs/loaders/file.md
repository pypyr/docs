---
title: pypyr.loaders.file
linktitle: file
description: The default file loader discovers and loads pipelines from the filesystem.
menu:
  docs:
    parent: loaders
    name: file
weight: -10
seo_article_headline: The file loader finds pipelines in the filesystem
seo_description: Load pipelines from the filesystem.
topics: [lookup-order]
---
# file loader
This is pypyr's default loader. It searches for pipelines on local disk based
on the pipeline-name and loads the pipeline when it finds it.

The important thing to understand with this loader is how pypyr matches the
pipeline name and the [look-up order]({{<
ref "/docs/pipelines/lookup-order" >}}) in which it traverses the filesystem
to resolve a pipeline name.

The loader's fully qualified name is `pypyr.loaders.file`. You're very unlikely
to have to specify this manually yourself, because if you don't specify any
value for [loader in the input args of the api]({{< ref
"/docs/api/run-pipeline#input-args" >}}) pypyr will use this loader by default.
