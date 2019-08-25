---
title: pypyr.steps.cmd
linktitle: cmd
date: 2020-06-13T21:38:57+01:00
description: Run any external program, command, script.
draft: false
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
the context value `cmd` as a sub-process.

In `cmd`, you cannot use things like exit, return, shell pipes, filename
wildcards, environment variable expansion, and expansion of \~ to a
user's home directory. Use [pypyr.steps.shell]({{< ref "shell.md" >}}) for
this instead. `cmd` runs a command, it does not invoke the shell.

## set command input
Input context can take one of two forms:

```yaml
- name: pypyr.steps.cmd
  description: passing cmd as a string does not save the output to context.
               it prints stdout in real-time.
  in:
    cmd: 'echo ${PWD}'
- name: pypyr.steps.cmd
  description: passing cmd as a dict allows you to specify if you want to
               save the output to context.
               it prints command output only AFTER it has finished running.
  in:
    cmd:
      run: 'echo ${PWD}'
      save: False # optional. Set True to save cmd output to context.
      cwd: './current/working/dir/here' # optional. Default to pypyr execution dir.
```

Supports string [substitutions]({{< ref "docs/substitutions">}}).

### changing the working directory
If you specify `cwd`, pypyr will change the current working directory to
`cwd` to execute this command. The directory change is only for the
duration of this step, not any subsequent steps. If you specify `cwd`,
the executable or program set in `run` is relative to the `cwd` if
the `run` cmd specifies a relative path.

If you do not specify `cwd`, it defaults to the current working directory,
which is from wherever you are running pypyr.

### save cmd output stdout & stderr
Set `save: True` to capture the command's output.

Be aware that if `save` is `True`, all of the command output ends up in
memory. Don't specify it unless your pipeline uses the stdout/stderr
response in subsequent steps.

Keep in mind that if the invoked command return-code is non-zero pypyr will automatically raise a `CalledProcessError` and stop the pipeline, so assuming 
the command returns a non-zero error code you do not have to do anything 
special to get the pipeline to fail automatically. If this is not the behavior 
you want, you can suppress errors with the `swallow` decorator and use the 
`runErrors` list on subsequent steps. Only use `save: True` when you actually 
need the stdout or stderr output.

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

## example
Example pipeline yaml:

```yaml
steps:
  - name: pypyr.steps.cmd
    in:
      cmd: ls -a
```

See a worked [example for cmd](https://github.com/pypyr/pypyr-example/tree/master/pipelines/shell.yaml).
