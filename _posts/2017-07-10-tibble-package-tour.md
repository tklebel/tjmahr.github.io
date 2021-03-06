---
title: A tour of the tibble package 
excerpt: The tidyverse version of dataframes
tags:
  - r
---

Dataframes are used in R to hold tabular data. Think of the prototypical 
spreadsheet or database table: a grid of data arranged into rows and columns. 
That's a dataframe. The [tibble R package](http://tibble.tidyverse.org/)
provides a fresh take on dataframes to fix some longstanding annoyances with
them. For example, printing a large tibble shows just the first 10 rows instead
of the flooding the console with the first 1,000 rows.

In this post, I provide a tour of the tibble package. Because the package 
provides tools for working with tabular data, it also contain some less
well-known helper functions that I would like to advertise in this post. In
particular, I find `add_column()`, `rownames_to_column()` and
`rowid_to_column()` to be useful tools in my work.

**Why the name "tibble"?** Tibbles first appeared in the dplyr package in 
January 2014, but they weren't called "tibbles" yet. dplyr used a subclass 
`tbl_df` for its dataframe objects and they behaved like modern tibbles: Better 
printing, not converting strings to factors, etc. We loved them, and we would 
convert our plain-old dataframes into these `tbl_df`s for these features. 
However, the name tee-bee-ell-dee-eff is quite a mouthful. On Twitter,
[@JennyBryan raised the question of how to pronounce
`tbl_df`](https://twitter.com/JennyBryan/status/514652585488576512), and
[@kevin_ushey suggested "tibble 
diff"](https://twitter.com/kevin_ushey/status/514659005621219329). The name was 
enthusiastically received.
{: .notice--info}


## Creating tibbles

Create a fresh tibble using `tibble()` and vectors of values for each column.
The column definitions are evaluated sequentially, so additional columns can be 
created by manipulating earlier defined ones. Below `x` is defined and then the 
values of `x` are manipulated to create the column `x_squared`.


```r
library(tibble)
library(magrittr)

tibble(x = 1:5, x_squared = x ^ 2)
#> # A tibble: 5 x 2
#>       x x_squared
#>   <int>     <dbl>
#> 1     1         1
#> 2     2         4
#> 3     3         9
#> 4     4        16
#> 5     5        25
```

Note that this sequential evaluation does not work on classical dataframes.


```r
data.frame(x = 1:5, x_squared = x ^ 2)
#> Error in data.frame(x = 1:5, x_squared = x^2): object 'x' not found
```

The function `data_frame()`---note the underscore instead of a dot---is an alias
for `tibble()`, which might be more transparent if your audience has never 
heard of tibbles.


```r
data_frame(x = 1:5, x_squared = x ^ 2)
#> # A tibble: 5 x 2
#>       x x_squared
#>   <int>     <dbl>
#> 1     1         1
#> 2     2         4
#> 3     3         9
#> 4     4        16
#> 5     5        25
```

In `tibble()`, the data are defined column-by-column. We can use `tribble()` to 
write out tibbles row-by-row. Formulas like `~x` are used to denote column names.


```r
tribble(
  ~ Film, ~ Year,
  "A New Hope", 1977,
  "The Empire Strikes Back", 1980,
  "Return of the Jedi", 1983)
#> # A tibble: 3 x 2
#>                      Film  Year
#>                     <chr> <dbl>
#> 1              A New Hope  1977
#> 2 The Empire Strikes Back  1980
#> 3      Return of the Jedi  1983
```

The name "tribble" is short for "transposed tibble" (the _transposed_
part referring to change from column-wise creation in `tibble()` to row-wise
creation in `tribble()`).

I like to use light-weight tribbles for two particular tasks:

* Recoding: Create a tribble of, say, labels for a plot and join it onto a dataset.
* Exclusion: Identify observations to exclude, and remove them with an anti-join.

Pretend that we have a tibble called `dataset`. The code below shows
examples of these tasks with `dataset`.


```r
library(dplyr)

# Recoding example
plotting_labels <- tribble(
  ~ Group, ~ GroupLabel,
  "TD", "Typically Developing",
  "CI", "Cochlear Implant",
  "ASD", "Autism Spectrum"
)

# Attach labels to dataset
dataset <- left_join(dataset, plotting_labels, by = "Group")

# Exclusion example
ids_to_exclude <- tibble::tribble(
  ~ Study, ~ ResearchID,
  "TimePoint1", "053L",
  "TimePoint1", "102L",
  "TimePoint1", "116L"
)

reduced_dataset <- anti_join(dataset, ids_to_exclude)
```


## Converting things into tibbles

`as_tibble()` will convert dataframes, matrices, and some other types into
tibbles.


```r
as_tibble(mtcars)
#> # A tibble: 32 x 11
#>      mpg   cyl  disp    hp  drat    wt  qsec    vs    am  gear  carb
#>  * <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#>  1  21.0     6 160.0   110  3.90 2.620 16.46     0     1     4     4
#>  2  21.0     6 160.0   110  3.90 2.875 17.02     0     1     4     4
#>  3  22.8     4 108.0    93  3.85 2.320 18.61     1     1     4     1
#>  4  21.4     6 258.0   110  3.08 3.215 19.44     1     0     3     1
#>  5  18.7     8 360.0   175  3.15 3.440 17.02     0     0     3     2
#>  6  18.1     6 225.0   105  2.76 3.460 20.22     1     0     3     1
#>  7  14.3     8 360.0   245  3.21 3.570 15.84     0     0     3     4
#>  8  24.4     4 146.7    62  3.69 3.190 20.00     1     0     4     2
#>  9  22.8     4 140.8    95  3.92 3.150 22.90     1     0     4     2
#> 10  19.2     6 167.6   123  3.92 3.440 18.30     1     0     4     4
#> # ... with 22 more rows
```

We can convert simple named vectors into tibbles with `enframe()`. 
For example, `quantile()` returns a named vector which we can `enframe()`.


```r
quantiles <- quantile(mtcars$hp, probs = c(.1, .25, .5, .75, .9))
quantiles
#>   10%   25%   50%   75%   90% 
#>  66.0  96.5 123.0 180.0 243.5

enframe(quantiles, "quantile", "value")
#> # A tibble: 5 x 2
#>   quantile value
#>      <chr> <dbl>
#> 1      10%  66.0
#> 2      25%  96.5
#> 3      50% 123.0
#> 4      75% 180.0
#> 5      90% 243.5
```

I have not had an opportunity to use `enframe()` since I learned about it,
but I definitely have created dataframes from name-value pairs in the past.

It's also worth noting the most common way I create tibbles: Reading in files.
The [readr](http://readr.tidyverse.org/) package will create tibbles when 
reading in data files like csvs.



## Viewing some values from each column

When we `print()` a tibble, we only see enough dataframe columns to fill the
width of the console. For example, we will not see every column in this
tibble.


```r
# Create a 200 x 26 dataframe
df <- as.data.frame(replicate(26, 1:200)) %>% 
  setNames(letters) %>% 
  as_tibble()

df
#> # A tibble: 200 x 26
#>        a     b     c     d     e     f     g     h     i     j     k     l
#>    <int> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int> <int>
#>  1     1     1     1     1     1     1     1     1     1     1     1     1
#>  2     2     2     2     2     2     2     2     2     2     2     2     2
#>  3     3     3     3     3     3     3     3     3     3     3     3     3
#>  4     4     4     4     4     4     4     4     4     4     4     4     4
#>  5     5     5     5     5     5     5     5     5     5     5     5     5
#>  6     6     6     6     6     6     6     6     6     6     6     6     6
#>  7     7     7     7     7     7     7     7     7     7     7     7     7
#>  8     8     8     8     8     8     8     8     8     8     8     8     8
#>  9     9     9     9     9     9     9     9     9     9     9     9     9
#> 10    10    10    10    10    10    10    10    10    10    10    10    10
#> # ... with 190 more rows, and 14 more variables: m <int>, n <int>,
#> #   o <int>, p <int>, q <int>, r <int>, s <int>, t <int>, u <int>,
#> #   v <int>, w <int>, x <int>, y <int>, z <int>
```

We can transpose the printing with `glimpse()` to see a few values from every 
column. Once again, just enough data is shown to fill the width of the output
console.


```r
glimpse(df)
#> Observations: 200
#> Variables: 26
#> $ a <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ b <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ c <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ d <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ e <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ f <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ g <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ h <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ i <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ j <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ k <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ l <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ m <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ n <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ o <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ p <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ q <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ r <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ s <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ t <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ u <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ v <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ w <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ x <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ y <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
#> $ z <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 1...
```



## Growing a tibble

We can add new rows and columns with `add_row()` and `add_column()`.

Below we add rows to the bottom of the tibble (the default behavior) and to the 
top of the tibble by using the `.before` argument (add the new row _before_ row 1). 
There also is an `.after` argument, but I prefer to only add rows to the
tops and bottoms of tables. The values in the `add_row()` are computed
iteratively, so we can define the values of `x_squared` in terms of `x`.


```r
df <- tibble(comment = "original", x = 1:2, x_squared = x ^ 2)
df
#> # A tibble: 2 x 3
#>    comment     x x_squared
#>      <chr> <int>     <dbl>
#> 1 original     1         1
#> 2 original     2         4

df <- df %>% 
  add_row(comment = "append", x = 3:4, x_squared = x ^ 2) %>% 
  add_row(comment = "prepend", x = 0, x_squared = x ^ 2, .before = 1)
df
#> # A tibble: 5 x 3
#>    comment     x x_squared
#>      <chr> <dbl>     <dbl>
#> 1  prepend     0         0
#> 2 original     1         1
#> 3 original     2         4
#> 4   append     3         9
#> 5   append     4        16
```

The value `NA` is used when values are not provided for a certain column.
Also, because we provide the names of the columns when adding rows, we
don't have to write out the columns in any particular order.


```r
df %>% 
  add_row(x = 5, comment = "NA defaults") %>% 
  add_row(x_squared = 36, x = 6, comment = "order doesn't matter")
#> # A tibble: 7 x 3
#>                comment     x x_squared
#>                  <chr> <dbl>     <dbl>
#> 1              prepend     0         0
#> 2             original     1         1
#> 3             original     2         4
#> 4               append     3         9
#> 5               append     4        16
#> 6          NA defaults     5        NA
#> 7 order doesn't matter     6        36
```

We can similarly add columns with `add_column()`.


```r
df %>% add_column(comment2 = "inserted column", .after = "comment")
#> # A tibble: 5 x 4
#>    comment        comment2     x x_squared
#>      <chr>           <chr> <dbl>     <dbl>
#> 1  prepend inserted column     0         0
#> 2 original inserted column     1         1
#> 3 original inserted column     2         4
#> 4   append inserted column     3         9
#> 5   append inserted column     4        16
```

Typically, with dplyr loaded, you would create new columns by using `mutate()`,
although I have recently started to prefer using `add_column()` for cases like
the above example, where I add a column with a single recycled value. 

## Row names and identifiers

Look at the converted `mtcars` tibble again.


```r
as_tibble(mtcars)
#> # A tibble: 32 x 11
#>      mpg   cyl  disp    hp  drat    wt  qsec    vs    am  gear  carb
#>  * <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#>  1  21.0     6 160.0   110  3.90 2.620 16.46     0     1     4     4
#>  2  21.0     6 160.0   110  3.90 2.875 17.02     0     1     4     4
#>  3  22.8     4 108.0    93  3.85 2.320 18.61     1     1     4     1
#>  4  21.4     6 258.0   110  3.08 3.215 19.44     1     0     3     1
#>  5  18.7     8 360.0   175  3.15 3.440 17.02     0     0     3     2
#>  6  18.1     6 225.0   105  2.76 3.460 20.22     1     0     3     1
#>  7  14.3     8 360.0   245  3.21 3.570 15.84     0     0     3     4
#>  8  24.4     4 146.7    62  3.69 3.190 20.00     1     0     4     2
#>  9  22.8     4 140.8    95  3.92 3.150 22.90     1     0     4     2
#> 10  19.2     6 167.6   123  3.92 3.440 18.30     1     0     4     4
#> # ... with 22 more rows
```

The row numbers in the converted dataframe have an asterisk `*` above them. That
means that the dataframe has row-names. Row-names are clunky and quirky; they are 
just a column of data (labels) that umm :confused: we store away from the rest
of the data. 

We should move those row-names into an explicit column, and
`rownames_to_column()` does just that.


```r
mtcars %>% 
  as_tibble() %>% 
  rownames_to_column("model")
#> # A tibble: 32 x 12
#>                model   mpg   cyl  disp    hp  drat    wt  qsec    vs    am
#>                <chr> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
#>  1         Mazda RX4  21.0     6 160.0   110  3.90 2.620 16.46     0     1
#>  2     Mazda RX4 Wag  21.0     6 160.0   110  3.90 2.875 17.02     0     1
#>  3        Datsun 710  22.8     4 108.0    93  3.85 2.320 18.61     1     1
#>  4    Hornet 4 Drive  21.4     6 258.0   110  3.08 3.215 19.44     1     0
#>  5 Hornet Sportabout  18.7     8 360.0   175  3.15 3.440 17.02     0     0
#>  6           Valiant  18.1     6 225.0   105  2.76 3.460 20.22     1     0
#>  7        Duster 360  14.3     8 360.0   245  3.21 3.570 15.84     0     0
#>  8         Merc 240D  24.4     4 146.7    62  3.69 3.190 20.00     1     0
#>  9          Merc 230  22.8     4 140.8    95  3.92 3.150 22.90     1     0
#> 10          Merc 280  19.2     6 167.6   123  3.92 3.440 18.30     1     0
#> # ... with 22 more rows, and 2 more variables: gear <dbl>, carb <dbl>
```

When I fit Bayesian models, I end up with a bunch of samples from a posterior 
distribution. In my data-tidying, I need to assign a ID-number to each sample. 
The function `rowid_to_column()` automates this step by creating a new column in
a dataframe with the row-numbers. In the example below, I load some MCMC samples
from the coda package and create draw IDs.


```r
library(coda)
data(line, package = "coda")
line1 <- as.matrix(line$line1) %>% 
  as_tibble()
line1
#> # A tibble: 200 x 3
#>      alpha      beta     sigma
#>      <dbl>     <dbl>     <dbl>
#>  1 7.17313 -1.566200 11.233100
#>  2 2.95253  1.503370  4.886490
#>  3 3.66989  0.628157  1.397340
#>  4 3.31522  1.182720  0.662879
#>  5 3.70544  0.490437  1.362130
#>  6 3.57910  0.206970  1.043500
#>  7 2.70206  0.882553  1.290430
#>  8 2.96136  1.085150  0.459322
#>  9 3.53406  1.069260  0.634257
#> 10 2.09471  1.480770  0.912919
#> # ... with 190 more rows

line1 %>% rowid_to_column("draw")
#> # A tibble: 200 x 4
#>     draw   alpha      beta     sigma
#>    <int>   <dbl>     <dbl>     <dbl>
#>  1     1 7.17313 -1.566200 11.233100
#>  2     2 2.95253  1.503370  4.886490
#>  3     3 3.66989  0.628157  1.397340
#>  4     4 3.31522  1.182720  0.662879
#>  5     5 3.70544  0.490437  1.362130
#>  6     6 3.57910  0.206970  1.043500
#>  7     7 2.70206  0.882553  1.290430
#>  8     8 2.96136  1.085150  0.459322
#>  9     9 3.53406  1.069260  0.634257
#> 10    10 2.09471  1.480770  0.912919
#> # ... with 190 more rows
```

From here, I could reshape the data into a long format or draw some random
samples for use in a plot, all while preserving the draw number.

*** 

...And that covers the main functionality of the tibble package. I hope you
discovered a new useful feature of the tibble package. To learn more about the
technical differences between tibbles and dataframes, see [the tibble chapter in
_R for Data Science_](http://r4ds.had.co.nz/tibbles.html).
