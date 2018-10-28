


# Tidy data



## Exercises 12.2.1

Using prose, describe how the variables and observations are organised in
    each of the sample tables.
    
In `table1` each row is a (country, year) with variables `cases` and `population`.

```r
table1
#> # A tibble: 6 x 4
#>   country      year  cases population
#>   <chr>       <int>  <int>      <int>
#> 1 Afghanistan  1999    745   19987071
#> 2 Afghanistan  2000   2666   20595360
#> 3 Brazil       1999  37737  172006362
#> 4 Brazil       2000  80488  174504898
#> 5 China        1999 212258 1272915272
#> 6 China        2000 213766 1280428583
```

In `table2` each row is a (country, year, type) with variable count.  Type is the `cases` and `population`.  `count` variable is the numeric value of the combination.

```r
table2
#> # A tibble: 12 x 4
#>   country      year type           count
#>   <chr>       <int> <chr>          <int>
#> 1 Afghanistan  1999 cases            745
#> 2 Afghanistan  1999 population  19987071
#> 3 Afghanistan  2000 cases           2666
#> 4 Afghanistan  2000 population  20595360
#> 5 Brazil       1999 cases          37737
#> 6 Brazil       1999 population 172006362
#> # ... with 6 more rows
```

In `table3` each row is a (country, year) with variable rate (`cases` / `population`).  `rate` is the rate of cases to population as a character string in the format `"cases/rate"`.

```r
table3
#> # A tibble: 6 x 3
#>   country      year rate             
#> * <chr>       <int> <chr>            
#> 1 Afghanistan  1999 745/19987071     
#> 2 Afghanistan  2000 2666/20595360    
#> 3 Brazil       1999 37737/172006362  
#> 4 Brazil       2000 80488/174504898  
#> 5 China        1999 212258/1272915272
#> 6 China        2000 213766/1280428583
```

In `table4a` each row is a country with variables `1999` and `2000`.  The value is the the `cases`.

```r
table4a
#> # A tibble: 3 x 3
#>   country     `1999` `2000`
#> * <chr>        <int>  <int>
#> 1 Afghanistan    745   2666
#> 2 Brazil       37737  80488
#> 3 China       212258 213766
```

In `table4b` each row is a country with variables `1999` and `2000`.  The value is the the `population`.

```r
table4b
#> # A tibble: 3 x 3
#>   country         `1999`     `2000`
#> * <chr>            <int>      <int>
#> 1 Afghanistan   19987071   20595360
#> 2 Brazil       172006362  174504898
#> 3 China       1272915272 1280428583
```

In `table5` each row is a (country, century, year) with variable rate (`cases` / `population`).  The original `year` was split into (century, year).


```r
table5
#> # A tibble: 6 x 4
#>   country     century year  rate             
#> * <chr>       <chr>   <chr> <chr>            
#> 1 Afghanistan 19      99    745/19987071     
#> 2 Afghanistan 20      00    2666/20595360    
#> 3 Brazil      19      99    37737/172006362  
#> 4 Brazil      20      00    80488/174504898  
#> 5 China       19      99    212258/1272915272
#> 6 China       20      00    213766/1280428583
```

Compute the rate for table2, and table4a + table4b. You will need to perform four operations:

Compute the `rate` for `table2`, and `table4a` + `table4b`. 
    You will need to perform four operations:

    Extract the number of TB cases per country per year.
    Extract the matching population per country per year.
    Divide cases by population, and multiply by 10000.
    Store back in the appropriate place.
    
    Which representation is easiest to work with? Which is hardest? Why?
    
Extract the number of TB cases per country per year.


```r
t2_cases <- table2 %>% filter(type == "cases") %>% select(-type) %>% rename(cases = count )
t4_cases <- table4a %>% gather(`1999`, `2000`, key = "year", value = "cases")
```

Extract the matching population per country per year.


```r
t2_pop <- table2 %>% filter(type == "population") %>% select(-type) %>% rename(population = count )
t4_pop <- table4b %>% gather(`1999`, `2000`, key = "year", value = "population")
```

Divide cases by population, and multiply by 10000.


```r
t2_rate <- t2_cases %>% left_join(t2_pop, by = c("country", "year")) %>% mutate(count = (cases / population) * 10000) %>% select(-cases, -population) %>% mutate(type = "rate") %>% select(country, year, type, count)

t4_rate <- t4_cases %>% left_join(t4_pop, by = c("country", "year")) %>% mutate(count = (cases / population) * 10000) %>% select(-cases, -population) %>% mutate(type = "rate") %>% select(country, year, count)
```

Store back in the appropriate place.


```r

table2_new <- left_join(table2, t2_rate)
#> Joining, by = c("country", "year", "type", "count")
table2_new
#> # A tibble: 12 x 4
#>   country      year type           count
#>   <chr>       <int> <chr>          <dbl>
#> 1 Afghanistan  1999 cases            745
#> 2 Afghanistan  1999 population  19987071
#> 3 Afghanistan  2000 cases           2666
#> 4 Afghanistan  2000 population  20595360
#> 5 Brazil       1999 cases          37737
#> 6 Brazil       1999 population 172006362
#> # ... with 6 more rows

table4c <- t4_rate %>% spread(year, count)
table4c
#> # A tibble: 3 x 3
#>   country     `1999` `2000`
#>   <chr>        <dbl>  <dbl>
#> 1 Afghanistan  0.373   1.29
#> 2 Brazil       2.19    4.61
#> 3 China        1.67    1.67
```

Recreate the plot showing change in cases over time using `table2`
    instead of `table1`. What do you need to do first?
    

```r
table1 %>% 
  mutate(rate = cases / population * 10000)
#> # A tibble: 6 x 5
#>   country      year  cases population  rate
#>   <chr>       <int>  <int>      <int> <dbl>
#> 1 Afghanistan  1999    745   19987071 0.373
#> 2 Afghanistan  2000   2666   20595360 1.29 
#> 3 Brazil       1999  37737  172006362 2.19 
#> 4 Brazil       2000  80488  174504898 4.61 
#> 5 China        1999 212258 1272915272 1.67 
#> 6 China        2000 213766 1280428583 1.67

table2_s <- table2_new %>% spread(type, count) %>% mutate(cases = as.integer(cases), population = as.integer(population))

table1 %>% 
  count(year, wt = cases)
#> # A tibble: 2 x 2
#>    year      n
#>   <int>  <int>
#> 1  1999 250740
#> 2  2000 296920

table2_s %>% 
  count(year, wt = cases)
#> # A tibble: 2 x 2
#>    year      n
#>   <int>  <int>
#> 1  1999 250740
#> 2  2000 296920

library(ggplot2)
ggplot(table1, aes(year, cases)) + 
  geom_line(aes(group = country), colour = "grey50") + 
  geom_point(aes(colour = country))

ggplot(table2_s, aes(year, cases)) + 
  geom_line(aes(group = country), colour = "grey50") + 
  geom_point(aes(colour = country))
```

<img src="03-tidydata_files/figure-html/unnamed-chunk-14-1.png" width="70%" style="display: block; margin: auto;" /><img src="03-tidydata_files/figure-html/unnamed-chunk-14-2.png" width="70%" style="display: block; margin: auto;" />

## Exercises 12.3.3

Why are `gather()` and `spread()` not perfectly symmetrical?  
    Carefully consider the following example:


```r
stocks <- tibble(
  year   = c(2015, 2015, 2016, 2016),
  half  = c(   1,    2,     1,    2),
  return = c(1.88, 0.59, 0.92, 0.17)
)
stocks
#> # A tibble: 4 x 3
#>    year  half return
#>   <dbl> <dbl>  <dbl>
#> 1  2015     1   1.88
#> 2  2015     2   0.59
#> 3  2016     1   0.92
#> 4  2016     2   0.17
stocks %>% 
  spread(year, return) %>% 
  gather("year", "return", `2015`:`2016`)
#> # A tibble: 4 x 3
#>    half year  return
#>   <dbl> <chr>  <dbl>
#> 1     1 2015    1.88
#> 2     2 2015    0.59
#> 3     1 2016    0.92
#> 4     2 2016    0.17
```
    (Hint: look at the variable types and think about column _names_.)
    
    Both `spread()` and `gather()` have a `convert` argument. What does it 
    do?


```r
df <- data.frame(row = rep(c(1, 51), each = 3),
                 var = c("Sepal.Length", "Species", "Species_num"),
                 value = c(5.1, "setosa", 1, 7.0, "versicolor", 2))
df
#>   row          var      value
#> 1   1 Sepal.Length        5.1
#> 2   1      Species     setosa
#> 3   1  Species_num          1
#> 4  51 Sepal.Length          7
#> 5  51      Species versicolor
#> 6  51  Species_num          2

df1 <- df %>% spread(var, value) 
df1 %>% str
#> 'data.frame':	2 obs. of  4 variables:
#>  $ row         : num  1 51
#>  $ Sepal.Length: Factor w/ 6 levels "1","2","5.1",..: 3 4
#>  $ Species     : Factor w/ 6 levels "1","2","5.1",..: 5 6
#>  $ Species_num : Factor w/ 6 levels "1","2","5.1",..: 1 2
df1
#>   row Sepal.Length    Species Species_num
#> 1   1          5.1     setosa           1
#> 2  51            7 versicolor           2

df2 <- df %>% spread(var, value, convert = TRUE) 
df2 %>% str
#> 'data.frame':	2 obs. of  4 variables:
#>  $ row         : num  1 51
#>  $ Sepal.Length: num  5.1 7
#>  $ Species     : chr  "setosa" "versicolor"
#>  $ Species_num : int  1 2
df2
#>   row Sepal.Length    Species Species_num
#> 1   1          5.1     setosa           1
#> 2  51          7.0 versicolor           2

df3 <- df1 %>% gather("var", "value", Sepal.Length:Species_num, convert = TRUE)
df3 %>% str
#> 'data.frame':	6 obs. of  3 variables:
#>  $ row  : num  1 51 1 51 1 51
#>  $ var  : chr  "Sepal.Length" "Sepal.Length" "Species" "Species" ...
#>  $ value: chr  "5.1" "7" "setosa" "versicolor" ...
df3
#>   row          var      value
#> 1   1 Sepal.Length        5.1
#> 2  51 Sepal.Length          7
#> 3   1      Species     setosa
#> 4  51      Species versicolor
#> 5   1  Species_num          1
#> 6  51  Species_num          2
```

Why does this code fail?


```r
table4a %>% 
  gather(`1999`, `2000`, key = "year", value = "cases")
#> # A tibble: 6 x 3
#>   country     year   cases
#>   <chr>       <chr>  <int>
#> 1 Afghanistan 1999     745
#> 2 Brazil      1999   37737
#> 3 China       1999  212258
#> 4 Afghanistan 2000    2666
#> 5 Brazil      2000   80488
#> 6 China       2000  213766
```

Why does spreading this tibble fail? How could you add a new column to fix
    the problem?
    
Phillip Woods has two record for his age (45 and 50).


```r
people <- tribble(
  ~name,             ~key,    ~value,
  #-----------------|--------|------
  "Phillip Woods",   "age",       45,
  "Phillip Woods",   "height",   186,
  "Phillip Woods",   "age",       50,
  "Jessica Cordero", "age",       37,
  "Jessica Cordero", "height",   156
)

people %>%
  mutate(id = c(1, 2, 2, 3, 3)) %>%
  select(id, everything()) %>%
  spread(key, value)
#> # A tibble: 3 x 4
#>      id name              age height
#>   <dbl> <chr>           <dbl>  <dbl>
#> 1     1 Phillip Woods      45     NA
#> 2     2 Phillip Woods      50    186
#> 3     3 Jessica Cordero    37    156
```

Tidy the simple tibble below. Do you need to spread or gather it?
    What are the variables?
    

```r
preg <- tribble(
  ~pregnant, ~male, ~female,
  "yes",     NA,    10,
  "no",      20,    12
)

preg %>%
  gather(`male`, `female`, key = "sex", value = "count")
#> # A tibble: 4 x 3
#>   pregnant sex    count
#>   <chr>    <chr>  <dbl>
#> 1 yes      male      NA
#> 2 no       male      20
#> 3 yes      female    10
#> 4 no       female    12
```

## Exercises 12.4.3

What do the `extra` and `fill` arguments do in `separate()`? 
    Experiment with the various options for the following two toy datasets.


```r
tibble(x = c("a,b,c", "d,e,f,g", "h,i,j")) %>% 
  separate(x, c("one", "two", "three"))
#> # A tibble: 3 x 3
#>   one   two   three
#>   <chr> <chr> <chr>
#> 1 a     b     c    
#> 2 d     e     f    
#> 3 h     i     j

tibble(x = c("a,b,c", "d,e,f,g", "h,i,j")) %>% 
  separate(x, c("one", "two", "three"), extra = "drop")
#> # A tibble: 3 x 3
#>   one   two   three
#>   <chr> <chr> <chr>
#> 1 a     b     c    
#> 2 d     e     f    
#> 3 h     i     j

tibble(x = c("a,b,c", "d,e", "f,g,i")) %>% 
  separate(x, c("one", "two", "three"))
#> # A tibble: 3 x 3
#>   one   two   three
#>   <chr> <chr> <chr>
#> 1 a     b     c    
#> 2 d     e     <NA> 
#> 3 f     g     i

tibble(x = c("a,b,c", "d,e", "f,g,i")) %>% 
  separate(x, c("one", "two", "three"), fill = "right")
#> # A tibble: 3 x 3
#>   one   two   three
#>   <chr> <chr> <chr>
#> 1 a     b     c    
#> 2 d     e     <NA> 
#> 3 f     g     i
```
  
Both `unite()` and `separate()` have a `remove` argument. What does it
    do? Why would you set it to `FALSE`?


```r
tibble(x = c("a,b,c", "d,e,f,g", "h,i,j")) %>% 
  separate(x, c("one", "two", "three"), extra = "drop", remove = TRUE)
#> # A tibble: 3 x 3
#>   one   two   three
#>   <chr> <chr> <chr>
#> 1 a     b     c    
#> 2 d     e     f    
#> 3 h     i     j
tibble(x = c("a,b,c", "d,e,f,g", "h,i,j")) %>% 
  separate(x, c("one", "two", "three"), extra = "drop", remove = FALSE)
#> # A tibble: 3 x 4
#>   x       one   two   three
#>   <chr>   <chr> <chr> <chr>
#> 1 a,b,c   a     b     c    
#> 2 d,e,f,g d     e     f    
#> 3 h,i,j   h     i     j

tibble(one = c("a","d","h"), two = c("b","e","i"), three = c("c","f","j")) %>%
  unite(x, one, two, three, sep = "")
#> # A tibble: 3 x 1
#>   x    
#>   <chr>
#> 1 abc  
#> 2 def  
#> 3 hij
tibble(one = c("a","d","h"), two = c("b","e","i"), three = c("c","f","j")) %>%
  unite(x, one, two, three, sep = "", remove = FALSE)
#> # A tibble: 3 x 4
#>   x     one   two   three
#>   <chr> <chr> <chr> <chr>
#> 1 abc   a     b     c    
#> 2 def   d     e     f    
#> 3 hij   h     i     j
```

Compare and contrast `separate()` and `extract()`.  Why are there
    three variations of separation (by position, by separator, and with
    groups), but only one unite?
    

```r
tibble(x = c("a,b,c", "d,e,f", "h,i,j")) %>% 
  separate(x, c("one", "two", "three"))
#> # A tibble: 3 x 3
#>   one   two   three
#>   <chr> <chr> <chr>
#> 1 a     b     c    
#> 2 d     e     f    
#> 3 h     i     j
tibble(x = c("a|b|c", "d|e|f", "h|i|j")) %>% 
  separate(x, c("one", "two", "three"))
#> # A tibble: 3 x 3
#>   one   two   three
#>   <chr> <chr> <chr>
#> 1 a     b     c    
#> 2 d     e     f    
#> 3 h     i     j
```

    
## Exercises 12.5.1

Compare and contrast the fill arguments to spread() and complete().


```r
treatment <- tribble(
  ~ person,           ~ treatment, ~response,
  "Derrick Whitmore", 1,           7,
  "Joe Black",        2,           NA,
  "Mary Smith",       3,           9,
  "Katherine Burke",  NA,           4
)

treatmentg <- treatment %>% gather(unit, count, -person)

treatmentg <- treatmentg %>% add_row(person = "Pete James", unit = "response", count = 2)
treatmentg <- treatmentg %>% add_row(person = "Bob Kirk", unit = "treatment", count = 4)

treatments <- treatmentg %>% spread(unit, count, fill = -9) 

df <- tibble(
  group = c(1:2, 1),
  item_id = c(1:2, 2),
  item_name = c("a", "b", "b"),
  value1 = 1:3,
  value2 = 4:6
)
df2 <- df %>% complete(group, nesting(item_id, item_name))
df2 %>% complete(group, nesting(item_id, item_name), fill = list(value1 = 0, value2 = 5))
#> # A tibble: 4 x 5
#>   group item_id item_name value1 value2
#>   <dbl>   <dbl> <chr>      <dbl>  <dbl>
#> 1     1       1 a              1      4
#> 2     1       2 b              3      6
#> 3     2       1 a              0      5
#> 4     2       2 b              2      5
```

What does the direction argument to fill() do?


```r
treatment <- tribble(
  ~ person,           ~ treatment, ~response,
  "Derrick Whitmore", 1,           7,
  NA,                 2,           NA,
  "Mary Smith",       3,           9,
  "Katherine Burke",  NA,           4
)

treatment %>% fill(person)
#> # A tibble: 4 x 3
#>   person           treatment response
#>   <chr>                <dbl>    <dbl>
#> 1 Derrick Whitmore         1        7
#> 2 Derrick Whitmore         2       NA
#> 3 Mary Smith               3        9
#> 4 Katherine Burke         NA        4
treatment %>% fill(person, .direction = "up")
#> # A tibble: 4 x 3
#>   person           treatment response
#>   <chr>                <dbl>    <dbl>
#> 1 Derrick Whitmore         1        7
#> 2 Mary Smith               2       NA
#> 3 Mary Smith               3        9
#> 4 Katherine Burke         NA        4
```

## Exercises 12.6.1

In this case study I set na.rm = TRUE just to make it easier to check that we had the correct values. Is this reasonable? Think about how missing values are represented in this dataset. Are there implicit missing values? What’s the difference between an NA and zero?


```r
who
#> # A tibble: 7,240 x 60
#>   country iso2  iso3   year new_sp_m014 new_sp_m1524 new_sp_m2534
#>   <chr>   <chr> <chr> <int>       <int>        <int>        <int>
#> 1 Afghan… AF    AFG    1980          NA           NA           NA
#> 2 Afghan… AF    AFG    1981          NA           NA           NA
#> 3 Afghan… AF    AFG    1982          NA           NA           NA
#> 4 Afghan… AF    AFG    1983          NA           NA           NA
#> 5 Afghan… AF    AFG    1984          NA           NA           NA
#> 6 Afghan… AF    AFG    1985          NA           NA           NA
#> # ... with 7,234 more rows, and 53 more variables: new_sp_m3544 <int>,
#> #   new_sp_m4554 <int>, new_sp_m5564 <int>, new_sp_m65 <int>,
#> #   new_sp_f014 <int>, new_sp_f1524 <int>, new_sp_f2534 <int>,
#> #   new_sp_f3544 <int>, new_sp_f4554 <int>, new_sp_f5564 <int>,
#> #   new_sp_f65 <int>, new_sn_m014 <int>, new_sn_m1524 <int>,
#> #   new_sn_m2534 <int>, new_sn_m3544 <int>, new_sn_m4554 <int>,
#> #   new_sn_m5564 <int>, new_sn_m65 <int>, new_sn_f014 <int>,
#> #   new_sn_f1524 <int>, new_sn_f2534 <int>, new_sn_f3544 <int>,
#> #   new_sn_f4554 <int>, new_sn_f5564 <int>, new_sn_f65 <int>,
#> #   new_ep_m014 <int>, new_ep_m1524 <int>, new_ep_m2534 <int>,
#> #   new_ep_m3544 <int>, new_ep_m4554 <int>, new_ep_m5564 <int>,
#> #   new_ep_m65 <int>, new_ep_f014 <int>, new_ep_f1524 <int>,
#> #   new_ep_f2534 <int>, new_ep_f3544 <int>, new_ep_f4554 <int>,
#> #   new_ep_f5564 <int>, new_ep_f65 <int>, newrel_m014 <int>,
#> #   newrel_m1524 <int>, newrel_m2534 <int>, newrel_m3544 <int>,
#> #   newrel_m4554 <int>, newrel_m5564 <int>, newrel_m65 <int>,
#> #   newrel_f014 <int>, newrel_f1524 <int>, newrel_f2534 <int>,
#> #   newrel_f3544 <int>, newrel_f4554 <int>, newrel_f5564 <int>,
#> #   newrel_f65 <int>

who1 <- who %>% 
  gather(new_sp_m014:newrel_f65, key = "key", value = "cases", na.rm = TRUE)
who1
#> # A tibble: 76,046 x 6
#>   country     iso2  iso3   year key         cases
#> * <chr>       <chr> <chr> <int> <chr>       <int>
#> 1 Afghanistan AF    AFG    1997 new_sp_m014     0
#> 2 Afghanistan AF    AFG    1998 new_sp_m014    30
#> 3 Afghanistan AF    AFG    1999 new_sp_m014     8
#> 4 Afghanistan AF    AFG    2000 new_sp_m014    52
#> 5 Afghanistan AF    AFG    2001 new_sp_m014   129
#> 6 Afghanistan AF    AFG    2002 new_sp_m014    90
#> # ... with 7.604e+04 more rows

who1 %>% 
  count(key)
#> # A tibble: 56 x 2
#>   key              n
#>   <chr>        <int>
#> 1 new_ep_f014   1032
#> 2 new_ep_f1524  1021
#> 3 new_ep_f2534  1021
#> 4 new_ep_f3544  1021
#> 5 new_ep_f4554  1017
#> 6 new_ep_f5564  1017
#> # ... with 50 more rows


who2 <- who %>% 
  gather(new_sp_m014:newrel_f65, key = "key", value = "cases", na.rm = FALSE)
who2
#> # A tibble: 405,440 x 6
#>   country     iso2  iso3   year key         cases
#>   <chr>       <chr> <chr> <int> <chr>       <int>
#> 1 Afghanistan AF    AFG    1980 new_sp_m014    NA
#> 2 Afghanistan AF    AFG    1981 new_sp_m014    NA
#> 3 Afghanistan AF    AFG    1982 new_sp_m014    NA
#> 4 Afghanistan AF    AFG    1983 new_sp_m014    NA
#> 5 Afghanistan AF    AFG    1984 new_sp_m014    NA
#> 6 Afghanistan AF    AFG    1985 new_sp_m014    NA
#> # ... with 4.054e+05 more rows

who2 %>% 
  count(key)
#> # A tibble: 56 x 2
#>   key              n
#>   <chr>        <int>
#> 1 new_ep_f014   7240
#> 2 new_ep_f1524  7240
#> 3 new_ep_f2534  7240
#> 4 new_ep_f3544  7240
#> 5 new_ep_f4554  7240
#> 6 new_ep_f5564  7240
#> # ... with 50 more rows
```

What happens if you neglect the mutate() step? (mutate(key = stringr::str_replace(key, "newrel", "new_rel")))


```r
who3 <- who1 %>% 
  mutate(key = stringr::str_replace(key, "newrel", "new_rel"))
who4 <- who3 %>% 
  separate(key, c("new", "type", "sexage"), sep = "_")


who4 <- who1 %>% 
  separate(key, c("new", "type", "sexage"), sep = "_")
#> Warning: Expected 3 pieces. Missing pieces filled with `NA` in 2580 rows
#> [73467, 73468, 73469, 73470, 73471, 73472, 73473, 73474, 73475, 73476,
#> 73477, 73478, 73479, 73480, 73481, 73482, 73483, 73484, 73485, 73486, ...].
```

I claimed that iso2 and iso3 were redundant with country. Confirm this claim.


```r
select(who3, country, iso2, iso3) %>%
  distinct() %>%
  group_by(country) %>%
  filter(n() > 1)
#> # A tibble: 0 x 3
#> # Groups:   country [0]
#> # ... with 3 variables: country <chr>, iso2 <chr>, iso3 <chr>
```

For each country, year, and sex compute the total number of cases of TB. Make an informative visualisation of the data.


```r
who1 <- who %>% 
  gather(new_sp_m014:newrel_f65, key = "key", value = "cases", na.rm = TRUE)

who2 <- who1 %>% 
  mutate(key = stringr::str_replace(key, "newrel", "new_rel"))


who3 <- who2 %>% 
  separate(key, c("new", "type", "sexage"), sep = "_")
who3
#> # A tibble: 76,046 x 8
#>   country     iso2  iso3   year new   type  sexage cases
#>   <chr>       <chr> <chr> <int> <chr> <chr> <chr>  <int>
#> 1 Afghanistan AF    AFG    1997 new   sp    m014       0
#> 2 Afghanistan AF    AFG    1998 new   sp    m014      30
#> 3 Afghanistan AF    AFG    1999 new   sp    m014       8
#> 4 Afghanistan AF    AFG    2000 new   sp    m014      52
#> 5 Afghanistan AF    AFG    2001 new   sp    m014     129
#> 6 Afghanistan AF    AFG    2002 new   sp    m014      90
#> # ... with 7.604e+04 more rows

who4 <- who3 %>% 
  separate(sexage, c("sex", "age"), sep = 1, convert = TRUE)
who5 <- who4 %>% group_by(country, year, sex) %>% summarise(count = sum((cases)))
```

