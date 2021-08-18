---
layout: post
title: Some technical discussions on behaviour of decision tree splitting (cont'd)
tags: decision-tree
---

### Swapping elements to reduce $SS$

Although we omit the proof, we show how to reduce $SS$ by swapping using a demo. Let's take $L = \\{1,3\\}, R = \\{2,4,5\\}$ as an example.

```R
SS <- function(v) PreciseSums::fsum((v - mean(v))^2)

L_array <- c(1,3)
R_array <- c(2,4,5)

while (TRUE) {
  cat("------------------------------------\n")
  # Make sure L has smaller average
  if (mean(L_array) > mean(R_array)) {
    tp <- L_array
    L_array <- R_array
    R_array <- tp
  }
  
  ss_before_swap <- SS(L_array) + SS(R_array)
  cat("Before swapping SS: ", ss_before_swap, "\n")
  cat("Input: ", L_array, "░", R_array, "\n")
  
  L_array <- sort(L_array)
  R_array <- sort(R_array)
  
  cat("Sort:  ", L_array, "░", R_array, "\n")
  
  # Check if max(L_array) <= min(R_array). If not, swap them
  if (L_array[length(L_array)] > R_array[1]) {
    cat("Swap ",L_array[length(L_array)], " and ", R_array[1],"\n")
    right_most <- L_array[length(L_array)]
    left_most <- R_array[1]
    L_array[length(L_array)] <- left_most
    R_array[1] <- right_most
    
    ss_after_swap <- SS(L_array) + SS(R_array)
    
    cat("\nOutput: ", L_array, "░", R_array, "\n")
    cat("After swapping SS:  ", ss_after_swap, "\n")
  } else {
    cat("Paused. Condition L ⊲ R met.\n")
    break
  }
}
```

![]({{site.baseurl}}/assets/12_01.png)

Because there are only 15 partitions for $\\{1,2,3,4,5\\}$, we can even plot all of them.

![]({{site.baseurl}}/assets/12_02.png)
