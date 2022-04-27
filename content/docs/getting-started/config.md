---
title: config
description: Configure pypyr run-time settings with environment variables & config files.
date: 2021-02-14
# categories: [install]
weight: -30
menu:
  docs:
    parent: getting-started
seo_article_headline: How to configure pypyr
seo_description: Configure pypyr task-runner run-time settings & defaults with pyproject.toml or the yaml config file.
# topics: [config]
# topics_weight: -100
---
# configuration
You can configure pypyr run-time settings & defaults using environment variables
and configuration files.

This is entirely optional - by default pypyr runs out-of-box without any extra
configuration necessary. If you're just getting started with pypyr, don't bother
reading the rest of this, it's not essential - just be aware that you can
configure pypyr if the need does arise.

Rather than use config to change run-time settings, you're more likely to use
the config file(s) for extra functionality like [injecting variables](#vars) or 
defining your own [shortcuts](#shortcuts) to longer pypyr command sequences.

{{% note tip %}}
API users, be aware that you explicitly have to opt in to use the config file 
look-up functionality. See
[config initialization with the API]({{< ref "/docs/api/run-pipeline#initialize-config" >}}).

Don't worry, it's just one extra line of code!
{{% /note %}}

## toml vs yaml
You can configure pypyr using either the pypyr yaml config file format, or
`pyproject.toml`, or a combination of both.

Neither is "better". If you prefer YAML, use YAML. If your project already uses
`pyproject.toml`, or if you just like TOML more than YAML, use that. It doesn't
matter, use whichever works for you.

You can also use both at the same time, using `./pypyr-config.yaml` to over-ride
or augment settings in `./pyproject.toml`.

The underlying structure for pypyr settings in both the yaml and toml formats is
exactly the same, it's just that the syntax differs.

## config file locations
pypyr goes through a look-up sequence to find all the applicable config files
for a run. The general principle is that pypyr looks in the current working
directory (i.e your project directory) first, and then goes on to look for the
current user's specific config and then lastly system-wide shared global
configuration.

The effective configuration for a run is the merged combination of ALL the
config files found. The merge goes from specific to general - meaning config
from earlier in the order of precedence will over-ride config settings found
later in the sequence.

A project-specific config in the current working directory overrides user
config, which in turn over-rides system-wide global config.

This allows you to check shared config into a source control repo while still
over-riding individual settings for your individual/specific local machine by
adding your own `pypyr-config.yaml` next to the `pyproject.toml` file.

The general look-up sequence is:
1. ./pypyr-config.yaml
2. ./pyproject.toml
3. {user config dir}/pypyr/config.yaml
4. {global config dirs}/pypyr/config.yaml

Settings from earlier in the sequence take priority over settings later in the
sequence.

The location of `{user config dir}` and `{global config dirs}` depends on your
platform (operating system).

Using any or using all of these file locations is entirely optional. pypyr will
not raise an error if it doesn't find a config file.

### linux
pypyr follows the [XDG Base Directory
specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html)
for all POSIX systems.

pypyr will default to this config file look-up sequence on Linux (or any POSIX
system):
1. `./pypyr-config.yaml`
2. `./pyproject.toml`
3. `~/.config/pypyr/config.yaml`
4. `/etc/xdg/pypyr/config.yaml`

You can control the location of the user config directory with the
environment variable `${XDG_CONFIG_HOME}` and for shared system-wide global
config with `${XDG_CONFIG_DIRS}`. If these environment variables are not set,
pypyr will use the XDG Base Dir defaults as given above.

### macos
pypyr will default to this config file look-up sequence on MacOs:

1. `./pypyr-config.yaml`
2. `./pyproject.toml`
3. `~/.config/pypyr/config.yaml`
4. `/Library/Application Support/pypyr/config.yaml`

On MacOs pypyr will also honor the XDG Base Dir environment variables if these
exist.

### windows
pypyr will default to this config file look-up sequence on Windows:

1. `.\pypyr-config.yaml`
2. `.\pyproject.toml`
3. `~\.config\pypyr\config.yaml`
4. `C:\ProgramData\pypyr\config.yaml`

pypyr will honor however you've configured your Windows installation's home
and shared config folders.

`~` refers to the home directory, which is given by the `%USERPROFILE%`
environment variable. Generally this is going to be `C:\Users\YourUserName`.

Windows gives the location for the shared system-wide config with the
`%ALLUSERSPROFILE%` environment variable - the default is `C:\ProgramData`.

{{% note tip %}}
See Windows help for the proper method to change the default locations for
`%USERPROFILE%` and `%ALLUSERSPROFILE%`.

I'd love to give you a link, but it's not immediately obvious what, if any, the
One True Way is, so how you do it is up to you. Enjoy the research journey ;-).

(Feel free to get in touch if you find a definitive answer!)
{{% /note %}}

On Windows pypyr will also honor the XDG Base Dir environment variables if these
exist.

## yaml format
This is the pypyr yaml config format. The format is the same whether you are
configuring per project with `./pypyr-config.yaml` or user/global with 
`pypyr/config.yaml`.

This example shows all the possible properties with some typical values. If
you're copying & pasting, remove the ones you are not using - you only need to
specify properties you actually want to change.

```yaml
default_backoff: fixed
default_encoding: utf-8
default_failure_group: on_failure
default_group: steps
default_loader: pypyr.loaders.file
default_success_group: on_success
json_ascii: false
json_indent: 2
log_config:
  version: 1
  handlers:
    console:
      class: logging.StreamHandler
      formatter: brief
      level: INFO
      stream: ext://sys.stdout
    file:
      class: logging.handlers.RotatingFileHandler
      formatter: precise
      filename: logconfig.log
      maxBytes: 1024
      backupCount: 3
  formatters:
    brief:
      format: '%(message)s'
    precise:
      format: '%(asctime)s %(levelname)-8s %(name)-15s %(message)s'
      datefmt: '%Y-%m-%d %H:%M:%S'
  loggers:
    root:
      level: DEBUG
      handlers:
        - console
        - file
log_date_format: '%Y-%m-%d %H:%M:%S'
log_detail_format: '%(asctime)s %(levelname)s:%(name)s:%(funcName)s: %(message)s'
log_notify_format: '%(message)s'
pipelines_subdir: pipelines
shortcuts:
  my-shortcut:
    pipeline_name: /mydir/my-pipeline
    args:
      akey: a value
      anotherkey: 123
vars:
  myvar: myvalue
  mylist:
    - one
    - two
  mydict:
    key1: value1
    key2: value2
```

pypyr will read the yaml file in the default system encoding. This is usually
`utf-8`, except on Windows. You can explicitly set which encoding to use
with the [PYPYR_ENCODING](#pypyr_encoding) environment variable.

## pyproject.toml
pypyr configuration in `pyproject.toml` goes under the `[tool.pypyr]` table.

Note that the [yaml format](#yaml-format) shown above and toml configuration
under `[tool.pypyr]` is structurally identical, it's just the syntax that
differs between the formats.

Here is an excerpt showing all the possible pypyr config properties in
`pyproject.toml` with some typical values.  If you're copying & pasting, remove
the ones you're not using - you only need to specify properties you actually
want to change.

```toml
[tool.pypyr]
default_backoff = "fixed"
default_encoding = "utf-8"
default_failure_group = "on_failure"
default_group = "steps"
default_loader = "pypyr.loaders.file"
default_success_group = "on_success"
json_ascii = false
json_indent = 2
log_date_format = "%Y-%m-%d %H:%M:%S"
log_detail_format = "%(asctime)s %(levelname)s:%(name)s:%(funcName)s: %(message)s"
log_notify_format = "%(message)s"
pipelines_subdir = "pipelines"

[tool.pypyr.log_config]
version = 1

[tool.pypyr.log_config.handlers.console]
class = "logging.StreamHandler"
formatter = "brief"
level = "INFO"
stream = "ext://sys.stdout"

[tool.pypyr.log_config.handlers.file]
backupCount = 3
class = "logging.handlers.RotatingFileHandler"
filename = "logconfig.log"
formatter = "precise"
maxBytes = 1024

[tool.pypyr.log_config.formatters.brief]
format = "%(message)s"

[tool.pypyr.log_config.formatters.precise]
datefmt = "%Y-%m-%d %H:%M:%S"
format = "%(asctime)s %(levelname)-8s %(name)-15s %(message)s"

[tool.pypyr.log_config.loggers.root]
handlers = ["console", "file"]
level = "DEBUG"

[tool.pypyr.shortcuts]
[tool.pypyr.shortcuts.my-shortcut]
    pipeline_name = "/mydir/my-pipeline"
    args = {akey = "a value", anotherkey = 123 }

[tool.pypyr.vars]
mydict = {key1 = "value1", key2 = "value2"}
mylist = ["one", "two"]
myvar = "myvalue"
```

For the official specification on `pyproject.toml` see [PEP
518](https://www.python.org/dev/peps/pep-0518/) and [PEP
621](https://www.python.org/dev/peps/pep-0621/).

Per spec, TOML must always be in `utf-8`.

## config properties
### default_backoff
The default [retry backoff strategy]({{< ref
"/docs/decorators/retry#backoff-algorithms" >}}) to use on error retries.

Default value is `fixed`.

### default_encoding
Sets the default encoding to use for all text file operations.

If you explicitly set encoding on any [filesystem step]({{< ref
"/topics/filesystem/" >}}) it will override this default for that particular
step.

The default value of [None/null/not specified] means to use the platform default
encoding as given by the Python runtime. This is the eminently sensible `utf-8`
for most systems, but be aware on Windows it's still `cp1252`.

To check your platform's default encoding, do:
```python
import locale
locale.getpreferredencoding(False)
```

You can find the possible values for encoding here:
[Python standard encodings](https://docs.python.org/3/library/codecs.html#standard-encodings)

The value for `default_encoding` initializes from the `PYPYR_ENCODING`
environment variable. If the `PYPYR_ENCODING` environment variable is not set,
pypyr will take this to mean to use the system default. This subtlety is
important if you want to set the encoding for reading the pypyr yaml config
files - pypyr will use the value of the `PYPYR_ENCODING` $env to initialize
config and bootstrap reading the yaml config files, after which point any
`default_encoding` set in the config files will over-ride the environment
variable's value.

{{% note tip %}}
Windows users, it _might_ be an idea to set your over-all
Python runtime to use `utf-8` like all other modern platforms do. Then you
wouldn't need to worry about any of this.

See [PEP 0540](https://www.python.org/dev/peps/pep-0540/).

TLDR; Set the environment variable `PYTHONUTF8` to `1`.
 
{{% /note %}}

### default_failure_group
Default step-group to run on unhandled error condition in pipeline. 

You can over-ride this default with any of the following:
- [cli arg]({{< ref "/docs/cli/run-a-pipeline#pypyr-command-line-switches" >}}) `--failure`
- explicitly set `failure` on [pype]({{< ref "/docs/steps/pype#failure" >}})
- pass `failure` to the [api]({{<ref "/docs/api/run-pipeline#input-args" >}}).
- set [failure on shortcut]({{< ref "/docs/pipelines/shortcuts#failure" >}}).

Default value is `on_failure`.

### default_group
The default step-group name to run in a pipeline.

You can override this default with any of the following:
- [cli arg]({{< ref "/docs/cli/run-a-pipeline#pypyr-command-line-switches" >}}) `--groups`
- explicitly set `groups` on [pype]({{< ref "/docs/steps/pype#groups" >}})
- pass `groups` to the [api]({{< ref "/docs/api/run-pipeline#input-args" >}}).
- set [groups on shortcut]({{< ref "/docs/pipelines/shortcuts#groups" >}}).

Default value is `steps`.

### default_loader
The default [pipeline loader]({{< ref "/docs/api/pipeline-loader" >}}) to use to
find & load pipelines.

This is the absolute module name of the loader. If you've not installed it as a
package into your environment, it should resolve from the current working
directory.

You can override this default with any of the following:
- explicitly set `loader` on [pype]({{< ref "/docs/steps/pype#loader" >}})
- pass `loader` to the [api]({{< ref "/docs/api/run-pipeline#input-args" >}}).
- set [loader on shortcut]({{< ref "/docs/pipelines/shortcuts#loader" >}}).

Default value is `pypyr.loaders.file`.

### default_success_group
The default step-group to run on success completion of a pipeline.

You can over-ride this default with any of the following:
- [cli arg]({{< ref "/docs/cli/run-a-pipeline#pypyr-command-line-switches" >}}) `--success`
- explicitly set `success` on [pype]({{< ref "/docs/steps/pype#success" >}})
- pass `success` to the [api]({{<ref "/docs/api/run-pipeline#input-args" >}}).
- set [success on shortcut]({{< ref "/docs/pipelines/shortcuts#success" >}}).

Default value is `on_success`.

### json_ascii
If true, any of the steps that [dump json]({{< ref "/topics/json/" >}}) will
escape all non-ASCII characters so that the resulting output only contains ASCII
characters.

Default value is `false`, meaning to output as is.

### json_indent
Pretty print any [output json]({{< ref "/topics/json/" >}}) to this indent
level.

`None` means the most compact representation (everything in one line).

An empty string `""`, 0 or negative number will only insert newlines.

Default value is `2`.

### log_config
Initialize logging configuration from a dict/mapping.

The schema for this is here: [Python Configuration Dictionary Schema](https://docs.python.org/3/library/logging.config.html#configuration-dictionary-schema").

Note that if you set this `log_config` property it's up to you to configure the
logging entirely. pypyr will ignore all the other `log_` configuration settings.

Default value is None or empty.

### log_date_format
The date format to use in log output.

Note that pypyr ignores this setting if you set [log_config](#log_config).

Default value is `%Y-%m-%d %H:%M:%S`.

### log_detail_format
What logging output looks like when you explicitly set the log-level by passing
value to [cli arg]({{< ref
"/docs/cli/run-a-pipeline#pypyr-command-line-switches" >}}) `--log`

{{< app-window title="term" lang="fish" >}}
$ pypyr mypipeline --log 20
{{< /app-window >}}

When you do NOT pass the `--log` arg, pypyr will use the
[log_notify_format](#log_notify_format).

Note that pypyr ignores this setting if you set [log_config](#log_config).

Default value is `%(asctime)s %(levelname)s:%(name)s:%(funcName)s: %(message)s`.

### log_notify_format
What logging output looks like when you do NOT explicitly set the log-level by
passing value to [cli arg]({{< ref
"/docs/cli/run-a-pipeline#pypyr-command-line-switches" >}}) `--log`.

This is what you see when you run pypyr without doing anything special:

{{< app-window title="term" lang="fish" >}}
$ pypyr mypipeline
{{< /app-window >}}

When you also pass `--log`, pypyr will use the [log_detail_format](#log_detail_format).

Note that pypyr ignores this setting if you set [log_config](#log_config).

Default value is `%(message)s`.

### pipelines_subdir
The fallback [look-up location]({{< ref
"/docs/pipelines/lookup-order#directory-locations-lookup-order" >}})
sub-directory of the current working directory where the default file loader
looks for pipelines.

Default value is `pipelines`. This means to look in `{cwd}/pipelines/`.

### shortcuts
Create shortcuts to longer pypyr command sequences, saving you valuable typing
time and preventing keyboard wear-and-tear.

A shortcut allows you to save longer command sequences and input args so you can
use a friendly short alias to run a pipeline instead.

You can run your shortcut directly from the cli like this:

{{< app-window title="term" lang="fish" >}}
$ pypyr my-shortcut
{{< /app-window >}}

If pypyr finds a matching shortcut name, it will load & initialize the pipeline
based upon the inputs you set in the shortcut.

See [shortcuts]({{< ref "/docs/pipelines/shortcuts" >}}) for details & examples.

Default value is empty.

### vars
Inject these key-value pairs into your pipeline using [pypyr.steps.configvars]({{< ref "/docs/steps/configvars" >}}).

Default value is empty.


## environment variables
In order to bootstrap the configuration process, you can use the following
environment variables:

### PYPYR_ENCODING
Initialize the `default_encoding` config setting with this value. This is
especially useful to bootstrap the encoding to use to read the yaml config files
themselves.

If you set the `PYPYR_ENCODING` environment variable pypyr will use that for
everything by default unless you override it with the `default_encoding` setting
in one of your config files.

See [default_encoding](#default_encoding) for details.

### PYPYR_SKIP_INIT
Set to 1 to skip the [config file look-up sequence](#config-file-locations)
entirely. If 1, will just use the default values for everything - which is
everything pypyr needs for a no-frills vanilla run.

Skipping the config initialization will (somewhat) improve performance, but this
is unlikely to be something you'd notice if you're just running pypyr as a cli.

API users, note that even if you call `config.init()` explicitly, setting
`PYPYR_SKIP_INIT` to `1` will still bypass the file look-up sequence.

Defaults to 0. When `PYPYR_SKIP_INIT` does not exist or is 0, pypyr will do the
config file look-up sequence on each run from the CLI.

### PYPYR_CONFIG_GLOBAL
If set, do NOT look in the usual platform specific locations for user & global
config, but use this yaml file instead.

If set, the config file look-up sequence is:
1. ./pypyr-config.yaml
2. ./pyproject.toml
3. $PYPYR_CONFIG_GLOBAL

The value of `PYPYR_CONFIG_GLOBAL` should be the full path to the yaml
configuration file - for example `/mydir/myfile.yaml` or `C:\mydir\myfile.yaml`.

### PYPYR_CONFIG_LOCAL
The name of the project specific yaml configuration file to look for in the
current working directory.

The default value is `pypyr-config.yaml`.

## troubleshooting
You can see what config pypyr has found by running the built-in `config-show`
pipeline:

```text
$ pypyr config-show
```

Under `WRITEABLE PROPERTIES` the output shows a comprehensive listing of the
effective config settings merged from all the config sources found.

If you're trying to figure out why your config is not working, look under
`COMPUTED PROPERTIES` for:

- `config_loaded_paths` 
    - List of all the config files found and loaded, given in order loaded.
    - Config from later files overrides config from earlier files.
    - Only lists a file if there were pypyr properties found in it. So if your
      `pyproject.toml` does not have a `[tool.pypyr]` table pypyr will NOT list
      the file under `config_loaded_paths`.
- `platform_paths`
    - Locations where pypyr will look for config files, even if it doesn't find
      anything at those locations.
    - This shows you where pypyr will look for config files based on the
      platform and/or environment variables giving that platform's config
      directories. This does NOT mean there necessarily are files at those
      locations.
    - By contrast, `config_loaded_paths` gives the files that pypyr actually
      found.

In this example partial output of `$ pypyr config-show`, pypyr did not find
_any_ config files (`config_loaded_paths` is empty), whereas `platform_paths`
shows you where pypyr looked for those files:

```text
COMPUTED PROPERTIES:

config_loaded_paths: []
cwd: /Users/captainhook/git/pypyr/pypyr-example
is_macos: True
is_posix: False
is_windows: False
platform: darwin
platform_paths:
  config_user: /Users/captainhook/.config/pypyr/config.yaml
  config_common:
    - /Library/Application Support/pypyr/config.yaml
  data_dir_user: /Users/captainhook/.local/share/pypyr
  data_dir_common:
    - /Library/Application Support/pypyr
pyproject_toml:
skip_init: False
```

If pypyr found both `./pyproject.toml` and `./pypyr-config.yaml` in the current
directory for project specific config, you'll see:

```text
COMPUTED PROPERTIES:

config_loaded_paths:
  - pyproject.toml
  - pypyr-config.yaml
```

Remember that pypyr will merge the config settings found in all the loaded files
in the order given in [config file locations](#config-file-locations) - so here
settings in `pypyr-config.yaml` will override settings in `pyproject.toml`.

Properties from files later in the `config_loaded_paths` list take priority over
properties from files earlier in list.

You can see the effective settings (i.e the final result of the merge) under
`WRITEABLE PROPERTIES`.