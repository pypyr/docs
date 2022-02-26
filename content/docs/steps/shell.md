---
title: pypyr.steps.shell
linktitle: shell
date: 2020-07-07T11:53:28+01:00
description: Run commands in the default shell. Use for pipes, wildcards, $ENVs, \~
card_extra_summary:
  heading: input context property
  details: "`cmd` (dict | str)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: shell
seo_article_headline: Execute shell commands as a task in a pipeline step.
seo_description: Run shell commands using pipes, wildcards, environment variables and \~ expansion for home in a task-runner pipeline.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: shell -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [execute external program]
---
# pypyr.steps.shell
## run commands in default shell
Runs the context value `cmd` in the default shell. On a sensible O/S, this is 
`/bin/sh`, or more recently `zsh`.

Do all the things you can't do with [cmd]({{< ref "cmd" >}}). `cmd` runs a 
program or executable, `shell` invokes the actual system shell. This means
all your shell expressions are available, such as your favorite bashisms.

If you are just looking to run a command or executable with arguments, you
do not need to use `shell`, you can use `cmd` instead, which will incur less
processing overhead, because it won't have to instantiate the shell first. 

## set shell commands
Input context can take one of two forms:

{{< tabs id="shell-inputs" >}}
{{< tab name="posix" >}}
```yaml
steps:
- name: pypyr.steps.shell
  description: passing cmd as a string doesn't save the output to context.
               prints stdout in real-time.
  in:
    cmd: echo $PWD

- name: pypyr.steps.shell
  description: passing cmd as a dict allows you to specify if you want to
               save the output to context if save=True.
               prints command output only AFTER it has finished running.
  in:
    cmd:
      run: echo $PWD
      save: True
      cwd: ./set/current/working/dir/here
```
{{< /tab >}}
{{< tab name="windows" >}}
```yaml
steps:
- name: pypyr.steps.shell
  description: passing cmd as a string doesn't save the output to context.
               prints stdout in real-time.
  in:
    cmd: echo %cd%

- name: pypyr.steps.shell
  description: passing cmd as a dict allows you to specify if you want to
               save the output to context if save=True.
               prints command output only AFTER it has finished running.
  in:
    cmd:
      run: echo %cd%
      save: True
      cwd: ./set/current/working/dir/here
```
{{< /tab >}}
{{< /tabs >}}

All inputs support [substitutions]({{< ref "/docs/substitutions">}}).

### changing working directory
If you specify `cwd`, it will change the current working directory to `cwd` to 
execute this command. The directory change is only for the duration of this 
step, not any subsequent steps.

When you do set `cwd`, the executable or program specified in `run` is relative 
to the `cwd`. `cwd` itself is relative to the over-all pypyr working directory.

If you do not specify `cwd`, it defaults to the current working directory, 
which is from wherever you are running `pypyr`.

{{< tabs id="shell-cwd" >}}
{{< tab name="posix" >}}
```yaml
steps:
- name: pypyr.steps.shell
  in:
    cmd:
      run: pwd
      cwd: ..
```
{{< /tab >}}
{{< tab name="windows" >}}
```yaml
steps:
- name: pypyr.steps.shell
  comment: bare cd with no args prints current dir
  in:
    cmd:
      run: cd
      cwd: ..
```
{{< /tab >}}
{{< /tabs >}}

In the example we're using the `..` directive to go up one level to the parent
directory, but you can of course use an absolute path or relative path as you
require instead.

### capture shell output
Be aware that if `save` is `True`, all of the command output ends up in
memory. Don't specify it unless your pipeline actually uses the stdout/stderr
response in subsequent steps.

Keep in mind that if the invoked command has a non-zero return code pypyr will 
automatically raise a `CalledProcessError` and stop the pipeline - in other 
words, you do not have to set `save` to `True` to handle the output in
case of error, the pipeline will stop and report the error automatically. If 
this is not the behavior you want, you can suppress errors with the 
[swallow decorator]({{< ref "/docs/decorators/swallow">}}) and use the 
[runErrors]({{< ref "/docs/getting-started/error-handling" >}}) list on 
subsequent steps. Only use `save: True` when you actually need the stdout or 
stderr output.

If `save` is `True`, pypyr will save the output to context as follows:

```yaml
cmdOut:
    returncode: 0
    stdout: 'stdout str here. None if empty.'
    stderr: 'stderr str here. None if empty.'
```

`cmdOut.returncode` is the exit status of the called process. Typically
0 means OK. A negative value -N indicates that the child was terminated
by signal N (POSIX only).

You can use cmdOut in subsequent steps like this:

```yaml
- name: pypyr.steps.echo
  comment: you'll only see me if cmd ran successfully with return code 0
  run: !py cmdOut['returncode'] == 0
  in:
    echoMe: "the command output was: {cmdOut[stdout]}"
```

## spaces in paths & args
Depending on your O/S, shell and file-system, it's up to you to deal with 
special characters in the path of the command or program you want to run. 

Generally you can put either just the path-segment with a space into quotes,
or you can escape the entire path by putting quotes around the whole thing.

If a single argument contains a space, surround if with double-quotes.

To illustrate some of the options:
```yaml
- name: pypyr.steps.shell
  in:
    cmd: '"dir with space/file with space" arg1 "arg2 with space"'

- name: pypyr.steps.shell
  in:
    cmd: '"dir with space"/"file with space"'

- name: pypyr.steps.shell
  in:
    cmd: ./"dir with space"/"file with space"
```

Note the "extra" pair of single-quotes (') is there when the string begins and
ends with double-quotes so that the YAML parser reads the double quotes (")
literally instead of interpreting these double quotes as structural YAML markers
meaning string.

## curly braces
If you want to pass {curly braces} through to the shell, and NOT have pypyr
interpret these as [{formatting expressions}]({{< ref "/docs/substitutions">}}),
you can escape the braces by {{doubling}}, or by using a
[sic string]({{< ref "/docs/substitutions/sic-strings" >}}) literal:

```yaml
- name: pypyr.steps.shell
  comment: if you want to pass curlies to the shell, use sic strings
  in:
    cmd: !sic echo ${PWD};

- name: pypyr.steps.shell
  comment: alternatively, escape by doubling
  in:
    cmd: echo ${{PWD}};
```

## examples
Example pipeline yaml using a pipe:

```yaml
steps:
  - name: pypyr.steps.shell
    comment: if you have something pipey in working dir it should show up
    in:
      cmd: ls | grep pipe
```

See a worked [example for shell pipeline power](https://github.com/pypyr/pypyr-example/tree/main/pipelines/shell.yaml).

## multiple inline shell statements
### posix
Friendly reminder of the difference between separating your shell statements
with `;` or `&&`:

-   `;` will continue to the next statement even if the previous command
    errored. It won't exit with an error code if it wasn't the last
    statement.
-   `&&` stops and exits reporting error on first error.

### windows
Friendly reminder of the difference between separating your shell statements
with `&` or `&&`:

-   `&` will continue to the next statement even if the previous command
    errored. It won't exit with an error code if it wasn't the last
    statement.
-   `&&` stops and exits reporting error on first error.

### changing directory
You can change directory multiple times during a shell step using
`cd`, but dir changes are only in scope for subsequent commands in this
step, not for subsequent steps.

Instead prefer using the [cwd](#changing-working-directory) input as
described above for an easy life, which sets the working directory
for the entire step without you having to code it in with chained shell
commands.

{{< tabs id="multiple-inline" >}}
{{< tab name="posix" >}}
```yaml
steps:
- name: pypyr.steps.shell
  comment: hop one up from current working dir
  in:
    cmd: echo $PWD; cd ../; echo $PWD

- name: pypyr.steps.shell
  comment: back in your current working dir
  in:
    cmd: echo $PWD
```
{{< /tab >}}
{{< tab name="windows" >}}
```yaml
steps:
- name: pypyr.steps.shell
  comment: hop one up from current working dir.
  in:
    cmd: cd & cd .. & cd

- name: pypyr.steps.shell
  comment: back in your current working dir
  in:
    cmd: cd
```
{{< /tab >}}
{{< /tabs >}}

## invoking python
```yaml
steps:
  name: pypyr.steps.shell
  in:
    cmd: python -m stuff.do
```

Note that this is a bit of a silly example - since we're not using any
shell functions we might as well have used [pypyr.steps.cmd]({{< ref "cmd" >}})
instead.

If you want to invoke Python as a sub-process from pypyr, you might want to
specify the full path to the Python interpreter to avoid problems with
accidentally ending up running an unexpected interpreter on your system. This
is especially relevant if you're using virtual environments.

See [pypyr.steps.python]({{< ref "python" >}}) for details.

Remember you can also invoke your Python code directly by using
[pypyr.steps.py]({{< ref "py" >}}), which will automatically be in the
current Python environment.

Alternatively, if you do have an external Python file you want to run, you can
just add a `def run_step(context)` function to your file and run it natively as
a pypyr step. This is described in [how to make a custom step]({{< ref
"/docs/api/step" >}}). This will also automatically execute in the current
Python environment.