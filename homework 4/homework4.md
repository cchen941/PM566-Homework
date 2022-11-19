Homework4
================
Chen Chen 6381370662
2022-11-18

# HPC

## Problem 1: Make sure your code is nice

Rewrite the following R functions to make them faster. It is OK (and
recommended) to take a look at Stack overflow and Google

``` r
# Total row sums
fun1 <- function(mat) {
  n <- nrow(mat)
  ans <- double(n) 
  for (i in 1:n) {
    ans[i] <- sum(mat[i, ])
  }
  ans
}

fun1alt <- function(mat) {
  rowSums(mat)
}

# Cumulative sum by row
fun2 <- function(mat) {
  n <- nrow(mat)
  k <- ncol(mat)
  ans <- mat
  for (i in 1:n) {
    for (j in 2:k) {
      ans[i,j] <- mat[i, j] + ans[i, j - 1]
    }
  }
  ans
}

fun2alt <- function(mat) {
  t(apply(mat, 1, cumsum))
}


# Use the data with this code
set.seed(2315)
dat <- matrix(rnorm(200 * 100), nrow = 200)

# Test for the first
microbenchmark::microbenchmark(
  fun1(dat),
  fun1alt(dat), check = "equivalent"
)
```

    ## Unit: microseconds
    ##          expr     min      lq      mean  median       uq      max neval
    ##     fun1(dat) 371.174 423.729 501.50522 447.584 485.2745 2107.987   100
    ##  fun1alt(dat)  34.643  37.001  53.03763  39.299  43.1560 1189.215   100

``` r
# Test for the second
microbenchmark::microbenchmark(
  fun2(dat),
  fun2alt(dat), check = "equivalent"
)
```

    ## Unit: microseconds
    ##          expr      min       lq     mean    median       uq       max neval
    ##     fun2(dat) 1985.090 2089.718 2234.487 2108.9185 2304.060  3270.514   100
    ##  fun2alt(dat)  482.337  864.883 1335.431  934.8605 1041.863 19189.499   100

Comparison: It is obvious that the time cost by fun1&2alt is shorter
than cost by fun1&2.

## Problem 2: Make things run faster with parallel computing

The following function allows simulating PI

``` r
sim_pi <- function(n = 1000, i = NULL) {
  p <- matrix(runif(n*2), ncol = 2)
  mean(rowSums(p^2) < 1) * 4
}

# Here is an example of the run
set.seed(156)
sim_pi(1000) # 3.132
```

    ## [1] 3.132

In order to get accurate estimates, we can run this function multiple
times, with the following code:

``` r
# This runs the simulation a 4,000 times, each with 10,000 points
set.seed(1231)
system.time({
  ans <- unlist(lapply(1:4000, sim_pi, n = 10000))
  print(mean(ans))
})
```

    ## [1] 3.14124

    ##    user  system elapsed 
    ##   3.249   1.239   4.513

Rewrite the previous code using parLapply() to make it run faster. Make
sure you set the seed using clusterSetRNGStream():

``` r
library(parallel)
cl<- makePSOCKcluster(4)
clusterSetRNGStream(cl, 1231)
clusterExport(cl, c("sim_pi"), envir=environment())

system.time({
  ans <- unlist(parLapply(cl, 1:4000, sim_pi, n=10000))
  print(mean(ans))
})
```

    ## [1] 3.141578

    ##    user  system elapsed 
    ##   0.003   0.000   1.122

Comparison: It is obvious that the time by code re-written is lesser
than the original one.

# SQL

Setup a temporary database by running the following chunk

``` r
# install.packages(c("RSQLite", "DBI"))
if(!require(RSQLite))install.packages(c("RSQLite"))
```

    ## Loading required package: RSQLite

``` r
if(!require(DBI))install.packages(c("DBI"))
```

    ## Loading required package: DBI

``` r
library(RSQLite)
library(DBI)

# Initialize a temporary in memory database
con <- dbConnect(SQLite(), ":memory:")

# Download tables
film <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film.csv")
film_category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/film_category.csv")
category <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/category.csv")

# Copy data.frames to database
dbWriteTable(con, "film", film)
dbWriteTable(con, "film_category", film_category)
dbWriteTable(con, "category", category)
```

## Question 1

How many many movies is there available in each rating category.

``` sql
PRAGMA table_info(film)
```

| cid | name                 | type    | notnull | dflt_value |  pk |
|:----|:---------------------|:--------|--------:|:-----------|----:|
| 0   | film_id              | INTEGER |       0 | NA         |   0 |
| 1   | title                | TEXT    |       0 | NA         |   0 |
| 2   | description          | TEXT    |       0 | NA         |   0 |
| 3   | release_year         | INTEGER |       0 | NA         |   0 |
| 4   | language_id          | INTEGER |       0 | NA         |   0 |
| 5   | original_language_id | INTEGER |       0 | NA         |   0 |
| 6   | rental_duration      | INTEGER |       0 | NA         |   0 |
| 7   | rental_rate          | REAL    |       0 | NA         |   0 |
| 8   | length               | INTEGER |       0 | NA         |   0 |
| 9   | replacement_cost     | REAL    |       0 | NA         |   0 |

Displaying records 1 - 10

``` sql
SELECT rating,
  COUNT(*) AS movie_count
FROM film
GROUP BY rating
```

| rating | movie_count |
|:-------|------------:|
| G      |         180 |
| NC-17  |         210 |
| PG     |         194 |
| PG-13  |         223 |
| R      |         195 |

5 records

## Question 2

What is the average replacement cost and rental rate for each rating
category.

``` sql
SELECT rating,
       AVG(replacement_cost) AS avg_replacement_cost,
       AVG(rental_rate) AS avg_rental_rate
FROM film
GROUP BY rating
```

| rating | avg_replacement_cost | avg_rental_rate |
|:-------|---------------------:|----------------:|
| G      |             20.12333 |        2.912222 |
| NC-17  |             20.13762 |        2.970952 |
| PG     |             18.95907 |        3.051856 |
| PG-13  |             20.40256 |        3.034843 |
| R      |             20.23103 |        2.938718 |

5 records

## Question 3

Use table film_category together with film to find the how many films
there are with each category ID

``` sql
PRAGMA table_info(film_category)
```

| cid | name        | type    | notnull | dflt_value |  pk |
|:----|:------------|:--------|--------:|:-----------|----:|
| 0   | film_id     | INTEGER |       0 | NA         |   0 |
| 1   | category_id | INTEGER |       0 | NA         |   0 |
| 2   | last_update | TEXT    |       0 | NA         |   0 |

3 records

``` sql
SELECT c.category_id,
       count(*) AS film_count
FROM film_category AS c INNER JOIN film AS f
ON c.film_id = f.film_id
GROUP BY category_id
```

| category_id | film_count |
|:------------|-----------:|
| 1           |         64 |
| 2           |         66 |
| 3           |         60 |
| 4           |         57 |
| 5           |         58 |
| 6           |         68 |
| 7           |         62 |
| 8           |         69 |
| 9           |         73 |
| 10          |         61 |

Displaying records 1 - 10

## Question 4

Incorporate table category into the answer to the previous question to
find the name of the most popular category.

``` sql
PRAGMA table_info(category)
```

| cid | name        | type    | notnull | dflt_value |  pk |
|:----|:------------|:--------|--------:|:-----------|----:|
| 0   | category_id | INTEGER |       0 | NA         |   0 |
| 1   | name        | TEXT    |       0 | NA         |   0 |
| 2   | last_update | TEXT    |       0 | NA         |   0 |

3 records

``` sql
SELECT a.category_id,
       c.name,
       count(*) AS film_count
FROM film_category AS a
INNER JOIN film AS f ON a.film_id = f.film_id
INNER JOIN category AS c ON a.category_id = c.category_id
GROUP BY a.category_id
ORDER BY film_count DESC
```

| category_id | name        | film_count |
|------------:|:------------|-----------:|
|          15 | Sports      |         74 |
|           9 | Foreign     |         73 |
|           8 | Family      |         69 |
|           6 | Documentary |         68 |
|           2 | Animation   |         66 |
|           1 | Action      |         64 |
|          13 | New         |         63 |
|           7 | Drama       |         62 |
|          14 | Sci-Fi      |         61 |
|          10 | Games       |         61 |

Displaying records 1 - 10

From the table above, the most popular category is sports with category
id 15.
