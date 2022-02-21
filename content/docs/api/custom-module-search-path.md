---
title: custom module name resolution
# linktitle: 
description: How to reference custom modules in your pipelines.
date: 2021-11-15
# categories: [ ]
menu:
  docs:
    parent: api
    name: custom module search path
    weight: -90
seo_article_headline: How to import custom Python modules in your automation pipeline.
seo_description: You can use ad hoc custom Python modules (.py files) without doing pip install first.
topics: [ custom code ]
weight: -90
---
# reference custom modules in your pipeline
{{% note tip %}}
TLDR:

Custom modules resolve relative to the current pipeline.

Although you can also resolve from the current directory as a fallback, this
will make your pipeline less portable than it should be. 

Prefer only using absolute names relative to the pipeline itself.
{{% /note %}}

Each custom step, parser, loader & retry algo you write lives in a .py
file. In Python speak, this .py file is known as a "module".

To use these modules in your pipelines, you have to refer to the module's
absolute name:

```yaml
steps:
  - mystep # {pipeline dir}/mystep.py
  - mydir.mystep # {pipeline dir}/mydir/mystep.py
  - mydir.sub.mystep # {pipeline dir}/mydir/sub/mystep.py
```

`{pipeline dir}` is the directory on the filesystem that the pipeline is in.

## special characters
For an easy life, the more your directory & file names can stick to 
[PEP8 package & module names](https://www.python.org/dev/peps/pep-0008/#package-and-module-names)
the better. Simply put, avoid punctuation other than underscore _. Especially
avoid full-stops/periods (.).

Although pypyr is more flexible than the standard Python import rules - e.g you
can have `-` in directory/file names and it will work for pypyr custom modules
such as custom steps - be aware that if you have to do a standard `import
x.y`/`from x import y` style statement of such modules in a custom Python
code-block it will fail.

## ad hoc modules
Take _ad hoc_ modules to mean modules that are on your file-system somewhere but
that are NOT installed as packages into the Python environment.

To make your life easier, pypyr adds the current directory and also the
pipeline's parent directory to python's `sys.path`. This allows you to use
ad hoc modules relative to your current pipeline's location WITHOUT having to
install these first.

Simply put, you can just have a .py file relative to your pipeline, and pypyr
will find it without you having to do anything extra. pypyr will also look in
your current directory,

No need to package and `pip install` your code first, you can just use your
ad hoc modules on the fly!

So assuming a directory structure like this:
```text
- mydir/
    |- astep.py
    |- mypipe.yaml
    |- mypipestuff/
        |- mystep.py
```

You can reference your custom step in the pipeline like this:
```yaml
steps:
  - astep # {pipeline dir}/astep.py
  - mypipestuff.mystep # {pipeline dir}/mypipestuff/mystep.py
```

And run the pipeline like this:
{{< app-window title="term" lang="text" >}}
$ pypyr mydir/mypipe
{{< /app-window >}}

Notice that `astep` and `mypipestuff.mystep` both resolve relative to the
pipeline location itself, not the current directory.

### search paths
When you run pypyr from the CLI, pypyr automatically adds the current directory
($PWD) to `sys.path`, allowing you to use custom modules relative to the
current directory (without having to install your code as a package first).

For each pipeline you invoke with the builtin default file loader, pypyr also
adds the pipeline's parent directory to `sys.path`. This is the pipeline
directory.

The pypyr module resolution order is:
1. published packages in the current Python environment
2. directory set by `--dir`/`py_dir` (for cli this is current directory by
   default)
3. pipeline directory (for pipelines loaded with the default builtin file loader)
4. `py_dir` if set by child pipeline on [pypyr.steps.pype]({{< ref
   "/docs/steps/pype" >}})
5. pipeline directory of any child pipelines called with `pypyr.steps.pype` with
   the default builtin file loader.
   
Note that 2 - 5 might well refer to the same directory location if the pipeline
and any child pipelines it calls are all directly in the current directory, or
some of these might be equal if different pipelines in the call-chain are in the
same directory. In this case the import system only searches that directory
once.

### name hiding
A side effect of this (and indeed any) search order is that if you have .py
files with the same name in different locations, the file found earlier in the
resolution order will mask a file with the same name later in the search order.

Assuming two `mystep.py` files at different locations like this:
```text
|- mydir/
    |- mypipe.yaml
    |- mystep.py
|- mystep.py
```

And you ran this pipeline like this:
{{< app-window title="term" lang="text" >}}
$ pypyr mydir/mypipe
{{< /app-window >}}

If `mypipe` references `mystep`, it will use `./mystep.py`, not
`mydir/mystep.py`, because $PWD comes before the pipeline directory in the
search order.

If you are creating reusable child pipelines that you will call from other
pipelines with [pypyr.steps.pype]({{< ref "/docs/steps/pype" >}}), it is
therefore a good idea to put your custom code in some sort of namespace to
minimize the potential for naming clashes.

Simply put, save your custom modules inside a directory with a relatively unique
name next to your pipeline:
```text
|- mydir/
    |- mypipe.yaml
    |- mypipe/
        |- mystep.py
```

In the pipeline itself, you will reference `mypipe.mystep`
```yaml
# ./mydir/mypipe.yaml
steps:
    - mypipe.mystep
```

In this case, we are relying on the idea that the pipeline name will be
relatively unique in the pipeline call-stack, thereby serving as a namespace for
its custom modules.

It might be tempting to think of this resource directory name as a uri and use a
full-stop/period ("myarbdomain.com") - don't do this. See [special character
rules](#special-characters).

### resolve from current directory with the API
If you are calling pypyr from the API and you also want to
resolve ad hoc modules from the current directory like the CLI does, you have to
use the `py_dir` argument:

```python
from pathlib import Path
import pypyr.pipelinerunner

CWD = Path.cwd()

pypyr.pipelinerunner.run(pipeline_name='pipeline-name',
                         py_dir=CWD)
```

You do NOT need to specify `py_dir` if you only want to resolve modules
relative to the pipeline itself, because the default file-loader will add the
pipeline's parent directory to `sys.path` for you when it finds the pipeline.

### changing the custom module directory
If your pipeline's custom ad hoc modules are in a different directory, you can
use the `--dir` switch from the CLI:

{{< app-window title="term" lang="text" >}}
# run ./mydir/mypipe.yaml
# look for modules in /path/to/modules

$ pypyr --dir /path/to/modules mydir/mypipe
{{< /app-window >}}

In the API, you achieve the same thing with the `py_dir` argument of the `run()`
function:
```python
import pypyr.pipelinerunner

# run ./pipeline-name.yaml
# look for modules in /path/to/modules
pypyr.pipelinerunner.run(pipeline_name='pipeline-name',
                         py_dir='/path/to/modules')
```

## installed packages
You can reference any packages you've installed in your current python 
environment.

You normally install a package in python with `$ pip install mypackage`.

You reference modules inside packages by their absolute name:
```yaml
steps:
  - mypackage.mystep
  - name: mypackage.sub.mystep
    comment: loop over a custom step in a package
    foreach: [one, two]
```

If you want to install your own custom code as package, here is the [official
quick-start](https://setuptools.pypa.io/en/latest/userguide/quickstart.html) for
how to do so with PyPa's `setuptools`.