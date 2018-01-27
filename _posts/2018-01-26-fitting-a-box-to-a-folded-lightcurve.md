---
layout: post
title: "Where is my planet?"
excerpt: "Finding periods of transit-like events"
modified: 2018-01-26
tags: [box-least-squares, fitting, lightkurve, exoplanets]
comments: true
image:
  feature: gsoc_post.jpg
  credit: Hubble Space Telescope
---

I'm interested in finding a fast way to compute good enough estimates of periods
of transit-like events caused by planets.

One of the prime works in this direction has been carried out by
<a href="https://arxiv.org/abs/astro-ph/0206099">Kovács, Zucker, and Mazeh</a>.
Their algorithm, the Box-least Squares (or BLS for short), has been very
successful in finding planets (their paper has over 500 citations!).

In this blog post, I follow a similar approach to derive the parameters
of a square function the "best" describe the transiting event.

Assume the following relation for the folded lightcurve $$y \triangleq (y_1, y_2, ..., y_n)$$
\begin{align}
y_i = h - d\cdot \mathbb{I}(t_0 \leq t_i \leq t_0 + w) + \eta_i\nonumber,
\end{align}
where $$\eta_i \sim \mathcal{N}(0, \sigma^2)$$, $$\sigma$$ known, and $$\mathbb{I}$$
is the indicator function. The parameters to be estimated are $$\left\{h, d, t_0, w\right\}$$.

In this setting, $$y_i \sim \mathcal{N}(h - d\cdot \mathbb{I}(t_0 \leq t_i \leq t_0 + w), \sigma^2)$$,
and the joint density of $$y$$ follows as
$$
\begin{align} p(y) = \prod_{i=1}^{n} p(y_i) = \dfrac{1}{(2\pi\sigma^2)^{n/2}} \exp\left\{-\dfrac{1}{2\sigma^2}\left(\sum_{i \in I^c}(y_i - h)^2 + \sum_{i \in I}(y_i - (h - d))^2\right)\right\}
\end{align}
$$
where $$I \triangleq \{i | t_0 \leq t_i \leq t_0 + w\}$$.

The log-likelihood function (up to an additive constant) may be written as
\begin{align}
    \log p(y) = -\dfrac{1}{2\sigma^2}\left(\sum_{i \in I^c}(y_i - h)^2 + \sum_{i \in I}(y_i - (h - d))^2\right)
\end{align}

Therefore, the maximum likelihood estimator may be written as
\begin{align}
    {\theta}^{\star}({y}) &= \arg \min_{h, d, t_0, w} \sum_{i \in I^c}(y_i - h)^2 + \sum_{i \in I}(y_i - (h - d))^2
\end{align}

Note that, $$\log p({y})$$ has continuous partial derivatives with respect to $$h$$ and $$d$$. And
therefore, for fixed values of $$t_0$$ and $$w$$,
\begin{align}
    {\theta}^{\star}({y}) &= \arg \min_{h, d} \sum_{i \in I^c}(y_i - h)^2 + \sum_{i \in I}(y_i - (h - d))^2
\end{align}

Differentiating $$\log p({y})$$ with respect to $$h$$, and solving
$$\dfrac{\partial}{\partial h}\log p({y}) = 0$$, it follows that
\begin{align}
    h^{\star} = \dfrac{1}{n}\left(w\cdot d + \sum_{i=1}^{n}y_i\right).\nonumber
\end{align}

Now, differentiating $$\log p({y})$$ with respect to $$d$$, and solving
$$\dfrac{\partial}{\partial d}\log p({y}) = 0$$, it follows that
\begin{align}
    d^{\star} = \dfrac{\dfrac{1}{n}\displaystyle\sum_{i=1}^{n}y_i - \dfrac{1}{w}\sum_{i \in I}y_i}{1 - \bar{w}}\nonumber,
\end{align}
in which $$\bar{w} \triangleq \dfrac{w}{n}$$.

Hence, one can iterate between numerically optimizing $$\log p({y})$$ for $$t_0$$ and $$w$$
and analytically computing $$h^{\star}$$ and $$d^{\star}$$. On my experiments, I found that two
iterations of this procedure are enough for convergence.

See https://github.com/KeplerGO/lightkurve/pull/4 for a Python implementation of these maths.
