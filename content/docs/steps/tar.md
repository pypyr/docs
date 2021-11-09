---
title: pypyr.steps.tar
linktitle: tar
date: 2020-07-07T17:42:21+01:00
description: Archive and/or extract tars with/without compression. Supports gzip, bzip2 & lzma.
card_extra_summary:
  heading: input context property
  details: "`tar` (dict)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: tar
seo_article_headline: Archive, extract & compress tar files in pypyr task-runner.
seo_description: Archive, extract & compress tar files as a step in pypyr task-runner. Supports gzip, bzip2 & lzma.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: tar -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
# topics: [topic1, topic2, topic3]
---
# pypyr.steps.tar
## archive & extract tar files with compression
Archive and extract tars with or without compression.

```yaml
- name: pypyr.steps.tar
  comment: extract & archive tar
  in:
    tar:
        extract:
            - in: /path/my.tar
              out: /out/path
        archive:
            - in: /dir/to/archive
              out: /out/destination.tar
        format: '' # optional. '' | gz | bz2 | xz
```

Either `extract` or `archive` should exist, or both. But not neither.

Optionally, you can also specify the tar compression format with
`format`. Available options for `format`:

-   `''` - no compression
-   `gz` - (gzip)
-   `bz2` - (bzip2)
-   `xz` - (lzma)

If you do not specify format, it defaults to `xz` (lzma). 

This step will run whatever combination of Extract and Archive you
specify. Regardless of combination, execution order is Extract, then
Archive. So you could archive select files that you just extracted in the same
step.

You can use [substitutions]({{< ref "/docs/substitutions">}}) to format all
the inputs from context. This allows you to set `in` and `out` paths 
dynamically at run-time.

{{% note warn %}}
Never extract archives from untrusted sources without prior inspection.
It is possible for extraction to create files outside of your `out` path, e.g. 
members that have absolute filenames starting with `/` or filenames with two 
dots `..`
{{% /note %}}

See a worked [example for tar](https://github.com/pypyr/pypyr-example/tree/main/pipelines/tar.yaml).

## tar extract
`tar['extract']` must exist. It's a list of dictionaries.

```yaml
key1: here
key2: tar.xz
tar:
  extract:
    - in: path/to/my.tar.xz
      out: /path/extract/{key1}
    - in: another/{key2}
      out: .
```

This will:
- Extract `path/to/my.tar.xz` to `/path/extract/here`
- Extract `another/tar.xz` to the current execution directory
  - This is the directory you're running pypyr from, not the pypyr pipeline 
    working directory you set with the `--dir` flag.

## tar archive
`tar['archive']` must exist. It's a list of dictionaries.

```yaml
key1: destination.tar.xz
key2: value2
tar:
  archive:
    - in: path/{key2}/dir
      out: path/to/{key1}
    - in: another/my.file
      out: ./my.tar.xz
```

This will:
- Archive directory `path/value2/dir` to `path/to/destination.tar.xz`,
- Archive file `another/my.file` to `./my.tar.xz`

