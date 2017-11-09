---
layout: post
title: "Sum of Poisson and Gaussian Random Variables"
excerpt: "Dr. G's Conjecture part II"
modified: 2017-09-08
tags: [gsoc, astropy, openastronomy]
comments: true
image:
  feature: gsoc_post.jpg
  credit: Hubble Space Telescope
---

In this short blog post I am going to derive the probability density function of the sum between Poisson and Gaussian random variables.

This assumption appears in many practical scenarios, specially in imaging in which a photon noise component (usually Poisson distributed)
gets combined with a thermal noise component (usually assumed to be Gaussian distributed).

Consider an experiment that outputs $$Z_i = Y_i + X_i,~i=1, 2, ..., n$$. Assume that $$Y^{n}
\triangleq \{Y_i\}_{i=1}^{n}$$ is a sequence of independent but **not** necessarily identically distributed Poisson random variables,
each of which has mean $$\lambda_i$$. Assume further that $$X^{n}
\triangleq \{X_i\}_{i=1}^{n}$$ is a sequence of iid Gaussian random variables with zero mean and variance $$\sigma^2$$, $$\sigma^2 > 0$$.

The first step into deriving the likelihood function of $$Z^{n}$$ is to get the pdf of every $$Z_i$$. Since $$Z_i$$ is the sum
of a Poisson random variable and a Gaussian random variable, we can go ahead and perform the convolution between their pdfs in
order to get the pdf of $$Z_i$$. However, let's try a different approach.

Note that, conditonal on $$ Y_i = y_i$$, $$Z_i$$ follows a Gaussian distribution with mean $$y_i$$ and variance
$$ \sigma^2 $$, i.e.

$$
\begin{align}
p(z_i | y_i) = \dfrac{\exp\left\{-\dfrac{(z_i - y_i)^2}{2\sigma^2}\right\}}{\sqrt{2\pi\sigma^2}}
\end{align}
$$

Now, we can use the Law of Total Probability to derive $$p(z_i)$$ as follows

$$
\begin{align}
p(z_i) &= \mathbb{E}(p(z_i | Y_i)) = \sum_{y_i=0}^{\infty}p(z_i | y_i)p(y_i)\\
p(z_i) &= \sum_{y_i=0}^{\infty} \dfrac{\exp\left\{-\dfrac{(z_i - y_i)^2}{2\sigma^2}\right\}}{\sqrt{2\pi\sigma^2}} \dfrac{e^{-\lambda_i}\lambda_i^{y_i}}{y_i!}
\end{align}
$$

Using the fact that $$Z_i,~i=1, 2, ..., n$$ are independent random variables, the pdf of $$Z^n$$ follows as

$$
\begin{align}
p(z^n) = \prod_{i=1}^{n} p(z_i) = \prod_{i=1}^{n} \sum_{y_i=0}^{\infty} \dfrac{\exp\left\{-\dfrac{(z_i - y_i)^2}{2\sigma^2}\right\}}{\sqrt{2\pi\sigma^2}}\dfrac{e^{-\lambda_i}\lambda_i^{y_i}}{y_i!}
\end{align}
$$

and the log-likelihood can be written as

$$
\begin{align}
\log p(z^n) = \sum_{i=1}^{n} \log\left\{\sum_{y_i=0}^{\infty}\dfrac{\exp\left\{-\dfrac{(z_i - y_i)^2}{2\sigma^2} - \lambda_i\right\}}{\sqrt{2\pi\sigma^2}}\dfrac{\lambda_i^{y_i}}{y_i!}\right\}
\end{align}
$$

$$
\begin{align}
\log p(z^n) &\propto -n\log\sigma + \sum_{i=1}^{n} \log\left\{\sum_{y_i=0}^{\infty}\exp\left(-\dfrac{(z_i - y_i)^2}{2\sigma^2} - \lambda_i + y_i(\log\lambda_i - \log y_i!)\right)\right\}\\
& = -n\log\sigma + \sum_{i=1}^{n} \log\left\{\sum_{y_i=0}^{\infty}\exp\left(-\dfrac{(z_i - y_i)^2}{2\sigma^2} - \lambda_i + y_i\left(\log\lambda_i - \sum_{\gamma=1}^{y_i}\log \gamma\right)\right)\right\}
\end{align}
$$


Any suggestions on how to make this likelihood computationally tractable? Maybe via approximation theory?
