---
title: pypyraws.steps.waitfor
linktitle: waitfor
date: 2020-08-20
description: Wait for any aws client method to complete, even when it doesn't have an official waiter.
card_extra_summary:
  heading: input context property
  details: '`awsWaitFor` (dict)'
categories: [ steps ]
menu:
  docs:
    parent: aws-steps
    name: waitfor
seo_article_headline: Create a custom AWS waiter without writing code.
seo_description: AWS does not have waiters for all state changes. Use this to create your own waiter for any property state change.
# social_og_description: 200 chars - if blank fall back to seo_description then description
# social_og_title: Assert dynamic value as expected in pipeline.
# topics: [ ]
---
# pypyraws.steps.waitfor
## custom waiter for aws state changes
Custom waiter for any aws client operation. Where
[pypyraws.steps.wait]({{< ref "wait">}}) uses the official AWS
waiters from the low-level client api, this step allows you to execute
*any* aws low-level client method and wait for a specified field in the
response to become the value you want it to be.

AWS does not have waiters for all state changes. Use this to create your own 
waiter for any property state change.

This is especially handy for things like Beanstalk, because Elastic
Beanstalk does not have Waiters for environment creation.

## input
The input context looks like this:

```yaml
awsWaitFor:
  awsClientIn: # required. awsClientIn allows the same arguments as pypyraws.steps.client.
    serviceName: elasticbeanstalk
    methodName: describe_environments
    methodArgs:
        ApplicationName: my wonderful beanstalk default application
        EnvironmentNames:
          - my-wonderful-environment
        VersionLabel: v0.1
  waitForField: '{Environments[0][Status]}' # required. format expression for field name to check in awsClient response
  toBe: Ready # required. Stop waiting when waitForField equals this value
  pollInterval: 30 # optional. Seconds to wait between polling attempts. Defaults to 30 if not specified.
  maxAttempts: 10 # optional. Defaults to 10 if not specified.
  errorOnWaitTimeout: True # optional. Defaults to True if not specified. Stop processing if maxAttempts exhausted without reaching toBe value.
```

See [pypyraws.steps.client]({{< ref "client">}}) for a full listing of 
available arguments under `awsClientIn`.

If `errorOnWaitTimeout` is True and `max_attempts` exhaust before
reaching the desired target state, pypyr will stop processing with a
`pypyraws.errors.WaitTimeOut` error.

The `awsWaitFor` context supports [text {substitution} formatting expressions]({{< ref "/docs/substitutions">}}).

{{% note tip %}}
Do note that while `waitForField` uses substitution style format
strings, the substitutions are made against the response object that
returns from the aws client call specified in `awsClientIn`, and not
from the pypyr context itself.
{{% /note %}}

## output
Once this step completes it adds `awsWaitForTimedOut` to the pypyr
context. This is a boolean value with values:

  
| awsWaitForTimedOut | Description |
|--------------------|-------------|
| True | `errorOnWaitTimeout=False` and `max_attempts` exhausted without reaching `toBe`. |
| False | `waitForField`\'s value becomes `toBe` within `max_attempts`.


## example
See a worked example for an [elastic beanstalk custom waiter for
environmment creation
here](https://github.com/pypyr/pypyr-example/blob/main/pipelines/aws-beanstalk-waitfor.yaml).
