Lab 09
================
Yumeng Gao
2022-10-26

(Rscript –vanilla -e ‘rmarkdown::render(“README.Rmd”, output_format=
“all”)’)

## Problem 2.

1.  Create a n x k matrix of Poisson variables with mean lambda

``` r
set.seed(1235)
fun1 <- function(n = 100, k = 4, lambda = 4) {
  x <- NULL
  
  for (i in 1:n)
    x <- rbind(x, rpois(k, lambda))
  
  return(x)
}

f1=fun1(100,4)
mean(f1)
```

    ## [1] 4.1575

``` r
f1=fun1(1000,4)
#f1=fun1(10000,4)
#f1=fun1(50000,4) #longer time


fun1alt <- function(n = 100, k = 4, lambda = 4) {
  x= matrix( rpois(n*k, lambda) , ncol=4)  
  return(x)
}

# Benchmarking
microbenchmark::microbenchmark(
  fun1(),
  fun1alt()
)
```

    ## Unit: microseconds
    ##       expr     min       lq      mean  median        uq      max neval
    ##     fun1() 559.304 754.1980 967.13002 923.104 1043.1965 2670.241   100
    ##  fun1alt()  25.970  31.8475  79.76895  35.318   40.7435 3711.512   100

``` r
d= matrix(1:16, ncol=4)
d
```

    ##      [,1] [,2] [,3] [,4]
    ## [1,]    1    5    9   13
    ## [2,]    2    6   10   14
    ## [3,]    3    7   11   15
    ## [4,]    4    8   12   16

``` r
diag(d)
```

    ## [1]  1  6 11 16

``` r
d[2]
```

    ## [1] 2

``` r
d[2,1]
```

    ## [1] 2

``` r
d[c(1,6,11,16)]
```

    ## [1]  1  6 11 16

## Problem 3.

Find the column max (hint: Checkout the function max.col()).

``` r
# Data Generating Process (10 x 10,000 matrix)
set.seed(1234)
M <- matrix(runif(12,), ncol=4)
M
```

    ##           [,1]      [,2]        [,3]      [,4]
    ## [1,] 0.1137034 0.6233794 0.009495756 0.5142511
    ## [2,] 0.6222994 0.8609154 0.232550506 0.6935913
    ## [3,] 0.6092747 0.6403106 0.666083758 0.5449748

``` r
# Find each column's max value
fun2 <- function(x) {
  apply(x, 2, max)
  x
}
fun2(M)
```

    ##           [,1]      [,2]        [,3]      [,4]
    ## [1,] 0.1137034 0.6233794 0.009495756 0.5142511
    ## [2,] 0.6222994 0.8609154 0.232550506 0.6935913
    ## [3,] 0.6092747 0.6403106 0.666083758 0.5449748

``` r
fun2alt <- function(x) {
  idx= max.col(t(x))
  x[cbind(idx, 1:4)]
}
fun2alt(M)
```

    ## [1] 0.6222994 0.8609154 0.6660838 0.6935913

``` r
x <- matrix(rnorm(1e4), nrow=10)

# Benchmarking
microbenchmark::microbenchmark(
  fun2(x),
  fun2alt(x)
)
```

    ## Unit: microseconds
    ##        expr      min       lq      mean    median       uq       max neval
    ##     fun2(x) 2173.706 2361.104 3036.7854 2557.5070 3223.862 11522.175   100
    ##  fun2alt(x)  185.396  234.925  338.2975  264.7835  311.168  4440.228   100
