---
title: pypyr.steps.pype
linktitle: pype
date: 2020-07-06T18:44:14+01:00
description: Call another pipeline from the current pipeline.
card_extra_summary:
  heading: input context property
  details: "`pype` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: pype
seo_article_headline: Call pipeline from running pipeline.
seo_description: Invoke another pipeline from the current pipeline, passing over dynamic arguments & values to the child pipeline.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: pype -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [control-of-flow]
---
# pypyr.steps.pype
## call another pipeline from current pipeline
Run another pipeline from this step. This allows pipelines to invoke
other pipelines. Why pype? Because the pypyr can pipe that song again.

`pype` is handy if you want to split a larger, cumbersome pipeline into
smaller units. This helps testing, in that you can test smaller units as
separate pipelines without having to re-run the whole big all-encompassing 
parent pipeline each time.

This gets pretty useful for longer running sequences where the first steps are 
not idempotent but you do want to iterate over the last steps in the pipeline.

Provisioning or deployment scripts frequently have this sort of pattern: where 
the first steps provision expensive resources in the environment and later steps 
just tweak settings on the existing environment.

The parent pipeline is the current, executing pipeline. The child pipeline is 
the pipeline you are calling from this step.

See a worked [example of
pype](https://github.com/pypyr/pypyr-example/tree/master/pipelines/pype.yaml).

## child pipeline execution arguments
You only need to specify the pipeline name you want to call by way of minimal
input arguments:

```yaml
- name: pypyr.steps.pype
  comment: only pype name is required.
  in:
    pype:
      name: child-pipeline # this will run ./child-pipeline.yaml
```

There are many further optional properties you can set for more fine-grained
control of the child pipeline execution:

```yaml
- name: pypyr.steps.pype
  comment: call another pipeline from here
  in:
    pype:
      name: pipeline-name # mandatory. string.
      args: # optional. Defaults None.
        inputkey: value
        anotherkey: anothervalue
      out: # optional. Defaults None.
        parentkey: childkey
        parentkey2: childkey2
      groups: [group1, group2] # optional. Defaults "steps".
      success: success_group # optional. Defaults "on_success".
      failure: failure_group # optional. Defaults "on_failure".
      pipeArg: argument here # optional. string.
      raiseError: True # optional. bool. Defaults True.
      skipParse: True # optional. bool. Defaults True.
      useParentContext: True  # optional. bool. Defaults True.
      loader: None # optional. string. Defaults to standard file loader.
```

The default values apply when you do not specify an option at all.

All inputs support [substitutions]({{< ref "/docs/substitutions">}}). This 
means you can dynamically set which pipeline you want to run and also set at
run-time what parameters to use for the child pipeline.

### name
Name of child pipeline to execute. The pipeline name resolves to a pipeline in
exactly the same way as the pipeline name you call from the cli, following the 
[pipeline lookup order]({{< ref "/docs/pipelines/lookup-order">}}).

```bash
# call ./path/to/my-pipeline.yaml
$ pypyr path/to/my-pipeline
```

pypyr will try to resolve `{name}.yaml` you set here from the working directory 
from which you invoked pypyr originally.

```yaml
pype:
  # this will run ./path/to/my-pipeline.yaml
  name: path/to/my-pipeline
```

If you want to load pipelines from elsewhere, use a [custom loader](#loader).

### args
Initialize the child pipeline context with these args. 

Setting `args` creates a fresh context for the child pipeline that contains 
only the key/values that you set here. `args` should contain a mapping (aka a
dict, in python terms). You can use complex multi-level nested hierarchies.

```yaml
pype:
  name: my-pipeline # this will run ./my-pipeline.yaml
  args:
    key1: value1 # string
    key2: # nested map
      key2.1: value2.1
      key2.2: # list
        - list item 1
        - list item 2
    key3: 123 # integer
    key4: False # bool
    # formatting expression evaluate in parent before child runs
    key5: 'BEGIN {expression} END' 
```

If you set `args`, you implicitly set `useParentContext` to `False`. Thus by
default the parent context is not available in the child pipeline if you set
`args`. More often than not this is the desired behavior, because it allows each
pipeline in the execution chain to use its own context keys without having to
worry about resetting or changing context keys from different pipelines that 
share the same name.

{Format expressions} you use in `args` evaluate against the parent context 
before the child executes.

If you explicitly set `useParentContext` to `True` AND you specify `args`, 
pypyr will merge args into the parent context in addition to applying all 
{formatting expressions} before running the child pipeline.

Here is an [example with pipe args](https://github.com/pypyr/pypyr-example/blob/master/pipelines/pype-args.yaml).

### out
If the child pipeline ran with a fresh new Context, because you set `args` or 
you set `useParentContext` to `False`, `out` saves values from the child 
pipeline context back to the parent context.

`out` can take 3 forms:

```yaml
# save key1 from child to parent
out: 'key1'                     
# or save list of keys from child to parent
out: ['key1', 'key2']           
# or map child keys to different parent keys
out:   
  'parent-destination-key1': 'child-key1'  
  'parent-destination-key2': 'child-key2'  
```

### groups
Run only these step-groups in the child pipeline. 

```yaml
pype:
  name: my-pipeline # this will run ./my-pipeline.yaml
  # only run, in order, group1 -> group2 -> group3
  groups: [group1, group2, group3]
```

Equivalent to `groups` arg on the pypyr cli.

If you don't set this, pypyr will just run the `steps` step-group as per usual.

If you only want to run a single group, you can set it simply as a string, not 
a list, like this:

```yaml
pype:
  name: my-pipeline # this will run ./my-pipeline.yaml
  groups: mygroupname
```

If you set `groups`, success and failure do not default to `on_success` and 
`on_failure` anymore. In other words, `pype` will only run the groups you 
specifically specified. If you still want success/failure handlers explicitly 
set these with `success` & `failure`.

### success
Run this child step-group on successful completion of the child pipeline's step 
`groups`.

Equivalent to `success` arg on the pypyr cli.

If you don't set this, pypyr will just run the `on_success` step-group as per 
usual if it exists, assuming you did not also set `groups`.

If you specify `success`, but you don't set `groups`, pypyr will default to 
running the standard `steps` group as entry-point for the child pipeline.    

### failure
Run this child step-group on an error occurring in the child pipeline.

Equivalent to `failure` arg on the pypyr cli.

If you don't set this, pypyr will just run the `on_failure` step-group as per 
usual if it exists, assuming you did not also set `groups`.

If you specify `failure`, but you don't set `groups`, pypyr will default to 
running the standard `steps` group as entry-point for the child pipeline.

### pipeArg
String to pass to the child pipeline context_parser. Equivalent to `context` 
arg on the pypyr cli. pypyr only uses this if [skipParse](#skipparse) is 
`False`.

Assuming you have a pipeline you would call from the cli like this:

```bash
$ pypyr child-pipeline arg1=value1 arg2=value2
```

You can do the same thing via `pype` as follows:
```yaml
- name: pypyr.steps.pype
  comment: call child pipeline, passing arg string 
           exactly like you would from the cli.
  in:
    pype:
      name: child-pipeline
      pipeArg: arg1=value1 arg2=value2
      skipParse: False
```

{{% note tip %}}
Generally, prefer to use `args` or `useParentContext=True` instead of passing
parameters to the child pipeline via `pipeArg`. 

`pipeArg` is relatively brittle, in that the child pipeline's context parser
will have to parse the `pipeArg` string to initialize the child context, making
for extra inefficient processing, if nothing else.
{{% /note %}}

### raiseError
If `True`, errors in child raise up to parent.

If `False`, log and swallow any errors that happen during the child pipeline's
execution. Swallowing means that the parent pipeline will carry on with the 
next step even if an error occurs in the child pipeline. 

### skipParse
If `True`, skip the `context_parser` on the child pipeline. 

Your child-pipeline might use a `context_parser` to initialize context for when 
you test it in isolation by running it directly from the cli, but 
when calling from a parent pipeline the parent is responsible for creating
the appropriate context using `args` or `useParentContext=True`. The default
`False` for `skipParse` allows for exactly this.

If you still want the child's `context_parser_` to run when invoking the child
via `pype` in a parent pipeline, set `skipParse` to `False`. In this case, use
[pipeArg](#pipearg) to pass the input arg string to the child's `context_parser`.

### useParentContext
If `True`, passes the parent's context to the child. Any changes to the context 
by the child pipeline will be available to the parent after the child completes. 

If `False`, the child creates its own, fresh context that does not contain any 
of the parent's keys. pypyr destroys the child's context upon completion of the 
child pipeline and updates to the child context do not reach the parent context.      

If you use [args](#args) to initialize the child pipeline context, 
`useParentContext` is implicitly `False`, so you do not need to set it 
explicitly unless you really want to.

{{% note tip %}}
If the child pipeline shares the parent context, be careful that the child does
not accidentally overwrite context that is important to the parent.

If you have a chain of pipelines calling each other, where you are using `pype` 
in a child that a parent calls via `pype` in a `for` or `while` loop, `pype` 
will work for both child and parent loop iterations because pypyr takes care of 
this under the hood for you. Be careful of your own custom context properties,
however, the parent and child shares these.
{{% /note %}}

### loader
Load the child pipeline with this loader. The default is the standard pypyr 
`pypyr.pypeloaders.fileloader`, which looks for pipelines on the file system.

This is useful if you are loading your pipelines using a 
[custom loader]({{< ref "/docs/api/pipeline-loader" >}}) to fetch your 
pipelines from your own external systems or locations.

## recursion
Yes, you can call another pipeline recursively - i.e a child pipeline can call 
its parent pipeline. It's up to you to avoid infinite recursion, though. Since
we're all responsible adults here, pypyr does not protect you from
infinite recursion other than the default python recursion limit. 

So don't come crying if you blew your stack. Or a seal.

Here is a worked [example of pipeline
recursion](https://github.com/pypyr/pypyr-example/tree/master/pipelines/pype-recursion.yaml).
