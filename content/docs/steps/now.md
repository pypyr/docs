---
title: pypyr.steps.now
linktitle: now
date: 2020-07-07T10:14:55+01:00
description: Saves current local date-time to context `now`.
card_extra_summary:
  heading: input context property
  details: "`nowIn` (str)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: now
seo_article_headline: Get local timestamp in pipeline.
seo_description: Get & format the current timestamp in local time (aka wall time) during task-runner pipeline execution.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: now -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [time]
---
# pypyr.steps.now
## get current local timestamp
Writes the current local date & time to context `now`. Local time is also known 
as wall time.

If you want UTC time, check out [pypyr.steps.nowutc]({{< ref "nowutc" >}}) 
instead.

If you run this step as a simple step (with no input formatting specified in 
`nowIn`), the default datetime format is ISO8601. For example: 
`YYYY-MM-DDTHH:MM:SS.ffffff+00:00`.

You can use explicit format strings to control the datetime representation. For 
a full list of available formatting codes, check [python date & time formatting](https://docs.python.org/library/datetime.html#strftime-and-strptime-behavior)

```yaml
- pypyr.steps.now # this sets {now} to YYYY-MM-DDTHH:MM:SS.ffffff+00:00
- name: pypyr.steps.echo
  in:
    echoMe: 'timestamp in ISO8601 format: {now}'
- name: pypyr.steps.now
  comment: use a custom date format string instead of the default ISO8601
  in:
    nowIn: '%A %Y %m/%d %H:%M in timezone %Z offset %z, localized to %x'
- name: pypyr.steps.echo
  in:
    echoMe: 'custom formatting for now was set in the previous step. {now}'
```

All inputs support [substitutions]({{< ref "/docs/substitutions">}}).

See a worked [example for pipeline now timestamp](https://github.com/pypyr/pypyr-example/tree/master/pipelines/now.yaml).

## multiple timestamps in same pipeline
If you have a pipeline where you want to refresh the timestamp multiple times, 
while using the same formatting, you can save yourself some typing by using 
[set]({{< ref "set" >}}) to specify the formatting `nowIn` context property.

```yaml
- name: pypyr.steps.set
  comment: set datetime formatting string
           this will endure for the entire pipeline.
  in:
    set:
      nowIn: '%A %Y %m/%d %H:%M in timezone %Z offset %z, localized to %x'
- pypyr.steps.now # uses nowIn formatting from step 1
- name: pypyr.steps.echo
  in:
    echoMe: 'timestamp in custom format: {now}'
- pypyr.steps.now # still uses nowIn formatting from step 1
- name: pypyr.steps.echo
  in:
    echoMe: 'Using the refreshed timestamp but still same format: {now}'
```