---
title: pypyraws.steps.ecswaitprep
linktitle: ecswaitprep
date: 2020-08-20
description: Run me after an ecs task run or stop to prepare an ecs waiter.
card_extra_summary:
  heading: input context property
  details: |
    `awsClientOut` (dict)

    `awsEcsWaitPrepCluster` (str)
categories: [ steps ]
menu:
  docs:
    parent: aws-steps
    name: ecswaitprep
seo_article_headline: Wait for ECS state changes without writing code.
seo_description: Wait for ECS services or tasks to start or stop without writing code. 
# social_og_description: 200 chars - if blank fall back to seo_description then description
# social_og_title: Assert dynamic value as expected in pipeline.
# topics: [ ]
---
# pypyraws.steps.ecswaitprep
## wait for ecs state changes
Run me after an ecs task run or stop to prepare an ecs waiter.

Prepares the `awsWaitIn` context key for `pypyraws.steps.wait`.

Available ecs waiters are:
- ServicesInactive
- ServicesStable
- TasksRunning
- TasksStopped

Full details here:
<http://boto3.readthedocs.io/en/latest/reference/services/ecs.html#waiters>

Use this step after any of the following ecs client methods if you want
to use one of the ecs waiters to wait for a specific state:

- describe_services
- describe_tasks
- list_services
    - specify awsEcsWaitPrepCluster if you don't want default
- list_tasks
    - specify awsEcsWaitPrepCluster if you don't want default
- run_task
- start_task
- stop_task
- update_service

## input context
You don't have to use this step, you could always just construct the
`awsWaitIn` dictionary in context yourself. It just so happens this step
saves you some legwork to do so.

Required context:

- `awsClientOut`
    - dict. mandatory.
    - This is the context key that any ecs command executed by
      `pypyraws.steps.client` adds.
    - Chances are pretty good you don't want to construct this by hand 
      yourself - the idea is to use the output as generated by one of the 
      supported ecs methods.
- `awsEcsWaitPrepCluster`
    - string. optional.
    - The short name or full arn of the cluster that hosts the task to
        describe. If you do not specify a cluster, the default cluster
        is assumed. For most of the ecs methods the code automatically
        deduces the cluster from `awsClientOut`, so don't worry about it.
    - But, when following list_services and list_tasks, you have to
        specify this parameter.
    - Specifying this parameter will override any automatically deduced cluster 
        .arn

See a worked example for [pypyr aws ecs
here](https://github.com/pypyr/pypyr-example/blob/main/pipelines/aws-ecs.yaml).
