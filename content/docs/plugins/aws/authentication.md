---
title: authentication
linktitle: authentication
date: 2020-08-19
description: How to authenticate with AWS.
# categories: [ ]
menu:
  docs:
    parent: aws
    identifier: aws-authentication
    name: authentication
    weight: 10
seo_article_headline: Authenticate AWS credentials from task-runner pipeline.
seo_description: How to configure your credentials & authenticate your automation pipeline against AWS.
# social_og_description: 200 chars - if blank fall back to seo_description then description
# social_og_title: Assert dynamic value as expected in pipeline.
# topics: [ ]
---
# aws authentication
## configuring credentials
pypyr-aws pretty much just uses the underlying boto3 authentication
mechanisms. More info here:
<http://boto3.readthedocs.io/en/latest/guide/configuration.html>

This means any of the following will work. The authentication settings lookup 
order is as follows:

### IAM credentials when inside AWS
If you are running inside of AWS - on EC2 or inside an ECS container, it will 
automatically use IAM role credentials if it does not find credentials in any 
of the other places listed below.

### set clientArgs in pypyr context
In the pypyr context

```yaml
awsClientIn:
    clientArgs:
        aws_access_key_id: ACCESS_KEY
        aws_secret_access_key: SECRET_KEY
        aws_session_token: SESSION_TOKEN
```

Remember that you can use 
[text {substitution} formatting expressions]({{< ref "/docs/substitutions">}}) 
to set these values dynamically. 

### environment variables 
You can set the following $ENV variables:

- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY
- AWS_SESSION_TOKEN

### credentials file
-   Credentials file at `~/.aws/credentials` or `~/.aws/config`

    -   If you have the aws-cli installed, run `aws configure` to get
        these configured for you automatically.

{{% note tip %}}
On dev boxes I generally don\'t bother with credentials, because
chances are pretty good that I have the aws-cli installed already
anyway, so pypyr will just re-use the aws shared configuration files
that are there anyway.
{{% /note %}}

## Ensure secrets stay secret
Be safe! Don't hard-code your aws credentials. Don't check credentials into a 
public repo. 😱

{{% note tip %}}
If you're running pypyr inside of aws - e.g in an ec2 instance or
an ecs container that is running under an IAM role, you don't actually
need explicitly to configure credentials for pypyr-aws.
{{% /note %}}

Do remember not to fling your key & secret around as shell arguments -
it could very easily leak that way into logs or expose via a `ps`. I
generally use one of the pypyr built-in context parsers like
`pypyr.parser.jsonfile` or `pypyr.parser.yamlfile`, see [pypyr built-in
context parsers]({{< ref "/docs/context-parsers" >}}).

Do remember also that $ENV variables are not a particularly secure place to 
keep your secrets.