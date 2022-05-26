---
title: pypyr.steps.shell
linktitle: shell
date: 2020-07-07T11:53:28+01:00
description: Run commands in the default shell. Use for pipes, wildcards, $ENVs, \~
card_extra_summary:
  heading: input context property
  details: "`cmd` (dict | str | list)"
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
## run shell statements
Runs the context value `cmd` in the default shell. On a sensible O/S, this is 
`/bin/sh`.

Where the [cmd step]({{< ref "cmd" >}}) runs a program or executable, `shell`
passes the command through to the system shell. This means all your usual shell
expressions are available, such as ~ expansions and your favorite bashisms.

If you are just looking to run a command or executable with arguments, you do
not need to use `shell`, you can use [pypyr.steps.cmd]({{< ref "cmd" >}})
instead. This will incur less processing overhead, because it won't have to
instantiate the shell first. 

You do NOT need shell to run a console-based executable or a script file (like
.sh, .bat, .ps1), you can just use [cmd]({{< ref "cmd" >}}) instead.

This step runs shell statements serially one after the other. Use
[shells for asynchronous parallel execution]({{< ref "shells" >}}) instead if
you want concurrent execution.

{{% note tip %}}
Windows users, the COMSPEC environment variable specifies the default system
shell.

This might well be `C:\Windows\System32\cmd.exe` rather than Powershell.
{{% /note %}}

## single shell statement
Input context can take one of two forms for a single shell instruction.

- Simple syntax is just a simple string giving a single run instruction.
- Expanded syntax allows you to set additional options to control how you want
to run the executable.

{{< tabs id="shell-inputs" >}}
{{< tab name="posix" >}}
```yaml
steps:
#  simple syntax
- name: pypyr.steps.shell
  comment: passing cmd as a string doesn't save the output to context.
           prints stdout in real-time.
  in:
    cmd: echo $PWD

#  expanded syntax
- name: pypyr.steps.shell
  comment: passing cmd as a dict allows you to save the output to context 
           when save=True.
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
#  simple syntax
- name: pypyr.steps.shell
  comment: passing cmd as a string doesn't save the output to context.
           prints stdout in real-time.
  in:
    cmd: echo %cd%

#  expanded syntax
- name: pypyr.steps.shell
  comment: passing cmd as a dict allows you to specify if you want to
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

The full expanded syntax for shell input is:
```yaml
# when save: True
- name: pypyr.steps.shell
  in:
    cmd:
      run: echo one
      save: True
      cwd: ./
      bytes: False
      encoding: utf-8

# when save: False
- name: pypyr.steps.shell
  in:
    cmd:
      run: echo two
      cwd: ..
      stdout: ./path/out.txt
      stderr: ./path/err.txt
      append: False
```

Only `run` is mandatory in expanded syntax. All other inputs are optional. The
example shows which options are relevant depending on whether `save` is `True`
or `False`.

## inject variables into shell
You can inject context variables into your shell. This means you can use any
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

- name: pypyr.steps.shell
  comment: use substitution expressions to inject variables into shell
  in:
    cmd: ffmpeg -i {in_file} -b:v {kbits}k -bufsize {kbits}k {out_file}

- name: pypyr.steps.shell
  comment: substitutions work on all inputs
  in:
    cmd:
      run: git {arb-key} --oneline {my-var}
      cwd: '{my-path}' # set any step input option with formatting expression
```

See the section on [escaping curly braces](#curly-braces) if you want to pass
literal curly braces to your shell statement.

On Posix, in the following example `{arb}` is from the pypyr context, and
`${PATH}` comes from the environment variable:

```yaml
- name: pypyr.steps.shell
  comment: combining pypyr substitutions with literal curly braces.
  in:
    cmd:
      run: echo "{arb} and now the $PATH env ${{PATH}}"
```

Remember that depending on your shell `$var` might also get the environment
variable for you without needing {curly braces}.

## run multiple shell statements in same step
You can use a list input to run multiple shell statements in the same step:

```yaml
#  simple syntax
- name: pypyr.steps.shell
  comment: list of simple cmd strings to run
  in:
    cmd:
      - echo one
      - echo two
```

Any of the list items can be in expanded syntax to specify extra options:
```yaml
- name: pypyr.steps.shell
  in:
    cmd:
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

Notice that in expanded syntax `run` can be a single run instruction or a list
of run instructions:

```yaml
- name: pypyr.steps.shell
  comment: apply the same settings to multiple shell commands
  in:
    cmd:
      run:
        - echo one
        - echo two
      save: True
      cwd: ./path/mydir
```

Note that each run instruction executes in its own shell session. If you instead
want to run multiple shell statements in the same shell session, see [multiple
inline shell statements](#multiple-inline-shell-statements) for details.

## change working directory
If you specify `cwd`, it will change the current working directory to `cwd` to 
execute this command.

The directory change is only for the duration of this step, not any subsequent
steps.

When you do set `cwd`, the executable or program specified in `run` is relative 
to the `cwd` if you use a relative path. You can of course use an absolute path
instead.

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

As usual for paths, you can use `.` for current and `..` for parent directory.

In the example we're using the `..` directive to go up one level to the parent
directory, but you can of course use an absolute path or relative path as you
require instead.

The `cwd` you set applies to all the shell statements in `run`.
```yaml
- name: pypyr.steps.shell
  comment: each cmd in run will execute in cwd
  in:
    cmd:
      run:
        - echo one %cd%
        - echo two %cd%
      cwd: path/mydir # cwd applies to each instruction in run
```

## capture shell output
Set `save: True` to capture the shell statement's output.

```yaml
- name: pypyr.steps.shell
  comment: save output to context.
  in:
    cmd:
      run: ./my-executable --arg1 value
      save: True
```

If `save` is `True`, pypyr will save the output to context `cmdOut` as follows:

```yaml
cmdOut.returncode: 0
cmdOut.stdout: 'stdout str here. None if empty.'
cmdOut.stderr: 'stderr str here. None if empty.'
cmdOut.cmd: './my-executable --arg1 value'
```

`cmdOut.returncode` is the exit status of the called process. Typically
0 means OK. A negative value -N indicates that the child was terminated
by signal N (POSIX only).

You can use `cmdOut` in subsequent steps like this:

```yaml
- name: pypyr.steps.shell
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

The `cmdOut` key contains a `SubprocessResult` instance. The full schema for
this object is:
```python
SubProcessResult():
  cmd: str # the original shell input
  returncode: int
  stdout: str | bytes | None
  stderr: str | bytes | None
```

### save output for multiple shell statements
When the shell step runs more than one instruction where `save: True`, the
`cmdOut` output context will be a list. Each list item is a dict/mapping in the
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

The list is in the order the shell statements were executed. The list will
contain only commands where `save` is `True`. This means that if your step input
is a list where only some commands have `save: True`, only those will be in
`cmdOut` and the other commands will not be:

```yaml
- name: pypyr.steps.shell
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

### encoding
By default pypyr treats the shell output as text. It decodes the bytes
returned by the shell using the default system encoding and strips
white-space like line-feeds from the end. 

You can set the `encoding` explicitly:
```yaml
- name: pypyr.steps.shell
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
By default pypyr deals with the shell's output as text in the system's
default encoding. If you want to capture the output as raw bytes without any
text decoding & line-ending handling, you can set `bytes: True`.

```yaml
- name: pypyr.steps.shell
  comment: save the output as raw bytes.
           this will bypass all text decoding.
  in:
    cmd:
      run: ./my-executable --arg1 value
      save: True
      bytes: True
```

The `bytes` setting is only applicable when `save` is `True`. Both stdout &
stderr will save the raw bytes returned by the shell when `bytes: True`.

If you combine `bytes: True` and `encoding` pypyr will decode the output in the
specified encoding but it won't perform line-ending normalization - so your
output might have line-endings at the end depending on the shell & executables
you're calling.

## save output to file
By default, the shell output writes to the parent process' stdout &
stderr streams. Normally this means that any program output will print
in the terminal from where you are running pypyr.

You can save the output to file instead:
```yaml
- name: pypyr.steps.shell
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
- name: pypyr.steps.shell
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
want to use instead of redirecting stdout via pypyr.

For example, with `curl` you use the `-o`/`--output` switch to specify an
output file:

```text
curl https://myurl.arb/blah -o myfile.txt
```
{{% /note %}}

If you set either/both of stdout & stderr, you cannot also set `save: True`,
since these are mutually exclusive.

Since you're in the shell already, you could of course instead use a shell
redirect in the shell statement itself:

```yaml
- name: pypyr.steps.shell
  comment: use a shell redirect to dump to file
  in:
    cmd:
      run: echo "do stuff" >file.log 2>&1
```

## redirect output to null device
You can discard the shell's output when you redirect any or both of stdout
& stderr to the system's Null device with the special value `/dev/null`:

```yaml
- name: pypyr.steps.shell
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
- name: pypyr.steps.shell
  comment: only suppress stdout
  in:
    cmd:
      run: ./my-executable --arg1 value
      stdout: /dev/null
```

When you redirect output to the null device it will not print to console.

Since you're in the shell already, you could of course instead use a shell
redirect in the shell statement itself:

```yaml
- name: pypyr.steps.shell
  comment: use a shell redirect to dump to null
  in:
    cmd:
      run: echo "do stuff" > /dev/null 2>&1
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
You can use shell expressions to combine multiple shell statements into a single
run instruction.

In this example, both steps do the the same thing:
```yaml
# multiple shell statements combined into single pypyr run instruction
- name: pypyr.steps.shell
  comment: run in same shell session
  in:
    cmd: echo one && echo two && echo three

# multiple shell statements as separate pypyr run instructions
- name: pypyr.steps.shell
  comment: each run instruction in its own shell session
  in:
    cmd:
      - echo one
      - echo two
      - echo three
```

The difference is that when you combine different shell commands into the same
run instruction (`echo one && echo two && echo three`) the entire combined
instruction executes in the same shell session.

In the second example each run instruction executes in its own shell scope.

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

You can change directory multiple times during a shell step using `cd`, but dir
changes are only in scope for subsequent statements in that same run
instruction, not for subsequent run instructions or steps.

```yaml
- name: pypyr.steps.shell
  comment: cd only applies to the same run instruction
  in:
    cmd:
      - cd mydir && echo hello from mydir
      - echo not in mydir anymore

- name: pypyr.steps.shell
  comment: cd only applies to the same run instruction
  in:
    cmd:
      run:
        - cd mydir && echo hello from mydir
        - echo not in mydir anymore
```

Instead prefer using the [cwd](#change-working-directory) input as
described above for an easy life, which sets the working directory
for the entire step without you having to code it in with chained shell
commands.

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
accidentally ending up running a different, unexpected interpreter on your
system. This is especially relevant if you're using virtual environments.

See [pypyr.steps.python]({{< ref "python" >}}) for details on getting the full
path to the current Python executable.

Remember you can also invoke your Python code directly by using
[pypyr.steps.py]({{< ref "py" >}}), which will automatically be in the
current Python environment.

Alternatively, if you do have an external Python file you want to run, you can
just add a `def run_step(context)` function to your file and run it natively as
a pypyr step. This is described in [how to make a custom step]({{< ref
"/docs/api/step" >}}). This will also automatically execute in the current
Python environment.