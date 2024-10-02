---
layout: post
title: Double machine learning for causal inference
tags: causal-inference
date: 2022-06-24
---

Recently I have been working on causal inference problems. Mainly I am trying to understand how many more units of sales Amazon can expect if we are able to offer faster delivery speed. This is a typical causal inference problem, where the treatment would be the faster delivery speed, and the control would be the slower delivery speed. Upon research, I was introduced to this interesting method called "double machine learning". This post will document some of the learnings I have had so far.

## Notation and Setup

Following convention, let's denote the treatment indicator for each unit $i$ as $T_i$. In the case where there is only one possible treatment, we write $T_i = 0$ if unit $i$ receives the control, and $T_i = 1$ if unit $i$ receives the treatment. We follow the potential outcome framework, so let $D_i(0)$ be the potential outcome of unit $i$ under control, and $D_i(1)$ be the potential outcome of unit $i$ under treatment.

Now, the observed outcome $D_i$ is

$$ D_i (T_i) = \mathbf{1}_{\{T_i = 0\}}\cdot D_i(0) + \mathbf{1}_{\{T_i = 1\}}\cdot D_i(1) $$

Obviously, observed outcome depends on the treatment it receives. For a fixed unit $i$, $D_i$ is either $D_i(0)$ or $D_i(1)$.

We can also write the observed outcome as

$$ D_i = D_i(0) + \mathbf{1}_{\{T_i = 1\}} \cdot [D_i(1) - D_i(0)] $$

In other words, we can express the observed outcome in terms of the individual treatment effect (ICE), $D_i(1) - D_i(0)$. This quantity is very difficult to estimate, and might not be needed after all.

## Double machine learning

Let's talk about how double machine learning works now. Again, we have two possible scenarios for each unit: control vs treatment. The treatment assignment of unit $i$ (i.e. whether unit $i$ is under control or not) itself is random, and we denote it as $T_i$. Once the treatment assignment is fixed, the potential outcomes $D_i(0)$ and $D_i(1)$ are also random, and depend on a set of covariates $X_i$. In other words, $D_i(0), D_i(1)$ as well as $T_i$ are all random variables.

Dropping the index $i$, we assume that we can write $D$ in terms of conditional expectations plus a white noise. That is,
$$
    D = \mathbb{E}\left[D(0)\mid X\right] + \mathbf{1}_{\{T=1\}}\cdot \mathbb{E}\left[D(1) - D(0) \mid X\right] + \epsilon
$$

Let's take conditional expectation on both sides. When we condition on $X$, terms like  $\mathbb{E}[\cdot \mid X]$ behave like constants and can be pulled out directly. So

$$ \mathbb{E}[D\mid X]=\mathbb{E}[D(0)\mid X] + \mathbb{E}[\mathbf{1}_{\{T=1\}} \mid X]\cdot  \mathbb{E}\left[D(1) - D(0) \mid X\right] $$

But since it's an indicator function in the expectation, the conditional expectation $ \mathbb{E}[\mathbf{1}\\_{\{T=1\}}] \mid X] $ really is just the conditional _probability_ of receiving treatment instead of control. Let $p_T(t) = \mathbb{P}\left[T=t\right]$. So let's rewrite it as


$$ \mathbb{E}[D\mid X]=\mathbb{E}[D(0)\mid X] +  p_T(1 \mid X)\cdot  \mathbb{E}\left[D(1) - D(0) \mid X\right] $$

Here comes the interesting part. We can group terms by their treatment status. Under the ignorability assumption, we finally arrive at

$$
    \mathbb{E}\left[D\mid X\right] = p_T(0\mid X)\cdot \mathbb{E}\left[D(0)\mid X\right] + p_T(1\mid X)\cdot \mathbb{E}\left[D(1)\mid X\right]
$$

We call this quantity __mean outcome__ and denote it as $m(X)$, and the reason should be clear. The conditional expectation of outcome $\mathbb{E}\left[D\mid X\right]$ can be viewed as an weighted average. We weigh each potential outcome by their propensity score (i.e. probability of receiving the corresponding treatment).

Subtract mean outcome from both sides of equation (2) and recall that $p_T(0) + p_t(1) = 1$, we have
$$
    D - m = \left(\mathbf{1}_{T = 1} - p_T(1)\right) \cdot \mathbb{E}\left[ D(1) - D(0)\mid X\right] + \epsilon
$$

Assuming that $m(\cdot)$ and $p_T(\cdot)$ are known, then there is a natural estimator for $\mathbb{E}\left[D(1) - D(0)\right]$. Namely, we can run a regression of $ D_i-m(X_i) $ with respect to $ \[\mathbf{1}_{\{T_i=1\}} - p_{T_i}(1)\] \phi(X)$ where $\phi(\cdot)$ is some function. For simplicity, we will assume that the lift is linear in covariate, even though this is not required to achieve good asymptotic consistency. That is,
$$
    \mathbb{E}\left[D(1) - D(0)\mid X\right] = X^\top \beta.
$$

In practice, $m(\cdot)$ and $p_T(\cdot)$ have to be estimated. In terms of model class, we have many choices. Neural networks can be used to model the mean outcome $m(\cdot)$, while logistic regression can be used to model propensity score $p_T(\cdot)$.

The final regression will yield a set of coefficients $\hat{\beta}$. In production, we simply need to multiply the covariates with $\hat{\beta}$ to generate lift estimates.
