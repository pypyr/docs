---
title: pipeline look-up order
linktitle: lookup order
description: pypyr looks for pipeline files & custom code modules on the filesystem in these directories.
date: 2020-08-12
publishdate: 2020-08-13
categories: [pipelines]
menu:
  docs:
    parent: pipelines
    name: lookup order
weight: -20
seo_article_headline: Directory lookup order for task-runner pipelines on the filesystem.
seo_description: Use absolute or relative paths to find a matching pipeline in the file location lookup sequence.
topics: [lookup-order]
---
# pipeline look-up order
## absolute vs relative paths
You can pass absolute or relative paths to pypyr.

{{< tabs id="abs-v-rel" >}}
{{< tab name="posix" >}}
{{< app-window title="term" lang="fish" >}}
$ pypyr pipeline-name # relative path: ./pipeline-name.yaml

$ pypyr subdir/pipeline-name # relative path: ./subdir/pipeline-name.yaml

$ pypyr /subdir/pipeline-name # absolute path: /subdir/pipeline-name.yaml

$ pypyr ~/subdir/pipeline-name # absolute path: /Users/username/subdir/pipeline-name.yaml
_
{{< /app-window >}}
{{< /tab >}}
{{< tab name="windows" >}}
{{< app-window title="term" lang="fish" >}}
$ pypyr pipeline-name # relative path: .\pipeline-name.yaml

$ pypyr subdir/pipeline-name # relative path: .\subdir\pipeline-name.yaml

$ pypyr c:/subdir/pipeline-name # absolute path: c:\subdir\pipeline-name.yaml

$ pypyr $home/subdir/pipeline-name # absolute path: C:\Users\username\subdir\pipeline-name.yaml
_
{{< /app-window >}}
{{< /tab >}}
{{< /tabs >}}

For relative paths, pypyr first looks for pipelines, any custom steps & other
code in the current working directory. This is the directory from which you
invoke pypyr.

Simply put, by default, this is the directory you're currently in when you 
invoke pypyr from the cli.

{{< note tip >}}
Windows users, you can specify paths with / or \ as directory
separator. pypyr will resolve either correctly for you.

When you're working with files and you want to have a
cross-platform compatible pipeline and you're using relative paths,
front-slash / is the way to go.
{{< /note >}}

## directory locations lookup order
pypyr first checks for a matching [shortcut]({{< ref "/docs/pipelines/shortcuts"
>}}) name.

If no matching shortcut exists, pypyr moves on to look for the pipeline with
the default loader.

By default pypyr uses the built-in file loader to find & load pipelines from the
filesystem. 

For absolute paths, pypyr just looks for the pipeline at that specific location.

For relative paths, pypyr searches a specific sequence of locations
for pipelines matching the pipeline name:

1. `{current dir}/`
2. `{current dir}/pipelines/`
3. `{pypyr install dir}/pipelines/`

Instead of `{current dir}/pipelines/` you can set the name of the sub-directory
in {current dir} using [pipelines_subdir in config]({{< ref
"/docs/getting-started/config#pipelines_subdir" >}}).

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
you under the hood, for both absolute and relative paths.

When you run:
{{< app-window title="term" lang="fish" >}}
$ pypyr pipeline-name
{{< /app-window >}}

pypyr will look in these locations in this order:

1. [shortcut name]({{< ref "/docs/pipelines/shortcuts#shortcut-name" >}}) == `pipeline-name`
2. `./pipeline-name.yaml`
3. `./pipelines/pipeline-name.yaml`
4. `{pypyr-install-dir}/pipelines/pipeline-name.yaml`

## pipelines in sub-directories
To organize your pipelines, you can save your pipelines in whatever directory 
structure you please.

```text
./
    |-- pipe-0.yaml
    |-- mydir
         |-- subdir1
             |-- pipe-1.yaml
             |-- pipe-2.yaml
         |-- subdir2
             |-- pipe-3.yaml
         |-- pipe-4.yaml
```

You can run these pipelines like this:

{{< app-window title="term" lang="text" >}}
$ pypyr pipe-0

$ pypyr mydir/subdir1/pipe-1

$ pypyr mydir/subdir1/pipe-2

$ pypyr mydir/subdir2/pipe-3

$ pypyr mydir/pipe-4
{{< /app-window >}}

## shared pipeline libraries
If you had a shared pipeline library on your file-system, for example at
`/Users/captainhook/shared-pipelines/`, you can run your pipelines from
anywhere else like this:

{{< tabs id="shared-pipe-libs" >}}
{{< tab name="posix" >}}
{{< app-window title="term" lang="fish" >}}
# print current dir
$ echo $PWD
/git/myproject

# run pipeline in another dir
$ pypyr ~/shared-pipelines/subdir/my-shared-pipe
{{< /app-window >}}
{{< /tab >}}
{{< tab name="windows" >}}
{{< app-window title="term" lang="text" >}}
# assuming powershell
# print current dir
$ echo $pwd
Path
----
C:\git\myproject

# run pipeline in another dir
$ pypyr $home/shared-pipelines/subdir/my-shared-pipe
{{< /app-window >}}
{{< /tab >}}
{{< /tabs >}}

The above example will run `~/shared-pipelines/subdir/my-shared-pipe.yaml`. Any
child pipelines or custom code modules called by `my-shared-pipe` will resolve
relative to `my-shared-pipe`'s location.

```text
/Users
    |- captainhook/
        |- shared-pipelines/
            |- subdir/
                |- my-shared-pipe.yaml
                |- mystep.py
                |- sub-pipe.yaml
```

This means you can code your pipeline to be portable by calling child pipelines
or custom python modules relative to the parent pipeline's location on the
file-system. Here is `my-shared-pipe`, using a child pipeline and a custom step
relative to itself:

```yaml
# ~/shared-pipelines/subdir/my-shared-pipe.yaml
steps:
    - name: pypyr.steps.pype
      comment: call child pipeline sub-pipe.yaml
               resolves relative to current pipeline dir.
      in:
        pype:
            name: sub-pipe # sub-pipe in same dir as current pipeline
    - mystep # looks for mystep.py relative to current pipeline
```

You can run `my-shared-pipe` from any location and it will work - all its
references resolves relative to itself.