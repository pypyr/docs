---
title: pypyraws.steps.s3fetchjson
linktitle: s3fetchjson
date: 2020-08-20
description: Fetch a json file from s3 into the pypyr context.
card_extra_summary:
  heading: input context property
  details: '`s3Fetch` (dict)'
categories: [ steps ]
menu:
  docs:
    parent: aws-steps
    name: s3fetchjson
seo_article_headline: Fetch a json file from S3 into the task-runner context.
seo_description: Get a JSON file from S3 and parse the JSON without writing code. 
# social_og_description: 200 chars - if blank fall back to seo_description then description
# social_og_title: Assert dynamic value as expected in pipeline.
topics: [ json ]
---
# pypyraws.steps.s3fetchjson
Fetch a json file from s3 and put the json values into context.

## input
Required input context is:

```yaml
s3Fetch:
  clientArgs: # optional
    arg1Name: arg1Value
  methodArgs: # mandatory
    Bucket: '{bucket}'
    Key: arb.json
  outKey: 'destination pypyr context key' # optional
```

- `clientArgs` go to the aws s3 client constructor. These are optional.
- `methodArgs` go to the the s3 `get_object` call. The minimum required 
   values are:
    - `Bucket`
    - `Key`
- `outKey` writes fetched json to this context key. If not specified, json 
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
pypyr will merge json parsed from the file into the pypyr context. Use the `outKey` input to control whether you want the resulting structure to merge into the root or into its own key.

This will overwrite existing values if the same keys are already in there.

I.e if file json has `{'eggs' : 'boiled'}`, but context
`{'eggs': 'fried'}` already exists, returned `context['eggs']` will be the 
updated value 'boiled'.

If you do not specify `outKey`, the json should not be an Array `[]` at the 
root level, but rather an Object `{}`.

## example
See a worked example for 
[pypyr aws s3fetch](https://github.com/pypyr/pypyr-example/blob/master/pipelines/aws-s3fetch.yaml).