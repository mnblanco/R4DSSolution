


# Data Import



## Exercises

What function would you use to read a file where fields were separated with “|”?

Use `read_delim` and specify "|" in the `delim` argument.


```r
read_delim("a|b|c
1|2|3", delim = "|")
#> # A tibble: 1 x 3
#>       a     b     c
#>   <int> <int> <int>
#> 1     1     2     3
```

Apart from file, skip, and comment, what other arguments do read_csv() and read_tsv() have in common?

What are the most important arguments to read_fwf()?

The `col_positions` is an important argument because this determine the width at which each column is separated. You can determine the width with the `fwf_*` helper functions.

Sometimes strings in a CSV file contain commas. To prevent them from causing problems they need to be surrounded by a quoting character, like " or '. By convention, read_csv() assumes that the quoting character will be ", and if you want to change it you’ll need to use read_delim() instead. What arguments do you need to specify to read the following text into a data frame?


```r
read_delim("x,y\n1,'a,b'", delim = ",", quote = "\'")
#> # A tibble: 1 x 2
#>       x y    
#>   <int> <chr>
#> 1     1 a,b
```

Identify what is wrong with each of the following inline CSV files. What happens when you run the code?


```r
read_csv("a,b\n1,2,3\n4,5,6")
#> # A tibble: 2 x 2
#>       a     b
#>   <int> <int>
#> 1     1     2
#> 2     4     5
read_csv("a,b,c\n1,2,3\n4,5,6")
#> # A tibble: 2 x 3
#>       a     b     c
#>   <int> <int> <int>
#> 1     1     2     3
#> 2     4     5     6

read_csv("a,b,c\n1,2\n1,2,3,4")
#> # A tibble: 2 x 3
#>       a     b     c
#>   <int> <int> <int>
#> 1     1     2    NA
#> 2     1     2     3
read_csv("a,b,c,d\n1,2\n1,2,3,4")
#> # A tibble: 2 x 4
#>       a     b     c     d
#>   <int> <int> <int> <int>
#> 1     1     2    NA    NA
#> 2     1     2     3     4

read_csv("a,b\n\"1")
#> # A tibble: 1 x 2
#>       a b    
#>   <int> <chr>
#> 1     1 <NA>

read_csv("a,b\n1,2\na,b")
#> # A tibble: 2 x 2
#>   a     b    
#>   <chr> <chr>
#> 1 1     2    
#> 2 a     b

read_csv("a;b\n1;3")
#> # A tibble: 1 x 1
#>   `a;b`
#>   <chr>
#> 1 1;3
```

## Exercises

What are the most important arguments to locale()?

- All are  important but `asciify` is hardly used.

What happens if you try and set decimal_mark and grouping_mark to the same character? What happens to the default value of grouping_mark when you set decimal_mark to “,”? What happens to the default value of decimal_mark when you set the grouping_mark to “.”?


```r
#locale(decimal_mark = ".", grouping_mark = ".")
```


```r
#locale(decimal_mark = ",", grouping_mark = ",")
```


```r
locale(decimal_mark = ".", grouping_mark = ",")
#> <locale>
#> Numbers:  123,456.78
#> Formats:  %AD / %AT
#> Timezone: UTC
#> Encoding: UTF-8
#> <date_names>
#> Days:   Sunday (Sun), Monday (Mon), Tuesday (Tue), Wednesday (Wed),
#>         Thursday (Thu), Friday (Fri), Saturday (Sat)
#> Months: January (Jan), February (Feb), March (Mar), April (Apr), May
#>         (May), June (Jun), July (Jul), August (Aug), September
#>         (Sep), October (Oct), November (Nov), December (Dec)
#> AM/PM:  AM/PM
```

I didn’t discuss the date_format and time_format options to locale(). What do they do? Construct an example that shows when they might be useful.


```r
#read_csv("a\n27-10-18")
read_csv("a\n27-10-18", locale = locale(date_format = "%d-%m-%y"))
#> # A tibble: 1 x 1
#>   a         
#>   <date>    
#> 1 2018-10-27
```


```r
#read_csv("a\n01-05-08 am")
read_csv("a\n01-05-08 am", locale = locale(time_format = "%M-%S-%I %p"))
#> # A tibble: 1 x 1
#>   a     
#>   <time>
#> 1 08:01
```

If you live outside the US, create a new locale object that encapsulates the settings for the types of file you read most commonly.


```r
locale("es",
       date_format = "%Y-%m-%d",
       time_format = "%M-%S-%I %p")
#> <locale>
#> Numbers:  123,456.78
#> Formats:  %Y-%m-%d / %M-%S-%I %p
#> Timezone: UTC
#> Encoding: UTF-8
#> <date_names>
#> Days:   domingo (dom.), lunes (lun.), martes (mar.), miércoles (mié.),
#>         jueves (jue.), viernes (vie.), sábado (sáb.)
#> Months: enero (ene.), febrero (feb.), marzo (mar.), abril (abr.), mayo
#>         (may.), junio (jun.), julio (jul.), agosto (ago.),
#>         septiembre (sept.), octubre (oct.), noviembre (nov.),
#>         diciembre (dic.)
#> AM/PM:  a. m./p. m.
```

What’s the difference between read_csv() and read_csv2()?

What are the most common encodings used in Europe? What are the most common encodings used in Asia? Do some googling to find out.

Generate the correct format string to parse each of the following dates and times:


```r
d1 <- "January 1, 2010"
parse_date(d1, format = "%B %d, %Y")
#> [1] "2010-01-01"

d2 <- "2015-Mar-07"
parse_date(d2, format = "%Y-%b-%d")
#> [1] "2015-03-07"

d3 <- "06-Jun-2017"
parse_date(d3, format = "%d-%b-%Y")
#> [1] "2017-06-06"

d4 <- c("August 19 (2015)", "July 1 (2015)")
parse_date(d4, format = "%B %d (%Y)")
#> [1] "2015-08-19" "2015-07-01"

d5 <- "12/30/14" # Dec 30, 2014
parse_date(d5, format = "%m/%d/%y")
#> [1] "2014-12-30"

t1 <- "1705"
parse_time(t1, format = "%H%M")
#> 17:05:00

t2 <- "11:15:10.12 PM"
parse_time(t2, "%I:%M:%OS %p")
#> 23:15:10.12
```
