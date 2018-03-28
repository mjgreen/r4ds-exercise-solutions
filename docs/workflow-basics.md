
---
output: html_document
editor_options:
  chunk_output_type: console
---
# Workflow Basics

## Coding basics

No exercises

## What's in a name?

No exercises

## Calling functions

No exercises

## Practice

### Exercise 1 {.exercise}

<div class='question'>
Why does this code not work?

```r
my_variable <- 10
my_varıable
#> Error in eval(expr, envir, enclos): object 'my_varıable' not found
```
</div>

The variable being printed is `my_varıable`, not `my_variable`:
the seventh character is "ı" ("LATIN SMALL LETTER DOTLESS I"), not "i".

While it wouldn't have helped much in this case, the importance of distinguishing characters in code is reasons why fonts which clearly distinguish similar characters are preferred in programming: especially important are distinguishing between zero (0), Latin small letter O (o), and Latin capital letter O (O); and the numeral one (1), Latin small letter I (i), Latin capital letter I (i), and Latin small letter L (l).
In these fonts, zero and the Latin letter O are often distinguished by using a glyph for zero that uses either a dot in the interior or a slash through it.

Also note that the error messages of the form "object '...' not found", means what it says: R cannot find an object with that name.
Unfortunately, the error does not tell you why that object cannot be found.
This is usually because you either (1) forgot to define the function (or had an error that prevented it from being defined earlier), (2) didn't load a package with the object, or (3) made a typo in the object's name (either when using it or when you originally defined it).

### Exercise 2 {.exercise}

<div class='question'>
Tweak each of the following R commands so that they run correctly:
</div>


```r
library("tidyverse")

ggplot(dota = mpg) +
  geom_point(mapping = aes(x = displ, y = hwy))
#> Error in structure(list(data = data, layers = list(), scales = scales_list(), : argument "data" is missing, with no default
```
The error message is `argument "data" is missing, with no default`.

It looks like a typo, `dota` instead of `data`.

```r
ggplot(data = mpg) +
  geom_point(mapping = aes(x = displ, y = hwy))
```

<img src="workflow-basics_files/figure-html/unnamed-chunk-4-1.png" width="70%" style="display: block; margin: auto;" />


```r
fliter(mpg, cyl = 8)
#> Error in fliter(mpg, cyl = 8): could not find function "fliter"
```

R could not find the function `fliter` because we made a typo: `fliter` instead of `filter`.


```r
filter(mpg, cyl = 8)
#> Error: `cyl` (`cyl = 8`) must not be named, do you need `==`?
```

We aren't done yet. But the error message gives a suggestion. Let's follow it.


```r
filter(mpg, cyl == 8)
#> # A tibble: 70 x 11
#>   manufacturer model displ  year   cyl trans drv     cty   hwy fl    class
#>   <chr>        <chr> <dbl> <int> <int> <chr> <chr> <int> <int> <chr> <chr>
#> 1 audi         a6 q…  4.20  2008     8 auto… 4        16    23 p     mids…
#> 2 chevrolet    c150…  5.30  2008     8 auto… r        14    20 r     suv  
#> 3 chevrolet    c150…  5.30  2008     8 auto… r        11    15 e     suv  
#> 4 chevrolet    c150…  5.30  2008     8 auto… r        14    20 r     suv  
#> 5 chevrolet    c150…  5.70  1999     8 auto… r        13    17 r     suv  
#> 6 chevrolet    c150…  6.00  2008     8 auto… r        12    17 r     suv  
#> # ... with 64 more rows
```


```r
filter(diamond, carat > 3)
#> Error in filter(diamond, carat > 3): object 'diamond' not found
```

R says it can't find the object `diamond`.
This is a typo; the data frame is named `diamonds`.

```r
filter(diamonds, carat > 3)
#> # A tibble: 32 x 10
#>   carat cut     color clarity depth table price     x     y     z
#>   <dbl> <ord>   <ord> <ord>   <dbl> <dbl> <int> <dbl> <dbl> <dbl>
#> 1  3.01 Premium I     I1       62.7   58.  8040  9.10  8.97  5.67
#> 2  3.11 Fair    J     I1       65.9   57.  9823  9.15  9.02  5.98
#> 3  3.01 Premium F     I1       62.2   56.  9925  9.24  9.13  5.73
#> 4  3.05 Premium E     I1       60.9   58. 10453  9.26  9.25  5.66
#> 5  3.02 Fair    I     I1       65.2   56. 10577  9.11  9.02  5.91
#> 6  3.01 Fair    H     I1       56.1   62. 10761  9.54  9.38  5.31
#> # ... with 26 more rows
```

How did I know? I started typing in `diamond` and RStudio completed it to `diamonds`.
Since `diamonds` includes the variable `carat` and the code works, that appears to have been the problem.

### Exercise 3 {.exercise}

<div class='question'>
Press *Alt + Shift + K*. What happens? How can you get to the same place using the menus?
</div>

This gives a menu with keyboard shortcuts. This can be found in the menu under `Tools -> Keyboard Shortcuts Help`.

