Lab 10
================
Yumeng Gao
2022-11-02

## Data Setup

``` r
# install.packages(c("RSQLite", "DBI"))
if(!require(RSQsite)) install.packages("RSQLite")
```

    ## Loading required package: RSQsite

    ## Warning in library(package, lib.loc = lib.loc, character.only = TRUE,
    ## logical.return = TRUE, : there is no package called 'RSQsite'

    ## 
    ## The downloaded binary packages are in
    ##  /var/folders/dk/hm0h1jcj2z98d3zw87pnp68m0000gn/T//RtmpxfgmOD/downloaded_packages

``` r
if(!require(DBI)) install.packages("DBI")
```

    ## Loading required package: DBI

``` r
library(RSQLite)
library(DBI)
```

``` r
# Initialize a temporary in memory database
con <- dbConnect(SQLite(), ":memory:")

# Download tables
actor <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/actor.csv")
rental <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/rental.csv")
customer <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/customer.csv")
payment <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/payment_p2007_01.csv")

# Copy data.frames to database
dbWriteTable(con, "actor", actor)
dbWriteTable(con, "rental", rental)
dbWriteTable(con, "customer", customer)
dbWriteTable(con, "payment", payment)

dbListTables(con)
```

    ## [1] "actor"    "customer" "payment"  "rental"

-   sql commands (similar to str() in r)

``` sql
PRAGMA table_info(actor)
```

``` r
x1
```

    ##   cid        name    type notnull dflt_value pk
    ## 1   0    actor_id INTEGER       0         NA  0
    ## 2   1  first_name    TEXT       0         NA  0
    ## 3   2   last_name    TEXT       0         NA  0
    ## 4   3 last_update    TEXT       0         NA  0

-   do sql from r

``` r
dbGetQuery(con,"
           PRAGMA table_info(actor)
           "
)
```

    ##   cid        name    type notnull dflt_value pk
    ## 1   0    actor_id INTEGER       0         NA  0
    ## 2   1  first_name    TEXT       0         NA  0
    ## 3   2   last_name    TEXT       0         NA  0
    ## 4   3 last_update    TEXT       0         NA  0

# Exercise 1

Retrive the actor ID, first name and last name for all actors using the
actor table. Sort by last name and then by first name.

``` sql
SELECT actor_id, first_name, last_name
FROM actor
ORDER by last_name, first_name
LIMIT 15
```

| actor_id | first_name | last_name |
|---------:|:-----------|:----------|
|       58 | CHRISTIAN  | AKROYD    |
|      182 | DEBBIE     | AKROYD    |
|       92 | KIRSTEN    | AKROYD    |
|      118 | CUBA       | ALLEN     |
|      145 | KIM        | ALLEN     |
|      194 | MERYL      | ALLEN     |
|       76 | ANGELINA   | ASTAIRE   |
|      112 | RUSSELL    | BACALL    |
|      190 | AUDREY     | BAILEY    |
|       67 | JESSICA    | BAILEY    |

Displaying records 1 - 10

``` r
dbGetQuery(con,"
SELECT actor_id, first_name, last_name
FROM actor
ORDER by last_name, first_name
LIMIT 15
")
```

    ##    actor_id first_name last_name
    ## 1        58  CHRISTIAN    AKROYD
    ## 2       182     DEBBIE    AKROYD
    ## 3        92    KIRSTEN    AKROYD
    ## 4       118       CUBA     ALLEN
    ## 5       145        KIM     ALLEN
    ## 6       194      MERYL     ALLEN
    ## 7        76   ANGELINA   ASTAIRE
    ## 8       112    RUSSELL    BACALL
    ## 9       190     AUDREY    BAILEY
    ## 10       67    JESSICA    BAILEY
    ## 11      115   HARRISON      BALE
    ## 12      187      RENEE      BALL
    ## 13       47      JULIA BARRYMORE
    ## 14      158     VIVIEN  BASINGER
    ## 15      174    MICHAEL    BENING

# Exercise 2

Retrive the actor ID, first name, and last name for actors whose last
name equals ‘WILLIAMS’ or ‘DAVIS’.

``` r
dbGetQuery(con,"
SELECT actor_id, first_name, last_name
FROM actor 
WHERE last_name IN ('WILLIAMS', 'DAVIS')
ORDER BY last_name
")
```

    ##   actor_id first_name last_name
    ## 1        4   JENNIFER     DAVIS
    ## 2      101      SUSAN     DAVIS
    ## 3      110      SUSAN     DAVIS
    ## 4       72       SEAN  WILLIAMS
    ## 5      137     MORGAN  WILLIAMS
    ## 6      172    GROUCHO  WILLIAMS

# Exercise 3

Write a query against the rental table that returns the IDs of the
customers who rented a film on July 5, 2005 (use the rental.rental_date
column, and you can use the date() function to ignore the time
component). Include a single row for each distinct customer ID.

``` r
dbGetQuery(con,"
           PRAGMA table_info(rental)
           "
)
```

    ##   cid         name    type notnull dflt_value pk
    ## 1   0    rental_id INTEGER       0         NA  0
    ## 2   1  rental_date    TEXT       0         NA  0
    ## 3   2 inventory_id INTEGER       0         NA  0
    ## 4   3  customer_id INTEGER       0         NA  0
    ## 5   4  return_date    TEXT       0         NA  0
    ## 6   5     staff_id INTEGER       0         NA  0
    ## 7   6  last_update    TEXT       0         NA  0

``` r
dbGetQuery(con,"
SELECT DISTINCT customer_id, rental_date
FROM rental
WHERE date(rental_date) = '2005-07-05'
")
```

    ##    customer_id         rental_date
    ## 1          565 2005-07-05 22:49:24
    ## 2          242 2005-07-05 22:51:44
    ## 3           37 2005-07-05 22:56:33
    ## 4           60 2005-07-05 22:57:34
    ## 5          594 2005-07-05 22:59:53
    ## 6            8 2005-07-05 23:01:21
    ## 7          490 2005-07-05 23:02:37
    ## 8          476 2005-07-05 23:05:17
    ## 9          322 2005-07-05 23:05:44
    ## 10         298 2005-07-05 23:08:53
    ## 11         382 2005-07-05 23:11:43
    ## 12         138 2005-07-05 23:13:07
    ## 13         520 2005-07-05 23:13:22
    ## 14         536 2005-07-05 23:13:51
    ## 15         114 2005-07-05 23:23:11
    ## 16         111 2005-07-05 23:25:54
    ## 17         296 2005-07-05 23:29:55
    ## 18         586 2005-07-05 23:30:36
    ## 19         349 2005-07-05 23:32:49
    ## 20         397 2005-07-05 23:33:40
    ## 21         369 2005-07-05 23:37:13
    ## 22         421 2005-07-05 23:41:08
    ## 23         142 2005-07-05 23:44:37
    ## 24         169 2005-07-05 23:46:19
    ## 25         348 2005-07-05 23:47:30
    ## 26         553 2005-07-05 23:50:04
    ## 27         295 2005-07-05 23:59:15

``` r
dbGetQuery(con,"
SELECT DISTINCT customer_id,
COUNT (*) AS N
FROM rental
WHERE date(rental_date) = '2005-07-05'
GROUP BY customer_id
")
```

    ##    customer_id N
    ## 1            8 1
    ## 2           37 1
    ## 3           60 1
    ## 4          111 1
    ## 5          114 1
    ## 6          138 1
    ## 7          142 1
    ## 8          169 1
    ## 9          242 1
    ## 10         295 1
    ## 11         296 1
    ## 12         298 1
    ## 13         322 1
    ## 14         348 1
    ## 15         349 1
    ## 16         369 1
    ## 17         382 1
    ## 18         397 1
    ## 19         421 1
    ## 20         476 1
    ## 21         490 1
    ## 22         520 1
    ## 23         536 1
    ## 24         553 1
    ## 25         565 1
    ## 26         586 1
    ## 27         594 1

\#Exercise 4

## Exercise 4.1

Construct a query that retrives all rows from the payment table where
the amount is either 1.99, 7.99, 9.99.

``` r
dbGetQuery(con,"
           PRAGMA table_info(payment)
           "
)
```

    ##   cid         name    type notnull dflt_value pk
    ## 1   0   payment_id INTEGER       0         NA  0
    ## 2   1  customer_id INTEGER       0         NA  0
    ## 3   2     staff_id INTEGER       0         NA  0
    ## 4   3    rental_id INTEGER       0         NA  0
    ## 5   4       amount    REAL       0         NA  0
    ## 6   5 payment_date    TEXT       0         NA  0

``` r
dbGetQuery(con,"
SELECT *
FROM payment
WHERE amount IN (1.99, 7.99, 9.99)
LIMIT 15
")
```

    ##    payment_id customer_id staff_id rental_id amount               payment_date
    ## 1       16050         269        2         7   1.99 2007-01-24 21:40:19.996577
    ## 2       16056         270        1       193   1.99 2007-01-26 05:10:14.996577
    ## 3       16081         282        2        48   1.99 2007-01-25 04:49:12.996577
    ## 4       16103         294        1       595   1.99 2007-01-28 12:28:20.996577
    ## 5       16133         307        1       614   1.99 2007-01-28 14:01:54.996577
    ## 6       16158         316        1      1065   1.99 2007-01-31 07:23:22.996577
    ## 7       16160         318        1       224   9.99 2007-01-26 08:46:53.996577
    ## 8       16161         319        1        15   9.99 2007-01-24 23:07:48.996577
    ## 9       16180         330        2       967   7.99 2007-01-30 17:40:32.996577
    ## 10      16206         351        1      1137   1.99 2007-01-31 17:48:40.996577
    ## 11      16210         354        2       158   1.99 2007-01-25 23:55:37.996577
    ## 12      16240         369        2       913   7.99 2007-01-30 09:33:24.996577
    ## 13      16275         386        1       583   7.99 2007-01-28 10:17:21.996577
    ## 14      16277         387        1       697   7.99 2007-01-29 00:32:30.996577
    ## 15      16289         391        1       891   7.99 2007-01-30 06:11:38.996577

## Exercise 4.2

Construct a query that retrives all rows from the payment table where
the amount is greater then 5

``` r
dbGetQuery(con,"
SELECT *
FROM payment
WHERE amount >5
LIMIT 15
")
```

    ##    payment_id customer_id staff_id rental_id amount               payment_date
    ## 1       16052         269        2       678   6.99 2007-01-28 21:44:14.996577
    ## 2       16058         271        1      1096   8.99 2007-01-31 11:59:15.996577
    ## 3       16060         272        1       405   6.99 2007-01-27 12:01:05.996577
    ## 4       16061         272        1      1041   6.99 2007-01-31 04:14:49.996577
    ## 5       16068         274        1       394   5.99 2007-01-27 09:54:37.996577
    ## 6       16073         276        1       860  10.99 2007-01-30 01:13:42.996577
    ## 7       16074         277        2       308   6.99 2007-01-26 20:30:05.996577
    ## 8       16082         282        2       282   6.99 2007-01-26 17:24:52.996577
    ## 9       16086         284        1      1145   6.99 2007-01-31 18:42:11.996577
    ## 10      16087         286        2        81   6.99 2007-01-25 10:43:45.996577
    ## 11      16092         288        2       427   6.99 2007-01-27 14:38:30.996577
    ## 12      16094         288        2       565   5.99 2007-01-28 07:54:57.996577
    ## 13      16102         293        2      1034   8.99 2007-01-31 03:22:06.996577
    ## 14      16106         296        1       511   5.99 2007-01-28 01:32:30.996577
    ## 15      16112         299        1       332   5.99 2007-01-27 00:55:36.996577

## Exercise 4.3

Construct a query that retrives all rows from the payment table where
the amount is greater then 5 and less then 8

``` r
dbGetQuery(con,"
SELECT *
FROM payment
WHERE amount >5 AND amount <8
LIMIT 15
")
```

    ##    payment_id customer_id staff_id rental_id amount               payment_date
    ## 1       16052         269        2       678   6.99 2007-01-28 21:44:14.996577
    ## 2       16060         272        1       405   6.99 2007-01-27 12:01:05.996577
    ## 3       16061         272        1      1041   6.99 2007-01-31 04:14:49.996577
    ## 4       16068         274        1       394   5.99 2007-01-27 09:54:37.996577
    ## 5       16074         277        2       308   6.99 2007-01-26 20:30:05.996577
    ## 6       16082         282        2       282   6.99 2007-01-26 17:24:52.996577
    ## 7       16086         284        1      1145   6.99 2007-01-31 18:42:11.996577
    ## 8       16087         286        2        81   6.99 2007-01-25 10:43:45.996577
    ## 9       16092         288        2       427   6.99 2007-01-27 14:38:30.996577
    ## 10      16094         288        2       565   5.99 2007-01-28 07:54:57.996577
    ## 11      16106         296        1       511   5.99 2007-01-28 01:32:30.996577
    ## 12      16112         299        1       332   5.99 2007-01-27 00:55:36.996577
    ## 13      16118         301        2       227   5.99 2007-01-26 09:20:12.996577
    ## 14      16121         302        2        92   5.99 2007-01-25 14:07:12.996577
    ## 15      16130         306        2       672   6.99 2007-01-28 20:33:55.996577

# Exercise 5

Retrive all the payment IDs and their amount from the customers whose
last name is ‘DAVIS’.

``` r
dbGetQuery(con,"
           PRAGMA table_info(customer)
           "
)
```

    ##    cid        name    type notnull dflt_value pk
    ## 1    0 customer_id INTEGER       0         NA  0
    ## 2    1    store_id INTEGER       0         NA  0
    ## 3    2  first_name    TEXT       0         NA  0
    ## 4    3   last_name    TEXT       0         NA  0
    ## 5    4       email    TEXT       0         NA  0
    ## 6    5  address_id INTEGER       0         NA  0
    ## 7    6  activebool    TEXT       0         NA  0
    ## 8    7 create_date    TEXT       0         NA  0
    ## 9    8 last_update    TEXT       0         NA  0
    ## 10   9      active INTEGER       0         NA  0

``` r
dbGetQuery(con,"
SELECT c.customer_id, c.last_name, p.payment_id, p.amount
FROM customer AS c INNER JOIN payment AS p
  ON c.customer_id = p.customer_id
  WHERE c.last_name IN ('DAVIS')
/* WHERE c.last_name == 'DAVIS' */  /* This is a comment */
")
```

    ##   customer_id last_name payment_id amount
    ## 1           6     DAVIS      16685   4.99
    ## 2           6     DAVIS      16686   2.99
    ## 3           6     DAVIS      16687   0.99

# Exercise 6

## Exercise 6.1

Use COUNT(\*) to count the number of rows in rental

``` r
dbGetQuery(con,"
SELECT *
FROM payment
WHERE amount >5 AND amount <8
LIMIT 15
")
```

    ##    payment_id customer_id staff_id rental_id amount               payment_date
    ## 1       16052         269        2       678   6.99 2007-01-28 21:44:14.996577
    ## 2       16060         272        1       405   6.99 2007-01-27 12:01:05.996577
    ## 3       16061         272        1      1041   6.99 2007-01-31 04:14:49.996577
    ## 4       16068         274        1       394   5.99 2007-01-27 09:54:37.996577
    ## 5       16074         277        2       308   6.99 2007-01-26 20:30:05.996577
    ## 6       16082         282        2       282   6.99 2007-01-26 17:24:52.996577
    ## 7       16086         284        1      1145   6.99 2007-01-31 18:42:11.996577
    ## 8       16087         286        2        81   6.99 2007-01-25 10:43:45.996577
    ## 9       16092         288        2       427   6.99 2007-01-27 14:38:30.996577
    ## 10      16094         288        2       565   5.99 2007-01-28 07:54:57.996577
    ## 11      16106         296        1       511   5.99 2007-01-28 01:32:30.996577
    ## 12      16112         299        1       332   5.99 2007-01-27 00:55:36.996577
    ## 13      16118         301        2       227   5.99 2007-01-26 09:20:12.996577
    ## 14      16121         302        2        92   5.99 2007-01-25 14:07:12.996577
    ## 15      16130         306        2       672   6.99 2007-01-28 20:33:55.996577

## Exercise 6.2

Use COUNT(\*) and GROUP BY to count the number of rentals for each
customer_id

## Exercise 6.3

Repeat the previous query and sort by the count in descending order

## Exercise 6.4

Repeat the previous query but use HAVING to only keep the groups with 40
or more.

# Exercise 7

The following query calculates a number of summary statistics for the
payment table using MAX, MIN, AVG and SUM

## Exercise 7.1

Modify the above query to do those calculations for each customer_id

## Exercise 7.2

Modify the above query to only keep the customer_ids that have more then
5 payments

-   Run the following chunk to disconnect from the connection.

``` r
# clean up
#dbDisconnect(con)
```
