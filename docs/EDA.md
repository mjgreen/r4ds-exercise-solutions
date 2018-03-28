
---
output: html_document
editor_options:
  chunk_output_type: console
---
# Exploratory Data Analysis

## Introduction


```r
library("tidyverse")
library("viridis")
library("forcats")
```
This will also use data from **nycflights13**,

```r
library("nycflights13")
```


## Questions

## Variation

### Exercise 1 {.exercise}


<div class='question'>
Explore the distribution of each of the x, y, and z variables in diamonds. What do you learn? Think about a diamond and how you might decide which dimension is the length, width, and depth.
</div>


<div class='answer'>

In order to make it easier to plot them, I'll reshape the dataset so that I can use the variables as facets.

```r
diamonds %>%
  mutate(id = row_number()) %>%
  select(x, y, z, id) %>%
  gather(variable, value, -id)  %>%
  ggplot(aes(x = value)) +
  geom_density() +
  geom_rug() +
  facet_grid(variable ~ .)
```

<img src="EDA_files/figure-html/unnamed-chunk-4-1.png" width="70%" style="display: block; margin: auto;" />

There several noticeable features of the distributions

1.  They are right skewed, with most diamonds small, but a few very large ones.
2.  There is an outlier in `y`, and `z` (see the rug)
3.  All three distributions have a bimodality (perhaps due to some sort of threshold)

According to the documentation for `diamonds`:
`x` is length, `y` is width, and `z` is depth.
I don't know if I would have figured that out before; maybe if there was data on the type of cuts.


</div>

### Exercise 2.


<div class='question'>
Explore the distribution of price. Do you discover anything unusual or surprising? (Hint: Carefully think about the `binwidth` and make sure you try a wide range of values.)
</div>


<div class='answer'>

-   The price data has many spikes, but I can't tell what each spike corresponds to. The following plots don't show much difference in the distributions in the last one or two digits.
-   There are no diamonds with a price of $1,500
-   There's a bulge in the distribution around $7,500.


```r
ggplot(filter(diamonds, price < 2500), aes(x = price)) +
  geom_histogram(binwidth = 10, center = 0)
```

<img src="EDA_files/figure-html/unnamed-chunk-5-1.png" width="70%" style="display: block; margin: auto;" />


```r
ggplot(filter(diamonds), aes(x = price)) +
  geom_histogram(binwidth = 100, center = 0)
```

<img src="EDA_files/figure-html/unnamed-chunk-6-1.png" width="70%" style="display: block; margin: auto;" />

Distribution of last digit

```r
diamonds %>%
  mutate(ending = price %% 10) %>%
  ggplot(aes(x = ending)) +
  geom_histogram(binwidth = 1, center = 0) +
  geom_bar()
```

<img src="EDA_files/figure-html/unnamed-chunk-7-1.png" width="70%" style="display: block; margin: auto;" />


```r
diamonds %>%
  mutate(ending = price %% 100) %>%
  ggplot(aes(x = ending)) +
  geom_histogram(binwidth = 1) +
  geom_bar()
```

<img src="EDA_files/figure-html/unnamed-chunk-8-1.png" width="70%" style="display: block; margin: auto;" />



```r
diamonds %>%
  mutate(ending = price %% 1000) %>%
  filter(ending >= 500, ending <= 800)  %>%
  ggplot(aes(x = ending)) +
  geom_histogram(binwidth = 1) +
  geom_bar()
```

<img src="EDA_files/figure-html/unnamed-chunk-9-1.png" width="70%" style="display: block; margin: auto;" />

</div>

### Exercise 3.


<div class='question'>
How many diamonds are 0.99 carat? How many are 1 carat? What do you think is the cause of the difference?
</div>


<div class='answer'>

There are more than 70 times as many 1 carat diamonds as 0.99 carat diamond.

```r
diamonds %>%
  filter(carat >= 0.99, carat <= 1) %>%
  count(carat)
#> # A tibble: 2 x 2
#>   carat     n
#>   <dbl> <int>
#> 1 0.990    23
#> 2 1.00   1558
```

I don't know exactly the process behind how carats are measured, but some way or another some diamonds carat values are being "rounded up", because presumably there is a premium for a 1 carat diamond vs. a 0.99 carat diamond beyond the expected increase in price due to a 0.01 carat increase.

To check this intuition, we'd want to look at the number of diamonds in each carat range to seem if there is an abnormally low number at 0.99 carats, and an abnormally high number at 1 carat.


```r
diamonds %>%
   filter(carat >= 0.9, carat <= 1.1) %>%
   count(carat) %>%
   print(n = 30)
#> # A tibble: 21 x 2
#>    carat     n
#>    <dbl> <int>
#>  1 0.900  1485
#>  2 0.910   570
#>  3 0.920   226
#>  4 0.930   142
#>  5 0.940    59
#>  6 0.950    65
#>  7 0.960   103
#>  8 0.970    59
#>  9 0.980    31
#> 10 0.990    23
#> 11 1.00   1558
#> 12 1.01   2242
#> 13 1.02    883
#> 14 1.03    523
#> 15 1.04    475
#> 16 1.05    361
#> 17 1.06    373
#> 18 1.07    342
#> 19 1.08    246
#> 20 1.09    287
#> 21 1.10    278
```

</div>

### Exercise 4 {.exercise}


<div class='question'>
Compare and contrast `coord_cartesian()` vs `xlim()` or `ylim()` when zooming in on a histogram. What happens if you leave `binwidth` unset? What happens if you try and zoom so only half a bar shows?
</div>


<div class='answer'>

`coord_cartesian` simply zooms in on the area specified by the limits. The calculation of the histogram is unaffected.


```r
ggplot(diamonds) +
  geom_histogram(mapping = aes(x = price)) +
  coord_cartesian(xlim = c(100, 5000), ylim = c(0, 3000))
#> `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

<img src="EDA_files/figure-html/unnamed-chunk-12-1.png" width="70%" style="display: block; margin: auto;" />

However, the `xlim` and `ylim` functions first drop any values outside the limits (the `ylim` doesn't matter in this case), then calculates the histogram, and draws the graph with the given limits.


```r
ggplot(diamonds) +
  geom_histogram(mapping = aes(x = price)) +
  xlim(100, 5000) +
  ylim(0, 3000)
#> `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
#> Warning: Removed 14714 rows containing non-finite values (stat_bin).
#> Warning: Removed 5 rows containing missing values (geom_bar).
```

<img src="EDA_files/figure-html/unnamed-chunk-13-1.png" width="70%" style="display: block; margin: auto;" />


</div>

## Missing Values

### Exercise 1 {.exercise}


<div class='question'>
What happens to missing values in a histogram?
What happens to missing values in a bar chart? > Why is there a difference?
</div>


<div class='answer'>

Missing values are removed when the number of observations in each bin are calculated. See the warning message: `Removed 9 rows containing non-finite values (stat_bin)`

```r
diamonds2 <- diamonds %>%
  mutate(y = ifelse(y < 3 | y > 20, NA, y))

ggplot(diamonds2, aes(x = y)) +
  geom_histogram()
#> `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
#> Warning: Removed 9 rows containing non-finite values (stat_bin).
```

<img src="EDA_files/figure-html/unnamed-chunk-14-1.png" width="70%" style="display: block; margin: auto;" />

In `geom_bar`, `NA` is treated as another category. The `x` aesthetic in `geom_bar` requires a discrete (categorical) variable, and missing values act like another category.

```r
diamonds %>%
  mutate(cut = if_else(runif(n()) < 0.1, NA_character_, as.character(cut))) %>%
  ggplot() +
  geom_bar(mapping = aes(x = cut))
```

<img src="EDA_files/figure-html/unnamed-chunk-15-1.png" width="70%" style="display: block; margin: auto;" />

In a histogram, the `x` aesthetic variable needs to be numeric, and `stat_bin` groups the observations by ranges into bins.
Since the numeric value of the `NA` observations is unknown, they cannot be placed in a particular bin, and are dropped.


</div>

### Exercise 2 {.exercise}


<div class='question'>
What does `na.rm = TRUE` do in `mean()` and `sum()`?
</div>


<div class='answer'>

This option removes `NA` values from the vector prior to calculating the mean and sum.


```r
mean(c(0, 1, 2, NA), na.rm = TRUE)
#> [1] 1
sum(c(0, 1, 2, NA), na.rm = TRUE)
#> [1] 3
```

</div>

## Covariation

### A categorical and continuous variable

#### Exercise 1 {.exercise}


<div class='question'>
Use what you've learned to improve the visualization of the departure times
</div>


<div class='answer'>
of canceled vs. non-canceled flights.

Instead of a `freqplot` use a box-plot

```r
nycflights13::flights %>%
  mutate(
    canceled = is.na(dep_time),
    sched_hour = sched_dep_time %/% 100,
    sched_min = sched_dep_time %% 100,
    sched_dep_time = sched_hour + sched_min / 60
  ) %>%
  ggplot() +
    geom_boxplot(mapping = aes(y = sched_dep_time, x = canceled))
```

<img src="EDA_files/figure-html/unnamed-chunk-17-1.png" width="70%" style="display: block; margin: auto;" />

</div>

#### Exercise 2 {.exercise}


<div class='question'>
What variable in the diamonds dataset is most important for predicting the price of a diamond? > How is that variable correlated with
Why does the combination of those two relationships lead to lower quality diamonds being more expensive?
</div>


<div class='answer'>

**TODO** I'm unsure what this question is asking conditional on using only the tools introduced in the book thus far.

</div>

#### Exercise 3 {.exercise}


<div class='question'>
Install the **ggstance** package, and create a horizontal box plot.
How does this compare to using `coord_flip()`?
</div>


<div class='answer'>

Earlier we created a horizontal box plot of the distribution `hwy` by `class`, using `geom_boxplot` and `coord_flip`:   

```r
ggplot(data = mpg) +
  geom_boxplot(mapping = aes(x = reorder(class, hwy, FUN = median), y = hwy)) +
  coord_flip()
```

<img src="EDA_files/figure-html/unnamed-chunk-18-1.png" width="70%" style="display: block; margin: auto;" />

In this case the output looks the same, but in the aesthetics the `x` and `y` are flipped from the previous case.

```r
library("ggstance")
#> 
#> Attaching package: 'ggstance'
#> The following objects are masked from 'package:ggplot2':
#> 
#>     geom_errorbarh, GeomErrorbarh

ggplot(data = mpg) +
  geom_boxploth(mapping = aes(y = reorder(class, hwy, FUN = median), x = hwy))
```

<img src="EDA_files/figure-html/unnamed-chunk-19-1.png" width="70%" style="display: block; margin: auto;" />

</div>

#### Exercise 4 {.exercise}


<div class='question'>
One problem with box plots is that they were developed in an era of much smaller datasets and tend to display a prohibitively large number of ``outlying values''.
One approach to remedy this problem is the letter value plot.
Install the **lvplot** package, and try using `geom_lv()` to display the distribution of price vs cut.
What do you learn?
</div>


<div class='answer'>
How do you interpret the plots?

The boxes of the letter-value plot correspond to many more quantiles.
They are useful for larger datasets because

1.  larger datasets can give precise estimates of quantiles beyond the quartiles, and
2.  in expectation, larger datasets should have more outliers (in absolute numbers).

The letter-value plot is described in:

>  Heike Hofmann, Karen Kafadar, and Hadley Wickham. 2011. "Letter-value plots: Boxplots for large data" <http://vita.had.co.nz/papers/letter-value-plot.pdf>


```r
library("lvplot")
ggplot(diamonds, aes(x = cut, y = price)) +
  geom_lv()
```

<img src="EDA_files/figure-html/unnamed-chunk-20-1.png" width="70%" style="display: block; margin: auto;" />

</div>

#### Exercise 5 {.exercise}


<div class='question'>
Compare and contrast `geom_violin()` with a faceted `geom_histogram()`, or a colored `geom_freqpoly()`.
What are the pros and cons of each method?
</div>


<div class='answer'>

I produce plots for these three methods below. The `geom_freqpoly` is better for look-up: meaning that given a price, it is easy to tell which `cut` has the highest density. However, the overlapping lines makes it difficult to distinguish how the overall distributions relate to each other.
The `geom_violin` and faceted `geom_histogram` have similar strengths and weaknesses.
It is easy to visually distinguish differences in the overall shape of the distributions (skewness, central values, variance, etc).
However, since we can't easily compare the vertical values of the distribution, its difficult to look up which category has the highest density for a given price.
All of these methods depend on tuning parameters to determine the level of smoothness of the distribution.



```r
ggplot(data = diamonds, mapping = aes(x = price, y = ..density..)) +
  geom_freqpoly(mapping = aes(colour = cut), binwidth = 500)
```

<img src="EDA_files/figure-html/unnamed-chunk-21-1.png" width="70%" style="display: block; margin: auto;" />


```r
ggplot(data = diamonds, mapping = aes(x = price)) +
  geom_histogram() +
  facet_wrap(~ cut, ncol = 1, scales = "free_y")
#> `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

<img src="EDA_files/figure-html/unnamed-chunk-22-1.png" width="70%" style="display: block; margin: auto;" />



```r
ggplot(data = diamonds, mapping = aes(x = cut, y = price)) +
  geom_violin() +
  coord_flip()
```

<img src="EDA_files/figure-html/unnamed-chunk-23-1.png" width="70%" style="display: block; margin: auto;" />

The violin plot was first described in

>  Hintze JL, Nelson RD (1998). "Violin Plots: A Box Plot-Density Trace Synergism." The American Statistician, 52(2), 181–184


</div>

#### Exercise 6 {.exercise}


<div class='question'>
If you have a small dataset, it's sometimes useful to use `geom_jitter()` to see the relationship between a continuous and categorical variable.
The **ggbeeswarm** package provides a number of methods similar to `geom_jitter()`.
List them and briefly describe what each one does.
</div>


<div class='answer'>

There are two methods:

-   `geom_quasirandom` that produces plots that resemble something between jitter and violin. There are several different methods that determine exactly how the random location of the points is generated.
-   `geom_beeswarm` creates a shape similar to a violin plot, but by offsetting the points.

I'll use the `mpg`  box plot example since these methods display individual points, they are better suited for smaller datasets.


```r
library("ggbeeswarm")
ggplot(data = mpg) +
  geom_quasirandom(mapping = aes(x = reorder(class, hwy, FUN = median),
                                 y = hwy))
```

<img src="EDA_files/figure-html/unnamed-chunk-24-1.png" width="70%" style="display: block; margin: auto;" />


```r
ggplot(data = mpg) +
  geom_quasirandom(mapping = aes(x = reorder(class, hwy, FUN = median),
                                 y = hwy),
                   method = "tukey")
```

<img src="EDA_files/figure-html/unnamed-chunk-25-1.png" width="70%" style="display: block; margin: auto;" />


```r
ggplot(data = mpg) +
  geom_quasirandom(mapping = aes(x = reorder(class, hwy, FUN = median),
                                 y = hwy),
                   method = "tukeyDense")
```

<img src="EDA_files/figure-html/unnamed-chunk-26-1.png" width="70%" style="display: block; margin: auto;" />


```r
ggplot(data = mpg) +
  geom_quasirandom(mapping = aes(x = reorder(class, hwy, FUN = median),
                                 y = hwy),
                   method = "frowney")
```

<img src="EDA_files/figure-html/unnamed-chunk-27-1.png" width="70%" style="display: block; margin: auto;" />


```r
ggplot(data = mpg) +
  geom_quasirandom(mapping = aes(x = reorder(class, hwy, FUN = median),
                                 y = hwy),
                   method = "smiley")
```

<img src="EDA_files/figure-html/unnamed-chunk-28-1.png" width="70%" style="display: block; margin: auto;" />



```r
ggplot(data = mpg) +
  geom_beeswarm(mapping = aes(x = reorder(class, hwy, FUN = median),
                                 y = hwy))
```

<img src="EDA_files/figure-html/unnamed-chunk-29-1.png" width="70%" style="display: block; margin: auto;" />


</div>

### Two categorical variables

#### Exercise 1 {.exercise}


<div class='question'>
How could you rescale the count dataset above to more clearly show the distribution of cut within color, or color within cut?
</div>


<div class='answer'>

To clearly show the distribution of `cut` within `color`, calculate a new variable `prop` which is the proportion of each cut within a `color`.
This is done using a grouped mutate.

```r
diamonds %>%
  count(color, cut) %>%
  group_by(color) %>%
  mutate(prop = n / sum(n)) %>%
  ggplot(mapping = aes(x = color, y = cut)) +
  geom_tile(mapping = aes(fill = prop)) +
  scale_fill_viridis(limits = c(0, 1))
```

<img src="EDA_files/figure-html/unnamed-chunk-30-1.png" width="70%" style="display: block; margin: auto;" />

Similarly, to scale by the distribution of `color` within `cut`,

```r
diamonds %>%
  count(color, cut) %>%
  group_by(cut) %>%
  mutate(prop = n / sum(n)) %>%
  ggplot(mapping = aes(x = color, y = cut)) +
  geom_tile(mapping = aes(fill = prop)) +
  scale_fill_viridis(limits = c(0, 1))
```

<img src="EDA_files/figure-html/unnamed-chunk-31-1.png" width="70%" style="display: block; margin: auto;" />

I add `limit = c(0, 1)` to put the color scale between (0, 1).
These are the logical boundaries of proportions.
This makes it possible to compare each cell to its actual value, and would improve comparisons across multiple plots.
However, it ends up limiting the colors and makes it harder to compare within the dataset.
However, using the default limits of the minimum and maximum values makes it easier to compare within the dataset the emphasizing relative differences, but harder to compare across datasets.

</div>

#### Exercise 2 {.exercise}


<div class='question'>
Use `geom_tile()` together with **dplyr** to explore how average flight delays vary by destination and month of year.
What makes the plot difficult to read?
How could you improve it?
</div>


<div class='answer'>


```r
flights %>%
  group_by(month, dest) %>%
  summarise(dep_delay = mean(dep_delay, na.rm = TRUE)) %>%
  ggplot(aes(x = factor(month), y = dest, fill = dep_delay)) +
  geom_tile() +
  labs(x = "Month", y = "Destination", fill = "Departure Delay")
```

<img src="EDA_files/figure-html/unnamed-chunk-32-1.png" width="70%" style="display: block; margin: auto;" />

There are several things that could be done to improve it,

-   sort destinations by a meaningful quantity (distance, number of flights, average delay)
-   remove missing values
-   better color scheme (viridis)

How to treat missing values is difficult.
In this case, missing values correspond to airports which don't have regular flights (at least one flight each month) from NYC.
These are likely smaller airports (with higher variance in their average due to fewer observations).


```r
library("viridis")
flights %>%
  group_by(month, dest) %>%
  summarise(dep_delay = mean(dep_delay, na.rm = TRUE)) %>%
  group_by(dest) %>%
  filter(n() == 12) %>%
  ungroup() %>%
  mutate(dest = fct_reorder(dest, dep_delay)) %>%
  ggplot(aes(x = factor(month), y = dest, fill = dep_delay)) +
  geom_tile() +
  scale_fill_viridis() +
  labs(x = "Month", y = "Destination", fill = "Departure Delay")
```

<img src="EDA_files/figure-html/unnamed-chunk-33-1.png" width="70%" style="display: block; margin: auto;" />


</div>

#### Exercise 3 {.exercise}


<div class='question'>
Why is it slightly better to use `aes(x = color, y = cut)` rather than `aes(x = cut, y = color)` in the example above?
</div>


<div class='answer'>

It's usually better to use the categorical variable with a larger number of categories or the longer labels on the y axis.
If at all possible, labels should be horizontal because that is easier to read.

However, switching the order doesn't result in overlapping labels.

```r
diamonds %>%
  count(color, cut) %>%  
  ggplot(mapping = aes(y = color, x = cut)) +
    geom_tile(mapping = aes(fill = n))
```

<img src="EDA_files/figure-html/unnamed-chunk-34-1.png" width="70%" style="display: block; margin: auto;" />

Another justification, for switching the order is that the larger numbers are at the top when `x = color` and `y = cut`, and that lowers the cognitive burden of interpreting the plot.


</div>

### Two continuous variables

#### Exercise 1 {.exercise}


<div class='question'>
Instead of summarizing the conditional distribution with a box plot, you could use a frequency polygon.
What do you need to consider when using `cut_width()` vs `cut_number()`?
How does that impact a visualization of
</div>


<div class='answer'>
the 2d distribution of `carat` and `price`?

When using `cut_width` the number in each bin may be unequal.
The distribution of `carat` is right skewed so there are few diamonds in those bins.

```r
ggplot(data = diamonds,
       mapping = aes(x = price,
                     colour = cut_width(carat, 0.3))) +
  geom_freqpoly()
#> `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

<img src="EDA_files/figure-html/unnamed-chunk-35-1.png" width="70%" style="display: block; margin: auto;" />
Plotting the density instead of counts will make the distributions comparable, although the bins with few observations will still be hard to interpret.

```r
ggplot(data = diamonds,
       mapping = aes(x = price,
                     y = ..density..,
                     colour = cut_width(carat, 0.3))) +
  geom_freqpoly()
#> `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

<img src="EDA_files/figure-html/unnamed-chunk-36-1.png" width="70%" style="display: block; margin: auto;" />
Plotting the density instead of counts will make the distributions comparable, although the bins with few observations will still be hard to interpret.

```r
ggplot(data = diamonds,
       mapping = aes(x = price,
                     colour = cut_number(carat, 10))) +
  geom_freqpoly()
#> `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

<img src="EDA_files/figure-html/unnamed-chunk-37-1.png" width="70%" style="display: block; margin: auto;" />
Since there are equal numbers in each bin, the plot looks the same if density is used for the y aesthetic (although the values are on a different scale).

```r
ggplot(data = diamonds,
       mapping = aes(x = price,
                     y = ..density..,
                     colour = cut_number(carat, 10))) +
  geom_freqpoly()
#> `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

<img src="EDA_files/figure-html/unnamed-chunk-38-1.png" width="70%" style="display: block; margin: auto;" />

</div>

#### Exercise 2 {.exercise}


<div class='question'>
Visualize the distribution of `carat`, partitioned by `price`.
</div>


<div class='answer'>

With a box plot, partitioning into an 10 bins with the same number of observations:

```r
ggplot(diamonds, aes(x = cut_number(price, 10), y = carat)) +
  geom_boxplot() +
  coord_flip() +
  xlab("Price")
```

<img src="EDA_files/figure-html/unnamed-chunk-39-1.png" width="70%" style="display: block; margin: auto;" />
With a box plot, partitioning into an bins of \$2,000 with the width of the box determined by the number of observations. I use `boundary = 0` to ensure the first bin goes from \$0--\$2,000.

```r
ggplot(diamonds, aes(x = cut_width(price, 2000, boundary = 0), y = carat)) +
  geom_boxplot(varwidth = TRUE) +
  coord_flip() +
  xlab("Price")
```

<img src="EDA_files/figure-html/unnamed-chunk-40-1.png" width="70%" style="display: block; margin: auto;" />

</div>

#### Exercise 3 {.exercise}


<div class='question'>
How does the price distribution of very large diamonds compare to small diamonds.
Is it as you expect, or does it surprise you?
</div>


<div class='answer'>

The distribution of very large diamonds is more variable.
I'm not surprised, since I had a very weak prior about diamond prices.
Ex post, I would reason that above a certain size other factors such as cut, clarity, color play more of a role in the price.

</div>

#### Exercise 4 {.exercise}


<div class='question'>
Combine two of the techniques you've learned to visualize the combined distribution of cut, carat, and price.
</div>


<div class='answer'>

There's lots of options to try, so readers may prodocue a variety of solutions.
Here's a couple that I tried.


```r
ggplot(diamonds, aes(x = carat, y = price)) +
  geom_hex() +
  facet_wrap(~ cut, ncol = 1) +
  scale_fill_viridis()
#> Loading required package: methods
```

<img src="EDA_files/figure-html/unnamed-chunk-41-1.png" width="70%" style="display: block; margin: auto;" />


```r
ggplot(diamonds, aes(x = cut_number(carat, 5), y = price, colour = cut)) +
  geom_boxplot()
```

<img src="EDA_files/figure-html/unnamed-chunk-42-1.png" width="70%" style="display: block; margin: auto;" />


```r
ggplot(diamonds, aes(colour = cut_number(carat, 5), y = price, x = cut)) +
  geom_boxplot()
```

<img src="EDA_files/figure-html/unnamed-chunk-43-1.png" width="70%" style="display: block; margin: auto;" />

</div>

#### Exercise 5 {.exercise}


<div class='question'>
Two dimensional plots reveal outliers that are not visible in one dimensional plots.
For example, some points in the plot below have an unusual combination of `x` and `y` values, which makes the points outliers even though their `x` and `y` values appear normal when examined separately.
</div>


<div class='answer'>


```r
ggplot(data = diamonds) +
  geom_point(mapping = aes(x = x, y = y)) +
  coord_cartesian(xlim = c(4, 11), ylim = c(4, 11))
```

<img src="EDA_files/figure-html/unnamed-chunk-44-1.png" width="70%" style="display: block; margin: auto;" />

Why is a scatterplot a better display than a binned plot for this case?

In this case, there is a strong relationship between $x$ and $y$. The outliers in this case are not extreme in either $x$ or $y$.
A binned plot would not reveal these outliers, and may lead us to conclude that the largest value of $x$ was an outlier even though it appears to fit the bivariate pattern well.

The later chapter [Model Basics] discusses fitting models to bivariate data and plotting residuals, which would reveal this outliers.


</div>

## Patterns and models

No exercises

## ggplot2 calls

No exercises

## Learning more

No exercises.

