---
layout: post
title: A popular way of choosing base level in regression revisited (cont'd)
---

### General case: with other covariates

To recap, we want to examine the following statement about choosing base level for a regression model:

> Always choose the level with most observations as the reference level, because this gives us a coefficient estimate with smaller variance.

We have shown that this is true for simple regression model. Now we look at multiple regression model.

In general, we have a dummy variable as well as other covariates. Let $\mathbf{X}_{n\times(p+1)}$ be our design matrix where the dummy variable is the last column and the intercept is the first column. For example, our model might be $y_i = \beta_0 + \beta_1 \cdot \text{family_size} + \beta_2 \cdot \text{sex} + \epsilon_i$. Then

$$ \mathbf{X} = \begin{bmatrix} 1 & 3  &1 \\ 1 & 4 & 0 \\ 1 & 5 & 0 \\ 1 & 2 & 0\end{bmatrix}, \mathbf{Z} = \begin{bmatrix} 1 & 3  &0\\ 1 & 4 & 1\\ 1 & 5 & 1 \\ 1 & 2 & 1\end{bmatrix} $$

represents the same design matrix using different sex as its base level.

We want to show a similar result, that is
1. The variance of $\hat{\beta}_p$ (coefficient corresponds to the dummy variable) does not depend on the choice of reference level.
2. The variance of $\hat{\beta}_0$ depends on the choice of reference level, and we should choose the group with more observations (spoiler alert: this is not true).

#### Proof of statement 1

How do we find the variance of $\hat{\beta}_p$ ? We know that $\hat{\beta} = \left(\mathbf{X}'\mathbf{X}\right)^{-1} \mathbf{X}' \mathbf{y}$ under one representation, and $\hat{\beta} = \left(\mathbf{Z}'\mathbf{Z}\right)^{-1}\mathbf{Z}' \mathbf{y}$. These two expressions do not help us much. We also know that the variance of $\hat{\beta}_i$ is $\left(\mathbf{X}'\mathbf{X}\right)^{-1}_{ii} \sigma^2$. But finding an explicit expression for $\left(\mathbf{X}'\mathbf{X}\right)^{-1}$ can be difficult, and the algebra can get pretty involved.

Instead, we introduce another identity from [1] and use a geometric argument. Let $V_{p-1}$ be the vector space spanned by column $\mathbf{x}_1$ (the intercept), $\mathbf{x}_2, \cdots, \mathbf{x}_{p-1}$. We can project $\mathbf{x}_p$ onto $V_{p-1}$, and we denote the projected vector as $p\left(\mathbf{x}_p \mid V_{p-1}\right)$. Equivalently, we can write $\mathbf{P} \mathbf{x}_p$ where $\mathbf{P}$ is the corresponding projection matrix.

Then, we define $ \mathbf{x}_p^\perp = \mathbf{x}_p - p(\mathbf{x}_p | V_{p-1}) $. According to [1], we can express the variance as

$$ \text{Var}\left[\hat{\beta}_p\right] = \frac{\sigma^2}{\|\mathbf{x}_p^\perp\|^2} = \frac{\sigma^2}{\langle \mathbf{x}_p^\perp, \mathbf{x}_p\rangle} = \frac{\sigma^2}{\langle \mathbf{x}_p - \mathbf{P}\mathbf{x}_p, \mathbf{x}_p\rangle}. $$

Now let's study the variance of $\hat{\beta}_p$ under different base level. This reduces to comparing $\|\mathbf{x}_p^\perp\|^2$ with $\|\mathbf{z}_p^\perp\|^2$.

Assume that when we use female as base level, the last column is $\mathbf{x}_p$. Then if we use male as reference level, the last column becomes $\mathbf{z}_p := \mathbf{1} - \mathbf{x}_p$. To compare, we write

$$ \|\mathbf{x}_p^\perp\|^2 = \langle \mathbf{x}_p - \mathbf{P}\mathbf{x}_p, \mathbf{x}_p\rangle $$

$$ \|\mathbf{z}_p^\perp\|^2 =\|\left(\mathbf{1} - \mathbf{x}_p\right)^\perp\|^2 =  \langle \left(\mathbf{1} - \mathbf{x}_p\right) - \mathbf{P}\left(\mathbf{1} - \mathbf{x}_p\right), \mathbf{1} - \mathbf{x}_p\rangle $$

We want to show that those two are equal. After some algera, we get

$$ \|\left(\mathbf{1} - \mathbf{x}_p\right)^\perp\|^2  = \|\mathbf{x}_p^\perp\|^2 + \langle \mathbf{x}_p - \mathbf{P}\mathbf{x}_p, -\mathbf{1}\rangle + \langle \mathbf{P} \mathbf{1} - \mathbf{1}, \mathbf{x}_p - \mathbf{1}\rangle $$

Notice that since $\mathbf{1}$ is in the subspace $V_{p-1}$ (since $V_{p-1}$ contains the first column, which is the intercept), $\mathbf{P} \mathbf{1} = \mathbf{1}$. So the term $\langle \mathbf{P} \mathbf{1} - \mathbf{1}, \mathbf{x} - \mathbf{1}\rangle$ vanishes. Also, since $\mathbf{P}$ is a projection and $\mathbf{1}$ is in the space on which we project, $\mathbf{x}_p - \mathbf{P}\mathbf{x}_p$ is orthogonal to $\mathbf{1}$. So $\langle \mathbf{x}_p - \mathbf{P}\mathbf{x}_p, -\mathbf{1}\rangle$ vanishes, too.

This proves the first statement.

#### Counterexamples of statement 2

Statement 2 sounds plausible but it is false. To be fair, we have run into multiple cases where using more populous category as base level gives a smaller standard error on the intercept coefficient. However, there are numerous counterexamples. We have found one using the `mtcars` data that comes with R.

```R
> # vs stands for engine type (0 = V-shaped, 1 = straight).
> # we have 18 V-shapes, and 14 straight
> cars <- mtcars[, c("mpg","cyl","disp","vs")]
> cars$intercept <- rep(1, nrow(cars))
> colnames(cars)[4] <- "vs_straight"
> print(paste("Using group with more obs as reference level?", sum(cars$vs_straight == 1) < sum(cars$vs_straight == 0)))
[1] "Using group with more obs as reference level? TRUE"
> lm(mpg ~ . - 1, data = cars) %>% summary() %>% coef()
              Estimate Std. Error    t value     Pr(>|t|)
cyl         -1.7504373 0.87235809 -2.0065582 5.454156e-02
disp        -0.0202937 0.01045432 -1.9411790 6.236295e-02
vs_straight -0.6337231 1.89593627 -0.3342534 7.406791e-01
intercept   35.8809111 4.47351580  8.0207409 9.819906e-09
> 
> cars$vs_V_shapes <- 1 - cars$vs_straight
> cars$vs_straight <- NULL
> print(paste("Using group with more obs as reference level?", sum(cars$vs_V_shapes == 1) < sum(cars$vs_V_shapes == 0)))
[1] "Using group with more obs as reference level? FALSE"
> lm(mpg ~ . - 1, data = cars) %>% summary() %>% coef()
              Estimate Std. Error    t value     Pr(>|t|)
cyl         -1.7504373 0.87235809 -2.0065582 5.454156e-02
disp        -0.0202937 0.01045432 -1.9411790 6.236295e-02
intercept   35.2471880 3.12535016 11.2778365 6.348795e-12
vs_V_shapes  0.6337231 1.89593627  0.3342534 7.406791e-01
```

The first regression is run using V-shaped engine (which has more observations) as base level. Yet, the standard error of the intercept is 4.47, much larger than what we get by using straight engine (which has fewer observations) as base level. **Also, note that the standard error of `vs_straight` and `vs_V_shapes` are the same, as we expect.**

In this example, we have 32 observations. They are not equally distributed to two categories (14 obs vs 18 obs), but close. Here is another less balanced example, where we have 20 observations with 15 males. When we use male as base level, the standard error of the intercept coefficient is actually slightly larger.

```R
> set.seed(64)
> n <- 20
> n_m <- 15  # First 15 obs of 20 are males
> 
> dummy_var <- matrix(c(rep(1, n_m), rep(0, n - n_m)), nrow = n)  # Use minority group (female) as base level
> y <- rnorm(20)
> C <- matrix(rpois(n*6, lambda = 6), nrow = n)
> X <- cbind(C, dummy_var, y)
> X <- data.frame(X)
> colnames(X)[7] <- "dummy_var"
> 
> lm(y ~ ., X) %>% summary() %>% coef()
                Estimate Std. Error     t value  Pr(>|t|)
(Intercept)  1.418429502  2.8457756  0.49843337 0.6271886
V1          -0.138312337  0.1547772 -0.89362221 0.3890947
V2           0.161704256  0.1420289  1.13853095 0.2771187
V3           0.011231707  0.1439728  0.07801269 0.9391037
V4          -0.006552859  0.1418405 -0.04619879 0.9639117
V5          -0.329203038  0.1899600 -1.73301198 0.1086883
V6          -0.032312033  0.1480165 -0.21830024 0.8308635
dummy_var    0.256849636  0.6964661  0.36878986 0.7187088
> 
> X$dummy_var <- 1 - X$dummy_var  # Now use majority group (male) as base level
> lm(y ~ ., X) %>% summary() %>% coef()
                Estimate Std. Error     t value  Pr(>|t|)
(Intercept)  1.675279138  2.8950661  0.57866697 0.5735149
V1          -0.138312337  0.1547772 -0.89362221 0.3890947
V2           0.161704256  0.1420289  1.13853095 0.2771187
V3           0.011231707  0.1439728  0.07801269 0.9391037
V4          -0.006552859  0.1418405 -0.04619879 0.9639117
V5          -0.329203038  0.1899600 -1.73301198 0.1086883
V6          -0.032312033  0.1480165 -0.21830024 0.8308635
dummy_var   -0.256849636  0.6964661 -0.36878986 0.7187088
```


Reference:
1. Linear Statistical Models Second Edition. James H. Stapleton. Wiley. Section 3.5

_I appreciate my friend [Changyuan Lyu](https://www.linkedin.com/in/clyu/) for taking time to discuss this problem with me._
