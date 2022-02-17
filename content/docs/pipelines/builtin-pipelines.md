---
title: built-in pipelines
description: Ready-made pipelines that come with pypyr out of the box.
date: 2020-08-12
publishdate: 2020-08-13
categories: [pipelines]
menu:
  docs:
    parent: pipelines
seo_article_headline: pypyr comes with ready-made pipelines out of the box.
seo_description: pypyr comes with some basic pipelines to run easy smoke test & install verification functionality.
topics: [built-in summary tables]
---
# Built-in pipelines
pypyr comes with with some basic pipelines out of the box. These don't do much, 
nor can they: the whole idea is for you to write your own awesome pipelines 
because it so so easy!

pipeline  | description     | how to run
--------------|---------------------|---------------
config-show   | Display current config settings & sources. | `pypyr config-show`
donothing     | Does what it says. Nothing.| `pypyr donothing`
echo          | Echos input args to output.| `pypyr echo text goes here`            
pypyrversion  | Prints the python cli version number.| `pypyr pypyrversion`
magritte      | Thoughts about pipes.| `pypyr magritte`
