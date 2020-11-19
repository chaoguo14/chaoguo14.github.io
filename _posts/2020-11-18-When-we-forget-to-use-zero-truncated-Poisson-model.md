---
layout: post
title: When we forget to use zero-truncated Poisson model
tags: statistics regression
---

Poisson model (e.g. Poisso GLM) is often used to model count data. For example, the number of train arrivals for a given time period can be Poisson distributed. This is closely related to Poisson process.

A Poisson variable $X$ can take values in $\{0, 1, 2, \cdots\}$. However, sometimes the zero's are censored. For example, the number of car accidents might be Poisson distributed. But, if a vehicle had zero accident, it might not get reported in the first place. We can only observe the non-zero outcomes. In such cases, Poisson is not the appropriate distribution to use. Instead, we should use zero-truncated Poisson distribution. As we will see later, misspecification will lead to incorrect parameter estimates, especially when it is very likely to observe zero.

### Probability Mass Function

Recall that if $X$ is Poisson distributed with mean $\lambda$, then

$$ \mathbb{P}\{X = k\} = e^{-\lambda} \frac{\lambda^k}{k!}, k\in\{0, 1, 2, \cdots\}$$

A truncated Poisson random variable $Z$ has probability mass function

$$ \mathbb{P}\{Z = k\} = \mathbb{P}\{X = k \mid X >0 \} = \frac{e^{-\lambda} \lambda^k}{(1 - e^{-\lambda}) k!},k\in\{1, 2,\cdots\}$$

We can show that $\mathbb{E}[Z] = \frac{\lambda}{1 - e^{-\lambda}}$.

### MLE based on i.i.d. sample

Assume we have observed $z_1, \cdots, z_n$ from a truncated Poisson distribution with $\lambda$ (that is, the _underlying untruncated_ variable has mean $\lambda$). The correct way to set up MLE estimator is to solve

$$ \frac{\lambda}{1 - e^{-\lambda}} = \bar{z}_n$$

for $\lambda$, which we denote as $\hat{\lambda}_{MLE, trunc}$.

This has no closed form solution. And this is different from the MLE for the untruncated Poisson, which is just $\hat{\lambda}_{MLE} =\bar{z}\_n$. However, if the zero has a fairly small probability (i.e. $e^{-\lambda}$ is very small), then $\hat{\lambda}\_{MLE}$ and $\hat{\lambda}\_{MLE, trunc}$ will be quite close. This makes intuitive sense.

```R
library(countreg)
library(gridExtra)
library(extraDistr)

sample_size <- seq(10, 5000, 99)
set.seed(233)
# Simple MLE ----
simulate <- function(n = sample_size, lambda = lambda) {
  MLE_trunc <- c()
  MLE <- c()
  
  for (n in sample_size) {
    X_pois <- data.frame(y = rpois(n = n, lambda = lambda))
    X_tpois <- data.frame(y = rtpois(n = n, lambda = lambda, a = 0, b = Inf))
    
    MLE_trunc <- c(MLE_trunc, glm(y ~ ., data = X_tpois, family = ztpoisson)$coefficients %>% exp())
    MLE <- c(MLE, glm(y ~ ., data = X_tpois, family = poisson())$coefficients %>% exp())
  }
  
  plot_df <- data.frame(MLE_trunc, MLE, sample_size) %>%
    tidyr::pivot_longer(cols = c("MLE_trunc", "MLE"))
  
  plot <- ggplot(data = plot_df, aes(x = sample_size, y = value, color = name)) +
    geom_point() +
    geom_hline(yintercept = lambda) +
    labs(title = paste("Lambda: ", lambda)) + xlab("Sample Size") + ylab("Value") +
    theme_minimal()
  return(plot)
}

plot_1 <- simulate(sample_size, lambda = 1)
plot_2 <- simulate(sample_size, lambda = 4)

grid.arrange(plot_1, plot_2, ncol = 1)
```

![two_time_series]({{site.baseurl}}/assets/374021.jpeg)
