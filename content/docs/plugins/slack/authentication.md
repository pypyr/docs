---
title: authentication
linktitle: authentication
date: 2020-08-19
description: How to authenticate with slack.
# categories: [ ]
menu:
  docs:
    parent: slack
    identifier: slack-authentication
    name: authentication
    weight: 10
seo_article_headline: Authenticate slack credentials from task-runner pipeline.
seo_description: How to configure your credentials & authenticate your automation pipeline against slack.
# social_og_description: 200 chars - if blank fall back to seo_description then description
# social_og_title: Assert dynamic value as expected in pipeline.
# topics: [ ]
---
# slack authentication
## Get slack api token
To authenticate against your slack, you need to create an api key.
There're various ways of going about this, using legacy tokens, test
tokens or a bot.

I generally [create a bot](https://my.slack.com/services/new/bot). Given
you're likely to use it just to send notifications to slack, rather
than consume events from slack, it's a pretty simple setup just to get
your api key.

Remember to invite and add the bot you create to the slack channel(s) to
which you want to post. You invite the bot in like you would a normal
user.

## ensure secrets stay secret
Be safe! Don't hard-code your api token, don't check it into a public
repo. Here are some tips for handling api tokens from
[slack](http://slackapi.github.io/python-slackclient/auth.html#handling-tokens).

Do remember not to fling the api key around as a shell argument - it
could very easily leak that way into logs or expose via a `ps`. I
generally use one of the pypyr built-in context parsers like
`pypyr.parser.jsonfile` or `pypyr.parser.yamlfile`, see [pypyr built-in
context parsers]({{< ref "/docs/context-parsers" >}}).