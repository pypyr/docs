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
