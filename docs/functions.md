
# Functions

## Introduction


```r
library("tidyverse")
library("lubridate")
```


## When should you write a function?


### Exercise 1 {.exercise} 


<div class='question'>

Why is TRUE not a parameter to `rescale01()`? 
What would happen if `x` contained a single missing value, and `na.rm` was `FALSE`?

</div>

<div class='answer'>

First, note that by a a single missing value, this means that the vector `x` has at least one element equal to `NA`.

If there were any `NA` values, and `na.rm = FALSE`, then the function would 
return `NA`.

I can confirm this by testing a function that allows for `na.rm` as an argument,

```r
rescale01_alt <- function(x, finite = TRUE) {
  rng <- range(x, na.rm = finite, finite = finite)
  (x - rng[1]) / (rng[2] - rng[1])
}
rescale01_alt(c(NA, 1:5), finite = FALSE)
#> [1] NA NA NA NA NA NA
rescale01_alt(c(NA, 1:5), finite = TRUE)
#> [1]   NA 0.00 0.25 0.50 0.75 1.00
```

</div>



### Exercise 2 {.exercise}


<div class='question'>

In the second variant of `rescale01()`, infinite values are left unchanged. 
Rewrite `rescale01()` so that `-Inf` is mapped to 0, and `Inf` is mapped to 1.

</div>


<div class='answer'>


```r
rescale01 <- function(x) {
  rng <- range(x, na.rm = TRUE, finite = TRUE)
  y <- (x - rng[1]) / (rng[2] - rng[1])
  y[y == -Inf] <- 0
  y[y == Inf] <- 1
  y
}

rescale01(c(Inf, -Inf, 0:5, NA))
#> [1] 1.0 0.0 0.0 0.2 0.4 0.6 0.8 1.0  NA
```

</div>

### Exercise 3 {.exercise}

<div class='question'>

 Practice turning the following code snippets into functions. Think about what each function does. What would you call it? How many arguments does it need? Can you rewrite it to be more expressive or less duplicative?
 
</div>
 
<div class='answer'>


```r
mean(is.na(x))

x / sum(x, na.rm = TRUE)

sd(x, na.rm = TRUE) / mean(x, na.rm = TRUE)
```


This function calculates the proportion of `NA` values in a vector,

```r
prop_na <- function(x) {
  mean(is.na(x))
}
prop_na(c(NA, 0, NA, 0, NA))
#> [1] 0.6
```

This function standardizes a function to its weight. If all elements of `x` are non-negative, this will ensure the vector sums to 1. 

```r
weights <- function(x) {
  x / sum(x, na.rm = TRUE)
}
y <- weights(0:5)
y
#> [1] 0.0000 0.0667 0.1333 0.2000 0.2667 0.3333
sum(y)
#> [1] 1
```

This function calculates the [coefficient of variation](https://en.wikipedia.org/wiki/Coefficient_of_variation) (assuming that `x` can only take non-negative values). 
The coefficient of variation is the standard deviation divided by the mean

```r
coef_variation <- function(x) {
  sd(x, na.rm = TRUE) / mean(x, na.rm = TRUE)
}
coef_variation(runif(10))
#> [1] 0.672
```

</div>

### Exercise 4


<div class='question'>

Follow <http://nicercode.github.io/intro/writing-functions.html> to write your own functions to compute the variance and skew of a numeric vector.

</div>


<div class='answer'>

**Note** The math in <https://nicercode.github.io/intro/writing-functions.html> seems not to be rendering, but I'll write functions for the variance and skewness.

The sample variance is defined as 
$$
Var(x) = \frac{1}{n - 1} \sum_{i=1}^n (x_i - \bar{x}) ^2
$$
where the sample mean is $\bar{x} = (\sum x_i) / n$.

```r
variance <- function(x) {
  # remove missing values
  x <- x[!is.na(x)]
  n <- length(x)
  m <- mean(x)
  sq_err <- (x - m) ^ 2
  sum(sq_err) / (n - 1)
}
var(1:10)
#> [1] 9.17
variance(1:10)
#> [1] 9.17
```

There are multiple definitions of [skewness](https://en.wikipedia.org/wiki/Skewness), but I'll use the method of moments estimator of the population skewness,
$$
b_1 =  \frac{m_3}{s^3} = \frac{\frac{1}{n} \sum (x_i - \bar{x}) ^ 3}{{\left(\frac{1}{n - 1} \sum (x_i - \bar{x}) ^ 2\right)} ^ \frac{3}{2}}
$$

```r
skewness <- function(x) {
  x <- x[!is.na(x)] 
  n <- length(x)
  m <- mean(x)
  m3 <- sum((x - m) ^ 3) / n
  s3 <- sqrt(sum((x - m) ^ 2) / (n - 1))
  m3 / s3
}
skewness(rgamma(10, 1, 1))
#> [1] 1.56
```

</div>

#### Exercise 5 {.exercise}

<div class='question'>

Write `both_na()`, a function that takes two vectors of the same length and returns the number of positions that have an `NA` in both vectors.

</div>

<div class='answer'>


```r
both_na <- function(x, y) {
  sum(is.na(x) & is.na(y))
}
both_na(c(NA, NA,  1, 2),
        c(NA,  1, NA, 2))
#> [1] 1
both_na(c(NA, NA,  1, 2, NA, NA, 1), 
        c(NA,  1, NA, 2, NA, NA, 1))
#> [1] 3
```

</div>

### Exercise 6 {.exercise}


<div class='question'>
What do the following functions do? Why are they useful even though they are so short?
</div>


<div class='answer'>


```r
is_directory <- function(x) file.info(x)$isdir
is_readable <- function(x) file.access(x, 4) == 0
```

The function `is_directory` checks whether the path in `x` is a directory.
The function `is_readable` checks whether the path in `x` is readable, meaning that the file exists and the user has permission to open it.
These functions are useful even though they are short because their names make it much clearer what the code is doing.

</div>

### Exercise 7 {.exercise}

<div class='question'>

Read the complete lyrics to ``Little Bunny Foo Foo''. There’s a lot of duplication in this song. Extend the initial piping example to recreate the complete song, and use functions to reduce the duplication.

</div>

<div class='answer'>
The lyrics of one of the [most common versions](https://en.wikipedia.org/wiki/Little_Bunny_Foo_Foo) of this song are

Little bunny Foo Foo
Hopping through the forest
Scooping up the field mice
And bopping them on the head

Down came the Good Fairy, and she said
"Little bunny Foo Foo
I don't want to see you
Scooping up the field mice

And bopping them on the head.
I'll give you three chances,
And if you don't stop, I'll turn you into a GOON!"
And the next day...

The verses repeat with one chance fewer each time.
When there are no chances left, the Good Fairy says

> "I gave you three chances, and you didn't stop; so...."
> POOF. She turned him into a GOON!
> And the moral of this story is: *hare today, goon tomorrow.*

Here's one way of writing this
```r
threat <- function(chances) {
  give_chances(from = Good_Fairy,
               to = foo_foo,
               number = chances,
               condition = "Don't behave",
               consequence = turn_into_goon)  
}

lyric <- function() {
  foo_foo %>%
    hop(through = forest) %>%
    scoop(up = field_mouse) %>%
    bop(on = head)
  
  down_came(Good_Fairy)
  said(Good_Fairy, 
      c("Little bunny Foo Foo",
        "I don't want to see you",
        "Scooping up the field mice",
        "And bopping them on the head.")
}

lyric()
threat(3)
lyric()
threat(2)
lyric()
threat(1)
lyric()
turn_into_goon(Good_Fairy, foo_foo)
             
```


</div>

## Functions are for humans and computers


### Exercise 1 {.exercise}


<div class='question'>
Read the source code for each of the following three functions, puzzle out what they do, and then brainstorm better names.
</div>


<div class='answer'>


```r
f1 <- function(string, prefix) {
  substr(string, 1, nchar(prefix)) == prefix
}

f2 <- function(x) {
  if (length(x) <= 1) return(NULL)
  x[-length(x)]
}

f3 <- function(x, y) {
  rep(y, length.out = length(x))
}
```

The function `f1` returns whether a function has a common prefix.

```r
f1(c("str_c", "str_foo", "abc"), "str_")
#> [1]  TRUE  TRUE FALSE
```
A better name for `f1` is `has_prefix()`

The function `f2` drops the last element

```r
f2(1:3)
#> [1] 1 2
f2(1:2)
#> [1] 1
f2(1)
#> NULL
```
A better name for `f2` is `drop_last()`.

The function `f3` repeats `y` once for each element of `x`.

```r
f3(1:3, 4)
#> [1] 4 4 4
```
This is a harder one to name. I would say something like `recycle` (R's name for this behavior), or `epxand`.

</div>

### Exercise 2 {.exercise}


<div class='question'>
Take a function that you’ve written recently and spend 5 minutes brainstorming a better name for it and its arguments.
</div>


<div class='answer'>

Answer left to the reader.

</div>

### Exercise 3 {.exercise}


<div class='question'>
Compare and contrast `rnorm()` and `MASS::mvrnorm()`. How could you make them more consistent?
</div>


<div class='answer'>

*You can ignore*

`rnorm` samples from the univariate normal distribution, while `MASS::mvrnorm` samples from the multivariate normal distribution.
The main arguments in `rnorm` are `n`, `mean`, `sd`.
The main arguments is `MASS::mvrnorm` are `n`, `mu`, `Sigma`. 
To be consistent they should have the same names.
However, this is difficult. 
In general, it is better to be consistent with more widely used functions, e.g. `rmvnorm` should follow the conventions of `rnorm`. 
However, while `mean` is correct in the multivariate case, `sd` does not make sense in the multivariate case. 
Both functions an internally consistent though; it would be bad to have `mu` and `sd` or `mean` and `Sigma`.

</div>

### Exercise 4 {.exercise}


<div class='question'>
Make a case for why `norm_r()`, `norm_d()` etc would be better than `rnorm()`, `dnorm()`. Make a case for the opposite.
</div>


<div class='answer'>

If named `norm_r` and `norm_d`, it groups the family of functions related to the normal distribution.
If named `rnorm`, and `dnorm`, functions related to are grouped into families by the action they perform. `r*` functions always sample from distributions: `rnorm`, `rbinom`, `runif`, `rexp`. `d*` functions calculate the probability density or mass of a distribution: `dnorm`, `dbinom`, `dunif`, `dexp`.


</div>

## Conditional execution

### Exercise 1 {.exercise} 


<div class='question'>
What’s the difference between `if` and `ifelse()`? > Carefully read the help and construct three examples that illustrate the key differences.
</div>


<div class='answer'>

The keyword `if` tests a single condition, while `ifelse` tests each element.


</div>

### Exercise 2 {.exercise}


<div class='question'>
Write a greeting function that says “good morning”, “good afternoon”, or “good evening”, depending on the time of day. (Hint: use a time argument that defaults to `lubridate::now()`. That will make it easier to test your function.)
</div>


<div class='answer'>


```r
greet <- function(time = lubridate::now()) {
  hr <- hour(time)
  # I don't know what to do about times after midnight, 
  # are they evening or morning?
  if (hr < 12) {
    print("good morning")
  } else if (hr < 17) {
    print("good afternoon")
  } else {
    print("good evening")
  }
} 
greet()
#> [1] "good evening"
greet(ymd_h("2017-01-08:05"))
#> [1] "good morning"
greet(ymd_h("2017-01-08:13"))
#> [1] "good afternoon"
greet(ymd_h("2017-01-08:20"))
#> [1] "good evening"
```

</div>

### Exercise 3 {.exercise}


<div class='question'>
Implement a `fizzbuzz` function. It takes a single number as input. If the number is divisible by three, it returns “fizz”. If it’s divisible by five it returns “buzz”. If it’s divisible by three and five, it returns “fizzbuzz”. Otherwise, it returns the number. Make sure you first write working code before you create the function.
</div>


<div class='answer'>


```r
fizzbuzz <- function(x) {
  stopifnot(length(x) == 1)
  stopifnot(is.numeric(x))
  # this could be made more efficient by minimizing the
  # number of tests
  if (!(x %% 3) & !(x %% 5)) {
    print("fizzbuzz")
  } else if (!(x %% 3)) {
    print("fizz")
  } else if (!(x %% 5)) {
    print("buzz")
  }
}
fizzbuzz(6)
#> [1] "fizz"
fizzbuzz(10)
#> [1] "buzz"
fizzbuzz(15)
#> [1] "fizzbuzz"
fizzbuzz(2)
```

</div>

### Exercise 4 {.exercise}


<div class='question'>
How could you use `cut()` to simplify this set of nested if-else statements?
</div>


<div class='answer'>


```r
if (temp <= 0) {
  "freezing"
} else if (temp <= 10) {
  "cold"
} else if (temp <= 20) {
  "cool"
} else if (temp <= 30) {
  "warm"
} else {
  "hot"
}
```
How would you change the call to `cut()` if I’d used `<` instead of `<=`? What is the other chief advantage of cut() for this problem? (Hint: what happens if you have many values in temp?)


```r
temp <- seq(-10, 50, by = 5)
cut(temp, c(-Inf, 0, 10, 20, 30, Inf), right = TRUE,
    labels = c("freezing", "cold", "cool", "warm", "hot"))
#>  [1] freezing freezing freezing cold     cold     cool     cool    
#>  [8] warm     warm     hot      hot      hot      hot     
#> Levels: freezing cold cool warm hot
```

To have intervals open on the left (using `<`), I change the argument to `right = FALSE`,

```r
temp <- seq(-10, 50, by = 5)
cut(temp, c(-Inf, 0, 10, 20, 30, Inf), right = FALSE,
    labels = c("freezing", "cold", "cool", "warm", "hot"))
#>  [1] freezing freezing cold     cold     cool     cool     warm    
#>  [8] warm     hot      hot      hot      hot      hot     
#> Levels: freezing cold cool warm hot
```

Two advantages of using `cut` is that it works on vectors, whereas `if` only works on a single value (I already demonstrated this above),
and that to change comparisons I only needed to change the argument to `right`, but I would have had to change four operators in the `if` expression.

</div>

### Exercise 5 {.exercise}


<div class='question'>

What happens if you use `switch()` with numeric values?

</div>


<div class='answer'>

It selects that number argument from `...`.


```r
switch(2, "one", "two", "three")
#> [1] "two"
```


</div>

### Exercise 6 {.exercise}


<div class='question'>
What does this `switch()` call do? What happens if `x` is `"e"`?
</div>


<div class='answer'>

It will return the `"ab"` for `a` or `b`, `"cd"` for `c` or `d`, an `NULL` for `e`. It returns the first non-missing value for the first name it matches.

```r
x <- "e"
switch(x, 
  a = ,
  b = "ab",
  c = ,
  d = "cd"
)
```
Experiment, then carefully read the documentation.


```r
switcheroo <- function(x) {
  switch(x, 
  a = ,
  b = "ab",
  c = ,
  d = "cd"
  )
}
switcheroo("a")
#> [1] "ab"
switcheroo("b")
#> [1] "ab"
switcheroo("c")
#> [1] "cd"
switcheroo("d")
#> [1] "cd"
switcheroo("e")
```

</div>

## Function arguments


### Exercise 1 {.exercise}


<div class='question'>
What does `commas(letters, collapse = "-")` do? Why?
</div>


<div class='answer'>

The `commas` function in the chapter is defined as

```r
commas <- function(...) {
  stringr::str_c(..., collapse = ", ")
}
```

When `commas()` is given a collapse argument, it throws an error.

```r
commas(letters, collapse = "-")
#> Error in stringr::str_c(..., collapse = ", "): formal argument "collapse" matched by multiple actual arguments
```
This is because when the argument `collapse` is given to `commas`, it 
is passed to `str_c` as part of `...`.
In other words, the previous code is equivalent to

```r
str_c(letters, collapse = "-", collapse = ", ")
```
However, it is an error to give the same named argument to a function twice.

One way to allow the user to override the separator in `commas` is to add a `collapse`
argument to the function.

```r
commas <- function(..., collapse = ", ") {
  stringr::str_c(..., collapse = collapse)
}
```



</div>

### Exercise 2 {.exercise}


<div class='question'>

It’d be nice if you could supply multiple characters to the pad argument, e.g. `rule("Title", pad = "-+")`. 
Why doesn’t this currently work? How could you fix it?

</div>


<div class='answer'>


```r
rule <- function(..., pad = "-") {
  title <- paste0(...)
  width <- getOption("width") - nchar(title) - 5
  cat(title, " ", stringr::str_dup(pad, width), "\n", sep = "")
}
```


```r
rule("Important output")
#> Important output ------------------------------------------------------
rule("Important output", pad = "-+")
#> Important output -+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
It does not work because it duplicates pad by the width minus the length of the string.
This is implicitly assuming that pad is only one character.
I could adjust the code to calculate the length of pad.
The trickiest part is handling what to do if width is not a multiple of the number of characters of `pad`.


```r
rule <- function(..., pad = "-") {
  title <- paste0(...)
  width <- getOption("width") - nchar(title) - 5
  padchar <- nchar(pad)
  cat(title, " ",
      stringr::str_dup(pad, width %/% padchar),
      # if not multiple, fill in the remaining characters
      stringr::str_sub(pad, 1, width %% padchar),
      "\n", sep = "")
}
rule("Important output")
#> Important output ------------------------------------------------------
rule("Important output", pad = "-+")
#> Important output -+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
rule("Important output", pad = "-+-")
#> Important output -+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+-
```

</div>

### Exercise 3 {.exercise}


<div class='question'>
What does the `trim` argument to `mean()` do? When might you use it?
</div>


<div class='answer'>

The `trim` arguments trims a fraction of observations from each end of the vector (meaning the range) before calculating the mean.
This is useful for calculating a measure of central tendency that is robust to outliers.

</div>

### Exercise 4 {.exercise}


<div class='question'>
The default value for the `method` argument to `cor()` is `c("pearson", "kendall", "spearman")`. 
What does that mean? What value is used by default?
</div>


<div class='answer'>

It means that the `method` argument can take one of those three values. 
The first value, `"pearson"`, is used by default.

</div>

## Environment 

No Exercises


