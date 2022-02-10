---
title: pypyr docs
linktitle: docs
description: pypyr help documentation
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
seo_article_headline: pypyr technical help documentation & usage instructions home.
seo_description: pypyr is a task-runner cli & api for yaml pipelines. This is technical documentation for the open-source pypyr project.
seo_is_carousel: true
---
# pypyr technical documentation
Here you will find technical documentation for using the pypyr task-runner. The
documentation covers how to sequence your tasks in a pipeline in yaml format
and how to run your pipeline tasks from the CLI or the Python API.

[![build status](https://github.com/pypyr/pypyr/workflows/lint-test-build/badge.svg?branch=main)](https://github.com/pypyr/pypyr/actions)
[![coverage status](https://codecov.io/gh/pypyr/pypyr/branch/main/graph/badge.svg)](https://codecov.io/gh/pypyr/pypyr)
[![pypi version](https://badge.fury.io/py/pypyr.svg)](https://pypi.python.org/pypi/pypyr/)
[![apache 2.0 license](https://img.shields.io/github/license/pypyr/pypyr)](https://opensource.org/licenses/Apache-2.0)

## pypyr is a task-runner cli & api
pypyr is a command line interface & api to run pipelines defined in yaml.
Think of pypyr as a simple task runner that lets you define and run
sequential steps.

Like a turbo-charged shell script, but less finicky.

> *pypyr*
>
> pronounce how you like, but I generally say *piper* as in "piping down the 
  valleys wild"

You can run loops, conditionally execute steps based on conditions you
specify, wait for status changes before continuing, break on failure
conditions or swallow errors. Pretty useful for orchestrating continuous
integration, continuous deployment and devops operations.

Read, merge and write configuration files to and from yaml, json or just plain 
old text.

pypyr runs on Linux, MacOS & Windows - anywhere with a Python runtime, really.

## documentation sections