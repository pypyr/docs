---
title: pypyr.steps.pyimport
linktitle: pyimport
date: 2020-11-25T18:20:22Z
description: Use external libraries in your !py expressions.
draft: false
card_extra_summary:
  heading: input context property
  details: "`pyImport` (str)"
categories: [steps]
# keywords: ""
menu:
  docs:
    parent: steps
    name: pyimport
seo_article_headline: Import standard or external libraries in your pipeline.
seo_description: Use any importable python code in your task-runner pipeline expressions.
# social_og_description: 200 chars, if blank fall back to seo_description then description
# social_og_title: pyimport -- if blank fall back to seo_article_headline > .Title. Max 70 chars
# social_og_image_alt: max 420 chars
topics: [inline code]
---
# pypyr.steps.pyimport
## import references for py strings
Import module & object references to the 
[!py string]({{< ref "/docs/substitutions/py-strings" >}}) namespace.

This allows you to use any importable Python code in your !py strings.

```yaml
- name: pypyr.steps.pyimport
  comment: any subsequent !py strings can use these objects
  in:
    pyImport: |
      import itertools as itools
      import math
      import urllib.parse
      
      from pathlib import Path
      from fractions import Fraction as myfraction
      
- name: pypyr.steps.set
  comment: use your pyimports anywhere you can use a formatting expression.
           here using it to eval a bool for the "run" condition.
           and also in the step arguments.
  run: !py math.sqrt(1764) == 42
  in:
    set:
      my_number: 1764
      my_url: http://arbhost/blah
      my_filename: file

      math_result: !py math.sqrt(my_number)
      url_host: !py urllib.parse.urlparse(my_url).netloc
      path_out: !py Path('dir', 'subdir', my_filename).with_suffix('.ext')
      fraction_denominator: !py myfraction(1, 3).denominator
```

This pipeline will result in output context like this:
```py
{
  'fraction_denominator': 3,
  'math_result': 42.0,
  'my_filename': 'file',
  'my_number': 1764,
  'my_url': 'http://arbhost/blah',
  'path_out': PosixPath('dir/subdir/file.ext'),
  'url_host': 'arbhost'
}
```

Only use absolute imports, not relative.

When you call `pyimport` multiple times in a pipeline, it will merge the new
imports into the existing set. This means you can overwrite specific objects
from a previous `pyimport` if you use the same names or aliases on subsequent
imports.

## import syntax
You can use all the usual absolute Python import syntax variations:

```python
import x
import x as y

import x.y
import x.y as z

from x import y
from x import y as z

from x import y, z
from a.b import c as d, e as f
```

## import search path
Imports resolve in your current python environment. pypyr also looks in the
current working directory, so you can import & re-use modules in your current 
path without having to package & publish the code to the current environment 
first.

Assume you have a python file in your working directory, saved in a `mydir`
subdirectory like this:
```python
# ./mydir/mymodule.py

def arb_function(arb_arg):
    return f'arb_function says: {arb_arg}'
```

Because pypyr will also look in your working directory for modules,
`mydir.mymodule` will resolve on `import` without you have to do anything
special. 

So you can use this `./mydir/mymodule.py` file from your pipeline like this:
```yaml
- name: pypyr.steps.pyimport
  comment: import custom modules relative to your working dir
            this will look for arb_function in ./mydir/mymodule.py
  in:
    pyImport: from mydir.mymodule import arb_function

- name: pypyr.steps.echo
  comment: you can use arb_function anywhere you can use a formatting expression
  in:
    echoMe: !py arb_function('input from pipeline here')
```

## lifetime of imported references
Imports live on the context object. If you pass the same context to a child
pipeline, the child pipeline will have access to the parents' imports.

The references you import are valid for all subsequent steps (and pipelines)
using the same Context instance. This means that the imported objects are
available until the pipeline ends, unless you do any of the following:
- run another pipeline with a fresh new context using 
[pypyr.steps.pype]({{< ref "pype" >}}).
  - The child pipeline will not have the parent's imports available if 
  `useParentContext` is `False`.
  - The imports remain valid in the parent pipeline after the child completes.
- Explicitly clear the context with [contextclearall]({{< ref "contextclearall" >}}).
  - Subsequent steps will not have the previous imports anymore.