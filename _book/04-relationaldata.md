
# Relational data



## Exercises

Imagine you wanted to draw (approximately) the route each plane flies from its origin to its destination. What variables would you need? What tables would you need to combine?

-   `flights` table: `origin` and `dest` variables

```r
head(flights %>% select(origin, dest))
#> # A tibble: 6 x 2
#>   origin dest 
#>   <chr>  <chr>
#> 1 EWR    IAH  
#> 2 LGA    IAH  
#> 3 JFK    MIA  
#> 4 JFK    BQN  
#> 5 LGA    ATL  
#> 6 EWR    ORD
```
-   `airports` table: faa (matches the origin and destination), longitude and latitude variables


```r
head(airports %>% select(lon, lat, faa))
#> # A tibble: 6 x 3
#>     lon   lat faa  
#>   <dbl> <dbl> <chr>
#> 1 -80.6  41.1 04G  
#> 2 -85.7  32.5 06A  
#> 3 -88.1  42.0 06C  
#> 4 -74.4  41.4 06N  
#> 5 -81.4  31.1 09J  
#> 6 -82.2  36.4 0A9
```

-   join `flights` with `airports`. The first join adds the location of the origin airport (`origin`) and the second join adds the location of destination airport (`dest`).

I forgot to draw the relationship between weather and airports. What is the relationship and how should it appear in the diagram?

The variable `origin` in `weather` is the same as `faa` in `airports`.


```r
head(weather %>% select(origin))
#> # A tibble: 6 x 1
#>   origin
#>   <chr> 
#> 1 EWR   
#> 2 EWR   
#> 3 EWR   
#> 4 EWR   
#> 5 EWR   
#> 6 EWR
head(airports %>% select(faa) %>% filter(faa == "EWR"))
#> # A tibble: 1 x 1
#>   faa  
#>   <chr>
#> 1 EWR
```

`weather` only contains information for the origin (NYC) airports. If it contained weather records for all airports in the USA, what additional relation would it define with flights?

`origin` would need to change to `location`
`year`, `month`, `day`, `hour`, `origin` in `weather` should match to `year`, `month`, `day`, `arr_time`, `dest` in `flight` 


```r
head(weather)
#> # A tibble: 6 x 15
#>   origin  year month   day  hour  temp  dewp humid wind_dir wind_speed
#>   <chr>  <dbl> <dbl> <int> <int> <dbl> <dbl> <dbl>    <dbl>      <dbl>
#> 1 EWR     2013     1     1     1  39.0  26.1  59.4      270      10.4 
#> 2 EWR     2013     1     1     2  39.0  27.0  61.6      250       8.06
#> 3 EWR     2013     1     1     3  39.0  28.0  64.4      240      11.5 
#> 4 EWR     2013     1     1     4  39.9  28.0  62.2      250      12.7 
#> 5 EWR     2013     1     1     5  39.0  28.0  64.4      260      12.7 
#> 6 EWR     2013     1     1     6  37.9  28.0  67.2      240      11.5 
#> # ... with 5 more variables: wind_gust <dbl>, precip <dbl>,
#> #   pressure <dbl>, visib <dbl>, time_hour <dttm>
head(flights)
#> # A tibble: 6 x 19
#>    year month   day dep_time sched_dep_time dep_delay arr_time
#>   <int> <int> <int>    <int>          <int>     <dbl>    <int>
#> 1  2013     1     1      517            515         2      830
#> 2  2013     1     1      533            529         4      850
#> 3  2013     1     1      542            540         2      923
#> 4  2013     1     1      544            545        -1     1004
#> 5  2013     1     1      554            600        -6      812
#> 6  2013     1     1      554            558        -4      740
#> # ... with 12 more variables: sched_arr_time <int>, arr_delay <dbl>,
#> #   carrier <chr>, flight <int>, tailnum <chr>, origin <chr>, dest <chr>,
#> #   air_time <dbl>, distance <dbl>, hour <dbl>, minute <dbl>,
#> #   time_hour <dttm>
```

We know that some days of the year are “special”, and fewer people than usual fly on them. How might you represent that data as a data frame? What would be the primary keys of that table? How would it connect to the existing tables?

The table `special_days` would contain a list of day where fewer people travel than usual.
Primary key: year, month, day should match `year`, `month`, and `day` in `flight`



```r
special_days <- tribble(
  ~year, ~month, ~day, ~holiday,
  2019, 01, 01, "New Years Day",
  2018, 12, 25, "Christmas Day"
)
```

## Exercises 13.3.1 

Add a surrogate key to flights.


```r
flights
#> # A tibble: 336,776 x 19
#>    year month   day dep_time sched_dep_time dep_delay arr_time
#>   <int> <int> <int>    <int>          <int>     <dbl>    <int>
#> 1  2013     1     1      517            515         2      830
#> 2  2013     1     1      533            529         4      850
#> 3  2013     1     1      542            540         2      923
#> 4  2013     1     1      544            545        -1     1004
#> 5  2013     1     1      554            600        -6      812
#> 6  2013     1     1      554            558        -4      740
#> # ... with 3.368e+05 more rows, and 12 more variables:
#> #   sched_arr_time <int>, arr_delay <dbl>, carrier <chr>, flight <int>,
#> #   tailnum <chr>, origin <chr>, dest <chr>, air_time <dbl>,
#> #   distance <dbl>, hour <dbl>, minute <dbl>, time_hour <dttm>

flights %>%
  arrange(year, month, day, sched_dep_time, carrier, flight) %>%
  mutate(flight_id = row_number()) %>%
  glimpse()
#> Observations: 336,776
#> Variables: 20
#> $ year           <int> 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013,...
#> $ month          <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,...
#> $ day            <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,...
#> $ dep_time       <int> 517, 533, 542, 544, 554, 559, 558, 559, 558, 55...
#> $ sched_dep_time <int> 515, 529, 540, 545, 558, 559, 600, 600, 600, 60...
#> $ dep_delay      <dbl> 2, 4, 2, -1, -4, 0, -2, -1, -2, -2, -3, NA, 1, ...
#> $ arr_time       <int> 830, 850, 923, 1004, 740, 702, 753, 941, 849, 8...
#> $ sched_arr_time <int> 819, 830, 850, 1022, 728, 706, 745, 910, 851, 8...
#> $ arr_delay      <dbl> 11, 20, 33, -18, 12, -4, 8, 31, -2, -3, -8, NA,...
#> $ carrier        <chr> "UA", "UA", "AA", "B6", "UA", "B6", "AA", "AA",...
#> $ flight         <int> 1545, 1714, 1141, 725, 1696, 1806, 301, 707, 49...
#> $ tailnum        <chr> "N14228", "N24211", "N619AA", "N804JB", "N39463...
#> $ origin         <chr> "EWR", "LGA", "JFK", "JFK", "EWR", "JFK", "LGA"...
#> $ dest           <chr> "IAH", "IAH", "MIA", "BQN", "ORD", "BOS", "ORD"...
#> $ air_time       <dbl> 227, 227, 160, 183, 150, 44, 138, 257, 149, 158...
#> $ distance       <dbl> 1400, 1416, 1089, 1576, 719, 187, 733, 1389, 10...
#> $ hour           <dbl> 5, 5, 5, 5, 5, 5, 6, 6, 6, 6, 6, 6, 6, 6, 6, 6,...
#> $ minute         <dbl> 15, 29, 40, 45, 58, 59, 0, 0, 0, 0, 0, 0, 0, 0,...
#> $ time_hour      <dttm> 2013-01-01 05:00:00, 2013-01-01 05:00:00, 2013...
#> $ flight_id      <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, ...

flights %>% 
  count(year, month, day, sched_dep_time, carrier, flight) %>% 
  filter(n > 1)
#> # A tibble: 0 x 7
#> # ... with 7 variables: year <int>, month <int>, day <int>,
#> #   sched_dep_time <int>, carrier <chr>, flight <int>, n <int>
```

Identify the keys in the following datasets

Lahman::Batting


```r
head(Lahman::Batting)
#>    playerID yearID stint teamID lgID  G  AB  R  H X2B X3B HR RBI SB CS BB
#> 1 abercda01   1871     1    TRO   NA  1   4  0  0   0   0  0   0  0  0  0
#> 2  addybo01   1871     1    RC1   NA 25 118 30 32   6   0  0  13  8  1  4
#> 3 allisar01   1871     1    CL1   NA 29 137 28 40   4   5  0  19  3  1  2
#> 4 allisdo01   1871     1    WS3   NA 27 133 28 44  10   2  2  27  1  1  0
#> 5 ansonca01   1871     1    RC1   NA 25 120 29 39  11   3  0  16  6  2  2
#> 6 armstbo01   1871     1    FW1   NA 12  49  9 11   2   1  0   5  0  1  0
#>   SO IBB HBP SH SF GIDP
#> 1  0  NA  NA NA NA   NA
#> 2  0  NA  NA NA NA   NA
#> 3  5  NA  NA NA NA   NA
#> 4  2  NA  NA NA NA   NA
#> 5  1  NA  NA NA NA   NA
#> 6  1  NA  NA NA NA   NA
Lahman::Batting %>% 
  count(playerID, yearID, stint) %>% 
  filter(n > 1)
#> # A tibble: 0 x 4
#> # ... with 4 variables: playerID <chr>, yearID <int>, stint <int>, n <int>
```

babynames::babynames


```r
head(babynames::babynames)
#> # A tibble: 6 x 5
#>    year sex   name          n   prop
#>   <dbl> <chr> <chr>     <int>  <dbl>
#> 1  1880 F     Mary       7065 0.0724
#> 2  1880 F     Anna       2604 0.0267
#> 3  1880 F     Emma       2003 0.0205
#> 4  1880 F     Elizabeth  1939 0.0199
#> 5  1880 F     Minnie     1746 0.0179
#> 6  1880 F     Margaret   1578 0.0162
babynames::babynames %>% 
  count(year, sex, name) %>% 
  filter(nn > 1)
#> # A tibble: 0 x 4
#> # ... with 4 variables: year <dbl>, sex <chr>, name <chr>, nn <int>
```

nasaweather::atmos


```r
head(nasaweather::atmos)
#> # A tibble: 6 x 11
#>     lat  long  year month surftemp  temp pressure ozone cloudlow cloudmid
#>   <dbl> <dbl> <int> <int>    <dbl> <dbl>    <dbl> <dbl>    <dbl>    <dbl>
#> 1  36.2 -114.  1995     1     273.  272.      835   304      7.5     34.5
#> 2  33.7 -114.  1995     1     280.  282.      940   304     11.5     32.5
#> 3  31.2 -114.  1995     1     285.  285.      960   298     16.5     26  
#> 4  28.7 -114.  1995     1     289.  291.      990   276     20.5     14.5
#> 5  26.2 -114.  1995     1     292.  293.     1000   274     26       10.5
#> 6  23.7 -114.  1995     1     294.  294.     1000   264     30        9.5
#> # ... with 1 more variable: cloudhigh <dbl>
nasaweather::atmos %>%
      count(lat, long, year, month) %>%
      filter(n > 1)
#> # A tibble: 0 x 5
#> # ... with 5 variables: lat <dbl>, long <dbl>, year <int>, month <int>,
#> #   n <int>
```

fueleconomy::vehicles


```r
head(fueleconomy::vehicles)
#> # A tibble: 6 x 12
#>      id make   model  year class trans drive   cyl displ fuel    hwy   cty
#>   <int> <chr>  <chr> <int> <chr> <chr> <chr> <int> <dbl> <chr> <int> <int>
#> 1 27550 AM Ge… DJ P…  1984 Spec… Auto… 2-Wh…     4   2.5 Regu…    17    18
#> 2 28426 AM Ge… DJ P…  1984 Spec… Auto… 2-Wh…     4   2.5 Regu…    17    18
#> 3 27549 AM Ge… FJ8c…  1984 Spec… Auto… 2-Wh…     6   4.2 Regu…    13    13
#> 4 28425 AM Ge… FJ8c…  1984 Spec… Auto… 2-Wh…     6   4.2 Regu…    13    13
#> 5  1032 AM Ge… Post…  1985 Spec… Auto… Rear…     4   2.5 Regu…    17    16
#> 6  1033 AM Ge… Post…  1985 Spec… Auto… Rear…     6   4.2 Regu…    13    13
fueleconomy::vehicles %>%
      count(id) %>%
      filter(n > 1)
#> # A tibble: 0 x 2
#> # ... with 2 variables: id <int>, n <int>
```

ggplot2::diamonds


```r
head(ggplot2::diamonds)
#> # A tibble: 6 x 10
#>   carat cut       color clarity depth table price     x     y     z
#>   <dbl> <ord>     <ord> <ord>   <dbl> <dbl> <int> <dbl> <dbl> <dbl>
#> 1 0.23  Ideal     E     SI2      61.5    55   326  3.95  3.98  2.43
#> 2 0.21  Premium   E     SI1      59.8    61   326  3.89  3.84  2.31
#> 3 0.23  Good      E     VS1      56.9    65   327  4.05  4.07  2.31
#> 4 0.290 Premium   I     VS2      62.4    58   334  4.2   4.23  2.63
#> 5 0.31  Good      J     SI2      63.3    58   335  4.34  4.35  2.75
#> 6 0.24  Very Good J     VVS2     62.8    57   336  3.94  3.96  2.48
ggplot2::diamonds %>%
  distinct() %>%
  nrow()
#> [1] 53794
nrow(ggplot2::diamonds)
#> [1] 53940
```

(You might need to install some packages and read some documentation.)

Draw a diagram illustrating the connections between the Batting, Master, and Salaries tables in the Lahman package. Draw another diagram that shows the relationship between Master, Managers, AwardsManagers.

-   `Master`

    -   Primary keys: `playerID`

-   `Batting`

    -   Primary keys: `yearID`, `yearID`, `stint`

    -   Foreign Keys:

        -   `playerID` = `Master$playerID` (many-to-1)

-   `Salaries`

    -   Primary keys: `yearID`, `teamID`, `playerID`

    -   Foreign Keys

        -   `playerID` = `Master$playerID` (many-to-1)


```r
dm1 <- dm_from_data_frames(list(Batting = Lahman::Batting,
                                Master = Lahman::Master,
                                Salaries = Lahman::Salaries)) %>%
  dm_set_key("Batting", c("playerID", "yearID", "stint")) %>%
  dm_set_key("Master", "playerID") %>%
  dm_set_key("Salaries", c("yearID", "teamID", "playerID")) %>%
  dm_add_references(
    Batting$playerID == Master$playerID,
    Salaries$playerID == Master$playerID
  )

dm_create_graph(dm1, rankdir = "LR", columnArrows = TRUE)
```

How would you characterise the relationship between the Batting, Pitching, and Fielding tables?

## Exercises 13.4.6 

Compute the average delay by destination, then join on the airports data frame so you can show the spatial distribution of delays. Here’s an easy way to draw a map of the United States:

airports %>%
  semi_join(flights, c("faa" = "dest")) %>%
  ggplot(aes(lon, lat)) +
    borders("state") +
    geom_point() +
    coord_quickmap()
(Don’t worry if you don’t understand what semi_join() does — you’ll learn about it next.)

You might want to use the size or colour of the points to display the average delay for each airport.

Add the location of the origin and destination (i.e. the lat and lon) to flights.

Is there a relationship between the age of a plane and its delays?

What weather conditions make it more likely to see a delay?

What happened on June 13 2013? Display the spatial pattern of delays, and then use Google to cross-reference with the weather.
