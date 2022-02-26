---
title: run your first pipeline
description: How to run your first pypyr pipeline
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
# categories: [install]
menu:
  docs:
    title: run your first pipeline
    parent: getting-started
weight: -80
seo_article_headline: Tutorial showing you how to run your first task-runner pipeline.
seo_description: An introduction to how to write & run your first pipeline with the pypyr task-runner to automate your own tasks.
# topics: [pipeline, tutorial, args]
---
# run your first pipeline
## run a built-in pipeline
Run one of the built-in pipelines to get a feel for it:

```bash
$ pypyr echo "Ceci n'est pas une pipe"
```

`echo` is the name of a built-in pypyr pipeline. The pipeline simply echoes
the input string back to console output using the built-in
[echo step]({{< ref "/docs/steps/echo">}}). 

The actual pipeline looks like this:

```yaml
# To execute this pipeline, shell something like:
# pypyr echo text goes here
context_parser: pypyr.parser.string
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: '{argString}'
```

This a silly pipeline with only one step. It doesn't do much other than show
you a basic example as an introduction of what a basic pipeline looks like.

The [pypyr.parser.string]({{< ref "/docs/context-parsers/string" >}})
context_parser tells pypyr to concatenate all the input args you specify from
the CLI and make the result available to the pipeline as a variable named
`argString`. `pypyr.steps.echo` then uses a [substitution expression]({{< ref
"/docs/substitutions/format-string" >}}) to get the value of `argString` and
print it out to the console.

You can achieve the same end-result by hard-coding the string you want to echo
into the pipeline yaml rather than passing it in from the cli as a positional
argument:

```bash
$ pypyr magritte
```

```yaml
# ./magritte.yaml
# To execute this pipeline, shell something like:
# pypyr magritte
steps:
  - name: pypyr.steps.echo
    comment: output echoMe
    in:
      echoMe: Ceci n'est pas une pipe
```

## write your first pipeline
A pipeline is simply a yaml file.

The simplest pipeline just needs a `steps` list. This is the default entry
point. `steps` is a list that has a bunch of individual steps in it that will
run sequentially one after the other.

The following examples will only have one or two steps in them, for the sake of
an easy introduction, but you can mix and match as many steps as you need in
your pipelines.

{{< tabs id="first-pipe" >}}
{{< tab name="posix" >}}
```yaml
# ./my-first-pipeline.yaml
# To execute this pipeline, shell something like:
# pypyr my-first-pipeline
steps:
  - name: pypyr.steps.echo
    comment: output echoMe
    in:
      echoMe: this is step 1

  - name: pypyr.steps.cmd
    comment: actually invokes any program on your system.
             the program must be available in current path.
             run any program you fancy.
             we're just running /bin/echo here for an easy demo.
    in:
      cmd: echo this is step 2
```
{{< /tab >}}
{{< tab name="windows" >}}
```yaml
# ./my-first-pipeline.yaml
# To execute this pipeline, shell something like:
# pypyr my-first-pipeline
steps:
  - name: pypyr.steps.echo
    comment: output echoMe
    in:
      echoMe: this is step 1

  - name: pypyr.steps.cmd
    comment: Actually invokes any executable program on your system.
             The program must be available in current path.
             Run any program you fancy.
             Here we're running the powershell executable.
             We're just using powershell's echo for an easy demo,
             but you can invoke any powershell command you like instead.
    in:
      cmd: powershell.exe -NoProfile -Command "echo 'this is step 2'"
```
{{< /tab >}}
{{< /tabs >}}

Save this file as `./my-first-pipeline.yaml`.

Now you can run your pipeline like this:

{{< app-window title="term" lang="text" >}}
$ pypyr my-first-pipeline
this is step 1
this is step 2

$
{{< /app-window >}}

The second step here is the [cmd step]({{< ref "/docs/steps/cmd">}}) - this
built-in step allows you to run any executable available on your system. For the
sake of hello-world simplicity, we're just running good old `echo`, but you
can substitute this with any executable program available in your `$PATH`.

If you don't want to rely on `$PATH` to resolve your program location, you can
specify the full absolute path instead. You can also specify a relative path to
the current working directory.

We're starting with `pypyr.steps.cmd` because you're likely to use it a lot. But
pypyr also has a whole bunch of other ready-made [built-in steps]({{< ref
"/docs/steps"
>}}) that you can use for various tasks such as working with files, variables,
merging configuring, paths & custom inline code. You can also make your own
[custom step]({{< ref "/docs/api/step" >}}) without fuss.

### cross-platform compatibility
Linux & MacOS can run the `posix` sample yaml as is.

On Windows, if you're running pypyr via
[WSL](https://docs.microsoft.com/en-us/windows/wsl/) you can also use the
`posix` samples as is, but if you're running pypyr natively on Windows use the
`windows` sample yaml instead. The reason for this is that posix-compliant
systems have `/bin/echo` available as a discrete program, whereas on Windows
`echo` is baked into the shell instead.

Just for the purposes of an introductory demonstration of running an .exe on
Windows via `pypyr.steps.cmd`, we're invoking `powershell.exe` and running the
`echo` command inside it. In real world scenarios, if you did really want to run
a shell command, you could just use [pypyr.steps.shell]({{< ref
"/docs/steps/shell" >}}) instead.

In general, pypyr pipelines can absolutely be cross-platform. In fact, this is
one of the reasons you might be using pypyr. For example, generally you can
specify all filesystem paths with `/` as you would for POSIX, and this will
automatically resolve correctly to the Windows `\`.

## passing arguments to your pipeline
You can pass arguments from the cli to your pipeline easy peasy.

All you need to do is add a [context_parser]({{< ref "/docs/context-parsers" >}}).

In this example we'll be using a key-value pair parser, which will let you
specify your own key-value pairs so you can access the values you need by key.
There are other types of built-in context parser that can parse the cmd input
differently for you.

The context parser will add these values into the pypyr context for you. Your
pipeline steps can then use these values as they please with 
[{formatting expressions}]({{< ref "/docs/substitutions">}}). 

This time round, instead of running `/bin/echo`, we're going to execute the
`ping` program:

{{< tabs id="ping-example" >}}
{{< tab name="posix" >}}
```yaml
# ./pipe-with-args.yaml
# To execute this pipeline, shell something like:
# pypyr pipe-with-args akey=avalue anotherkey="another arbitrary value" count=1 ip=127.0.0.1
context_parser: pypyr.parser.keyvaluepairs
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: this is akey's value - {akey}. It came from the cli args. {anotherkey}.

  - name: pypyr.steps.cmd
    in:
      cmd: ping -c {count} {ip}
```
{{< /tab >}}
{{< tab name="windows" >}}
```yaml
# ./pipe-with-args.yaml
# To execute this pipeline, shell something like:
# pypyr pipe-with-args akey=avalue anotherkey="another arbitrary value" count=1 ip=127.0.0.1
context_parser: pypyr.parser.keyvaluepairs
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: this is akey's value - {akey}. It came from the cli args. {anotherkey}.

  - name: pypyr.steps.cmd
    comment: runs C:\Windows\System32\PING.EXE
    in:
      cmd: ping /n {count} {ip}
```
{{< /tab >}}
{{< /tabs >}}

Save this file as `./pipe-with-args.yaml`.

Now you can run this pipeline like this:

{{< app-window title="term" lang="text" >}}
$ pypyr pipe-with-args akey=avalue anotherkey="another arbitrary value" count=1 ip=127.0.0.1
this is akey's value - avalue. It came from the cli args. another arbitrary value.
PING 127.0.0.1 (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.063 ms

--- 127.0.0.1 ping statistics ---
1 packets transmitted, 1 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.063/0.063/0.063/0.000 ms

$
{{< /app-window >}}

## branching, loops, retries
You can add extra functionality to *any* step in pypyr by decorating the step
with a [decorator]({{< ref "/docs/decorators" >}}). pypyr gives you built-in
decorators to [loop]({{< ref "loops" >}}), [retry a failure condition
automatically]({{< ref "error-handling#retry-on-error" >}}) and [selectively run
a step only if a certain condition is true]({{< ref "conditional-logic" >}}) -
all of this without you having to code it yourself.

### conditional execution
You can dynamically [run]({{< ref "/docs/decorators/run">}}) or [skip]({{< ref
"/docs/decorators/skip">}}) any step you want:
```yaml
# ./conditional-logic.yaml
context_parser: pypyr.parser.keys
steps:
  - name: pypyr.steps.default
    comment: set default values for optional cli inputs
    in:
      defaults:
        A: False
        B: False

  - name: pypyr.steps.echo
    in:
      echoMe: begin

  - name: pypyr.steps.echo
    run: '{A}'
    in:
      echoMe: A

  - name: pypyr.steps.echo
    run: '{B}'
    in:
      echoMe: B

  - name: pypyr.steps.echo
    in:
      echoMe: end
```

The [pypyr.parser.keys]({{< ref "/docs/context-parsers/keys" >}}) context parser
creates a boolean variable set to `True` for every key you pass in via the cli.

The `run` condition on the A and B steps uses a [{formatting expression}]({{<
ref "/docs/substitutions/format-string">}}) to check whether the boolean value
of `A` or `B` is `True`.

You can now selectively control which steps you want to run by changing
the cli input arguments:
```text
$ pypyr conditional-logic
begin
end

$ pypyr conditional-logic A
begin
A
end

$ pypyr conditional-logic B
begin
B
end

$ pypyr conditional-logic A B
begin
A
B
end
```

Notice that we're just using the `pypyr.steps.echo` step here to demonstrate the
principle - you can apply the conditional [run]({{< ref "/docs/decorators/run"
>}}) decorator to any given step, including your own custom commands.

### automatic retry
You can automatically [retry]({{< ref "/docs/decorators/retry">}}) any given
step if it fails, without having to write any code.

```yaml
steps:
  - name: pypyr.steps.cmd
    retry:
      max: 4
    in:
      cmd: curl https://arb-unreliable-url-example-xyz/
```

This is calling the [curl](https://curl.se) program - we're just using this
because it's widely available on different operating systems, but you could use
[wget](https://www.gnu.org/software/wget/) or whatever takes your fancy instead.
If `curl` returns an error, pypyr will retry the command automatically up to 4
times.

If the 4th retry still fails, pypyr will not go to the next step. pypyr will
raise the error & report failure instead.

### static loop
Let's add a simple loop to a command using the
[foreach decorator]({{< ref "/docs/decorators/foreach" >}}):
{{< tabs id="static-loop" >}}
{{< tab name="posix" >}}
```yaml
# ./fruit-loop.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin

  - name: pypyr.steps.cmd
    foreach: [apple, pear, banana]
    in:
      cmd: echo arb-command --arg {i}

  - name: pypyr.steps.echo
    in:
      echoMe: end
```
{{< /tab >}}
{{< tab name="windows" >}}
```yaml
# ./fruit-loop.yaml
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin

  - name: pypyr.steps.cmd
    foreach: [apple, pear, banana]
    in:
      cmd: powershell.exe -NoProfile -Command "echo 'arb-command --arg {i}'"

  - name: pypyr.steps.echo
    in:
      echoMe: end
```
{{< /tab >}}
{{< /tabs >}}

Notice that for the `cmd` step we're just using `echo` for the sake of example,
you can instead use whatever executable is available on your system with
whatever input args & switches you want.

{{< note tip >}}
Just putting `echo` in front of a command is a handy way of troubleshooting
more complex command sequences - you print out exactly the final formatted
value pypyr passes to the underlying operating system.
{{< /note >}}

`{i}` is a context variable that contains the current item of the iterator.

Running this pipeline looks like this:
{{< app-window title="term" lang="text" >}}
$ pypyr fruit-loop
begin
arb-command --arg apple
arb-command --arg pear
arb-command --arg banana
end

$
{{< /app-window >}}

### dynamic loop cli input args
Of course, you don't have to hard-code the list the you want to iterate. Let's
inject the values from the cli. As always, use a context parser to access
cli arguments: in this case,
[pypyr.parser.list]({{< ref "/docs/context-parsers/list" >}}) will parse all
the input arguments into a list named `{argList}` for you.

{{< tabs id="dynamic-loop" >}}
{{< tab name="posix" >}}
```yaml
# ./fruit-loop
context_parser: pypyr.parser.list
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin

  - name: pypyr.steps.cmd
    foreach: '{argList}'
    in:
      cmd: echo arb-command --arg {i}

  - name: pypyr.steps.echo
    in:
      echoMe: end
```
{{< /tab >}}
{{< tab name="windows" >}}
```yaml
# ./fruit-loop
context_parser: pypyr.parser.list
steps:
  - name: pypyr.steps.echo
    in:
      echoMe: begin

  - name: pypyr.steps.cmd
    foreach: '{argList}'
    in:
      cmd: powershell.exe -NoProfile -Command "echo 'arb-command --arg {i}'"

  - name: pypyr.steps.echo
    in:
      echoMe: end
```
{{< /tab >}}
{{< /tabs >}}

Notice that the `foreach` input now uses a
[{formatting expression}]({{< ref "/docs/substitutions/format-string">}}) to get
the value of the `argList` variable set by the cli args input context parser,
instead of a hard-coded list.

Now you can control dynamically how many to times to run your command from the
cli like this:

{{< app-window title="term" lang="text" >}}
$ pypyr fruit-loop
begin
end

$ pypyr fruit-loop apple pear
begin
arb-command --arg apple
arb-command --arg pear
end

{{< /app-window >}}

The first run, without any arguments, just prints `begin` and `end` - since the
input list is empty there is nothing over which to loop.

## what next?
You probably should check out:
- understanding [basic pipeline structure]({{< ref "basic-concepts" >}})
- How to [declare & use variables]({{< ref "variables" >}})
- How to do [branching with conditional logic]({{< ref "conditional-logic" >}})
- How to use [loops]({{< ref "loops" >}})
- Deal with [errors]({{< ref "error-handling">}})