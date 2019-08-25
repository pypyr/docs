---
title: substitutions
description: Use string formatting expressions to make substitutions or token replacements in context strings, fields or config. 
date: 2019-08-21
publishdate: 2019-08-21
lastmod: 2019-08-21
draft: false
menu:
  docs:
    identifier: substitutions-overview
    name: overview
    parent: substitutions
    weight: -100
list_fields:
  - fallback: title
    field: linktitle
    heading: title
    islink: true
  - heading: description
    fallback: summary
    field: description
  - heading: example
    field: card_extra_summary.details
list_style: section-list/table
seo_article_headline: Formatting expressions, string interpolation & token replacement.
seo_description: Formatting expressions for type safe assignment, string interpolation & token replacement in a pipeline task-runner.
seo_is_carousel: true
topics: [built-in summary tables]
---
# substitution format expressions
Substitutions, or interpolation, allow you to use formatting expressions to 
manipulate values dynamically at run-time. Use this for type-safe assignment, 
string concatenation, simple string formatting replacement & python expressions 
for more complex logic like `min`, `max`, `abs` or constructing more complex 
conditional boolean evaluation expressions.

You can use any of the following anywhere that supports substitutions.