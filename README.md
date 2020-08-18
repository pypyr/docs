# docs
pypyr documentation source. edit documentation here.
=======
# the pypyr project documentation website source
https://pypyr.io/ website source. Edit website content & documentation here in 
this repo.

## how to edit documentation text
All copy is in markdown (.md) files in `./content`. Directory structure is 
important, it follows the urls taxonomy you'll find on _pypyr.io_.

An easy way to find content you want to edit is to browse the website at
[pypyr.io](https://pypyr.io), and clicking the _edit me_ link on the particular
page - this will hyperlink you back into the exact .md page to edit in this 
repo.

## how to edit code
The site plumbing is courtesy of the great static site generator [hugo](https://gohugo.io). 

Code (html, css, js) is either in `./layouts/` or in the site's custom theme. 

Colors and images specific to _pypyr_ are in `./layouts`, but in general most
of the functionality and structural html comes from the theme itself.

### edit the theme
This repo uses Hugo modules to link the [pypyr-docs](https://github.com/pypyr/pypyr-docs) 
custom theme into the current project. Edit the theme in its own repo. 

## building website
`$ pypyr ops/build`

This builds site in the `/public` directory. The `/public` directory is a git 
sub-module pointing to the GitHub Pages repo at 
[pypyr.github.io](https://github.com/pypyr/pypyr.github.io).

This build will wipe the `/public/` directory without destroying the git 
sub-module, allowing for incremental or differential updates (unlike 
`hugo --cleanDestinationDir`). 

## publishing website
`$ pypyr ops/publish update`

This 
- updates the theme (pulling the latest from `pypyr-docs`)
- builds the site using the `ops/build` pipeline
- commits & pushes the changes to both the linked `pypyr.github.io` repo and 
  the current parent repo.

To publish *without* getting the latest theme:
`$ pypyr ops/publish`

To build & commit changes *without* pushing to origin:
`$ pypyr ops/publish update nopush`
or without updating theme:
`$ pypyr ops/publish nopush`