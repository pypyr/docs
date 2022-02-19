---
title: pypyr.steps.cmd
linktitle: cmd
date: 2020-06-13T21:38:57+01:00
description: Run any external program, command, script.
card_extra_summary:
  heading: input context property
  details: "`cmd` (dict | str)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: cmd
seo_article_headline: Execute any external program, command or script as a task in a pipeline step.
seo_description: Execute any external program, command or script as a task in a task-runner pipeline step.
# social_og_description: 200 chars, if blank fall back to "description"
social_og_title: Execute external commands in a task-runner pipeline.
# social_og_image_alt: max 420 chars
topics: [execute external program]
---
# pypyr.steps.cmd
## execute arbitrary external commands, applications & scripts
Run a program, run an external script, application or command. Execute 
the executable you specify as a sub-process.

In `cmd`, you cannot use things like exit, return, shell pipes, filename
wildcards, environment variable expansion, and expansion of \~ to a
user's home directory. Use [pypyr.steps.shell]({{< ref "shell.md" >}}) for
this instead. `cmd` runs a command, it does not invoke the shell.

## set command input
Input context can take one of two forms:

```yaml
- name: pypyr.steps.cmd
  comment: passing cmd as a string does not save the output to context.
           it prints stdout in real-time.
  in:
    cmd: pwd

- name: pypyr.steps.cmd
  comment: passing cmd as a dict allows you to specify if you want to
           save the output to context.
           it prints command output only AFTER it has finished running.
  in:
    cmd:
      run: pwd
      save: False # optional. Set True to save cmd output to context.
      cwd: ./path/dir/here # optional. Defaults to current execution dir.
```

You can specify the full absolute path to the command, or use a relative path.
If you are using a relative path, the cmd will resolve relative the current
working directory before searching `$PATH`.

Supports string [substitutions]({{< ref "docs/substitutions">}}) for all inputs.

### changing the working directory
If you specify `cwd`, pypyr will change the current working directory to
`cwd` to execute this command. The directory change is only for the
duration of this step, not any subsequent steps. If you specify `cwd`,
the executable or program set in `run` is relative to the `cwd` if
the `run` cmd specifies a relative path.

If you do not specify `cwd`, it defaults to the current working directory,
which is from wherever you are running pypyr.

{{% note warn %}}
On Windows this is more complicated. When setting `cwd`, the cmd itself will
NOT resolve relative to your `cwd` setting. It will resolve relative to the
actual current working directory. You might want to use a full/absolute path
instead.

Once the cmd is actually running it will use the `cwd` value internally. 

See the Windows documentation of the `lpApplicationName` and `lpCommandLine`
parameters of WinAPI `CreateProcess` for gory details.
{{% /note %}}

### save cmd output stdout & stderr
Set `save: True` to capture the command's output.

Be aware that if `save` is `True`, all of the command output ends up in
memory. Don't specify it unless your pipeline uses the stdout/stderr
response in subsequent steps.

Keep in mind that if the invoked command return-code is non-zero pypyr will
automatically raise a `CalledProcessError` and stop the pipeline, so assuming
the command returns a non-zero error code you do not have to do anything special
to get the pipeline to fail automatically. If this is not the behavior you want,
you can suppress errors with the `swallow` decorator and use the `runErrors`
list on subsequent steps. Only use `save: True` when you actually need the
stdout or stderr output.

If `save: True`, pypyr will save the output to context as follows:

```yaml
cmdOut:
  returncode: 0
  stdout: 'stdout str here. None if empty.'
  stderr: 'stderr str here. None if empty.'
```

`cmdOut.returncode` is the exit status of the called process. Typically
0 means OK. A negative value -N indicates that the child was terminated
by signal N (POSIX only).

You can use `cmdOut` in subsequent steps like this:

```yaml
- name: pypyr.steps.echo
  run: !py "cmdOut['returncode'] == 0"
  in:
    echoMe: "you'll only see me if cmd ran successfully with return code 0.
            the command output was: {cmdOut[stdout]}"
```

## spaces in paths & args
Depending on your O/S and file-system, it's up to you to deal with special
characters in the path of the command or program you want to run. 

Generally you can put either just the path-segment with a space into quotes,
or you can escape the entire path by putting quotes around the whole thing.

If a single argument contains a space, surround if with double-quotes.

To illustrate some of the options:
```yaml
- name: pypyr.steps.cmd
  in:
    cmd: '"dir with space/file with space" arg1 "arg2 with space"'

- name: pypyr.steps.cmd
  in:
    cmd: '"dir with space"/"file with space"'

- name: pypyr.steps.cmd
  in:
    cmd: ./"dir with space"/"file with space"
```

Note the "extra" pair of single-quotes (') is there when the string begins and
ends with double-quotes so that the YAML parser reads the double quotes (")
literally instead of interpreting these double quotes as structural YAML markers
meaning string.

## microsoft windows
Especially on Windows the distinction between running a command and running a
shell is important - `echo`, `dir` or `copy` are not actually stand-alone
programs, these instead are built into the Windows shell. Use
[pypyr.steps.shell]({{< ref "shell.md" >}}) instead if you want to run a shell
command.

Generally in pypyr you can use forward slashes for paths even when you're on
Windows. This is pretty handy for cross-platform pipelines.

However, be aware that if you specify a relative path to your command you MUST
use backslashes in the path -  e.g `mydir\mycmd.exe`.

If you specify the full absolute path you can still use forward slashes - e.g
`C:/mydir/mycmd.exe`

## invoking python
```yaml
steps:
  name: pypyr.steps.cmd
  in:
    cmd: python -m stuff.do
```

If you want to invoke Python as a sub-process from pypyr, you might want to
specify the full path to the Python interpreter to avoid problems with
accidentally ending up running an unexpected interpreter on your system. This
is especially relevant if you're using virtual environments.

See [pypyr.steps.python]({{< ref "python" >}}) for details.

Remember you can also invoke your Python directly by using
[pypyr.steps.py]({{< ref "py" >}}), which will automatically be in the
current Python environment.

Alternatively, if you do have an external Python file you want to run, you can
just add a `def run_step(context)` function to your file and run it natively as
a pypyr step. This is described in [how to make a custom step]({{< ref
"/docs/api/step" >}}). This will also automatically execute in the current
Python environment.

## example
Example pipeline yaml:

```yaml
steps:
  - name: pypyr.steps.cmd
    in:
      cmd: ls -a
```

See a worked [example for cmd](https://github.com/pypyr/pypyr-example/tree/main/pipelines/shell.yaml).
