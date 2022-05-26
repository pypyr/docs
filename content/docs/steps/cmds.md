---
title: pypyr.steps.cmds
linktitle: cmds
date: 2022-05-23T08:35:40+01:00
description: Run external programs, commands & scripts concurrently in parallel.
card_extra_summary:
  heading: input context property
  details: "`cmds` (dict | list)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: cmds
seo_article_headline: Run any external programs, commands or scripts concurrently.
seo_description: Run executables asynchronously in parallel with a taskrunner without writing any code.
# social_og_description: 200 chars, if blank fall back to seo_description then description
social_og_title: Execute programs concurrently in parallel with a task-runner pipeline.
# social_og_image_alt: max 420 chars
topics: [execute external program]
---
# pypyr.steps.cmds
## run programs concurrently
Run programs, external scripts, applications or commands in parallel. This step
launches executables as asynchronous subprocesses that run concurrently in
parallel.

Step input can take two forms: simple syntax or expanded syntax. Simple syntax
is just a list of strings. This will run the commands in parallel with the
default options.

{{< tabs id="parallel-copy" >}}
{{< tab name="posix" >}}
```yaml
#  simple syntax
- name: pypyr.steps.cmds
  comment: copy 3 files concurrently
  in:
    cmds:
      - cp file1.ext /media/vol1/
      - cp file2.ext /media/vol2/file2-archive.ext
      - cp file3.ext /media/vol3/
```
{{< /tab >}}
{{< tab name="windows" >}}
```yaml
#  simple syntax
- name: pypyr.steps.cmds
  comment: copy 3 files concurrently
  in:
    cmds:
      - xcopy /i file1.ext X:\file1-archive.ext
      - robocopy c:\reports '\\marketing\videos' yearly-report.mov /mt /z
      - xcopy file3.ext Y:\mydir\
```
{{< /tab >}}
{{< /tabs >}}

If you want to override any of the default run options, you can use expanded
syntax instead:
```yaml
#  when save: True
- name: pypyr.steps.cmds
  comment: when save is True
  in:
    cmds:
      run:
        - ./myprogram --myswitch myarg1
        - ./anotherprogram arg1 "arg 2"
      save: True
      cwd: mydir/subdir
      bytes: False
      encoding: utf-8

#  when save: False
- name: pypyr.steps.cmds
  comment: when save is False
  in:
    cmds:
      run:
        - ./myprogram --myswitch myarg1
        - ./anotherprogram arg1 "arg 2"
      # save: False - false by default, no need to specify explicitly
      cwd: ..
      stdout: ./path/out.txt
      stderr: ./path/err.txt
      append: False
```

Only `run` is mandatory in expanded syntax. All other inputs are optional. The
example shows which options are relevant depending on whether `save` is `True`
or `False`. `save` is `False` by default.

You can combine simple and expanded syntax - any of the top-level list items
can be in expanded syntax:

```yaml
- name: pypyr.steps.cmds
  comment: you can mix & match simple strings and expanded dict inputs
  in:
    cmds:
      - ./a-executable --arg one
      - run:
          - ./b-executable --arg two
          - ./c-executable --arg three
        save: True
        cwd: ./path/mydir
      - ./d-executable --arg four
```

In this example a, b, c & d will all start at the same time and run in 
parallel.

If you want to run a sequence of executables serially (i.e one after the after),
check [pypyr.steps.cmd]({{< ref "cmd.md" >}}) instead. You can also embed serial
sequences in your parallel workload - see the section
[combine parallel & serial execution](#combine-parallel--serial-execution) for
details.

`cmds` runs executables directly, it does not invoke them through the shell. You
cannot use shell features like exit, return, shell pipes, filename wildcards,
environment variable expansion, and expansion of \~ to a user's home directory.
Use [pypyr.steps.shells]({{< ref "shells.md" >}}) for concurrent shells instead.

## inject variables into command
You can inject context variables into your command. This means you can use any
of the pypyr [context manipulation steps]({{< ref "/topics/context" >}}) to
manipulate your inputs and then pass these to any of this step's inputs.

Supports string [substitutions]({{< ref "docs/substitutions">}}) for all step
inputs in expanded syntax.

```yaml
- name: pypyr.steps.set
  comment: set some arb values in context
  in:
    set:
      user: myusername
      password: my-super-secret-password
      ftp_domain: ftp.mydomain.com
      file_name: file.ext
      url: https://my-domain.arb/subdir/myfile.arb
      local_file: filename.ext
      save_me: False
      my_path: /mydir/subdir

# simple syntax
- name: pypyr.steps.cmds
  comment: use substitution expressions to inject variables into cmd
  in:
    cmds:
      - wget ftp://{user}:{password}@{ftp_domain}/path/{file_name}
      - curl -o {local_file} {url}

# expanded syntax
- name: pypyr.steps.cmds
  comment: substitutions work on all inputs
  in:
    cmds:
      run:
        - wget ftp://{user}:{password}@{ftp_domain}/path/{file_name}
        - curl -o {local_file} {url}
      cwd: '{my_path}' # set any step input option with formatting expression
      save: '{save_me}'
```

## combine parallel & serial execution
If you want to start some commands at the same time, but still create a serial
sequence where some commands run in serial one after the other, you can use a
nested list to encode a serial sequence as part of the parallel workload:

```yaml
- name: pypyr.steps.cmds
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
- name: pypyr.steps.cmds
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
- name: pypyr.steps.cmds
  in:
    cmds:
      run:
        - echo A
        - [echo B.1, echo B.2, echo B.3]
        - echo C
      save: True

- name: pypyr.steps.cmds
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

## change the working directory
If you specify `cwd`, pypyr will change the current working directory to
`cwd` to execute the commands. The `cwd` you set applies to all the run
instructions in `run`.

The directory change is only for the duration of this step, not any subsequent
steps.

If you do not specify `cwd`, it defaults to the current working directory,
which is from wherever you are running pypyr.

```yaml
- name: pypyr.steps.cmds
  comment: run commands in different directory
  in:
    cmds:
      run:
        - ./a-executable --arg two
        - ./b-executable --arg three
      cwd: path/mydir # cwd applies to each instruction in run
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
- name: pypyr.steps.cmds
  comment: each cmd in run will execute in cwd
  in:
    cmds:
      run:
        - ./a-executable --arg two
        - ./b-executable --arg two
      cwd: path/mydir # cwd applies to each instruction in run
```

As usual for paths, you can use `.` for current and `..` for the parent
directory.

## save output stdout & stderr
Set `save: True` to capture the output of all the run instructions:
```yaml
- name: pypyr.steps.cmds
  in:
    cmds:
      - run:
          - ./a-executable --arg two
          - ./b-executable --arg three
        save: True
```

pypyr saves the results to context as a list `cmdOut` in the input order.

```yaml
cmdOut:
  - SubprocessResult(cmd=['./a-executable', '--arg', 'two'], returncode=0, stdout='2', stderr=b'')
  - SubprocessResult(cmd=['./b-executable', '--arg', 'three'], returncode=0, stdout='3', stderr=b'')
```

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

Be aware that if pypyr could not launch the program, `cmdOut` will contain an
`Exception` object instead of a `SubprocessResult` - see [error
handling](#error-handling) for details.

Be aware that if `save` is `True`, all of the command output ends up in
memory. Don't set it unless your pipeline actually uses the stdout/stderr
response in subsequent steps. Instead of saving the output to memory like this,
you can alternatively [write output to file](#save-output-to-file).

Only use `save: True` when you actually need to use the stdout or stderr output
in subsequent steps - you don't need to set it just to check that the return
code is 0. pypyr will raise an error automatically if ANY of the return-codes is
not 0.

If this is not what you want, you can suppress errors with the [swallow
decorator]({{< ref "docs/decorators/swallow">}}) and use the [runErrors]({{< ref
"/docs/getting-started/error-handling#use-runtime-error-details-inside-pipeline"
>}}) list on subsequent steps.

When you use a sub-list to embed a serial sequence in the parallel workload, the
`cmdOut` output will similarly contain a sub-list for the serial results, again
in the order specified by the input:

```yaml
- name: pypyr.steps.cmds
  in:
    cmds:
      - run:
          - echo A
          - [echo B.1, echo B.2, echo B.3]
          - echo C
        save: True
```

In this case the result will be:
```python
[
  SubprocessResult(cmd=['echo', 'A'], returncode=0, stdout='A', stderr=b''),
  # notice the B results are contained in a sub-list
  [
    SubprocessResult(cmd=['echo', 'B.1'], returncode=0, stdout='B.1', stderr=b''),
    SubprocessResult(cmd=['echo', 'B.2'], returncode=0, stdout='B.2', stderr=b''),
    SubprocessResult(cmd=['echo', 'B.3'], returncode=0, stdout='B.3', stderr=b'')
  ],
   SubprocessResult(cmd=['echo', 'C'], returncode=0, stdout='C', stderr=b'')
]
```

You can access nested lists in the following steps like this:
```yaml
- name: pypyr.steps.echo
  run: !py cmdOut[1][1].returncode == 0
  in:
    echoMe: |
            you'll only see me if cmd ran successfully with return code 0.

            This is B.1's output:
            the command output was: {cmdOut[1][0].stdout}
            the error output was: {cmdOut[1][0].stderr}
            
            This is B.3's output:
            the command output was: {cmdOut[1][2].stdout}
            the error output was: {cmdOut[1][2].stderr}
```

The `cmdOut` key contains a list of `SubprocessResult` instances. The full
schema for this object is:
```python
SubProcessResult():
  cmd: list[str] # the input cmd split into args
  returncode: int
  stdout: str | bytes | None
  stderr: str | bytes | None
```

### debugging output
A quick way of seeing exactly what is in `cmdOut` is just to echo it out:

```yaml
- name: pypyr.steps.echo
  in:
    echoMe: '{cmdOut}'
```

If something strange is going on with the arguments you pass to the run
instruction, pay attention to the `cmdOut.cmd` attribute, which will show you
how the command string was parsed into component arguments. If there is a
problem with your argument split, the problem is very likely to be that you
aren't escaping special characters or whitespace with quotes.

In this example the value for `--arg2` contains a space - to treat it as a
single argument you wrap it in quotes:
```yaml
- name: pypyr.steps.cmds
  comment: wrap arguments with spaces in quotes
  in:
    cmds:
      run:
        - ./my-executable --arg1 value
        - ./another-executable --arg1 value --arg2 "value with space"
```

### encoding
By default pypyr treats the command output as text. It decodes the bytes
returned by the executable using the default system encoding and strips
white-space like line-feeds from the end. 

If you are saving output from an executable that is not in the default encoding,
you can set the `encoding`:
```yaml
- name: pypyr.steps.cmds
  comment: save output & decode in specified encoding.
  in:
    cmds:
      run:
        - ./my-executable --arg1 value
        - ./another-executable --arg1 value --arg2 "value with space"
      save: True
      encoding: utf-16
```

The `encoding` setting is only applicable when `save` is `True`. Both stdout &
stderr will use the encoding you set.

The default system encoding is very likely to be `utf-8`, unless you're on
Windows. See here for a [list of available encoding
codecs](https://docs.python.org/3/library/codecs.html#standard-encodings).

You can set the
[default_cmd_encoding setting in config]({{< ref "/docs/getting-started/config#default_cmd_encoding" >}})
to override the system default.

### binary output
By default pypyr deals with the executable's output as text in the system's
default encoding. If you want to capture the output as raw bytes without any
text decoding & line-ending handling, you can set `bytes: True`.

```yaml
- name: pypyr.steps.cmds
  comment: save the output as raw bytes.
           this will bypass all text decoding.
  in:
    cmds:
      run:
        - ./my-executable --arg1 value
        - ./another-executable arg1 "arg with space"
      save: True
      bytes: True
```

The `bytes` setting is only applicable when `save` is `True`. Both stdout &
stderr will save the raw bytes returned by the executable when `bytes: True`.

When you set `bytes: True` pypyr will ignore `encoding` and it won't perform
line-ending normalization - so your output might have line-endings at the end
depending on the executable you're calling.

## error handling
pypyr will run all the commands specified in parallel. If any of the commands
raise an error, the other concurrent commands will continue running. pypyr
does not stop execution for other cmds if any one raises an error.

The exception to this is when you have a serial sequence inside your parallel
workload - in a serial sequence the preceding command has to succeed before
the next command will run.

```yaml
- name: pypyr.steps.cmds
  in:
    cmds:
      - A
      - [B.1, B.2, B.3]
      - C
```

In this example A, B.1 and C all start in parallel. If any of these fail, it
will not prevent the other commands from finishing. B.2, however, will only
run after B.1 succeeds, and B.3 will only run after B.2 succeeds. If B.2 fails,
B.3 will not run.

There are 2 types of errors:
- An exception trying to launch the application.
  - This is when the application couldn't run at all.
  - Typically this might be because the path to the application is wrong, or
    maybe you don't have permissions to launch the executable.
- The program ran but returned a non-zero return-code.
  - In this case the application ran, but it returned an error-code.

Once all the commands complete, if there were any errors, pypyr raises an
exception. This is an aggregate exception of type `pypyr.errors.MultiError`,
which contains a flattened collection of all the errors raised by the individual
commands in the same order the original commands were listed. Flattened means
that even where you specified serial sequences with a sub-list, the `MultiError`
collection is flat - so in the case of the above example you'd have `[A, B.1,
C]` if each command raised an error.

As always, when the step raises an exception, pypyr will stop processing the
pipeline and not run the next step. You will see an exception in the
terminal like this, with the MultiError listing each error:

```text
MultiError: The following error(s) occurred while running the async commands:
FileNotFoundError: [Errno 2] No such file or directory: 'xyz/does-not-exist'
SubprocessError: Command '['./raise-err.sh']' returned non-zero exit status 1. stderr: boom
```

If you want to ignore the error or continue to the next step in the pipeline,
you can suppress errors with the [swallow decorator]({{< ref
"docs/decorators/swallow">}}) and use the [runErrors]({{< ref
"/docs/getting-started/error-handling#use-runtime-error-details-inside-pipeline"
>}}) list on subsequent steps.

Here is an example to demonstrate the relation between `cmdOut` and `runErrors`:

```yaml
- name: pypyr.steps.cmds
  swallow: True
  in:
    cmds:
      run:
        - echo A
        - xyz/does-not-exist --arb
        - echo B
        - ./raise-err.sh
        - echo C
      save: True

- name: pypyr.steps.echo
  in:
    echoMe: '{cmdOut}'

- name: pypyr.steps.echo
  in:
    echoMe: '{runErrors}'
```

`xyz/does-not-exist --arb` will raise an exception because the path to the
executable is invalid. `./raise-err.sh` is a valid script and will run, but
it returns a return-code of `1`, which means an error.

`cmdOut` contains the saved output for the commands. `cmdOut` looks like this:
```python
[
  SubprocessResult(cmd=['echo', 'A'], returncode=0, stdout='A', stderr=b''),
  FileNotFoundError(2, 'No such file or directory'),
  SubprocessResult(cmd=['echo', 'B'], returncode=0, stdout='B', stderr=b''),
  SubprocessResult(cmd=['./raise-err.sh'], returncode=1, stdout=b'', stderr='boom'),
  SubprocessResult(cmd=['echo', 'C'], returncode=0, stdout='C', stderr=b'')
]
```

Notice that the `cmdOut` contains an exception object rather than a
`SubprocessResult` for the 2nd run instruction, because it couldn't launch the
non-existent executable. The 4th result did run, but returned 1 and therefore
counts as an error.

`runErrors` contains the errors the step would have raised but ignored because
`swallow: True`. If `swallow` is `False` (which is the default), you can access
`cmdOut` or the `runErrors` list in a
[failure handler]({{< ref "/docs/getting-started/error-handling#failure-handlers" >}}).

## save output to file
By default, the executable output writes to the parent process' stdout &
stderr streams. Normally this means that any program output will print
in the terminal from where you are running pypyr.

You can save the output to file instead:
```yaml
- name: pypyr.steps.cmds
  comment: save the output to file.
           this will NOT print the output to console,
           but redirect it to the output files instead.
  in:
    cmds:
      run:
        - ./my-executable --arg1 value
        - ./another-executable arg1 "arg with space"
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
- name: pypyr.steps.cmds
  comment: save both stdout & stderr output to the same file.
           this will NOT print the output to console,
           but redirect it to the output file instead.
  in:
    cmds:
      run:
        - ./my-executable --arg1 value
        - ./another-executable arg1 "arg with space"
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
- name: pypyr.steps.cmds
  comment: discard all output.
  in:
    cmds:
      run: ./my-executable --arg1 value
      stdout: /dev/null
      stderr: /dev/null
```

The previous example redirects both stdout & stderr. You can also selectively
redirect only one of these to dump to null - so if you want to see stderr output
but not standard output:
```yaml
- name: pypyr.steps.cmds
  comment: only suppress stdout
  in:
    cmds:
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
- name: pypyr.steps.cmds
  in:
    cmds:
      - '"dir with space/file with space" arg1 "arg2 with space"'
      - '"dir with space"/"file with space"'
      - ./"dir with space"/"file with space"
```

Note the "extra" pair of single-quotes (') is there when the string begins and
ends with literal double-quotes so that the YAML parser reads the double quotes
(") literally instead of interpreting these double quotes as structural YAML
markers meaning string.

## microsoft windows
On Windows the distinction between running a command and running a shell can
lead to surprises - `echo`, `dir` or `copy` are not actually stand-alone
programs, these instead are built into the Windows shell. Use
[pypyr.steps.shells]({{< ref "shells.md" >}}) instead if you want to run
concurrent shell commands.

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

How Windows resolves the program name is relatively complex. When in doubt,
use the full absolute path and extension:

```yaml
steps:
  - name: pypyr.steps.cmds
    in:
      cmds:
        - "C:/program files/acme corp/program.exe" arg1 "arg2 with space"
        - '"C:/program files/anon corp/arb-program.exe"'
```

## invoking python
```yaml
steps:
  - name: pypyr.steps.cmds
    in:
      cmds:
        - python -m stuff.do
        - python mydir/myfile.py
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