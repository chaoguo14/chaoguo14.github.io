---
layout: post
title: An interesting bug in the package RandomForest
---

## Environment and a reproducible example

The other day I ran into this interesting bug in the randomForest package in R. I was running on R version 3.5.2, and the version of the randomForest package was 4.6.14.

```r
> library(randomForest)
> packageVersion("randomForest")
[1] ‘4.6.14’
```

To replicate the bug, run the following codes:
```r
rf_model <- randomForest(x = iris[, colnames(iris) != "Species"], y = iris$Species)

# This works fine
partialPlot(rf_model, iris, "Sepal.Length")
partialPlot(rf_model, iris, Sepal.Length)
partialPlot(rf_model, iris, c("Sepal.Length", "Sepal.Width")[1])

# But this does not work
var_interested <- "Sepal.Length"
partialPlot(rf_model, iris, var_interested)
# Error in `[.data.frame`(pred.data, , xname) : undefined columns selected
```

According to the documentation, `partialPlot()` makes a partial dependence plot, which gives a graphical depiction of the marginal effect of a variable. More explicitly, `partialPlot(x = rf_model, pred.data = iris, x.var = "Sepal.Length")` makes a PDP of the model `rf_model` on the data set `iris`, and it plots the marginal effect of the variable `Sepal.Length`.

Obviously, some sort of nonstandard evaluation technique is used to write the function, since both `"Sepal.Length"` and `Sepal.Length` can be correctly recognized by the function.

However, the function fails if we assign `"Sepal.Length"` to a new variable `var_interested`, and we pass `var_interested` to the function.

## Getting the source code

To figure out what is going wrong, we need to look at the source code. A common way to do this is to use the `print()` function.

```r
> print(randomForest::partialPlot)
function (x, ...) 
UseMethod("partialPlot")
<bytecode: 0x7f8f80b01bc8>
<environment: namespace:randomForest>
```

This does not give us much. Since the first argument passed to `partialPlot()` has class `randomForest`, we should try

```r
> print(randomForest::partialPlot.randomForest)
Error: 'partialPlot.randomForest' is not an exported object from 'namespace:randomForest'
> print(randomForest:::partialPlot.randomForest)
```

And this will gives us the source code.

## What really happened?

The first couple of lines of the source code read

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

As we can see, `xname` is created based on `x.var`, so we just need to understand what `substitue()` is doing here. According to the documentation,
>`substitute()` returns the parse tree for the (unevaluated) expression expr, substituting any variables bound in env

>Substitution takes place by examining each component of the parse tree as follows: If it is not a bound symbol in env, it is unchanged. **If it is a promise object, i.e., a formal argument to a function or explicitly created using delayedAssign(), the expression slot of the promise replaces the symbol.** If it is an ordinary variable, its value is substituted, unless env is .GlobalEnv in which case the symbol is left unchanged...

Although the explanation is terse, we only need to pay attention to the bolded sentence, since in our case, `substitute()` is used inside a function. For any expression supplied to the argument `x.var`, `substitue()` will simply let `x.var` be that expression. Consider the following example.

```r
f <- function(x.var) {
    print(pryr::ast(x.var))  # show the parse tree
    x.var <- substitute(x.var)
    print(class(x.var))
    print(x.var)
}

> f("Sepal.Length")
\- `x.var 
NULL
[1] "character"
[1] "Sepal.Length"

> f(Sepal.Length)
\- `x.var 
NULL
[1] "name"
Sepal.Length
```

As we can see, if we provide a quoted string, then the string gets assigned to `x.var`. Then `is.character(x.var)` is true, so everything goes through. If we provide an unquoted name, the name gets assigned to `x.var`. Now, `x.var` has class `name`. When we deparse it, we get the quoted name.

So why would the function fail if we assign the quoted names `"Sepal.Length"` to `var_interested`?

```r
> var_interested <- "Sepal.Length"
> f(x.var = var_interested)
\- `x.var 
NULL
[1] "name"
var_interested
```

When `substitute()` looks at the parse tree of the expression `x.var`, it still returns `x.var`. However, since we wrote `x.var = q`, it is replaced with `q`, not `"haha"`, which is what `q` points to. This causes the problem, because there is no column in the data named `"var_interested"`!

But why would this example work? 
```r
partialPlot(rf_model, iris, c("Sepal.Length", "Sepal.Width")[1])
```

When we call `substitute(x.var)`, it first looks at the parse tree of the expression `x.var`, which is just `x.var` itself. Then it tries to replace it with the expression `c("Sepal.Length", "Sepal.Width")[1]`. Recall that any R expression has 4 components: call, constant, pairlist, and names. The expression `c("Sepal.Length", "Sepal.Width")[1]`, since we are calling the function that gives us the 1st element of an atomic vector. To verify this, we can type

```r
> is.call(substitute(c("Sepal.Length", "Sepal.Width")[1]))
[1] TRUE
> is.name(substitute(c("Sepal.Length", "Sepal.Width")[1]))
[1] FALSE
```

Since it is not a character, nor a name, it gets evaluated directly in the if-else statements, thus gives `x.var` the right value (which is `"Sepal.Length"`) in the end.

## Final thoughts: should we encourage non-standard evaluation?

This is a strange bug, and it requires good understanding of advanced R to debug it. However, if we require user to supply a character string in the first place, then we would have not encounter this bug. For example, a common approach is to raise an error if `is.character(x.var)` returns `FALSE`.

Non-standard evaluation can be handy. Yet my personal opinion is that, under many circumstances, it only saves us from typing many quotes. For users coming from a different programming language (e.g. Python, Java), non-standard evaluation can cause more confusions than benefit.
