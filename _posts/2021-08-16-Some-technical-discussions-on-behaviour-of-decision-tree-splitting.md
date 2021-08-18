---
layout: post
title: Some technical discussions on behaviour of decision tree splitting
tags: decision-tree
---

### How does a regression decision tree choose on which feature to split?

This is like Data Science 101, so I will not go through the details. You can read [1] to refresh the memory. Here is the rough idea:

- Do the following steps for every feature $\mathbf{x}\_1,\cdots,\mathbf{x}\_p$
  - Sort $(x\_i, y)$ in non-descending order of $x_i$. Call the sorted pair $(x_i', y')$. Note that for different $x_i$'s we will get different $y_i'$.
  - Divide $\mathbf{y}'$ (i.e. the sorted $\mathbb{y}$) into a left part $L = \\{y_i' \le y_s = y_1'\\}$ and a right part $R = \\{y_i > y_1'\\}$, and compute something called _impurity_. For regression tree, a commonly used impurity measure is $SS(L) + SS(R)$. Here, $SS$ is the sum of squared error, defined as $SS(x_1,\cdots,x_n) = \sum_{i=1}^n (x_i - \bar{x}_n)^2$, where $\bar{x}_n$ is the sample average.
  - Repeat the previous step, but change $y_1'$ to $y_2', y_3'$ and so on.
  - We have examined $n$ different partitions. Keep the one that gives the smallest impurity $SS(L) + SS(R)$
- Now we have the smallest impurity $SS(L) + SS(R)$ for each feature. Choose the feature with the smallest overall impurity.

The question I was interested in was this:

> **Question 0**: If there any smarter way to find the best feature $\mathbf{x}$, without going through all the features? If not, is there any smarter way to understand the whole process at least?
 
If you examine the algorithm, obviously the magic happens when we create partition $L$ and $R$ on the _sorted_ vector $\mathbb{y}'$. To get to the essence of the problem, let's ignore $\mathbf{x}\_i$ for now, and think about the following problem:

> **Question 1**: Given a vector $\mathbf{y}$, and re-order it (or equivalently, permute it) to get another vector $\mathbf{y}' := \pi(\mathbf{y})$. We then partition the re-ordered vector $\mathbb{y}'$ into a left part $L$ and a right part $R$. If we want to minimize $SS(L) + SS(R)$, how should we re-order $\mathbf{y}$ (i.e. what is the optimal $\pi^*(\cdot)$? How should we partition it after that?

For simplicity, let's define order function $o(\mathbf{x})$. This is the same as `order()` in R. For example, $o([5,2,7]) = [2,1,3]$. If we can answer Question 1, then we can think of Question 0 in this way:

> There is a permutation $\pi^*$ such that if you partition $\pi^*(\mathbf{y})$ into a left part and a right part, you can minimize $SS(L) + SS(R)$.
> To find the best feature, we just need to find the feature $\mathbf{x}^*$ such that $\|o(\mathbf{x}^*) - o(\pi(\mathbf{y}))\|$ is as small as possible.

Or at least that's what I thought...

### How to find the best partition if $\mathbf{y}$ is sorted already?

I started with a seemingly easier question: Assume $\mathbf{y}$ is already sorted. Is there any _analytical_ way to find the optimal split point (Note that we can always just scan through all $n$ possibilities)? Formally,

> **Question 2**: Assume $\mathbf{y} = (y_1, y_2, \cdots, y_n)$ is sorted in ascending order. We want to divide $\mathbf{y}$ into a left part $L = \\{y \le y_s\\}$ and a right part $R = \\{y > y_s\\}$. How to find a $y_s$ that minimizes $SS(L) + SS(R)$?

At first, I thought sample average and sample median were two promising candidates. Intuitively (at least to me), if we want to make $SS(L) + SS(R)$ small, then the points within each group should be close to each other. As a result, we probably shouldn't put sample maximum and minimum in the same group.

And indeed, for many samples, the best splitting point $y_s$ is the median or the mean. For example, if $\mathbf{y} = \\{1, 2, 3\\}$, then we should split it into $L=\\{1,2\\}$ and $R =\\{3\\}$. Another good example is $\mathbf{y} = \\{1, 2, 3, 4\\}$. Here, $y_s = 2$, which is a median.

But it's also easy to find counterexamples. Consider $\mathbf{y} = \\{1, 3, 6, 8, 10\\}$. It turns out that the best split point is $y_s = 3$. This is neither mean nor median. In fact, sometimes it's not even close to the "center" of the sample. In the following example, the best split point is the black vertical line. It is quite far from the sample median (in blue) and the sample average (in red).

![]({{site.baseurl}}/assets/11_01.png)

It seems that this seemingly easier question has no simple answer. I guess you can say that you are not surprised. After all, this is a special case of the more difficult _$k$-means clustering problem_. Here, we are dealing with a 1-dimensional 2-means clustering problem. For 1-dimensional $k$-means clustering where each cluster is one interval, you can use dynamic programming to find a global optimizer. See refernece [2] for details.

In conclusion, even when $\mathbf{y}$ is sorted, there is no analytical way to find the best split $y_s$. We can always linearly scan through it. So that's not too bad.

### What if $\mathbf{y}$ is not sorted?

In general, $\mathbf{y}$ will not be sorted. We can still scan the whole thing, and find the best partition _for that specific ordering_. But for each different ordered $\mathbf{y}'$, there is a different optimal partition, and thus a different minimum SS.

![]({{site.baseurl}}/assets/11_02.png)

### No, but really, _how_ does a decision tree choose on which feature to split?


Intuitively, it should choose the feature $\mathbb{x}_i$ that is "mostly correlated" to the response $\mathbb{y}$. In order to make this statement more precise, I started by consider Pearson correlation.

> An incorrect statement: Given features $X_1,\cdots,X_n$ and a response $Y$, a decision tree always chooses the $X_i$ that maximizes its Pearson correlation with $Y$.

However, it was very easy to construct counterexample. So the above statement is wrong.

```R
Ckmeans.1d.dp::Ckmeans.1d.dp(c(1,2,3,4,5,6), k = 2)

train_df <- data.frame(X1 = c(1,3,4,5,6,7),
                       X2 = c(1,4,5,6,7,8),
                       Y  = c(1,2,3,4,5,6))

stats::cor.test(train_df$X1, train_df$Y, method = 'spearman')$estimate
stats::cor.test(train_df$X2, train_df$Y, method = 'spearman')$estimate
```

One can verify that the minimum SS is achieved by partitioning $Y$ into $\\{1,2,3\\}$ and $\\{4, 5, 6\\}$. We can achieve this partition by using either $X_1$ or $X_2$. Yet, $X_1$ has higher Pearson correlation with $Y$ than $X_2$.

The special thing about this example is this: Even though $X_1$ has higher correlation with $Y$ than $X_2$, they share the same ranking with $Y$. In order words, sorting $Y$ using $X_1$'s rank or $X_2$'s rank does not make any difference.

So maybe we can fix the incorrect statement. Instead of using Pearson's correlation, we can use some type of rank-based correlation measure. What about

> Another incorrect statement: Given features $X_1,\cdots,X_n$ and a response $Y$, a decision tree always chooses the $X_i$ that maximizes its Spearman's correlation with $Y$.

This statement also turned out to be wrong. With a little help from computer, we can quickly find a counterexample.
```R
train_df <- data.frame(X1 = c(1,2,6,3,4,5),
                       X2 = c(3,1,2,6,5,4),
                       Y  = c(1,2,3,4,5,6))
```

$X_1$ has higher Spearman's correlation with $Y$ than $X_2$ (0.657 versus 0.6). However, if we use $X_1$ to sort our $Y$ and look for the best partition, the minimum SS will be $5.5$. On the contrary, if we use $X_2$, the minimum SS is $4$, which is better.

At this point, it seems like there is no easy answer to the question "how does a decision tree choose on which feature to split?" except then reciting the algorithm itself.



_To be continued in the next post._

References
1. [https://scikit-learn.org/stable/modules/tree.html#regression-criteria](https://scikit-learn.org/stable/modules/tree.html#regression-criteria)
2. [https://journal.r-project.org/archive/2011-2/RJournal_2011-2_Wang+Song.pdf](https://journal.r-project.org/archive/2011-2/RJournal_2011-2_Wang+Song.pdf)
