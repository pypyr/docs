---
title: slack plugin
linktitle: slack
description: Send messages to Slack.
date: 2020-08-19
menu:
  docs:
    identifier: slack-overview
    name: overview
    parent: slack
    weight: -100
seo_is_carousel: true
seo_article_headline: Send slack notifications from an automation pipeline.
seo_description: Send messages to slack for notifications and status updates when your task-runner pipeline runs.
# topics: []
cascade:
  feedback_bug: https://github.com/pypyr/pypyr-slack/issues/new?title=Re:%%20%s&labels=bug
---
# pypyr slack plug-in
Send messages to [slack](https://slack.com/) from pypyr. This is useful
for sending notifications on success or failure conditions in your
pipelines. Or for sending a message just because you can.

[![build status](https://github.com/pypyr/pypyr-slack/workflows/lint-test-build/badge.svg)](https://github.com/pypyr/pypyr-slack/actions)
[![coverage status](https://codecov.io/gh/pypyr/pypyr-slack/branch/master/graph/badge.svg)](https://codecov.io/gh/pypyr/pypyr-slack)
[![pypi version](https://badge.fury.io/py/pypyrslack.svg)](https://pypi.python.org/pypi/pypyrslack/)
[![apache 2.0 license](https://img.shields.io/github/license/pypyr/pypyr-slack)](https://opensource.org/licenses/Apache-2.0)

## installation
```bash
$ pip install --upgrade pypyrslack
```

pypyrslack depends on the pypyr core and the offical slack Python library. 
The above `pip` will install these for you if you don't have it already.

## in this section