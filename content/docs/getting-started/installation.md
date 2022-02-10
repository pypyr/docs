---
title: install pypyr
description: How to install pypyr with pip & how to run pypyr via docker
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
categories: [install]
menu:
  docs:
    title: install pypyr
    parent: getting-started
weight: -90
seo_article_headline: How to install the pypyr pipeline task-runner.
seo_description: pypyr installs quickly & easily via pip. Or, the ready-made docker container is a drop-in replacement for the cli.
# topics: [install, pypi, docker]
---
# install pypyr
## pip
```bash
$ pip install pypyr
```

### upgrades
Use the standard pip upgrade switch:

```bash
$ pip install --upgrade pypyr
```

## supported o/s
pypyr runs on Linux, MacOS & Windows.

The [automated CI process](https://github.com/pypyr/pypyr/actions) with [100%
test coverage](https://app.codecov.io/gh/pypyr/pypyr) checks for cross-platform
compatibility.

pypyr also runs on CI servers & containers - pretty much anywhere with a Python
run-time will work.

{{% note tip %}}
Windows users, please don't feel left out!

pypyr does run on Windows and the full automated test suite with 100% test
coverage ensures Windows compatibility on every code change going into the main
branch, same as for POSIX systems.

The documentation examples tend to show POSIX style paths & commands more so
than Windows, but this is not a limitation in the pypyr tool itself, it's down
to time-constraints in producing exhaustive documentation.

If you have the time, feel free to help out open source software and [contribute
by improving the docs]({{< ref
"/docs/contributing/contribute-to-pypyr#documentation" >}})!

{{%/ note %}}

## python version
Tested against Python >=3.7

## docker
Stuck with an older version of python? Want to run pypyr in an
environment that you don't control, like a CI server somewhere?

You can use the official [pypyr docker image](https://hub.docker.com/r/pypyr/pypyr/)
as a drop-in replacement for the pypyr executable.

```bash
$ docker run pypyr/pypyr echo "Ceci n'est pas une pipe"
```
