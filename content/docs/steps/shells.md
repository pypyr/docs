---
title: pypyr.steps.shells
linktitle: shells
date: 2022-05-23T08:35:53+01:00
description: Run shell statements concurrently in parallel.
card_extra_summary:
  heading: input context property
  details: "`cmds` (dict | list)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: shells
seo_article_headline: Execute shell statements concurrently
seo_description: Execute shell statements asynchronously in parallel with a taskrunner without writing any code.
# social_og_description: 200 chars, if blank fall back to seo_description then description
social_og_title: Run concurrent shell statements in parallel in a task-runner pipeline.
# social_og_image_alt: max 420 chars
topics: [execute external program]
---
# pypyr.steps.shells
## run shell statements concurrently
Runs shell statements in parallel in the default shell. The default shell is
usually `/bin/sh` on POSIX, and on Windows it's `cmd.exe`.

Where the [cmds step]({{< ref "cmds" >}}) runs programs or executables, `shells`
passes the commands through to the system shell. This means all your usual shell
expressions are available, such as ~ expansions and your favorite bashisms.

If you just want to run parallel programs, scripts or executables with
arguments, you do NOT need to use `shells`, you can use [pypyr.steps.cmds]({{<
ref "cmds" >}}) instead. This will incur less processing overhead, because it
won't have to instantiate the shell first. 

You do NOT need shell to run a console-based executable or a script file (like
.sh, .bat, .ps1), you can just use [cmds]({{< ref "cmds" >}}) instead.

Step input can take two forms: simple syntax or expanded syntax. Simple syntax
is just a list of strings. This will run the shell statements in parallel with
the default options.
```yaml
#  simple syntax
- name: pypyr.steps.shells
  comment: list of simple cmd strings to run concurrently
  in:
    cmds:
      - echo one
      - echo two
```

If you want to override any of the default run options, you can use expanded
syntax instead:
```yaml
# expanded syntax
# when save: True
- name: pypyr.steps.shells
  in:
    cmds:
      run:
        - echo one
        - echo two
      save: True
      cwd: ./
      bytes: False
      encoding: utf-8

# when save: False
- name: pypyr.steps.shells
  in:
    cmds:
      run:
        - echo one
        - echo two
      cwd: ..
      stdout: ./path/out.txt
      stderr: ./path/err.txt
      append: False
```

Only `run` is mandatory in expanded syntax. All other inputs are optional. The
example shows which options are relevant depending on whether `save` is `True`
or `False`. `save` is `False` by default.

All inputs support [substitutions]({{< ref "/docs/substitutions">}}). This means
you can inject variables into any of the step inputs - see here for an example
using the cmds step
[cmds variable substitution]({{< ref "cmds#inject-variables-into-command" >}}) -
you can do the exact same thing here with the shells step.

You can combine simple and expanded syntax - any of the top-level list items
can be in expanded syntax to specify extra options:
```yaml
- name: pypyr.steps.shells
  in:
    cmds:
      - echo one

      - run: echo two
        save: True
        cwd: ./
        bytes: False
        encoding: utf-8

      - run:
          - echo three
          - echo four
        cwd: ../
        stdout: ./path/out.txt
        stderr: ./path/err.txt
        append: False
      
      - echo five
```

In this example all the commands will start at the same time and run in
parallel. In other words, one, two, three, four & five will run concurrently.

Note that each run instruction executes in its own shell session. If you instead
want to run multiple shell statements in the same shell session, see
[multiple inline shell statements]({{< ref "shell.md#multiple-inline-shell-statements" >}})
for details - the same principles of the serial `shell` step apply here to the
parallel `shells` step.

If you want to run a sequence of shell statements serially (i.e one after the
after), see [pypyr.steps.shell]({{< ref "shell.md" >}}) instead. You can also
embed serial sequences in your parallel workload - see the next section [combine
parallel & serial execution](#combine-parallel--serial-execution) for details.

## combine parallel & serial execution
If you want to start some shell statements at the same time, but still create a
serial sequence where some commands run in serial one after the other, you can
use a nested list to encode a serial sequence as part of the parallel workload:

```yaml
- name: pypyr.steps.shells
  in:
    cmds:
      - echo A
      - [echo B.1, echo B.2, echo B.3]
      - echo C
```

This example will run as follows:
```text
A
B.1 -> B.2 -> B3
C
```

- A, B.1 & C will all start concurrently.
- B.2 will only start after B.1 completes successfully.
- B.3 will only start after B.2 completes successfully.

In a serial sequence, subsequent commands will only run when the previous
command completes successfully. B.2 and B.3 will NOT run if B.1 raised an error.

You can specify the serial sequence sub-list in the pipeline using `[cmd1,
cmd2]` square brackets, or by using nested list syntax as per usual in yaml:

```yaml
- name: pypyr.steps.shells
  in:
    cmds:
      - echo A
      - 
        - echo B.1
        - echo B.2
        - echo B.3
      - echo C
```

You can do exactly the same thing in expanded syntax - all these are equivalent
ways of achieving the same result:

```yaml
- name: pypyr.steps.shells
  in:
    cmds:
      run:
        - echo A
        - [echo B.1, echo B.2, echo B.3]
        - echo C
      save: True

- name: pypyr.steps.shells
  in:
    cmds:
      run:
        - echo A
        - 
          - echo B.1
          - echo B.2
          - echo B.3
        - echo C
      cwd: mydir/subdir
```

## capture shell output
Refer to [cmds save output to context]({{< ref "cmds#save-output-stdout--stderr"
>}}) for details on how to capture stdout & stderr output and use it in
subsequent steps in the pipeline.

As with `cmds` you can control how to parse the results with the optional
inputs:
- [bytes]({{< ref "cmds#binary-output" >}}) to treat output as raw bytes
- [encoding]({{< ref "cmds#encoding" >}})  to decode output in a different
  encoding

Be aware that `cmdOut.cmd` will be the original input shell statement as a
string, it will not be split into a list of its component arguments like it does
for the `cmds` step.

## save output to file
Refer to [cmds save output to file]({{< ref "cmds#save-output-to-file" >}}) for
details on how to use the following optional inputs:
- stdout
- stderr
- append

These inputs allow you to output to a file, or to redirect stderr to stdout.

## redirect output to null device
See
[redirect cmd output to null device]({{< ref "cmds#redirect-output-to-null-device" >}})
for details on how to redirect the shell's output to system null.

## error handling
The error handling principles are identical to that documented for
asynchronous [parallel cmds error handling]({{< ref "cmds#error-handling" >}}).

## special characters
See the documentation for the serial `shell` step for details:
- [spaces in paths & args]({{< ref "shell#spaces-in-paths--args" >}})
- [curly braces]({{< ref "shell#curly-braces" >}})