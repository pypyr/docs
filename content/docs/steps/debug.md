---
title: pypyr.steps.debug
linktitle: debug
date: 2020-07-01T12:03:17+01:00
description: Pretty print context to output.
card_extra_summary:
  heading: input context property
  details: "`debug` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: debug
seo_article_headline: Debug task-runner pipelines by printing context values. 
seo_description: Pretty print nested hierarchy to output to assist with troubleshooting and debugging.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: debug -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [debug, print]
---
# pypyr.steps.debug
## debug the context
Pretty print the context to output.

Print the pypyr context to the pypyr output. This is likely to be the
console. This may assist in debugging when trying to see what values are
what.

debug prints to the INFO (20) log-level. This means you won't see debug
output unless you specify `pypyr mypype --log 20` or lower value for `log`. If
you have values you that _always_ want to print to output, 
[echo]({{< ref "echo" >}}) is the more natural step to use.

Obviously, be aware that if you have sensitive values like passwords in
your context you probably want to be careful about this. No duh.

All inputs are optional. This means you can run debug in a pipeline as a
simple step just with:

```yaml
steps:
  - name: my.arb.step
    in:
      arb: arb1
  - pypyr.steps.debug # use debug as a simple step, with no config
  - name: another.arb.step
    in:
      another: value
```

In this case it will dump the entire context as is without applying
formatting. 

## optional inputs
Debug supports the following optional inputs:

```yaml
- name: pypyr.steps.debug
  in:
    debug: # optional
      keys: keytodump # optional. str for a single key name to print.
                      # or a list of key names to print ['key1', 'key2'].
                      # if not specified, print entire context.
      format: False # optional. Boolean, defaults False.
                    # Applies formatting expressions to output.
```

See some worked examples to [use debug to pretty print context](https://github.com/pypyr/pypyr-example/blob/main/pipelines/debug.yaml).