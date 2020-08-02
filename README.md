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
This repo uses a git sub-module to link the [pypyr-docs](https://github.com/pypyr/pypyr-docs) 
custom theme into the local directory `./themes/pypyr-docs`. 

Colors and images specific to _pypyr_ are in `./layouts`, but in general most
of the functionality and structural html comes from the theme itself.

### edit the theme
You can deal with source control for editing the theme in two ways:
1. You can absolutely edit the theme itself from this repo, but keep in mind when 
you do anything in `./themes/pypyr-docs/**/*` you have to deal with git 
sub-module functionality.
2. Alternatively, head over to [pypyr-docs](https://github.com/pypyr/pypyr-docs).
Use git like you normally do.


## building website


## publishing website
