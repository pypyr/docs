---
title: pipeline shortcuts
linktitle: shortcuts
description: Create re-usable shortcuts to pipeline run commands in config.
date: 2022-03-07
categories: [pipelines]
menu:
  docs:
    parent: pipelines
    name: shortcuts
weight: -10
seo_article_headline: Save shortcuts to pipeline run commands in config.
seo_description: Save shortcuts to complex pipeline taskrunner automation command sequences.
# topics: [control-of-flow]
---
# shortcuts
You can create shortcuts to complex command inputs in [pypyr config]({{< ref
"/docs/getting-started/config" >}}).

A shortcut allows you to save longer command sequences so you can use a friendly
short alias to run a pipeline with complex input arguments.

As with all pypyr config, you can create your shortcut in any of the yaml
config files or in `pyproject.toml`.

{{< tabs id="yaml-vs-toml" >}}
{{< tab name="config.yaml" >}}
```yaml
shortcuts:
  sc1:
    pipeline_name: /mydir/my-pipeline
    args:
      akey: a value
      anotherkey: 123
  sc2:
    pipeline_name: /mydir/another-pipeline
    args:
      boolinput: true
      mylist:
        - one
        - two
```
{{< /tab >}}
{{< tab name="pyproject.toml" >}}
```toml
[tool.pypyr.shortcuts]
[tool.pypyr.shortcuts.sc2]
    pipeline_name = "/mydir/my-pipeline"
    args = {akey = "a value", anotherkey = 123 }
[tool.pypyr.shortcuts.sc3]
    pipeline_name = "/mydir/another-pipeline"
    args = {boolinput = true, mylist = ["one", "two"] }
```
{{< /tab >}}
{{< /tabs >}}

You can now use the shortcut name to run the pipeline specified by
`pipeline_name` and pass its `args` to it without all the typing:

{{< app-window title="term" lang="fish" >}}
$ pypyr sc1

$ pypyr sc2
{{< /app-window >}}

## schema
These are all the available shortcut properties. The yaml and toml is
structurally equivalent so you can directly convert between the formats as you
prefer.

{{< tabs id="schema" >}}
{{< tab name="yaml" >}}
```yaml
shortcuts:
  my-shortcut-name:
    pipeline_name: pipe name here
    parser_args: ['arg1', 'arg2=value2']
    skip_parse: true
    args:
      a: b
      c: d
    groups: ['steps']
    success: 'on_success'
    failure: 'on_falure'
    loader: pypyr.loaders.file
    py_dir: /arb/dir
```
{{< /tab >}}
{{< tab name="toml" >}}
```toml
[tool.pypyr.shortcuts]
[tool.pypyr.shortcuts.my-shortcut-name]
pipeline_name = "pipe name here"
parser_args = [ "arg1", "arg2=value2" ]
skip_parse = true
args = {a = "b", c = "d"}
groups = [ "steps" ]
success = "on_success"
failure = "on_falure"
loader = "pypyr.loaders.file"
py_dir = "/arb/dir"
```
{{< /tab >}}
{{< /tabs >}}

Note that for either yaml or toml you could use the inline format for maps (aka
tables) or lists (aka arrays) as you see fit. For example, the toml example
shows inline tables for `args` just for the sake of brevity, but you could use
the standard full syntax if you prefer.

### shortcut name
The shortcut name is the key that identifies the shortcut. You pass the name
from the cli or the [api]({{< ref"/docs/api/run-pipeline#shortcuts" >}}) to
invoke the shortcut.

The shortcut name here is `my-shortcut-name`, meaning you can run this shortcut
as follows:

{{< app-window title="term" lang="fish" >}}
$ pypyr my-shortcut-name
{{< /app-window >}}

### pipeline_name
The pipeline name is the only required property for a shortcut.

This is the absolute or relative path to the pipeline you want to run (only
without the .yaml at the end). pypyr always appends .yaml to your input pipeline
name for you behind the scenes, for both absolute and relative paths.

Remember that relative paths will resolve relative to your current working
directory. This might not be what you want if you're creating a shortcut in
your user or global config files: since you could invoke a shortcut defined in
the user or global config from any given directory on the filesystem.

| pipeline_name | resolves to location |
|---------------|----------------------|
| my-pipe | ./my-pipe.yaml
| mydir/my-pipe | ./mydir/my-pipe.yaml
| /mydir/my-pipe | /mydir/my-pipe.yaml
| c:/mydir/my-pipe | c:\\mydir\\my-pipe.yaml

Take `./` to mean whatever the current working directory is.

Windows users, you can use either \ or / for paths, it doesn't matter.

How the pipeline_name resolves depends on the [loader](#loader). The default
loader `pypyr.loaders.file` will follow the usual [pipeline look-up order]({{<
ref "lookup-order" >}}) sequence to find the pipeline on the filesystem.

### args
Initialize the pipeline context with `args`. This allows you to inject variables
into your pipeline.

`args` is a dict (aka a mapping in yaml, or a table in toml).

You can use complex multi-level nested hierarchies. pypyr will honor the
data-types from the yaml/toml - in other words, string, booleans, integers etc.

If you specify your pipeline input variables with `args`, pypyr will disable the
`context_parser` for you. If you still want the pipeline's `context_parser` to
run, either also specify `parser_args` or set `skip_parse` to `False`.

{{< tabs id="args" >}}
{{< tab name="yaml" >}}
```yaml
shortcuts:
  my-shortcut-name:
    pipeline_name: my-pipeline # this will run ./my-pipeline.yaml
    args:
      key1: value1 # string
      key2: # nested map
        key2.1: value2.1
        key2.2: # list
          - list item 1
          - list item 2
      key3: 123 # integer
      key4: False # bool
      key5: 'BEGIN {expression} END' 
```
{{< /tab >}}
{{< tab name="toml" >}}
```toml
[shortcuts.my-shortcut-name]
pipeline_name = "my-pipeline"

[shortcuts.my-shortcut-name.args]
key1 = "value1"
key3 = 123
key4 = false
key5 = "BEGIN {expression} END"

[shortcuts.my-shortcut-name.args.key2]
"key2.1" = "value2.1"
"key2.2" = [ "list item 1", "list item 2" ]
```
{{< /tab >}}
{{< /tabs >}}

### parser_args
This is a list/array of string arguments to pass to the pipeline's
[context_parser]({{< ref "/docs/context-parsers" >}}).

This list contains each argument you would normally have passed from the cli
following the pipeline name. If you're trying to figure out how to convert your
cli call to a `parser_args` list, the principle is that the cli splits the input
string by spaces. I.e each arg is separated by a space, unless it's escaped
inside quotes.

Assuming you have a pipeline that you would call from the cli like this:

```fish
$ pypyr my-pipeline arg1=value1 arg2="value 2" arg3=123
```

The exact shortcut equivalent is:
{{< tabs id="parser-args" >}}
{{< tab name="yaml" >}}
```yaml
shortcuts:
  my-shortcut-name:
    pipeline_name: my-pipeline
    parser_args: ['arg1=value1', 'arg2=value 2', 'arg3=123']
```
{{< /tab >}}
{{< tab name="toml" >}}
```toml
[tool.pypyr.shortcuts]
[tool.pypyr.shortcuts.my-shortcut-name]
pipeline_name = "my-pipeline"
parser_args = ["arg1=value1", "arg2=value 2", "arg3=123"]
```
{{< /tab >}}
{{< /tabs >}}

pypyr only uses these if [skip_parse](#skip_parse) is `False`. pypyr will
automatically implicitly set `skip_parse` to `False` for you whenever you
specify `parser_args`.

If you explicitly set `skip_parse` to `True`, the pipeline will skip the context
parser even if you set a value for `parser_args`.

If you combine `args` and `parser_args` in the same shortcut, pypyr will merge
values resulting from both into the pipeline's context.

{{% note tip %}}
Should you use `args` or `parser_args `to pass parameters to the pipeline?

It doesn't really matter. `args` is marginally more efficient, but given typical
pipeline functionality you're very unlikely to notice performance improvements
here.

`args` gives you more flexibility to initialize your context with complex,
nested structures. Such structures are generally harder to do with `parser_args`
because the context parser has to parse its values from a flat string input to
the cli. But if your pipeline inputs map easily onto cli context arg inputs,
that might not be all that relevant to you either.

So pick whichever works for you, it doesn't really matter. You can combine both
at the same time too.

{{% /note %}}

### skip_parse
If `True`, skip the `context_parser` on the pipeline. 

Your pipeline might use a [context_parser]({{< ref "/docs/context-parsers" >}})
to initialize context arguments you pass in from the cli.

But if you use the [args](#args) property on your shortcut directly to set the
input arguments for your pipeline, you probably want to bypass the context
parser. In this case your pipeline's inputs come directly from the shortcut's
`args` and not from the cli.

If you still want the pipeline's `context_parser` to run, set `skip_parse` to
`False`. When you set `skip_parse` to `False` , use [parser_args](#parser_args)
to pass cli-style input arguments to the pipeline's `context_parser`.

The default value for `skip_parse` depends on whether you specified
`parser_args` or not. If you set `parser_args`, `skip_parse` will always default
to `True`. If you set `args` instead and `parser_args` is not set, `skip_parse`
will default to `False`.

You are very unlikely to need to set this property yourself. A reason would be
if your `parser_args` are not specified but you still want the `context_parser`
to run.

### groups
This is a list/array of the [step groups]({{< ref
"/docs/pipelines/pipeline-structure#step-groups" >}}) to run in the pipeline.

Equivalent to `groups` arg on the pypyr cli.

If you don't set this, pypyr will just run the `steps` step-group by default as
per usual.

If you only want to run a single group, you can set it simply as a string, not 
a list, like this:

{{< tabs id="schema-groups" >}}
{{< tab name="yaml" >}}
```yaml
my-shortcut:
  pipeline_name: my-pipeline # this will run ./my-pipeline.yaml
  groups: mygroupname
```
{{< /tab >}}
{{< tab name="toml" >}}
```toml
[tool.pypyr.shortcuts.my-shortcut]
pipeline_name = "my-pipeline"
groups = "steps"
```
{{< /tab >}}
{{< /tabs >}}

If you set `groups`, `success` and `failure` do not default to `on_success` and 
`on_failure` anymore. In other words, pypyr will only run the groups you 
explicitly specified. If you still want success/failure handlers explicitly 
set these with `success` & `failure`.

### success
Run this step-group on successful completion of the pipeline's
[groups](#groups).

Equivalent to `success` arg on the pypyr cli.

If you don't set this, pypyr will just run the `on_success` step-group as per 
usual (if it exists in the pipeline), assuming you did not also set `groups`.

If you specify `success`, but you don't set `groups`, pypyr will default to 
running the standard `steps` group as entry-point the pipeline.

### failure
Run this step-group on an error occurring in the pipeline.

Equivalent to `failure` arg on the pypyr cli.

If you don't set this, pypyr will just run the `on_failure` step-group as per 
usual (if it exists in the pipeline), assuming you did not also set `groups`.

If you specify `failure`, but you don't set `groups`, pypyr will default to 
running the standard `steps` group as entry-point for the pipeline.

### loader
Load the pipeline with this loader. This is useful if you are using a [custom
loader]({{< ref "/docs/api/pipeline-loader" >}}) to fetch your pipelines from
your own external systems or locations.

The default pipeline loader is the built-in `pypyr.loaders.file`, which looks
for pipelines on the file system. You can set the [default loader for all of
pypyr in config]({{< ref "/docs/getting-started/config#default_loader" >}}).

You only need to set this property if you do not want to use the default for
the particular pipeline referenced by this shortcut.

### py_dir
Load ad hoc custom python modules from this directory. You only need to set this
if your pipeline imports custom modules that do not resolve from the pipeline's
parent directory AND the modules are not installed to the current python
environment.

By default, the current directory ($PWD) and the pipeline's directory are in the
module resolution path by already, so if your custom modules are relative to
these locations you do NOT need to set `py_dir`.

{{< app-window title="term" lang="fish" >}}
$ cd mydir
# run mydir/subdir/mypipeline.yaml
$ pypyr subdir/my-pipeline
{{< /app-window >}}

In this example, the current directory `mydir` and the pipeline's directory
`mydir/subdir` are in the custom module resolution lookup locations already, so
you do not need to set `py_dir` if your custom modules resolve relative to any
of these locations.

See [custom module import resolution rules]({{< ref
"/docs/api/custom-module-search-path" >}}) for details.

## combine shortcuts from multiple config files
You can create your shortcuts in any of the local, user or global config files.
1. `./pypyr-config.yaml` - project/local
2. `./pyproject.toml` - project/local
3. `~/.config/pypyr/config.yaml` - user
4. `{global config dirs}/pypyr/config.yaml` - global

pypyr will merge the `shortcuts` found in all the available pypyr config files
on your system. So you could create some shortcuts in the global config file,
some in the user config file and some project specific ones in your current
project's directory. The merge order means you can create a system-wide global
shortcut in the global config file, but override it on a per-user or per-project
basis. pypyr follows a [configuration file look-up sequence]({{< ref
"/docs/getting-started/config#config-file-locations" >}}) where pypyr will
merge the config from the various config sources in order of precedence to
produce a final effective combined config.

Note that pypyr merges the [shortcut name](#shortcut-name), not the contents of
the shortcut. The shortcut name is treated as the merge key. So if you had
`myshortcut` in global config and also in your project config, `myshortcut` from
the project config will supersede the global config's `myshortcut` entirely. It
won't additively merge the shortcut's child keys, like `args`.

## bypass context parser
When you set `args` instead of `parser_args`, you bypass the `context_parser`
of the pipeline.

You probably want your shortcut pipeline to remain directly runnable from the
cli, but also to initialize your pipeline variables from the shortcut with
`args`. Not to worry, this is easy. Just set your `args` to whatever your
`context_parser` would have constructed.

### pypyr.parser.string
For example, the [pypyr.parser.string]({{< ref "/docs/context-parsers/string">}})
parser creates an `argString` key in context. Since the context parser won't
run if you specify `args`, your `args` just have to create the same variable.

```yaml
# To execute this pipeline, shell something like:
# pypyr my-pipeline text goes here
context_parser: pypyr.parser.string
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: '{argString}'
```

You can encode this in a shortcut as follows:

{{< tabs id="inject-argstring" >}}
{{< tab name="yaml" >}}
```yaml
shortcuts:
  my-shortcut:
    pipeline_name: my-pipeline
    args:
      argString: is that a penguin?
```
{{< /tab >}}
{{< tab name="toml" >}}
```toml
[tool.pypyr.shortcuts.my-shortcut]
pipeline_name = "my-pipeline"
args = { argString = "is that a penguin?" }
```
{{< /tab >}}
{{< /tabs >}}

You can now run the shortcut and you can also still directly run the pipeline
from the cli like this:

{{< app-window title="term" lang="fish" >}}
# run shortcut
$ pypyr my-shortcut
is that a penguin?

# run same pipeline directly
$ pypyr my-pipeline "is that a penguin?"
is that a penguin?
{{< /app-window >}}

If you prefer, you can have your config treat the input args like the cli does
by setting `parser_args` instead of `args`:

{{< tabs id="argstring-context-args" >}}
{{< tab name="yaml" >}}
```yaml
shortcuts:
  my-shortcut:
    pipeline_name: my-pipeline
    parser_args: ["is that a penguin?"]
```
{{< /tab >}}
{{< tab name="toml" >}}
```toml
[tool.pypyr.shortcuts.my-shortcut]
pipeline_name = "my-pipeline"
parser_args = [ "is that a penguin?" ]
```
{{< /tab >}}
{{< /tabs >}}

In this case the `context_parser` creates the `argString` variable in context
for you.

### pypyr.parser.keyvaluepairs
Now let's follow the same principle with the [pypyr.parser.keyvaluepairs]({{<
ref "/docs/context-parsers/keyvaluepairs">}}) parser.

This parser takes key=value pairs from the cli arg input and creates these
as variables in the pypyr context.

```yaml
# ./my-pipeline.yaml
context_parser: pypyr.parser.keyvaluepairs
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: Piping {var1} the {var2}
```

When you create a shortcut for this pipeline with `args`, you set the
variables yourself:

{{< tabs id="inject-kvp" >}}
{{< tab name="yaml" >}}
```yaml
shortcuts:
  my-shortcut:
    pipeline_name: my-pipeline
    args:
      var1: down
      var2: valleys wild
```
{{< /tab >}}
{{< tab name="toml" >}}
```toml
[tool.pypyr.shortcuts.my-shortcut]
pipeline_name = "my-pipeline"
args = { var1 = "down", var2="valleys wild" }
```
{{< /tab >}}
{{< /tabs >}}

You can now run the shortcut and also still directly run the pipeline from the
cli like this:

{{< app-window title="term" lang="fish" >}}
# run shortcut
$ pypyr my-shortcut
Piping down the valleys wild

# run same pipeline directly
$ pypyr my-pipeline var1=down var2="valleys wild"
Piping down the valleys wild
{{< /app-window >}}

If you wanted to achieve the same thing with `parser_args` instead and rely
on the context_parser to initialize context for you, the shortcut looks like
this:

{{< tabs id="inject-kvp-parser" >}}
{{< tab name="yaml" >}}
```yaml
shortcuts:
  my-shortcut:
    pipeline_name: my-pipeline
    parser_args:
      - var1=down
      - var2=valleys wild
```
{{< /tab >}}
{{< tab name="toml" >}}
```toml
[tool.pypyr.shortcuts.my-shortcut]
pipeline_name = "my-pipeline"
parser_args = [ "var1=down", "var2=valleys wild" ]
```
{{< /tab >}}
{{< /tabs >}}

Remember that `parser_args` is a list/array of the arguments passed from the
cli. Per usual shell parsing rules, each argument is separated by a space,
unless it's enclosed in quotes. Since you've already split the arguments by
hand, you don't have to quote spaces on each array/list item.