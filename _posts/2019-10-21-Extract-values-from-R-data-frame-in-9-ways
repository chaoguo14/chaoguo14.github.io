---
layout: post
title: Extract values from an R data frame in 9 ways
---

Compared to pandas, subsetting in R is slightly more complicated. Most of the time, we use `.iloc` and `.loc` to subset a pandas data frame. (There was also `.ix`, but luckily, it is deprecated.) So there are only 2 functions to master. However, R has so many different ways to subset a data frame. In fact, there is an exercise in [Advanced R](https://adv-r.hadley.nz/subsetting.html#subset-single):

> Brainstorm as many ways as possible to extract the third value from the cyl variable in the mtcars dataset.

I am sure that the readers already know several different ways of doing this (e.g. `mtcars[3, "cyl"]`). But I will try to come up with as many different solutions as possible by considering the order of subsetting. That is, we can
1. First select the column "cyl", and then find its 3rd element. Or
2. First select the 3rd row, and the find the "cyl" element. Or
3. Directly find the element.

### Directly select the element

The most straightforward solution would be `mtcars[3, "cyl"]`.

### Column, then row

A data frame in R is just a list, where its name is just the column name. As a result, we have the following 2 solutions:

1. `mtcars$cyl[3]`
2. `mtcars[["cyl"]][3]`

Both `$` and `[[ ]]` are standard way to get elements from a list. Both `mtcars$cyl` and `mtcars[["cyl"]]` are atomic vectors.

Another way would be

3. `mtcars[, "cyl"][3]`

`mtcars[, "cyl"]` also returns an atomic vector. This is the default behaviour of `[, ]` when the second index is a single integer/character string, and `drop=FALSE` is not specified. When `drop=FALSE` is specified, it maintains the dimension and returns a n-by-1 data frame.

We can also first obtain a "sub-data frame" (instead of an atomic vector), and then proceed. That is,

4. `mtcars["cyl"][3, ]`
5. `mtcars[, "cyl", drop=FALSE][3, ]`

Here, `mtcars["cyl"]` returns a n-by-1 data frame (unlike `mtcars[["cyl"]]`). As a result, it is incorrect to write `mtcars["cyl"][3]` because `[3]` is interpreted as "select the 3rd column" when `mtcars["cyl"]` has only 1 column.

### Row, then column

We can also first select the 3rd row, and then select the "cyl" from that row:

6. `mtcars[3, ][, "cyl"]`
7. `mtcars[3, ][["cyl"]]`
8. ` mtcars[3, ]$cyl`

Here `mtcars[3, ]` returns a 1-by-p data frame, not an atomic vector. This makes sense, since an atomic vector can only contain one type of elements, but different columns of a data frame can be of different types (e.g. factor, numeric). **A follow-up question: What does `mtcars[3, , drop=TRUE]` return? Why?**
