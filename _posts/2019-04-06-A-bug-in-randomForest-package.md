---
layout: post
title: An interesting bug in the package RandomForest
---

## Environment and a reproducible example

I ran into this interesting bug with the `randomForest` package in R when I was trying to make some [partial dependence plots](https://journal.r-project.org/archive/2017/RJ-2017-016/RJ-2017-016.pdf). I was running R version 3.5.2, and the version of the randomForest package was 4.6.14.

```r
> library(randomForest)
> packageVersion("randomForest")
[1] '4.6.14'
```

To replicate the bug, run the following codes:
```r
rf_model <- randomForest(x = iris[, colnames(iris) != "Species"], y = iris$Species)

# This works fine
partialPlot(rf_model, iris, x.var = "Sepal.Length")
partialPlot(rf_model, iris, x.var = Sepal.Length)
partialPlot(rf_model, iris, x.var = c("Sepal.Length", "Sepal.Width")[1])

# But this does not work
var_interested <- "Sepal.Length"
partialPlot(rf_model, iris, x.var = var_interested)
# Error in `[.data.frame`(pred.data, , xname) : undefined columns selected
```

The failure is surprising, and we will find out why it happens.

## Getting the source code

To figure out what is going wrong, we need to look at the source code. A common way to do this is to use the `print()` function.

```r
> print(randomForest::partialPlot)
function (x, ...) 
UseMethod("partialPlot")
<bytecode: 0x7f8f80b01bc8>
<environment: namespace:randomForest>
```

This does not give us much information. Since the first argument passed to `partialPlot()` has class `randomForest`, we try

```r
> print(randomForest::partialPlot.randomForest)
Error: 'partialPlot.randomForest' is not an exported object from 'namespace:randomForest'
> print(randomForest:::partialPlot.randomForest)  # recall the difference between :: and :::
```

And this will gives us the source code for `partialPlot.randomForest()`. The first couple of lines read

```r
function (x, pred.data, x.var, which.class, w, plot = TRUE, add = FALSE, 
    n.pt = min(length(unique(pred.data[, xname])), 51), rug = TRUE, 
    xlab = deparse(substitute(x.var)), ylab = "", main = paste("Partial Dependence on", 
        deparse(substitute(x.var))), ...) 
{
    classRF <- x$type != "regression"
    
    if (is.null(x$forest)) 
        stop("The randomForest object must contain the forest.\n")
        
    x.var <- substitute(x.var)
    xname <- if (is.character(x.var)) 
                x.var
            else {
                if (is.name(x.var)) 
                    deparse(x.var)
                else {
                    eval(x.var)
                }
            }
    ...
```

Apparently, when we write `x.var = var_interested`, the function does not assign the correct value to `xname`.

## What does `substitute()` do?

According to the documentation,

>_substitute()_ returns the parse tree for the (unevaluated) expression _expr_, substituting any variables bound in env

>Substitution takes place by examining each component of the parse tree as follows: If it is not a bound symbol in env, it is unchanged. **If it is a promise object, i.e., a formal argument to a function or explicitly created using delayedAssign(), the expression slot of the promise replaces the symbol.** If it is an ordinary variable, its value is substituted, unless env is .GlobalEnv in which case the symbol is left unchanged...

Although the explanation is terse, we only need to pay attention to the bolded sentence, since in our case, `substitute()` is used inside a function. For any expression supplied to the argument `x.var`, `substitue()` will simply let `x.var` be that expression. Consider the following example.

```r
f <- function(x.var) {
    print(pryr::ast(x.var))  # show the parse tree
    x.var <- substitute(x.var)
    print(x.var)
    print(class(x.var))

}

> f(x.var = "Sepal.Length")
\- `x.var 
NULL
[1] "Sepal.Length"  # "Sepal.Length" is supplied to x.var
[1] "character"  # And obviously, "Sepal.Length" is character

> f(x.var = Sepal.Length)
\- `x.var 
NULL
Sepal.Length  # Sepal.Length is supplied to x.var, without quotes
[1] "name"  # Sepal.Length is not character since there is no quote. It's just a name.
```

So why does the function fail if we assign the quoted names `"Sepal.Length"` to `var_interested`?

```r
> var_interested <- "Sepal.Length"
> f(x.var = var_interested)
\- `x.var 
NULL
var_interested
[1] "name"
```

Let's be a little bit more careful here. We first assigned `"Sepal.Length"` (which is a character) to the variable `var_interested`. Then, we assigned `var_interested` to the argument `x.var`. You might expect `substitute(x.var)` to return `"Sepal.Length"`, but this is not the case! `substitute()` will only substitute `x.var` with whatever is assigned to it (in this case, it's the variable name `var_interested`), not the underlying evaluated value. So when we enter the if-else statements, since `x.var` is not character, but a name, `xname <- deparse(x.var)` is executed. Now, the function will look for a column with name "var_interested", and of course it cannot find it. **In conclusion, we have this bug because the author assumed that if the user pass something without quotes to `x.var`, it is always the name of the column without quotes.**

But why would this example work? 
```r
partialPlot(rf_model, iris, c("Sepal.Length", "Sepal.Width")[1])
```

When we call `substitute(x.var)`, it first looks at the parse tree of the expression `x.var`, which is just `x.var` itself. Then it tries to replace it with the expression `c("Sepal.Length", "Sepal.Width")[1]`. Recall that any R expression has 4 components: call, constant, pairlist, and names. The expression `c("Sepal.Length", "Sepal.Width")[1]` is a call, since we are calling the function `c()` first, then the subsetting function `[]`. To verify this, we can type

```r
> is.call(substitute(c("Sepal.Length", "Sepal.Width")[1]))
[1] TRUE
> is.name(substitute(c("Sepal.Length", "Sepal.Width")[1]))
[1] FALSE
```

Since it is not a character, nor a name, it gets evaluated directly in the if-else statements, thus giving `x.var` the correct value (which is `"Sepal.Length"`).

## Final thoughts: should we encourage non-standard evaluation?

This is a strange bug, and it requires good understanding of advanced R to debug. However, if we require user to supply a character string in the first place, then we would have not encounter this bug. For example, a common approach is to raise an error if `is.character(x.var)` returns `FALSE`.

Non-standard evaluation can be handy. Yet my personal opinion is that, under many circumstances, it only saves us from typing many quotes. For users coming from a different programming language (e.g. Python, Java), non-standard evaluation can cause more confusions than benefit.
