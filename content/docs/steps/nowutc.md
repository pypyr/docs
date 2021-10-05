---
title: pypyr.steps.nowutc
linktitle: nowutc
date: 2020-07-07T10:19:23+01:00
description: Saves current utc date-time to context `nowUtc`.
card_extra_summary:
  heading: input context property
  details: "`nowUtcIn` (str)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: nowutc
seo_article_headline: Get UTC timestamp in pipeline.
seo_description: Get & format the current timestamp in UTC during task-runner pipeline execution.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: nowutc -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [time]
---
# pypyr.steps.nowutc
## get current utc timestamp
Writes the current UTC date & time to context *nowUtc*.

If you want local or wall time, check out [pypyr.steps.now]({{< ref "now" >}}) 
instead.

If you run this step as a simple step (with no input `nowUtcIn`
formatting), the default datetime format is ISO8601. For example: 
`YYYY-MM-DDTHH:MM:SS.ffffff+00:00`

You can use explicit format strings to control the datetime representation. For 
a full list of available formatting codes, check [python date & time formatting](https://docs.python.org/library/datetime.html#strftime-and-strptime-behavior)

```yaml
- pypyr.steps.nowutc # this sets {nowUtc} to YYYY-MM-DDTHH:MM:SS.ffffff+00:00
- name: pypyr.steps.echo
  in:
    echoMe: 'utc timestamp in ISO8601 format: {nowUtc}'
- name: pypyr.steps.nowutc
  description: use a custom date format string instead of the default ISO8601
  in:
    nowUtcIn: '%A %Y %m/%d %H:%M in timezone %Z offset %z, localized to %x'
- name: pypyr.steps.echo
  in:
    echoMe: 'custom formatting was set in the previous step: {nowUtc}'
```

All inputs support [substitutions]({{< ref "/docs/substitutions">}}).

See a worked [example for pipeline with now utc timestamp](https://github.com/pypyr/pypyr-example/tree/master/pipelines/now.yaml).

## multiple timestamps in same pipeline
If you have a pipeline where you want to refresh the timestamp multiple times, 
while using the same formatting, you can save yourself some typing by using 
[set]({{< ref "set" >}}) to specify the formatting `nowUtcIn` context property.

```yaml
- name: pypyr.steps.set
  comment: set datetime formatting string
           this will endure for the entire pipeline.
  in:
    set:
      nowUtcIn: '%A %Y %m/%d %H:%M in timezone %Z offset %z, localized to %x'
- pypyr.steps.nowutc # uses nowUtcIn formatting from step 1
- name: pypyr.steps.echo
  in:
    echoMe: 'timestamp in custom format: {nowUtc}'
- pypyr.steps.nowutc # still uses nowUtcIn formatting from step 1
- name: pypyr.steps.echo
  in:
    echoMe: 'Using the refreshed timestamp but still same format: {nowUtc}'
```