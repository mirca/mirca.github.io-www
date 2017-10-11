---
layout: post
title: "The Likelihood Function for Poisson Measurements"
excerpt: "Dr. G's Conjecture"
modified: 2017-08-06
tags: [gsoc, astropy, openastronomy]
comments: true
image:
  feature: gsoc_post.jpg
  credit: Hubble Space Telescope
---

Assume you perform an experiment that returns $$n$$ independent **not** identically distributed Poisson random variables $$Y^{n}
\triangleq \{Y_i\}_{i=1}^{n}$$ each of which has mean $$\lambda_i(\theta)$$, where
$$\theta$$ is a vector of parameters which belongs to some set $$\Theta \subseteq \mathbb{R}^m$$ called parameter space.
In this setting, $$\{\lambda_i\}_{i=1}^{n}$$ are called parametric models.

The likelihood function can be expressed as

$$
\begin{align}
P_{\theta}(Y^{n} = y^{n}) & = \prod_{i=1}^{n} p_{\theta}(y_i) = \prod_{i=1}^{n} \exp{-\lambda_i(\theta)}\dfrac{\lambda_i^{y_i}(\theta)}{y_i!} \\ P_{\theta}(Y^{n} = y^{n}) & = \exp\left({-\sum_{i=1}^{n}\lambda_i(\theta)}\right)\prod_{i=1}^{n}\dfrac{\lambda_i^{y_i}(\theta)}{y_i!}
\end{align}
$$

Taking the natural logarithm of $$P_{\theta}(Y^{n} = y^{n})$$, it follows that

\begin{align}
\log P_{\theta}(Y^{n} = y^{n}) = \sum_{i=1}^{n}\left(- \lambda_i(\theta) + y_i\log\lambda_i(\theta) - \log y_i !\right)
\end{align}

The Maximum Likelihood Estimator can be stated as the solution of the following optimization problem

\begin{align}
\arg \max_{\theta \in \Theta} \log P_{\theta}(Y^{n}) & = \arg \min_{\theta \in \Theta} - \log P_{\theta}(Y^{n} = y^{n}) \\\\ \arg \max_{\theta \in \Theta} \log P_{\theta}(Y^{n}) & = \arg \min_{\theta \in \Theta} \sum_{i=1}^{n}\left(\lambda_i(\theta) - y_i\log\lambda_i(\theta)\right)
\end{align}

To solve $$\arg \max_{\theta \in \Theta} \log P_{\theta}(Y^{n})$$ is necessary to solve the following set of equations (let's talk about 2nd order derivatives and regularization conditions later)

$$
\begin{align}
\dfrac{\partial}{\partial \theta_j} \log P_{\theta}(Y^{n} = y^{n})\Bigr|_{\substack{\theta=\hat{\theta}}} &= 0 \\ \sum_{i=1}^{n}\left(\dfrac{\partial \lambda_i(\theta)}{\partial \theta_j} - \dfrac{y_i}{\lambda_i(\theta)}\dfrac{\partial \lambda_i(\theta)}{\partial \theta_j}\right)\Bigr|_{\substack{\theta=\hat{\theta}}} &= 0 \\ \sum_{i=1}^{n}\dfrac{\partial \lambda_i(\theta)}{\partial \theta_j}\left(1 - \dfrac{y_i}{\lambda_i(\theta)} \right)\Bigr|_{\substack{\theta=\hat{\theta}}} &= 0 ~~~~~~~~~~~(1)
\end{align}
$$

for $$ j=1, 2, ..., m$$. In other words, we need to find $$\theta^{*} \in \Theta$$ such that the above statements are verified to be true.

We can't get any further unless we make some assumptions about the parametric model $$\lambda_i$$ and the parameter space.

In some problems, such as [point spread function photometry](https://photutils.readthedocs.io/en/stable/photutils/psf.html) and [single molecule localization microscopy](http://q-bio.org/w/images/1/1d/SR_review2.pdf), one is interested in estimating the total light flux emitted by a star (or a molecule) and its subpixel position in a detector. Therefore, our parameter vector can be written as $$\theta = (A, c_o, r_o)$$.

It is also practical to assume that the rate of change of the expected number of counts, in the $$i$$-th pixel, with respect to the total integrated flux is proportional to the expected number of counts. Mathematically,

$$
\dfrac{\partial{\lambda_i(\theta)}}{\partial A} = \alpha \lambda_i(\theta)
$$

for some non-zero constant $$ \alpha $$.

Substituting this assumption in (1), it follows that

$$
\begin{align}
\sum_{i=1}^{n} \alpha \lambda_i({\theta}^{*})\left(1 - \dfrac{y_i}{\lambda_i({\theta}^{*})} \right) & = 0 \\
\sum_{i=1}^{n} \lambda_i({\theta}^{*}) & = \sum_{i=1}^{n} y_i
\end{align}
$$

The mathematical result above tells us that the total flux is exactly estimated by our model $$\lambda_i$$ at the solution $${\theta^{*}}$$.
