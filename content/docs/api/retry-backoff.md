---
title: custom retry backoff
linktitle: retry backoff
description: Create your own custom retry backoff algorithm.
date: 2021-01-18
# card_extra_summary:
#   heading: input context property
#   details: "`assert` (dict)"
# categories: [ steps ]
menu:
  docs:
    parent: api
    name: retry backoff
seo_article_headline: Create a custom retry backoff strategy for an automation pipeline.
seo_description: Create a custom task-runner retry backoff implementation in a few lines of Python code.
topics: [ custom code ]
---
# create a custom retry backoff algorithm
## custom retry backoff algorithms
A retry backoff strategy is an algorithm that varies the backoff interval
between retries. If you do not want to use one of the builtin [common backoff
retry strategies]({{< ref "/docs/decorators/retry#backoff-algorithms">}}), you
can implement your own by deriving a callable from `BackoffBase`, which lives in
`pypyr.retries`.

For real life inspiration, you can see all of [pypyr's builtin retry backoff
implementations on
github](https://github.com/pypyr/pypyr/blob/main/pypyr/retries.py).

## custom retry signature
You can find `BackoffBase` in `pypyr.retries`.

```python
class BackoffBase(abc.ABC):
    """Derive from me for a custom back-off strategy.

    Attributes:
        sleep (float): initial sleep interval in seconds.
        max_sleep (float): upper bound for sleep interval calculation.
        jrc (float): Jitter Range Coefficient.
        kwargs (dict): arbitrary keyword args for use in deriving classes.
    """

    def __init__(self, sleep=1, max_sleep=None, jrc=0, kwargs=None):
        """Initialize back-off strategy.

        Args:
            sleep (float): positive sleep interval in seconds.
            max_sleep (float): upper bound for sleep interval calculation.
            jrc (float): jitter randomizes between [sleep*jrc] and [sleep].
            kwargs (dict): arbitrary arguments for use by deriving classes.
        """
        self.sleep = sleep
        self.max_sleep = max_sleep
        self.jrc = jrc
        self.kwargs = kwargs

    @abc.abstractmethod
    def __call__(self, n):
        """Implement back-off sleep interval calculation here.

        Args:
            n (int): The iteration counter.

        Returns:
            Float. The sleep interval for iteration n.
        """
        raise NotImplementedError()

    def min(self, sleep):
        """Return the smaller of sleep vs max_sleep."""
        max_sleep = self.max_sleep
        return min(sleep, max_sleep) if max_sleep else sleep

    def randomize(self, sleep):
        """Get random number in between (jrc*sleep) and (sleep).

        Does NOT guard against result < max_sleep. This is on purpose.
        It allows you to jitter under or over the sleep interval depending on
        if jrc > 1.
        """
        return random.uniform(sleep*self.jrc, sleep)
```

## custom backoff example
```python
import pypyr.retries

class MyBackoff(pypyr.retries.BackoffBase):
    """Your own custom backoff algorithm."""

    def __call__(self, n):
        """Just return n + sleep + custom input myArg."""
        # probably do something more useful here.
        # myArg comes from backoffArgs you set in pipeline.
        # this will sleep for:
        # (iteration count + sleep + myArg) in seconds.
        return n + self.sleep + self.kwargs['myArg']
```

You can use `self.min(my_interval)` to honor `sleepMax` and 
`self.randomize(my_interval)` to add jitter with `jrc` in the same way the
builtin retry algorithms do.

## use a custom backoff in a pipeline
You can use your custom backoff in a pipeline by setting the `backoff` property
on the step's [retry decorator]({{< ref "/docs/decorators/retry">}}).

Specify your custom backoff callable by using its containing module's absolute
name followed by the class name.

To load a custom callable, the name should be in format
`package.module.ClassName` or `module.ClassName`.

The usual [custom module import resolution rules]({{< ref
"/docs/api/custom-module-search-path" >}}) apply.

Assuming your callable class `MyBackoff` exists in a file like this 
`{pipelinedir}/mydir/mybackoffs.py`, you can use use it in your pipeline like
this:

```yaml
- name: pypyr.steps.cmd
  retry:
    backoff: mydir.mybackoffs.MyBackoff
    max: 123
    sleep: 456
    backoffArgs:
      myArg: 789.01
  in:
    cmd: curl xyz://manifestly-wrong-url
```

If you package your code and you install the package into the active python
environment, you can of course use the usual python package name instead,
followed by the class name:

```yaml
- name: pypyr.steps.cmd
  retry:
    backoff: mypackage.mymodule.MyBackoff
    max: 123
    sleep: 456
    backoffArgs:
      myArg: 789.01
  in:
    cmd: curl xyz://manifestly-wrong-url
```