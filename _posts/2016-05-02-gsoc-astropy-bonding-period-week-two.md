---
layout: post
title: "Community Bonding Period: Week Two"
excerpt: "Week one of the community bonding period is over!"
modified: 2016-05-04
tags: [gsoc, astropy, openastronomy]
comments: true
image:
  feature: gsoc_post.jpg
  credit: Hubble Space Telescope
---
<p style='text-align: justify;'>
Week one of the community bonding period is over!
</p>

<p style='text-align: justify;'>
So far, I mostly read photutils source code and I would say I am getting very well acquainted with it, mainly with the PSF photometry code. 
</p>

<p style='text-align: justify;'>
Of course, the most interesting thing last week was my very first meeting with mentors via Skype. They are amazing! I'm sure I'm going to have a very fun summer this year :)
</p>

<p style='text-align: justify;'>
So, in that meeting, we did discuss my first tasks for the project, and they are be the following:
<ol>
<li> Write a consistent API proposal for an interface that performs point spread function photometry in crowded fields</li>
<li> Provide uncertainties on fitted parameters (Astropy issue #1827)</li>
</ol>
</p>

<p style='text-align: justify;'>
For the first task, my goal should be to mimic DAOPHOT. DAOPHOT was chosen primarily because it is the standard software package to perform stellar photometry in crowded fields, and because it works :). 
</p>

<p style='text-align: justify;'>
In the first versions of DAOPHOT one would basically iterate over FIND-GROUP-NSTAR-SUBSTAR-FIND-GROUP... until no more sources are found. FIND is a routine that basically searches for potential stars in a given frame. This routine is already implemented in photutils, namely, <b>daofind</b>. It returns, amongst other things, an estimate of the centroid coordinates and the brightness (which may be further passed to the psf photometry function). GROUP is used to group (:o) stars which are "close enough" from each other. For instance, assume that FIND has found N stars in a frame, then GROUP takes those initial estimates from FIND and computes the distances from each star to every other one, and whenever one star is "close enough" to another they are assigned to the same group. If, however, one star is not "close enough" to any other, then the star in question is fitted by a single model. After grouping, the NSTAR routine is used to simultaneously fit the stars in every group. Further, the SUBSTAR routine is applied to subtracted all fitted stars from the given frame and FIND is again applied to search for any remaining star. This process is then iterated until no more stars are found. Quite simple, but not simpler, isn't it? :)
</p>

<p style='text-align: justify;'>
The second task is crucial and I have some insigths. Although in a stellar field image the photon counts in each pixel follow basically a Poisson distribution, the photon counts in each pixel can be quite well approximated by a Gaussian distribution in the limit of a large number of photon counts (see Central Limit Theorem or <a href="http://etc.cfht.hawaii.edu/PSF-photometry/PSF-photometry.html">here</a>), so that one can obtain quite satisfactory results using least squares.
</p>

<p style='text-align: justify;'>
The least squares fitter is a particular result of the Maximum Likelihood estimator in case that the observations are Gaussian distributed. Hence, one way to compute the uncertainties on each of the fitted parameters is to analitically construct the Fisher Information Matrix (see, e.g., <a href="http://etc.cfht.hawaii.edu/PSF-photometry/PSF-photometry.html">here</a>), which depends only on the derivatives of PSF model with respect to each of the fitted parameters. While this is a good solution, it may be not the more convinient.
</p>

<p style='text-align: justify;'>
The other way is to take for free an estimate of the inverse of the Hessian matrix that is returned by some optimization routines like scipy.optimize.minimize and scipy.optimize.curve_fit.
</p>

<p style='text-align: justify;'>
It seems rather obvious that the latter approach is the easier to go, however I'm not sure how actually stable it is. See <a href="http://stackoverflow.com/questions/29443318/scipy-optimize-minimize-hess-inv-strongly-depends-on-initial-guess">here</a>, for instance.
</p>

<p style='text-align: justify;'>
For this second week of the community bonding period I'm going to sketch an implementation for the GROUP and NSTAR routines and play with them a bit to have some more insights.
</p>

<p style='text-align: justify;'>
Other thing to do is to compare the uncertainties obtained experimentally with those provided by scipy.optimize.minimize or scipy.optimize.curve_fit. This may be achieved by generating an image with, let's say, 1000 isolated stars, and comparing the mean of the uncertainties provided by scipy.optimize.minimize with the mean of the squared errors between the estimated centroids and the actual known values.
</p>

<p style='text-align: justify;'>
<i>Now, to work! :)</i>
</p>

