---
title: pypyrslack.steps.send
linktitle: send
date: 2020-08-20
description: Send messages or notifications to slack.
card_extra_summary:
  heading: input context property
  details: |
            `slackToken` (str)

            `slackChannel` (str)

            `slackText` (str)
categories: [ steps ]
menu:
  docs:
    parent: slack-steps
    name: send
seo_article_headline: Send notification to slack from task runner automation pipeline.
seo_description: Send slack messages or notifications from an automation pipeline, without writing code. 
# social_og_description: 200 chars - if blank fall back to seo_description then description
# social_og_title: Assert dynamic value as expected in pipeline.
# topics: [ ]
---
# pypyrslack.steps.send
Send a message to slack.

## input
Requires the following context items:

- `slackToken`
    - your slack api token. Keep this secure.
- `slackChannel`
    - send to this slack channel (include \# in front)
- `slackText`
    - the body of your message. Use your usual slack formatting chars.

## substitutions
For all inputs you can use substitution tokens, aka string interpolation. 
This substitutes anything between curly braces  with the context value for that 
key. For example, if your context looked like this:

```yaml
arbitraryValue: pypyrchannel
arbitraryText: down the
moreArbText: wild
slackChannel: "#{arbitraryValue}"
slackText: "piping {arbitraryText} valleys {moreArbText}"
```

This will result in sending a message to `#pypyrchannel` with text:

> piping down the values wild

Escape literal curly braces with doubles: `{{` for `{`, `}}` for `}`

Substitutions support much more powerful functionality, which you can find at 
[text {substitution} formatting expressions]({{< ref "/docs/substitutions">}}).

## example
Here is some sample yaml of what a pipeline using the pypyr-slack
plug-in could look like:

```yaml
steps:
  - name: pypyr.steps.env
    comment: slack api token in environment variable
    in:
      env:
        get:
          slackToken: SLACK_TOKEN
  - name: pypyr.steps.set
    in:
      set:
        slackChannel: "#random"
  - name: pypyrslack.steps.send
    in:
      slackText: "pypyr is busy doing things :construction:"

# The slackToken and slackChannel have already been set in steps
# on_success and on_failure are just changing the text for the message.
on_success:
  - name: pypyrslack.steps.send
    in:
      slackText: "that went well! :hotdog:"

on_failure:
  - name: pypyrslack.steps.send
    comment: override the channel to send failures to their
             own special channel.
    in:
      slackChannel: "#failure-notification-channel"
      slackText: "whoops! :rage1:"
```

If you saved this yaml as `./hoping-for-a-hotdog.yaml`, you  can run the 
following:

```bash
$ pypyr hoping-for-a-hotdog
```

See a worked example for sending messages via [pypyr slack](https://github.com/pypyr/pypyr-example/tree/main/pipelines/slack.yaml).