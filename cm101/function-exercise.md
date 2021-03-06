---
title: "STAT 547 Class Meeting 01: Writing your own Functions"
output: github_document
---


```r
library(gapminder)
library(tidyverse)
library(testthat)
```

This worksheet is a condensed version of Jenny's stat545.com functions [part1](http://stat545.com/block011_write-your-own-function-01.html), [part2](http://stat545.com/block011_write-your-own-function-02.html), and [part3](http://stat545.com/block011_write-your-own-function-03.html).

## Syntax Demo

Let's demo the syntax of function-making.


```r
square <- function(x) {
  x^2
}

square(4)
```

```
## [1] 16
```

## Motivating example: max minus min.

Find the max minus min of the gapminder life expectancy:


```r
max(gapminder$lifeExp) - min(gapminder$lifeExp)
```

```
## [1] 59.004
```

Exercise: turn this into a function! i.e., write a function that returns the max minus min of a vector. Try it out on the gapminder variables.


```r
max_minus_min <- function(x) {
  max(x) - min(x)
}

identical(max(gapminder$lifeExp) - min(gapminder$lifeExp), max_minus_min(gapminder$lifeExp))
```

```
## [1] TRUE
```

```r
# [1] TRUE
```

We'll be building on this. Development philosophy [widely attributed to the Spotify development team](http://blog.fastmonkeys.com/?utm_content=bufferc2d6e&utm_medium=social&utm_source=twitter.com&utm_campaign=buffer):

![](http://stat545.com/img/spotify-howtobuildmvp.gif)

## Testing

Check your function using your own eyeballs:

- Apply to the vector 1:10. Do you get the intended result?
- Apply to a random uniform vector. Do you get meaningful results?


```r
max_minus_min(1:10)
```

```
## [1] 9
```

```r
# [1] 9
max_minus_min(runif(100))
```

```
## [1] 0.9510273
```

```r
# [1] 0.9799598
```

Let's formalize this testing with the `testthat` package. `expect_*()` functions:


```r
expect_equal(0.1 + 0.2, 0.3)
expect_identical(0.1 + 0.2, 0.3)
```

```
## Error: 0.1 + 0.2 not identical to 0.3.
## Objects equal but not identical
```

```r
# Objects equal but not identical
# there are floating point differences between args in expect_identical
```

Add another check to the following unit test, based on the uniform random numbers:


```r
test_that("Simple cases work", {
    expect_equal(max_minus_min(1:10), 9)
    expect_lt(max_minus_min(runif(100)),1) # expect less than
    expect_gt(max_minus_min(runif(100)),0.97) # expect greater than
    expect_that(max_minus_min(runif(100)), function(x) x < 1 & x > 0.97)
} )
```

```
## Error: Test failed: 'Simple cases work'
## * max_minus_min(runif(100)) is not strictly more than 0.97. Difference: -0.00523
```

## Try and break your function

Because you will eventually forget the function specifics.


```r
max_minus_min(numeric(0))
```

```
## Warning in max(x): no non-missing arguments to max; returning -Inf
```

```
## Warning in min(x): no non-missing arguments to min; returning Inf
```

```
## [1] -Inf
```

```r
max_minus_min(gapminder)
```

```
## Error in FUN(X[[i]], ...): only defined on a data frame with all numeric variables
```

```r
max_minus_min(gapminder$country)
```

```
## Error in Summary.factor(structure(c(1L, 1L, 1L, 1L, 1L, 1L, 1L, 1L, 1L, : 'max' not meaningful for factors
```

These don't break!


```r
max_minus_min(gapminder[c('lifeExp', 'gdpPercap', 'pop')])
```

```
## [1] 1318683072
```

```r
max_minus_min(c(TRUE, TRUE, FALSE, TRUE, TRUE))
```

```
## [1] 1
```

We want:

1. Prevent the latter cases from happening, and
2. Make a more informative error message in the former.

Check out `stopifnot` and `stop`:


```r
stopifnot(FALSE)
```

```
## Error in eval(expr, envir, enclos): FALSE is not TRUE
```

```r
stop("Here's my little error message.")
```

```
## Error in eval(expr, envir, enclos): Here's my little error message.
```

Your turn:  Use two methods:

1. Using `stopifnot`, modify the max-min function to throw an error if an input is not numeric (the `is.numeric` function is useful).


```r
mmm1 <- function(x) {
    stopifnot(is.numeric(x))
    max(x) - min(x)
}
```

2. Using `stop` and an `if` statement, Modify the max-min function to:
    - throw an error if an input is not numeric. In the error message, indicate what's expected as an argument, and what was recieved.
    - return `NULL` if the input is length-0, with a warning using the `warning` function.


```r
mmm2 <- function(x) {
    if (!is.numeric(x)) {
        stop(paste("Expecting x to be numeric. You gave me", typeof(x)))
    }
    max(x) - min(x)
}
mmm2(gapminder)
```

```
## Error in mmm2(gapminder): Expecting x to be numeric. You gave me list
```

Try breaking the function now:


```r
mmm1((numeric(0)))
```

```
## Warning in max(x): no non-missing arguments to max; returning -Inf
```

```
## Warning in min(x): no non-missing arguments to min; returning Inf
```

```
## [1] -Inf
```

```r
mmm1(gapminder)
```

```
## Error in mmm1(gapminder): is.numeric(x) is not TRUE
```

```r
mmm1(gapminder$country)
```

```
## Error in mmm1(gapminder$country): is.numeric(x) is not TRUE
```

```r
mmm1(gapminder[c('lifeExp', 'gdpPercap', 'pop')])
```

```
## Error in mmm1(gapminder[c("lifeExp", "gdpPercap", "pop")]): is.numeric(x) is not TRUE
```

```r
mmm1(c(TRUE, TRUE, FALSE, TRUE, TRUE))
```

```
## Error in mmm1(c(TRUE, TRUE, FALSE, TRUE, TRUE)): is.numeric(x) is not TRUE
```

```r
mmm2((numeric(0)))
```

```
## Warning in max(x): no non-missing arguments to max; returning -Inf

## Warning in max(x): no non-missing arguments to min; returning Inf
```

```
## [1] -Inf
```

```r
mmm2(gapminder)
```

```
## Error in mmm2(gapminder): Expecting x to be numeric. You gave me list
```

```r
mmm2(gapminder$country)
```

```
## Error in mmm2(gapminder$country): Expecting x to be numeric. You gave me integer
```

```r
mmm2(gapminder[c('lifeExp', 'gdpPercap', 'pop')])
```

```
## Error in mmm2(gapminder[c("lifeExp", "gdpPercap", "pop")]): Expecting x to be numeric. You gave me list
```

```r
mmm2(c(TRUE, TRUE, FALSE, TRUE, TRUE))
```

```
## Error in mmm2(c(TRUE, TRUE, FALSE, TRUE, TRUE)): Expecting x to be numeric. You gave me logical
```

## Naming, and generalizing to quantile difference

Let's generalize the function to take the difference in two quantiles:


```r
qd <- function(x, probs = c(0,1)) {
    stopifnot(is.numeric(x))
    if (length(x) == 0) {
        warning("You inputted a length-0 x. Expecting length >=1. Returning NULL.")
        return(NULL)
    }
    qvec <- quantile(x, probs)
    max(qvec) - min(qvec)
}
```

Try it out:


```r
x <- runif(100)
qd(x, c(0.25, 0.75))
```

```
## [1] 0.5005617
```

```r
IQR(x)
```

```
## [1] 0.5005617
```

```r
qd(x) # added default probs
```

```
## [1] 0.9557066
```

```r
mmm2(x)
```

```
## [1] 0.9557066
```

Why did I call the arguments `x` and `probs`? Check out `?quantile`.

If we input a vector stored in some variable, need that variable be named `x`?

## Defaults

Would be nice to have defaults for `probs`, right? Add them to the below code (which is copied and pasted from above):


```r
qd2 <- function(x, probs = c(0,1)) {
    stopifnot(is.numeric(x))
    if (length(x) == 0) {
        warning("You inputted a length-0 x. Expecting length >=1. Returning NULL.")
        return(NULL)
    }
    qvec <- quantile(x, probs)
    max(qvec) - min(qvec)
}

qd2(rnorm(100))
```

```
## [1] 4.860784
```

## NA handling

Does this return what we were expecting?


```r
v <- c(1:10, NA)
qd(v)
```

```
## Error in quantile.default(x, probs): missing values and NaN's not allowed if 'na.rm' is FALSE
```

Notice that `quantile()` has a `na.rm` option. Let's use it in our `qd` function. Modify the code below:


```r
qd2 <- function(x, probs=c(0,1), na.rm = F) {
    stopifnot(is.numeric(x))
    if (length(x) == 0) {
        warning("You inputted a length-0 x. Expecting length >=1. Returning NULL.")
        return(NULL)
    }
    qvec <- quantile(x, probs, na.rm = na.rm)
    max(qvec) - min(qvec)
}

qd2(v, na.rm = T)
```

```
## [1] 9
```

## Ellipses

There are other arguments to `quantile`, like `type`, that are not used all that much. Put them in as ellipses:


```r
qd3 <- function(x, probs=c(0,1), ...) {
    stopifnot(is.numeric(x))
    if (length(x) == 0) {
        warning("You inputted a length-0 x. Expecting length >=1. Returning NULL.")
        return(NULL)
    }
    qvec <- quantile(x, probs, ...)
    max(qvec) - min(qvec)
}

qd3(v, c(0.25,0.75), na.rm = T, type = 1)
```

```
## [1] 5
```
