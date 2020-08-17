---
title: pipeline look-up order
linktitle: lookup order
description: pypyr looks for pipeline files & custom code modules on disk in these directories.
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
categories: [pipelines]
menu:
  docs:
    parent: pipelines
    name: lookup order
seo_article_headline: Directory lookup order for task-runner pipelines on the filesystem.
seo_description: Search for a matching pipeline first in the working directory & alternate location lookup sequence.
# topics: [control-of-flow]
---
# pipeline look-up order
## the working directory
pypyr first looks for pipelines, any custom steps & other code in the 
current working directory. This is the directory from which you invoke pypyr.

Simply put, by default, this is the directory you're currently in when you 
invoke pypyr from the cli.

You can change the default working directory by passing the `--dir` flag to 
pypyr CLI. pypyr will use whatever path you specify in `--dir` as the working 
directory base path. If you're using the API, use the `working_dir` argument 
for the same effect.

Changing the working directory not only means that pypyr looks for pipelines in 
that location first, but will also resolve any custom code modules like your own 
steps or context-parsers from there.

## directory locations lookup order
By default pypyr uses a file loader to find & load pipelines from the 
filesystem. pypyr searches a specific sequence of locations for pipelines 
matching the pipeline name:

1. `{working dir}/`
2. `{working dir}/pipelines`
3. `{pypyr install dir}/pipelines`

`{pypyr install dir}` is where-ever you installed the currently running 
instance of pypyr. Very likely, this is either in your default python path, or 
the currently active virtual environment:

```text
{your python environment}/lib/python3.8/site-packages/pypyr/pipelines/
```

{{% note tip %}}
The last look-up  in `{pypyr-install-dir}` is for pypyr built-in pipelines. You 
probably shouldn't be saving your own pipelines there, they might get 
over-written or wiped by upgrades or re-installs.
{{% /note %}}

## pipeline name matching
To make your life easier when you edit your pipelines in your favorite yaml 
editor, do use the .yaml extension for your pipeline files.

Still to make your life easier, don't add the .yaml when you invoke pypyr with
your pipeline-name. pypyr always appends .yaml to your input pipeline name for 
you under the hood.

When you run:
{{< app-window title="term" lang="fish" >}}
$ pypyr pipeline-name
{{< /app-window >}}

pypyr will look in these locations in this order:

1. `./pipeline-name.yaml`
2. `./pipelines/pipeline-name.yaml`
3. `{pypyr-install-dir}/pipelines/pipeline-name.yaml`

## pipelines in sub-directories
To organize your pipelines, you can save your pipelines in whatever directory 
structure you please.

```text
./
    |-- pipe-0.yaml
    |-- dir
         |-- subdir1
             |-- pipe-1.yaml
             |-- pipe-2.yaml
         |-- subdir2
             |-- pipe-3.yaml
         |-- pipe-4.yaml
```

You can run these pipelines like this:

{{< app-window title="term" lang="fish" >}}
$ pypyr pipe-0

$ pypyr dir/subdir1/pipe-1

$ pypyr dir/subdir1/pipe-2

$ pypyr dir/subdir2/pipe-3

$ pypyr dir/pipe-4
{{< /app-window >}}


You can achieve the same effect for everything in the `dir` sub-directory by 
using the `--dir` flag. The difference is that if you use the `--dir` flag 
pypyr will also load any custom python modules from this directory.

{{< app-window title="term" lang="fish" >}}
$ pypyr --dir dir subdir1/pipe-1

$ pypyr --dir dir subdir1/pipe-2

$ pypyr --dir dir subdir2/pipe-3

$ pypyr --dir pipe-4
{{< /app-window >}}