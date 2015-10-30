---
layout: post
title: "Coding Period: Weeks three and four!"
excerpt: "Viva la subpixel resolucíon"
modified: 2016-06-26
tags: [gsoc, astropy, openastronomy]
comments: true
image:
  feature: gsoc_post.jpg
  credit: Hubble Space Telescope
---

Hi all!

<p style='text-align: justify;'>
This post regards the coding the I have done during the third and fourth
weeks of my awesome GSoC project.
</p>

<p style='text-align: justify;'>
First things first. My mentor Erik Tollerud (aka @eteq) gave me a really nice
opportunity to listen and also contribute with some of the super devs
from Space Telescope Science Institute (STScI). More precisely, I participated
in a code sprint organized by the JWST Data Analysis Development Forum.
</p>

<p style='text-align: justify;'>
Basically, the intention of this sprint was to build a framework for PSF fitting
photometry, which is precisely the subject of my GSoC project.
So, thanks Erik =D.
</p>

<p style='text-align: justify;'>
Also, during those meetings, I could see a future development for my GSoC
project, which is to consider psf photometry in crowded fields from a set
of dithered images instead of just one image of a given field. If time allows,
this would be, maybe, my first "extra" task.
</p>

<p style='text-align: justify;'>
Now, talking about coding, I've submitted quite a few PRs to photutils.
Some of them were just minor organizational things like <a href="https://github.com/astropy/photutils/pull/367">creating a PSF subpackage</a> or <a href="https://github.com/astropy/photutils/pull/379">changing the interface of the findstars module</a>.
</p>

<p style='text-align: justify;'>
Yes, we decided to "change" the interface from a functional programming paradigm to
nicier, more maintanable object-oriented paradigm.
</p>

<p style='text-align: justify;'>
As a first step into that, <a href="https://github.com/astropy/photutils/pull/369">I changed the GROUP functionality</a>, which is now much faster and cleaner thanks to Moritz's reviews xD
</p>

<p style='text-align: justify;'>
I also submitted a <a href="https://github.com/mirca/ze-gsoc16-photutils/pull/3">PR to my own branch</a> as a proposal to an interface of class which will encapsulate the whole
PSF photometry process and it will still provide to users the flexibility to either change or
include their own functionalities into it. For instance, suppose that an user has a better
algorithm to detect stars, then one would just write it as a StarFinder subclass and then pass it to the PSFPhotometry object. Simple, but not simpler :)
</p>

<p style='text-align: justify;'>
Also, thanks to Brigitta, I managed to rebase a branch which adds uncertainty computation to the current sequential psf photometry in photutils.
</p>

<p style='text-align: justify;'>
For now, I'm concentrating my efforts in the GROUP PR and also making NSTAR compatible with
our new interface.  
</p>

<p style='text-align: justify;'>
<i>Now, to work! :)</i>
</p>
