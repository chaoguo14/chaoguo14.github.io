---
layout: post
title: A popular way of choosing base level in regression: Revisited
---

## Set the base level to be one with populous data?
Let's say you are running a multiple linear regression model, and you have a categorical variable (e.g. sex, which can be male or female). How do you choose the reference level (or base level) for the dummy variable? For example, if you choose "male" as the reference level, then you would define the dummy variable like this:

$$ D_i = \begin{cases} 1 &\text{if i-th observation is female}\\0 &\text{if i-th observation is male} \end{cases} $$

Interestingly, there is a common "urban legend" rule on how to choose the reference level:

> Always choose the level with most observations as the reference level, because this gives us a coefficient estimate with smaller variance.

Personally, many people have told me something similar and they work in different industries as statisticians/data scientists. I am not the only one who has experience that, neither. For example, see [1]. There are some resources that approve this approach although they fail to present a rigorous demonstration/proof. For example, see [2] and [3].

This is a strong statement, and I will investigate to what extent it is true. To save you some time,

>**TL;DR:**
>1. In simple linear regression, this statement is true and can be proved mathematically.
>2. In multiple linear regression, this is not always true, as we can find counter-examples easily. But it is 50% true.
>3. In GLM, I do not know.

Let's dive into it, shall we?

## Simple case: $y_i = \beta_0 + \beta_1 \cdot \text{sex} + \epsilon_i$

Let's consider a simple case first. We have observations $\left(d_1, y_1\right), \cdots, \left(d_n, y_n\right)$ and commonly-seen assumptions of linear regression are satisfied. The dummy variable is the only covariate, and indicates the sex of each observation. We assume there are only 2 possibilities: male, female. We consider model

$$ Y_i = \beta_0 + \beta_1 D_i + \epsilon_i ,\,\,\, \epsilon_i \sim \mathcal{N}\left(0, \sigma^2\right) $$

Assume there are $n_m$ males, and $n_f$ females. Should we choose male or female as the base level?

We claim that we should choose whichever level that has more observations. Specifically, we will prove 2 statements:

1. Regardless of the reference level, $\text{Var}\left[\hat{\beta}_1\right] = \frac{\sigma^2}{n_m} + \frac{\sigma^2}{n_f}$.
2. If female is the base level (we mark $D_i = 1$ when $i$ is male), then $\text{Var}\left[\hat{\beta}_0\right] = \frac{\sigma^2}{n_f}$. If male is the base level, then $\text{Var}\left[\hat{\beta}_0\right] = \frac{\sigma^2}{n_m}$. So we should choose the female as base level if $n_f \ge n_m$.

One interesting things: the choice of base level has no impact on the variance of $\hat{\beta}_1$, the coefficient estimate of the dummy variable. In other words, choosing the "correct" base level leads to a better model by estimating the intercept better.

**Proof of 1:** We know that

$$ \hat{\beta}_1 = \frac{\sum \left(d_i - \bar{d}\right)\left(y_i - \bar{y}\right)}{\sum \left(d_i - \bar{d}\right)^2} $$

from which we can derive that

$$ \text{Var}\left[\hat{\beta}_1\right] = \frac{\sigma^2}{\sum \left(d_i - \bar{d}\right)^2} $$

If female is the reference level, then $d_i = 1$ when the $i$-th observation is male. So

$$ \sum \left(d_i - \bar{d}\right)^2 = \sum d_i^2 - n\left(\bar{d}\right)^2 = n_m - \frac{n_m^2}{n} = n_m\cdot \frac{n_f}{n} $$

if we recall that $n_m + n_f = n$.

**Proof of 2:** We know that

$$ \hat{\beta}_0 = \bar{y} - \hat{\beta}_1 \bar{d} $$

from which we can derive that

$$ \text{Var}\left[\hat{\beta}_0\right] = \left(\frac{1}{n} + \frac{\bar{d}^2}{\sum \left(d_i - \bar{d}\right)^2}\right) \sigma^2 $$

Assume female is the reference level, then $\bar{d}^2 = n_m^2 / n^2$, and $\sum \left(d_i - \bar{d}\right)^2 =  n_m n_f/n$. So the variance (omitting $\sigma^2$) becomes

$$ \frac{1}{n} + \frac{\bar{d}^2}{\sum \left(d_i - \bar{d}\right)^2} = \frac{1}{n} + \frac{n_m^2}{n^2} \frac{n}{n_m n_f} = \frac{1}{n} + \frac{n_m}{n n_f} = \frac{n}{n n_f} = \frac{1}{n_f} $$

Now assume male is the reference level, then $\bar{d}^2 = n_f^2/n$. Similar derivation shows that statement 2 is true.

_To be continued in the next post._

References:
1. https://stats.stackexchange.com/questions/208329/reference-level-in-glm-regression
2. https://www.casact.org/pubs/monographs/papers/05-Goldburd-Khare-Tevet.pdf  page 15
3. https://www.theanalysisfactor.com/strategies-dummy-coding/
