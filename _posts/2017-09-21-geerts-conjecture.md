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

Assume you perform an experiment that returns $n$ Poisson distributed random variables $$Y^{n}
\triangleq \{Y_i\}_{i=1}^{n}$$ each of which has mean $$\lambda_i(\theta)$$, where
$$\theta$$ is a vector of parameters which belongs to some $$\Theta$$ called parameter space.
In this setting, $$\{\lambda_i\}_{i=1}^{n}$$ are called parametric models.

The likelihood function can be expressed as:

$$
\begin{align}
P_{\theta}(Y^{n} = y^{n}) & = \prod_{i=1}^{n} p_{\theta}(y_i) = \prod_{i=1}^{n} \exp{-\lambda_i(\theta)}\dfrac{\lambda_i^{y_i}(\theta)}{y_i!} \\ P_{\theta}(Y^{n} = y^{n}) & = \exp\left({-\sum_{i=1}^{n}\lambda_i(\theta)}\right)\prod_{i=1}^{n}\dfrac{\lambda_i^{y_i}(\theta)}{y_i!}
\end{align}
$$

Taking the Naperian logarithm of $$P_{\theta}(Y^{n} = y^{n})$$, it follows that:

\begin{align}
\log P_{\theta}(Y^{n} = y^{n}) = \sum_{i=1}^{n}\left(- \lambda_i(\theta) + y_i\log\lambda_i(\theta) - \log y_i !\right)
\end{align}

The Maximum Likelihood Estimator is the solution of the following optimization problem:

\begin{align}
\arg \max_{\theta \in \Theta} \log P_{\theta}(Y^{n}) & = \arg \min_{\theta \in \Theta} - \log P_{\theta}(Y^{n} = y^{n}) \\\\ \arg \max_{\theta \in \Theta} \log P_{\theta}(Y^{n}) & = \arg \min_{\theta \in \Theta} \sum_{i=1}^{n}\left(\lambda_i(\theta) - y_i\log\lambda_i(\theta)\right)
\end{align}

To solve $$ \arg \min_{\theta \in \Theta} - \log P_{\theta}(Y^{n} = y^{n}) $$ is necessary to solve the following set of equations

\begin{align}
\dfrac{\partial}{\partial \theta_j} \log P_{\theta}(Y^{n} = y^{n}) = 0, j=1, 2, ..., m \\\\
\sum_{i=1}^{n}\left(\dfrac{\partial}{\partial \theta_j}\lambda_i(\theta) - \dfrac{y_i}{\lambda_i(\theta)}\dfrac{\partial}{\partial \theta_j}\lambda_i(\theta)\right) = 0, j=1, 2, ..., m

\end{align}
