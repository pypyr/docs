---
title: plugins overview
linktitle: plugins
description: A listing of pypyr's official plug-ins. Add AWS & Slack functions to your pipelines.
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
menu:
  docs:
    identifier: plugins-overview
    name: overview
    parent: plugins
    weight: -100
list_fields:
  - fallback: title
    field: linktitle
    heading: title
    islink: true
  - heading: description
    fallback: summary
    field: description
  - heading: input context
    field: card_extra_summary.details
list_style: section-list/table
seo_is_carousel: true
seo_article_headline: Extend pypyr task-runner functionality with plug-ins.
seo_description: Plug-ins provide extra functionality for external libraries & dependencies outside of the pypyr core.
topics: [built-in summary tables]
---
# plugins
pypyr core stays deliberately light so the dependencies are down to the 
minimum. I loathe installs where there're a raft of extra deps
that I don't use clogging up the system.

When and if you need other libraries, you can selectively choose to add this 
functionality by installing a pypyr plugin.


| boss pypyr plug-in | description |
---------------------|-------------|
[pypyr-aws](https://github.com/pypyr/pypyr-aws/) | Interact with the AWS sdk api. Supports all AWS Client functions, such as S3, EC2, ECS & co. via the AWS low-level Client API. |
|[pypyr-slack](https://github.com/pypyr/pypyr-slack/) | Send messages to Slack |