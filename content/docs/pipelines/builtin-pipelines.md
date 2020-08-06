---
title: built-in pipelines
description: Ready-made pipelines that come with pypyr out of the box.
date: 2019-08-21
publishdate: 2019-08-21
lastmod: 2019-08-21
categories: [pipelines]
menu:
  docs:
    parent: pipelines
draft: false
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
donothing     | Does what it says. Nothing.| `pypyr donothing`
echo          | Echos context value echoMe to output.| `pypyr echo text goes here`            
pypyrversion  | Prints the python cli version number.| `pypyr pypyrversion`
magritte      | Thoughts about pipes.| `pypyr magritte`
