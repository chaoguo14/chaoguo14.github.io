---
layout: post
title: Logistic regression complete separation - Behind the scene
---

## What is complete separation?

Assume we have data points $(x_i, y_i)$ where $y_i \in \{0, 1\}$, a binary label. Logistic regression seeks a hyperplane defined by a coefficient vector $\beta$, such that the hyperplane will separate $0$'s and $1$'s nicely. Specifically, if $x_i \beta > 0$, then we predict that $y_i = 1$. Else, then we predict that $y_i = 0$.

Often, it is impossible to correctly classify $0$'s and $1$'s perfectly with a hyperplane, and we will make some mistakes. However, occasionally, we can find a hyperplane that separates them perfectly. This is called **complete separation**.

Complete separation does not play well with logistic regression. This is a widely known fact to practitioners. When there is complete separation, logistic regression gives extreme coefficient estimate and estimated standard error, which pretty much renders meaningful statistical inference impossible.

## A typical R output and its error messages

Typically, this is what you would see in R if there is complete separation. Let's create some fake data points.

```R
x <- c(-1,-2,-3,1,2,3)
y <- as.factor(c(1,1,1,0,0,0))
df <- data.frame(x, y)
model <- glm(y ~ x + 0, data = df, family = "binomial")
```

R would give you the following result and warning

```
Warning message:
glm.fit: fitted probabilities numerically 0 or 1 occurred 
```

If you choose to ignore the warning and proceed to see the summary anyway,

```R
> summary(model)
```



```
Call:
glm(formula = y ~ x + 0, family = "binomial", data = df)

Deviance Residuals: 
         1           2           3           4           5           6  
 1.253e-05   2.110e-08   2.110e-08  -1.253e-05  -2.110e-08  -2.110e-08  

Coefficients:
  Estimate Std. Error z value Pr(>|z|)
x   -23.27   48418.86       0        1

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 8.3178e+00  on 6  degrees of freedom
Residual deviance: 3.1384e-10  on 5  degrees of freedom
AIC: 2

Number of Fisher Scoring iterations: 24
```

The first thing that should pop out is the estimated s.e. being 48418.85, which is quite big. A big estimated s.e. leads to a strange looking Z-value and p-value, too.

If you know what "deviance" or "AIC" means, you would notice something else that is strange. For example, we know that $\text{AIC} = -2\ell + 2p$ where $\ell$ is the log-likelihood, and $p$ is the number of parameters. Here, $\text{AIC} = 2$, and we know that $p = 1$. Solving for $\ell$, we get that $\ell = 0$. This means that the likelihood of the model is $1$, which is extremely huge. You can also use `logLik(model)` to find the log-likelihood of a model object.

```R
> logLik(model)
'log Lik.' -1.569194e-10 (df=1)
```

## Why huge log-likelihood?

Recall that logistic regression is essentially a GLM where $y_i$'s are Bernoulli random variables such that

$$ \mathbb{P} \{ y_i = 1\} = p_i = \frac{1}{1 + e^{-x_i \beta}}$$

MLE seeks to maximize

$$\ell \left(\beta; y_1, \cdots, y_n\right) =  \sum_{i:y_i = 1}\ln \frac{1}{1 + e^{-x_i \beta}} + \sum_{i:y_i = 0}\ln \frac{1}{1 + e^{x_i \beta}}$$

with respect to $\beta$.

Mathematically, if there exists some $\beta$ such that

1. $x_i \beta > 0$ for all $i$ such that $y_i = 1$, and
2. $x_i \beta < 0$ for all $i$ such that $y_i = 0$,

then the hyperplane defined by $\beta$ completely separate $y_i$'s.

A key observation is that if $\beta$ completely separate $y_i$'s, then making $\beta$ arbitrarily larger still completely separates $y_i$'s (since the two inequalities still hold). As a result, if we let $\beta \to \infty$, both terms (which are logarithms of estimated probabilities) in the log-likelihood $\ell$ will go to zero.

This explains why the log-likelihood $\ell$ is close to zero and $\hat{\beta}$ is big as well when there is complete separation, as well as the error message "fitted probabilities numerically 0 or 1 occurred ".

## Why huge standard error?

The standard error comes from the Hessian matrix $\mathbf{H}$ of the log-likelihood $\ell$. To be specific, the standard error of $\hat{\beta}_i$ is the $i$-th diagonal entry of $[-\mathbf{H}]^{-1}$. It can be shown that the Hessian matrix of $\ell$ is

$$ \mathbf{H} = -\sum_{i=1}^n \mathbf{x}_i \mathbf{x}_i^T \hat{p}_i (1 - \hat{p}_i)$$

where $\hat{p}_i$ is the predicted probability on the $i$-th observation. That is,

$$ \hat{p}_i = \frac{1}{1 + e^{-\mathbf{x}_i^T \hat{\beta}}}$$

Thus, to explain the huge standard error, we can show that

$$ \left[ -\mathbf{H}\right]^{-1}_{ii} \ge \frac{1}{\left[- \mathbf{H}\right]_{ii}}$$

and $\left[-\mathbf{H}\right]_{ii}$ is small when there is complete separation.

If we consider the simple case where we only have 1 covariate, then the estimated standard error is

$$\hat{\text{Var}} \left[\beta\right] = \left[\sum_{i=1}^n x_i^2 \hat{p}_i (1 - \hat{p}_i)\right]^{-1}$$

Due to complete separation, $\hat{p}_i$'s are very close to 1 for each $i$'s, so $\text{s.e.}(\beta)$ will be large, since we are taking reciprocal of something that is close to zero.

In the case where we have $p$ covariates $\beta_1,\cdots,\beta_p$. we need to invert the matrix $-\mathbf{H}$. Consider a SVD written as $\mathbf{P}\mathbf{D}\mathbf{P}^T$ whose inverse is $\mathbf{P} \mathbf{D}^{-1} \mathbf{P}^T$. We need to compare

$$ \left[ \mathbf{P} \mathbf{D}^{-1} \mathbf{P}^T\right]_{ii} \text{ versus } \left[ \mathbf{P} \mathbf{D}\mathbf{P}^T \right]_{ii}$$

We have

$$ \left[ \mathbf{P} \mathbf{D}^{-1} \mathbf{P}^T\right]_{ii} =  \frac{P_{i1}^2}{d_1} + \cdots + \frac{P_{ip}^2}{d_p}, \text{ and that }\left[ \mathbf{P} \mathbf{D}\mathbf{P}^T \right]_{ii} = P_{i1}^2 d_1 + \cdots + P_{ip}^2 d_p$$

Using AM-GM inequality, we conclude that

$$ \left[ \mathbf{P} \mathbf{D}^{-1} \mathbf{P}^T\right]_{ii} \ge \frac{1}{\left[ \mathbf{P} \mathbf{D}\mathbf{P}^T \right]_{ii}} $$

As a result, we have

$$\hat{\text{Var}}\left[ \beta_i\right] \ge \frac{1}{\left[ -\mathbf{H}\right]_{ii}}$$

Since

$$\left[-\mathbf{H}\right]_{ii} = \sum_{j=1}^n x_{ji}^2 \cdot \hat{p}_i (1 - \hat{p}_i)$

and this is close to zero when $\hat{p}_i$ is almost 1, we will get a large estimated s.e.
