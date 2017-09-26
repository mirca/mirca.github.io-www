---
layout: post
title: "The Likelihood Function for Independent Non-identically Distributed Poisson Measurements"
excerpt: ""
modified: 2017-08-06
tags: [gsoc, astropy, openastronomy]
comments: true
image:
  feature: gsoc_post.jpg
  credit: Hubble Space Telescope
---

Assume you perform an experiment that returns $n$ Poisson distributed random variables $$Y^{n}
\triangleq {Y_i}_{i=1}^{n}$$ each of which has mean $$\lambda_i(\theta)$$, where
\theta is a vector of parameters which belongs to some [compact space](https://en.wikipedia.org/wiki/Compact_space)
$$\Theta$$ called parameter space.
In this setting, $$\lambda_i$$, $$i=1, 2, ..., n$$ are called parametric models.

The likelihood function is can be expressed as:

\begin{align}
P_{\theta}(Y^{n} = y^{n}) = \prod_{i=1}^{n} p_{\theta}(y_i) \\
P_{\theta}(Y^{n} = y^{n}) = \prod_{i=1}^{n} \exp{-\lambda_i(\theta)}\dfrac{\lambda_i^{y_i}(\theta)}{y_i!} \\
P_{\theta}(Y^{n} = y^{n}) = \exp{-\sum_{i=1}^{n}\lambda_i(\theta)}\prod_{i=1}^{n}\dfrac{\lambda_i^{y_i}(\theta)}{y_i!}
\end{align}

Taking the Naperian logarithm of $$P_{\theta}(Y^{n} = y^{n})$$, it follows that:

\begin{align}
\log P_{\theta}(Y^{n} = y^{n}) = - \sum_{i=1}^{n}\lambda_i(\theta) + \sum_{i=1}^{n}y_i\lambda_i(\theta) - \sum_{i=1}^{n}\log y_i !
\end{align}
