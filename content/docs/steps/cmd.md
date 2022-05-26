---
title: pypyr.steps.cmd
linktitle: cmd
date: 2020-06-13T21:38:57+01:00
description: Run any external program, command, script.
card_extra_summary:
  heading: input context property
  details: "`cmd` (dict | str | list)"
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
## execute external commands, applications & scripts
Run a program, run an external script, application or command. Execute an
executable as a sub-process.

`cmd` runs an executable, it does not invoke the shell. You cannot use shell
features like exit, return, shell pipes, filename wildcards, environment
variable expansion, and expansion of \~ to a user's home directory. Use
[pypyr.steps.shell]({{< ref "shell.md" >}}) for that instead.

This step runs executables serially one after the other. Use
[cmds for asynchronous parallel execution]({{< ref "cmds" >}}) instead if you
want concurrent execution.

## run single command
Input context for a single command instruction can take one of two forms:

- Simple syntax is just a simple string giving a single run instruction.
- Expanded syntax allows you to set additional options to control how you want
to run the executable.

```yaml
#  simple syntax just runs with defaults
- name: pypyr.steps.cmd
  comment: passing cmd as a string does not save output to context.
           it prints stdout & stderr in real-time.
  in:
    cmd: git log --oneline -1

# expanded syntax lets you change the default run settings
- name: pypyr.steps.cmd
  comment: passing cmd as a dict allows you to override defaults
  in:
    cmd:
      run: git log --oneline -1
      save: False # optional. Set True to save cmd output to context.
      cwd: ./path/dir/here # optional. Defaults to current execution dir.
```

For the sake of example we're just invoking a read-only operation of the `git`
program to show the last commit. If you're not in a git repo this would
obviously error.

You can specify the full absolute path to the command, or use a relative path.
If you are using a relative path, the cmd will try to resolve relative the
current working directory before searching `$PATH`.

These are all the options available in expanded syntax:
```yaml
#  when save: True
- name: pypyr.steps.cmd
  comment: when save is True
  in:
    cmd:
      run: curl https://myurl.blah/diblah
      save: True
      cwd: .
      bytes: False
      encoding: utf-8

#  when save: False
- name: pypyr.steps.cmd
  comment: when save is False (the default when `save` not set)
  in:
    cmd:
      run: curl --cert certfile --key keyfile https://myurl.blah/diblah
      # save: False - false by default, no need to specify explicitly
      cwd: ..
      stdout: ./path/out.txt
      stderr: ./path/err.txt
      append: False
```

Only `run` is mandatory in expanded syntax. All other inputs are optional. The
example shows which options are relevant depending on whether `save` is `True`
or `False`. `save` is `False` by default.

## inject variables into command
You can inject context variables into your command. This means you can use any
of the pypyr [context manipulation steps]({{< ref "/topics/context" >}}) to
manipulate your inputs and then pass these to the command.

Supports string [substitutions]({{< ref "docs/substitutions">}}) for all step
inputs in expanded syntax.

```yaml
- name: pypyr.steps.set
  comment: set some arb values in context
  in:
    set:
      in_file: input.avi
      out_file: output.avi
      kbits: 64
      arb-key: log
      my-var: -1
      my-path: ./mypath/mydir

# simple syntax
- name: pypyr.steps.cmd
  comment: use substitution expressions to inject variables into cmd
  in:
    cmd: ffmpeg -i {in_file} -b:v {kbits}k -bufsize {kbits}k {out_file}

# expanded syntax
- name: pypyr.steps.cmd
  comment: substitutions work on all inputs
  in:
    cmd:
      run: git {arb-key} --oneline {my-var}
      cwd: '{my-path}' # set any step input option with formatting expression
```

## run multiple commands in same step
You can use a list input to run multiple executables in the same step:

```yaml
#  simple syntax
- name: pypyr.steps.cmd
  comment: list of run instructions
  in:
    cmd:
      - git log --oneline -1
      - git config -l

# any or all of the list items can be in expanded syntax
- name: pypyr.steps.cmd
  comment: you can mix & match simple strings and expanded dict inputs
  in:
    cmd:
      - git log --oneline -1
      - run: git config -l
        cwd: ./path/mydir
      - git status --porcelain
```

In expanded syntax, the value of `run` can be a single run instruction, or it
can be a list of run instructions:

```yaml
- name: pypyr.steps.cmd
  comment: apply the same settings to multiple commands
  in:
    cmd:
      run:
        - git log --oneline -1
        - git config -l
      save: True
      cwd: ./path/mydir
```

This allows you to set non-default options for the entire sequence of commands
in `run`. In this example each command in the `run` list will run in the
specified `cwd` directory.

You can also make `run` a list of instructions even when the expanded input is
already part of a list of run instructions:

```yaml
- name: pypyr.steps.cmd
  comment: you can mix & match simple strings and expanded dict inputs
  in:
    cmd:
      - ./a-executable --arg one
      - run:
          - ./b-executable --arg two
          - ./c-executable --arg three
        save: True
        cwd: ./path/mydir
      - ./d-executable --arg four
```

In all of these cases you can use any valid combination of step input
properties with the expanded syntax:

```yaml
- name: pypyr.steps.cmd
  in:
    cmd:
      - ./a-executable --arg one

      - run: ./b-executable --arg two
        save: True
        cwd: ./
        bytes: False
        encoding: utf-8

      - run: # run can be a single str or a list of str
          - c-executable --arg three
          - d-executable --arg four
        cwd: ../
        stdout: ./path/out.txt
        stderr: ./path/err.txt
        append: False
      
      - ./e-executable --arg five
```

## change the working directory
If you specify `cwd`, pypyr will change the current working directory to
`cwd` to execute the command.

The directory change is only for the duration of this step, not any subsequent
steps.

If you do not specify `cwd`, it defaults to the current working directory,
which is from wherever you are running pypyr.

```yaml
- name: pypyr.steps.cmd
  comment: run command in different directory
  in:
    cmd:
      run: git log --oneline -1
      cwd: path/mydir
```

If you do specify `cwd`, the executable or program set in `run` is relative to
the `cwd` if the `run` cmd specifies a relative path.

{{% note warn %}}
On Windows this is more complicated. When setting `cwd`, the cmd itself will
NOT resolve relative to your `cwd` setting. It will resolve relative to the
actual current working directory. You might want to use a full/absolute path
instead. If you do use a relative path, you have to use \ rather than / in the
path. Both \ and / works for full/absolute paths. See the
[Windows section](#microsoft-windows) further down for details.

Once the cmd is actually running it will use the `cwd` value internally. 

See the Windows documentation of the `lpApplicationName` and `lpCommandLine`
parameters of WinAPI `CreateProcess` for gory details.
{{% /note %}}

The `cwd` you set applies to all the run instructions in `run`.
```yaml
- name: pypyr.steps.cmd
  comment: each cmd in run will execute in cwd
  in:
    cmd:
      run:
        - git log --oneline -1
        - git status
      cwd: path/mydir # cwd applies to each instruction in run
```

As usual for paths, you can use `.` for current and `..` for the parent
directory.

## save output stdout & stderr
Set `save: True` to capture the command's output.

```yaml
- name: pypyr.steps.cmd
  comment: save output to context.
  in:
    cmd:
      run: ./my-executable --arg1 value
      save: True
```

If `save: True`, pypyr will save the output to context `cmdOut` as follows:

```yaml
cmdOut.returncode: 0
cmdOut.stdout: 'stdout str here. None if empty.'
cmdOut.stderr: 'stderr str here. None if empty.'
cmdOut.cmd: ['./my-executable', '--arg1', 'value']
```

`cmdOut.returncode` is the exit status of the called process. Typically
0 means OK. A negative value -N indicates that the child was terminated
by signal N (POSIX only).

You can use `cmdOut` in subsequent steps like this:

```yaml
- name: pypyr.steps.cmd
  in:
    cmd:
      run: echo 1
      save: True

- name: pypyr.steps.echo
  run: !py cmdOut.returncode == 0
  in:
    echoMe: "you'll only see me if cmd ran successfully with return code 0.
            the command output was: {cmdOut.stdout}.
            the error output was: {cmdOut.stderr}."
```

Be aware that if `save` is `True`, all of the command output ends up in
memory. Don't set it unless your pipeline actually uses the stdout/stderr
response in subsequent steps. Instead of saving the output to memory like this,
you can alternatively [write output to file](#save-output-to-file).

Only use `save: True` when you actually need to use the stdout or stderr output
in subsequent steps - you don't need to set it just to check that the return
code is 0, since pypyr will raise an error automatically if it's not. If this is
not what you want, you can suppress errors with the [swallow decorator]({{< ref
"docs/decorators/swallow">}}) and use the [runErrors]({{< ref
"/docs/getting-started/error-handling#use-runtime-error-details-inside-pipeline"
>}}) list on subsequent steps.

If `swallow` is False (which is the default), you can access `cmdOut` and/or the
`runErrors` list in a
[failure handler]({{< ref "/docs/getting-started/error-handling#failure-handlers" >}}).

The `cmdOut` key contains a `SubprocessResult` instance. The full schema for
this object is:
```python
SubProcessResult():
  cmd: str | list[str] # the input cmd split into args
  returncode: int
  stdout: str | bytes | None
  stderr: str | bytes | None
```

Note on Windows that the result's `cmd` attribute is the original string, on all
other platforms it is a list of args split from the original input string.

### save output for multiple commands
When the cmd step runs more than one instruction with `save: True`, the `cmdOut`
variable in context will be a list. Each list item is a SubprocessResult in the
same format as the output for a single command:

```yaml
cmdOut:
  - returncode: 0
    stdout: 'stdout str here. None if empty.'
    stderr: 'stderr str here. None if empty.'
  - returncode: 0
    stdout: 'stdout str here. None if empty.'
    stderr: 'stderr str here. None if empty.'
```

The list is in the order the commands were executed. The list will contain only
commands where `save` is `True`. This means that if your step input is a list
where only some commands have `save: True`, only those will be in `cmdOut` and
the other commands will not be:

```yaml
- name: pypyr.steps.cmd
  comment: only 2, 3 & 5 saves to cmdOut
  in:
    cmd:
      - a-executable --arg one

      - run:
          - b-executable --arg two
          - c-executable --arg three
        save: True

      - d-executable --arg four

      - run: e-executable --arg five
        save: True
```

In this example the `cmdOut` list will contain 3 items - the output for b, c
and e-executable.

You can use the `cmdOut` list in subsequent steps like this:

```yaml
- name: pypyr.steps.echo
  run: !py cmdOut[0].returncode == 0
  in:
    echoMe: |
            you'll only see me if cmd ran successfully with return code 0.

            For the first command,
            the command output was: {cmdOut[0].stdout}
            the error output was: {cmdOut[0].stderr}
            
            And for the second command,
            the command output was: {cmdOut[1].stdout}
            the error output was: {cmdOut[1].stderr}
```

Notice that you reference each cmd's output on a zero-based index - i.e the
1st item is in position 0.

Notice the difference in formatting expression when the step only ran a single
command vs when multiple commands had `save` set to True:

```python
# single cmd output
'{cmdOut.stdout}'

# list of command outputs
# the 1st output is at index 0
'{cmdOut[0].stdout}'
```

### debugging output
A quick way of seeing exactly what is in `cmdOut` is just to echo it out:

```yaml
- name: pypyr.steps.echo
  in:
    echoMe: '{cmdOut}'
```

OP debugging FTW!

### encoding
By default pypyr treats the command output as text. It decodes the bytes
returned by the executable using the default system encoding and strips
white-space like line-feeds from the end. 

If you are saving output from an executable that is not in the default encoding,
you can set the `encoding`:
```yaml
- name: pypyr.steps.cmd
  comment: save output & decode in specified encoding.
  in:
    cmd:
      run: ./my-executable --arg1 value
      save: True
      encoding: utf-16
```

The `encoding` setting is only applicable when `save` is `True`. Both stdout &
stderr will use the encoding you set.

The default system encoding is very likely to be `utf-8`, unless you're on
Windows. See here for a [list of available encoding
codecs](https://docs.python.org/3/library/codecs.html#standard-encodings).

You can change the
[default_cmd_encoding setting in config]({{< ref "/docs/getting-started/config#default_cmd_encoding" >}})
to use a different default here.

### binary output
By default pypyr deals with the executable's output as text in the system's
default encoding. If you want to capture the output as raw bytes without any
text decoding & line-ending handling, you can set `bytes: True`.

```yaml
- name: pypyr.steps.cmd
  comment: save the output as raw bytes.
           this will bypass all text decoding.
  in:
    cmd:
      run: ./my-executable --arg1 value
      save: True
      bytes: True
```

The `bytes` setting is only applicable when `save` is `True`. Both stdout &
stderr will save the raw bytes returned by the executable when `bytes: True`.

If you combine `bytes: True` and `encoding` pypyr will decode the output in
the specified encoding but it won't perform line-ending normalization - so
your output might have line-endings at the end depending on the executable
you're calling.

## save output to file
By default, the executable output writes to the parent process' stdout &
stderr streams. Normally this means that any program output will print
in the terminal from where you are running pypyr.

You can save the output to file instead:
```yaml
- name: pypyr.steps.cmd
  comment: save the output to file.
           this will NOT print the output to console,
           but redirect it to the output files instead.
  in:
    cmd:
      run: ./my-executable --arg1 value
      stdout: mydir/out.txt # optional. write stdout to this file.
      stderr: mydir/err.txt # optional. write stderr to this file.
      append: False # optional. Default False.
```

Set `append` to `True` to append output if the file(s) exists already.
If `append` is `False` it will overwrite any existing file - this is the default
option when you don't explicitly set `append`.

If you want error output to write to the same file as `stdout`, you can use
the special value `/dev/stdout` to redirect stderr to stdout:

```yaml
- name: pypyr.steps.cmd
  comment: save both stdout & stderr output to the same file.
           this will NOT print the output to console,
           but redirect it to the output file instead.
  in:
    cmd:
      run: ./my-executable --arg1 value
      stdout: mydir/out.txt
      stderr: /dev/stdout
```

{{% note tip %}}
A lot of cli tools have a built-in option to output to file, which you might
want to use instead of redirecting stdout to file via pypyr.

For example, with `curl` you use the `-o`/`--output` switch to specify an
output file:

```text
curl https://myurl.arb/blah -o myfile.txt
```
{{% /note %}}

When you set either/both of stdout & stderr, you cannot also set `save: True`,
since these are mutually exclusive.

## redirect output to null device
You can discard the executable's output when you redirect any or both of stdout
& stderr to the system's Null device with the special value `/dev/null`:

```yaml
- name: pypyr.steps.cmd
  comment: discard all output.
  in:
    cmd:
      run: ./my-executable --arg1 value
      stdout: /dev/null
      stderr: /dev/null
```

The previous example redirects both stdout & stderr. You can also selectively
redirect only one of these to dump to null - so if you want to see stderr output
but not standard output:
```yaml
- name: pypyr.steps.cmd
  comment: only suppress stdout
  in:
    cmd:
      run: ./my-executable --arg1 value
      stdout: /dev/null
```

When you redirect output to the null device it will not print to console.

## spaces in paths & args
Depending on your O/S and file-system, it's up to you to deal with special
characters in the path of the command or program you want to run. 

Generally you can put either just the path-segment with a space into quotes,
or you can escape the entire path by putting quotes around the whole thing.

If a single argument contains a space, surround it with double-quotes.

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
ends with literal double-quotes so that the YAML parser reads the double quotes
(") literally instead of interpreting these double quotes as structural YAML
markers meaning string.

## microsoft windows
On Windows the distinction between running a command and running a shell can
lead to surprises - `echo`, `dir` or `copy` are not actually stand-alone
programs, these instead are built into the Windows shell. Use
[pypyr.steps.shell]({{< ref "shell.md" >}}) instead if you want to run a shell
command.

Generally in pypyr you can use forward slashes for paths even when you're on
Windows. This is pretty handy for cross-platform pipelines.

However, be aware that if you specify a relative path to your command you MUST
use backslashes in the path -  e.g `mydir\mycmd.exe`.

If you specify the full absolute path you can still use forward slashes - e.g
`C:/mydir/mycmd.exe`

Windows will generally automatically put known executable extensions at the end
of the file for you - so both these are equivalent:

```text
ping 127.0.0.1
ping.exe 127.0.0.1
```

## invoking python
```yaml
steps:
  - name: pypyr.steps.cmd
    in:
      cmd: python -m stuff.do
```

If you want to invoke Python as a sub-process from pypyr, you might want to
specify the full path to the Python interpreter to avoid problems with
accidentally running a different, unexpected interpreter on your system. This is
especially relevant if you're using virtual environments.

See [pypyr.steps.python]({{< ref "python" >}}) for details on getting the full
path to the current Python executable.

Remember you can also invoke Python code directly by using
[pypyr.steps.py]({{< ref "py" >}}), which will automatically be in the
current Python environment.

Alternatively, if you do have an external Python file you want to run, you can
just add a `def run_step(context)` function to your file and run it natively as
a pypyr step. This is described in [how to make a custom step]({{< ref
"/docs/api/step" >}}). This will also automatically execute in the current
Python environment.