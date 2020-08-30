---
title: aws plugin
linktitle: aws
description: Automate AWS devops. Interact with the AWS sdk api without writing any code. Supports all AWS Client functions, such as S3, EC2, ECS & co. via the AWS low-level Client API.
date: 2020-08-19
menu:
  docs:
    identifier: aws-overview
    name: overview
    parent: aws
    weight: -100
seo_is_carousel: true
seo_article_headline: Automate AWS scripting with an automation task-runner.
seo_description: AWS automation without code. Auto retries, conditional execution & loops. Prepare and parse complex inputs & outputs.
# topics: []
cascade:
  feedback_bug: https://github.com/pypyr/pypyr-aws/issues/new?title=Re:%%20%s&labels=bug
---
# pypyr aws plug-in
## aws automation without code
Run anything on aws. No really, anything! If the aws api supports it, the pypyr 
aws plug-in supports it.

It's a pretty easy way of invoking the aws api as a step in a series of steps 
without having to write code. 

Why use this when you could just use the aws cli instead? The aws cli is all 
kinds of awesome, but I find more often than not it's not just one or two 
ad hoc cli or aws api methods you have to execute - especially when automating 
and scripting you actually need to run a sequence of commands, where the output 
of a previous command influences what you pass to the next command.

Sure, you can bash it up, and I do that too, but running it as a pipeline via 
pypyr has actually made my life quite a bit easier because of not having to 
deal with conditionals, error traps and input validation.

pypyr is especially helpful with preparing the configuration inputs for aws 
function inputs. The json is pretty gnarly to work with from the cli and within 
a casual script, whereas with pypyr you can work with context, variable 
substitution and, critically, pretty much copy & paste from the AWS help 
documentation to get your inputs right.

[![build status](https://github.com/pypyr/pypyr-aws/workflows/lint-test-build/badge.svg)](https://github.com/pypyr/pypyr-aws/actions)
[![coverage status](https://codecov.io/gh/pypyr/pypyr-aws/branch/master/graph/badge.svg)](https://codecov.io/gh/pypyr/pypyr-aws)[![pypi version](https://badge.fury.io/py/pypyraws.svg)](https://pypi.python.org/pypi/pypyraws/)
[![apache 2.0 license](https://img.shields.io/github/license/pypyr/pypyr-aws)](https://opensource.org/licenses/Apache-2.0)

## installation
```bash
$ pip install --upgrade pypyraws
```

`pypyraws` depends on the `pypyr` cli & api core. The above `pip` will install 
it for you if you don't have it already.

## in this section
