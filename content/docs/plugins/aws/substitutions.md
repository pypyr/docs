---
title: substitutions
linktitle: substitutions
date: 2020-08-19
description: Use substitutions for string interpolation & dynamically setting values in the AWS client.
# categories: [ ]
menu:
  docs:
    parent: aws
    identifier: aws-substitutions
    name: substitutions
    weight: 10
seo_article_headline: Set AWS client inputs using interpolation formatting expressions.
seo_description: Dynamically set AWS client arguments without writing code for manual string parsing. 
# social_og_description: 200 chars - if blank fall back to seo_description then description
# social_og_title: Assert dynamic value as expected in pipeline.
# topics: [ ]
---
# substitutions
You can use substitution tokens, aka string interpolation, where
specified for context items. This substitutes anything between `{curly
braces}` with the context value for that key. 

This also works where you have dictionaries/lists inside dictionaries/lists. 
For example, if your context looked like this:

```yaml
bucketValue: the.bucket
keyValue: dont.kick
moreArbText: wild
awsClientIn:
  serviceName: s3
  methodName: get_object
  methodArgs:
    Bucket: '{bucketValue}'
    Key: '{keyValue}'
```

This will run s3 `get_object` to retrieve file `dont.kick` from `the.bucket`.

- `Bucket: '{bucketValue}'` becomes `Bucket: the.bucket`
- `Key: '{keyValue}'` becomes `Key: dont.kick`

In json & yaml, curlies need to be inside quotes to make sure they parse
as strings.

Escape literal curly braces with doubles: `{{` for `{`, `}}` for `}`

Substitutions support much more powerful functionality, which you can find at 
[text {substitution} formatting expressions]({{< ref "/docs/substitutions">}}).

