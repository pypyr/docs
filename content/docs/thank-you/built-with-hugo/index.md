---
title: technical documentation built with hugo
linktitle: built with hugo
date: 2020-09-07T12:09:06+01:00
description: Open-source technical documentation & product site template built with Hugo static site generator.
draft: false
lastmod: 2020-09-07T12:09:06+01:00
menu:
  docs:
    parent: thank-you
publishdate: 2020-09-07T12:09:06+01:00
# seo_article_headline: built-with-hugo
# seo_description: fall back to description
---
# technical documentation built with hugo
This website builds on top of the popular open source static site generator 
[hugo](https://gohugo.io).

I was hoping to create a friendly & helpful product site more so than just a 
cut-and-dry API technical information sheet. There are many great no- to 
low-code frameworks for publishing technical documentation out there, but 
generally these are more constrained when you need to step outside of the 
prescribed boundaries of the given layouts. And finding one which looks nice 
and also modern is surprisingly hard (yes, yes, subjective, of course).

So I did something bespoke with Hugo instead. And it was a lot of fun.

## what makes hugo so great?
- Hugo has mastered the trick of being the sort of framework that gets out of 
  your way. When you start noticing your framework, it's generally a bad sign. 
  
  It doesn't force you into prescriptive layouts and output. You control the 
  output html exactly. Hugo just makes it easy to put that html exactly where 
  you want it.

- Layout & partial look-up sequences are very powerful with intuitive defaults, 
  so you can override your generic layouts with more specific renderings where 
  you want. With no extra gunk as side-effect.

- The built-in dev server is a dream productivity booster. Auto 
  delta/differential rebuild & reloads of the site each time you hit 
  Save in your code editor, and those reloads take only ~100ms. Just go 
  `hugo server` and you're off to the races! 
  
  Your code editor probably takes longer to format-on-save than your changes to
  rebuild & reload.

- Actively developed with helpful new features & enhancements that keep 
  backward compatibility. 
  
  It's a good sign when you greet new features & refinements with 
  "oh, that's handy!" rather than "oh, what now?!".

- Hugo is blazingly fast. The sort of fast that you don't notice while you're 
  coding. Which is the sort of fast that matters. A from-scratch build of ~200 
  pages for this site takes less than one second. 

- Hugo is lightweight. It's a single binary, so you don't have to concern 
  yourself with configuring a bunch of plumbing for supporting frameworks to 
  get it to work.

## features
So all of that is great, but what's actually important is what it lets you do. 
When you don't have to worry about cajoling the underlying framework to do what 
you want, you can focus on building great features to make using the site a joy 
(... I hope!):

- Fully responsive down to 320px.
- Different page layouts & behaviors all configurable via front matter.
- Tree menu for documentation section where hierarchy is important for 
  navigation over different pages.
- Designed for dark mode.
- SEO with Structured Data
- Social media sharing with configurable meta tags.
- Lightweight site - no dependency on jquery, bootstrap or any of the heavier
  frameworks and 50 million dependencies for simple functionality that can as 
  easily just live in modern html, css & js. Back to basics FTW!
- Minimalist coding gets to a >=98% score on Lighthouse.
  {{< figure-bundle src="pypyr-lighthouse-score.png" title="pypyr lighthouse" caption="lighthouse score >98%" alt="static site generator like hugo can get you to a lighthouse score >98%" class="paragraph-80-width">}}
- Automated release process with free hosting on [Github Pages](https://pages.github.com/). 
- Intuitive product site taxonomy with different sections for technical 
  documentation, release notes and news. 
- Encourage feedback with prominent bug reporting on any given feature and 
  direct links to edit & improve the documentation.

## tech docs static site template
If you like the look of this site, most of the code is open-source itself and 
in the 
[pypyr-docs technical documentation theme](https://github.com/pypyr/pypyr-docs) 
template.

You can use this template for your own projects via the magic of Hugo's 
excellent theme & templating support.

{{% note warn %}}
The theme is not particularly documented yet. . . watch this space, coming soon!
{{% /note %}}
