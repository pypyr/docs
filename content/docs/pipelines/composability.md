---
title: composability
linktitle: composability
description: Encapsulate pipeline functionality in re-usable components.
date: 2022-10-07
categories: [pipelines]
menu:
  docs:
    parent: pipelines
    name: composability
weight: -10
seo_article_headline: Create composable automation pipelines with re-usable components.
seo_description: Encapsulate your workflow automation tasks in re-usable modular components with pypyr task-runner.
# topics: [control-of-flow]
---
# composability
You can encapsulate your automation tasks as modular components that you can
re-use and re-combine in different ways in other places.

Here are some common patterns for how to achieve modular & composable pipelines.
You can mix and match ideas from each in the same pipeline or suite of pipelines
depending on what works for you, you don't exclusively have to stick to one way.

## groups
- have a single pipeline with multiple step-groups.
- there are default step groups that run each time.
- additional optional step-groups that run depending on input switches from the
  cli.

```yaml
context_parser: pypyr.parser.list
steps:
  - name: pypyr.steps.call
    comment: set default config & environment values.
    in:
      call: set_config
  - name: pypyr.steps.call
    comment: common code that runs each time
    in:
      call: 
        - common_1
        - common_2
  - name: pypyr.steps.call
    comment: optionally call whichever extras were set as input args
    run: '{argList}'
    in:
      call: '{argList}'

set_config:
  - name: pypyr.steps.echo
    in:
      echoMe: first config step
  - name: pypyr.steps.echo
    in:
      echoMe: second config step

common_1:
  - name: pypyr.steps.echo
    in:
      echoMe: first common step
  - name: pypyr.steps.echo
    in:
      echoMe: second common step

common_2:
  - name: pypyr.steps.echo
    in:
      echoMe: first common 2 step

a:
  - name: pypyr.steps.echo
    in:
      echoMe: optional a

b:
  - name: pypyr.steps.echo
    in:
      echoMe: optional b

c:
  - name: pypyr.steps.echo
    in:
      echoMe: optional c
```

If the you save the above pipeline as `compose/groups/single-pipe.yaml`, you can
run it like this:
{{< app-window title="term" lang="fish" >}}
# run just the defaults:
$ pypyr compose/groups/single-pipe
 
# run, in order, any combination of a, b, c:
$ pypyr compose/groups/single-pipe a
$ pypyr compose/groups/single-pipe a b c
$ pypyr compose/groups/single-pipe c a
$ pypyr compose/groups/single-pipe c b a
{{< /app-window >}}

## pipes
- encapsulate functionality in different pipelines.
- each pipeline can run individually.
- the parent (or main entry-point, or orchestrator) pipeline, runs some default
  setup steps on each run.
- the various child pipelines run optionally depending on input switches from
  the cli.

Imagine we create the following pipelines:
- `compose/pipes/parent.yaml` - entry-point or orchestrator pipeline.
- `compose/pipes/a.yaml`
- `compose/pipes/b.yaml`
- `compose/pipes/c.yaml`

We can run these pipelines in different combinations like this:
{{< app-window title="term" lang="fish" >}}
# run just the defaults:
$ pypyr compose/pipes/parent

# run, in order, any combination of a, b, c:
$ pypyr compose/pipes/parent a
$ pypyr compose/pipes/parent a b c
$ pypyr compose/pipes/parent c a
$ pypyr compose/pipes/parent c b a

# you can still run each child pipeline individually:
$ pypyr compose/pipes/a
$ pypyr compose/pipes/b
$ pypyr compose/pipes/c

{{< /app-window >}}

### parent pipeline
```yaml
# compose/pipes/parent.yaml
context_parser: pypyr.parser.list
steps:
  - name: pypyr.steps.call
    comment: set default config & environment values.
    in:
      call: set_config
  - name: pypyr.steps.call
    comment: common code that runs each time
    in:
      call: 
        - common_1
        - common_2
  - name: pypyr.steps.pype
    comment: optionally call whichever extra pipelines were set as input args
    foreach: '{argList}'
    in:
      pype:
        name: '{i}'

set_config:
  - name: pypyr.steps.echo
    in:
      echoMe: first config step
  - name: pypyr.steps.echo
    in:
      echoMe: second config step

common_1:
  - name: pypyr.steps.echo
    in:
      echoMe: first common step
  - name: pypyr.steps.echo
    in:
      echoMe: second common step

common_2:
  - name: pypyr.steps.echo
    in:
      echoMe: first common 2 step
```

By default child pipelines will have access to the parent's context - this means
that the child can use variables you create in the parent.

Alternatively, you can pass only selected values to the child as arguments by
using [args]({{< ref "/docs/steps/pype#args" >}}) on the `pype` step.

You can further modularize this by only running specific step-groups in a child
pipeline by specifying [groups]({{< ref "/docs/steps/pype#groups" >}}) on the
`pype` step.

### child pipelines
The idea is that each child pipeline can run on its own directly as an
individual, stand-alone component, but you can also orchestrate it from the
parent pipeline to run as part of a sequence with other pipelines.

Just for the sake of example, we have pipelines `a`, `b` and `c`:

```yaml
# compose/pipes/a.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: optional a
```

```yaml
# compose/pipes/b.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: optional b
```

```yaml
# compose/pipes/c.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: optional c
```

## shared
- common or shared functionality exists in its own pipeline.
- other pipelines can call this helper pipeline to execute the shared/common
  steps.
- this way helper/common code exist in one place.
- the shared pipeline can also run individually.

Imagine we create the following pipelines:
- `compose/shared/pipe-a.yaml`
- `compose/shared/pipe-b.yaml`
- `compose/shared/shared-stuff.yaml`

Now when you run either pipeline `a` or `b`, each will call the `shared-stuff`
helper pipeline so you don't have to duplicate shared functionality in both
pipelines.

{{< app-window title="term" lang="fish" >}}
# run pipeline a
$ pypyr compose/shared/pipe-a

# run pipeline b
$ pypyr compose/shared/pipe-b

{{< /app-window >}}

The pipelines look like this:
```yaml
# compose/shared/pipe-a.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: this is pipe A

  - name: pypyr.steps.pype
    comment: call common helper pipeline
    in:
      pype:
        name: shared-stuff

  - name: pypyr.steps.echo
    in:
      echoMe: done!
```

```yaml
# compose/shared/pipe-b.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: this is pipe B

  - name: pypyr.steps.pype
    comment: call common helper pipeline
    in:
      pype:
        name: shared-stuff

  - name: pypyr.steps.echo
    in:
      echoMe: done!
```

```yaml
# compose/shared/shared-stuff.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: this is shared stuff.
```

By default the shared pipeline will have access to the calling pipeline's
context - this means that the shared child can use variables you create in the
caller.

Alternatively, you can pass only selected values to the shared pipeline as
arguments by using [args]({{< ref "/docs/steps/pype#args" >}}) on the
`pype` step.

### modularize your shared pipeline
You can further modularize this by only running specific step-groups in the
shared pipeline by specifying [groups]({{< ref "/docs/steps/pype#groups" >}})
on the `pype` step in the calling pipeline.

In this case, your shared pipeline might look like this:
```yaml
# compose/shared/shared-stuff.yaml
group-1:
  - name: pypyr.steps.echo
    in:
      echoMe: this is shared stuff group 1.

group-2:
  - name: pypyr.steps.echo
    in:
      echoMe: this is shared stuff group 2.

group-3:
  - name: pypyr.steps.echo
    in:
      echoMe: this is shared stuff group 3.
```

And now `pipe-a` and `pipe-b` can selectively call step-groups in the shared
pipeline like this:
```yaml
# compose/shared/pipe-a.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: this is pipe A

  - name: pypyr.steps.pype
    comment: call group-1 in common helper pipeline
    in:
      pype:
        name: shared-stuff
        groups: group-1

  - name: pypyr.steps.echo
    in:
      echoMe: done!
```

```yaml
# compose/shared/pipe-b.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: this is pipe B

  - name: pypyr.steps.pype
    comment: call group-1 and group-3 in common helper pipeline
    in:
      pype:
        name: shared-stuff
        groups: [group-1, group-3]

  - name: pypyr.steps.echo
    in:
      echoMe: done!
```

As ever, you run your pipelines like this:
{{< app-window title="term" lang="fish" >}}
# run pipeline a
$ pypyr compose/shared/pipe-a

# run pipeline b
$ pypyr compose/shared/pipe-b

{{< /app-window >}}