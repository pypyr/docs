---
title: pypyraws.steps.s3fetchyaml
linktitle: s3fetchyaml
date: 2020-08-20
description: Fetch a yaml file from s3 into the pypyr context.
card_extra_summary:
  heading: input context property
  details: '`s3Fetch` (dict)'
categories: [ steps ]
menu:
  docs:
    parent: aws-steps
    name: s3fetchyaml
seo_article_headline: Fetch a yaml file from S3 into the task-runner context.
seo_description: Get a YAML file from S3 and parse the YAML without writing code. 
# social_og_description: 200 chars - if blank fall back to seo_description then description
# social_og_title: Assert dynamic value as expected in pipeline.
topics: [ yaml ]
---
# pypyraws.steps.s3fetchyaml
Fetch a yaml file from s3 and put the yaml structure into context.

## input
Required input context is:

```yaml
s3Fetch:
  clientArgs: # optional
    arg1Name: arg1Value
  methodArgs: # mandatory
    Bucket: '{bucket}'
    Key: arb.yaml
  key: 'destination pypyr context key' # optional
```

- `clientArgs` go to the aws s3 client constructor. These are optional.
- `methodArgs` go to the the s3 `get_object` call. The minimum required 
   values are:
    - `Bucket`
    - `Key`
- `key` writes fetched yaml to this context key. If not specified, yaml 
  writes directly to context root.

The `s3Fetch` input context supports [text {substitution} formatting expressions]({{< ref "/docs/substitutions">}}).

## method arguments
The only required inputs for `methodArgs` are `Bucket` and `Key`, but you can 
use any of the following inputs to enable extra functionality like cache, 
version and SSE service side encryption:

```yaml
Bucket='string',
IfMatch='string',
IfModifiedSince=datetime(2015, 1, 1),
IfNoneMatch='string',
IfUnmodifiedSince=datetime(2015, 1, 1),
Key='string',
Range='string',
ResponseCacheControl='string',
ResponseContentDisposition='string',
ResponseContentEncoding='string',
ResponseContentLanguage='string',
ResponseContentType='string',
ResponseExpires=datetime(2015, 1, 1),
VersionId='string',
SSECustomerAlgorithm='string',
SSECustomerKey='string',
RequestPayer='requester',
PartNumber=123
```

Check here for all available arguments with detailed explanations of each: 
<http://boto3.readthedocs.io/en/latest/reference/services/s3.html#S3.Client.get_object>

## output
pypyr will merge yaml parsed from the file into the pypyr context. Use the 
`key` input to control whether you want the resulting structure to merge into 
the root or into its own key. 

This will overwrite existing values if the same keys are already in there.

I.e if the yaml from S3 has

```yaml
eggs: boiled
```

but context `{'eggs': 'fried'}` already exists, resulting `context['eggs']` will be 'boiled'.

If you do not specify `key`, the yaml should not be a list at the top
level, but rather a mapping. So the top-level yaml should not look like
this:

```yaml
- eggs
- ham
```

but rather like this:

```yaml
breakfastOfChampions:
  - eggs
  - ham
```

## example
See a worked example for [pypyr aws s3fetch
here](https://github.com/pypyr/pypyr-example/blob/master/pipelines/aws-s3fetch.yaml).
