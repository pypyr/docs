---
title: contribute to pypyr
description: pypyr is open-source and your contributions are very welcome!
date: 2020-08-12
publishdate: 2020-08-13
lastmod: 2020-08-16
# categories: [contributing]
menu:
  docs:
    parent: contributing
draft: false
seo_article_headline: How to contribute to pypyr open source software.
seo_description: This contribution guide will help you contribute to open-source software even if you don't code.
# topics: []
---
# contribute to pypyr
pypyr is open source. Your contributions are most welcome.

Feel free to join pypyr on
[discord](https://discordapp.com/invite/8353JkB).

## bugs
Well, you know. No one's perfect. Feel free to [create an
issue](https://github.com/pypyr/pypyr/issues/new).

## contribute to the core cli
The usual jazz - create an issue, fork, code, test, PR. It might be an
idea to discuss your idea on discord or via the Issues list first before
you go off and write a huge amount of code - you never know, something
might already be in the works, or maybe it's not quite right for the
core-cli (you're still welcome to fork and go wild regardless, of
course, it just mightn't get merged back in here).


## roll your own plug-in
You've probably noticed by now that pypyr is built to be pretty
extensible. You've probably also noticed that the core pypyr cli is
deliberately kept light. The core cli is philosophically only a way of
running a sequence of steps. Dependencies to external libraries should
generally get their own package, so end-users can selectively install
what they need rather than have a monolithic batteries-included
application.

If you've got some custom context_parser or steps code that are
useful, create a repo and bask in the glow of sharing with the open
source community. Honor the pypyr Apache license please.

I generally name plug-ins `pypyr-myplugin`, where `myplugin`
is likely some sort of dependency that you don't want in the pypyr core
cli. For example, pypyr-aws contains pypyr steps for the
AWS `boto3` library. This is kept separate so that you don't have to deal
with yet another dependency you don't need if your current project
isn't using AWS.

If you want, for cred on the [official plug-in list]({{< ref "docs/plugins">}}), 
please feel free to get in touch via the Feedback link on this page. Get in 
touch on twitter anyway, would love to hear from you!
