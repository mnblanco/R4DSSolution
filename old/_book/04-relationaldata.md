
# Relational data



## Exercises 13.2.1

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


```r
flights_dest_delay <-flights %>% group_by(dest) %>% summarise(avg_arr_delay=mean(arr_delay, na.rm = TRUE)) %>%
  inner_join(airports, by = c(dest = "faa")) %>% select(dest, lon, lat, avg_arr_delay)

flights_dest_delay %>%
  ggplot(aes(lon, lat, colour = avg_arr_delay)) +
    borders("state") +
    geom_point() +
    coord_quickmap()
#> 
#> Attaching package: 'maps'
#> The following object is masked from 'package:purrr':
#> 
#>     map
```

<img src="04-relationaldata_files/figure-html/unnamed-chunk-15-1.png" width="70%" style="display: block; margin: auto;" />
(Don’t worry if you don’t understand what semi_join() does — you’ll learn about it next.)

You might want to use the size or colour of the points to display the average delay for each airport.

Add the location of the origin and destination (i.e. the lat and lon) to flights.


```r
airport_locations <- airports %>%
  select(faa, lat, lon)

flights %>%
    select(year:day, hour, origin, dest) %>%
  left_join(
    airport_locations,
    by = c("origin" = "faa")
  ) %>%
  left_join(
    airport_locations,
    by = c("dest" = "faa")
  )
#> # A tibble: 336,776 x 10
#>    year month   day  hour origin dest  lat.x lon.x lat.y lon.y
#>   <int> <int> <int> <dbl> <chr>  <chr> <dbl> <dbl> <dbl> <dbl>
#> 1  2013     1     1     5 EWR    IAH    40.7 -74.2  30.0 -95.3
#> 2  2013     1     1     5 LGA    IAH    40.8 -73.9  30.0 -95.3
#> 3  2013     1     1     5 JFK    MIA    40.6 -73.8  25.8 -80.3
#> 4  2013     1     1     5 JFK    BQN    40.6 -73.8  NA    NA  
#> 5  2013     1     1     6 LGA    ATL    40.8 -73.9  33.6 -84.4
#> 6  2013     1     1     5 EWR    ORD    40.7 -74.2  42.0 -87.9
#> # ... with 3.368e+05 more rows
```

Is there a relationship between the age of a plane and its delays?


```r
plane_ages<- flights %>% 
  inner_join(planes, by = c("tailnum")) %>%
  mutate(age = year.x - year.y) %>%
  group_by(age) %>%
  summarise(avg_dep_delay = mean(dep_delay, na.rm = TRUE), avg_arr_delay = mean(arr_delay,  na.rm = TRUE)) %>%
  na.omit()

plane_ages
#> # A tibble: 46 x 3
#>     age avg_dep_delay avg_arr_delay
#>   <int>         <dbl>         <dbl>
#> 1     0         10.6           4.01
#> 2     1          9.64          2.85
#> 3     2         11.8           5.70
#> 4     3         12.5           5.18
#> 5     4         11.0           4.92
#> 6     5         13.2           5.57
#> # ... with 40 more rows
plane_ages %>%
  ggplot(aes(x = age, y = avg_dep_delay)) +
  geom_point() +
  geom_line()

plane_ages %>%
  ggplot(aes(x = age, y = avg_arr_delay)) +
  geom_point() +
  geom_line()
```

<img src="04-relationaldata_files/figure-html/unnamed-chunk-17-1.png" width="70%" style="display: block; margin: auto;" /><img src="04-relationaldata_files/figure-html/unnamed-chunk-17-2.png" width="70%" style="display: block; margin: auto;" />

What weather conditions make it more likely to see a delay?


```r
flight_weather <-
  flights %>%
  inner_join(weather, by = c("origin" = "origin",
                            "year" = "year",
                            "month" = "month",
                            "day" = "day",
                            "hour" = "hour"))

flight_weather %>%
  group_by(temp) %>%
  summarise(delay = mean(dep_delay, na.rm = TRUE)) %>%
  na.omit() %>%
  ggplot(aes(x = temp, y = delay)) +
    geom_line() + 
  geom_point()

flight_weather %>%
  group_by(dewp) %>%
  summarise(delay = mean(dep_delay, na.rm = TRUE)) %>%
  na.omit() %>%
  ggplot(aes(x = dewp, y = delay)) +
    geom_line() + 
  geom_point()

flight_weather %>%
  group_by(humid) %>%
  summarise(delay = mean(dep_delay, na.rm = TRUE)) %>%
  na.omit() %>%
  ggplot(aes(x = humid, y = delay)) +
    geom_line() + 
  geom_point()

flight_weather %>%
  group_by(wind_speed) %>%
  summarise(delay = mean(dep_delay, na.rm = TRUE)) %>%
  na.omit() %>%
  ggplot(aes(x = wind_speed, y = delay)) +
    geom_line() + 
  geom_point()

flight_weather %>%
  group_by(wind_gust) %>%
  summarise(delay = mean(dep_delay, na.rm = TRUE)) %>%
  na.omit() %>%
  ggplot(aes(x = wind_gust, y = delay)) +
    geom_line() + 
  geom_point()

flight_weather %>%
  group_by(precip) %>%
  summarise(delay = mean(dep_delay, na.rm = TRUE)) %>%
  na.omit() %>%
  ggplot(aes(x = precip, y = delay)) +
    geom_line() + 
  geom_point()

flight_weather %>%
  group_by(pressure) %>%
  summarise(delay = mean(dep_delay, na.rm = TRUE)) %>%
  na.omit() %>%
  ggplot(aes(x = pressure, y = delay)) +
    geom_line() + 
  geom_point()

flight_weather %>%
  group_by(visib) %>%
  summarise(delay = mean(dep_delay, na.rm = TRUE)) %>%
  na.omit() %>%
  ggplot(aes(x = visib, y = delay)) +
    geom_line() + 
  geom_point()
```

<img src="04-relationaldata_files/figure-html/unnamed-chunk-18-1.png" width="70%" style="display: block; margin: auto;" /><img src="04-relationaldata_files/figure-html/unnamed-chunk-18-2.png" width="70%" style="display: block; margin: auto;" /><img src="04-relationaldata_files/figure-html/unnamed-chunk-18-3.png" width="70%" style="display: block; margin: auto;" /><img src="04-relationaldata_files/figure-html/unnamed-chunk-18-4.png" width="70%" style="display: block; margin: auto;" /><img src="04-relationaldata_files/figure-html/unnamed-chunk-18-5.png" width="70%" style="display: block; margin: auto;" /><img src="04-relationaldata_files/figure-html/unnamed-chunk-18-6.png" width="70%" style="display: block; margin: auto;" /><img src="04-relationaldata_files/figure-html/unnamed-chunk-18-7.png" width="70%" style="display: block; margin: auto;" /><img src="04-relationaldata_files/figure-html/unnamed-chunk-18-8.png" width="70%" style="display: block; margin: auto;" />

What happened on June 13 2013? Display the spatial pattern of delays, and then use Google to cross-reference with the weather.

In June 13 2013 there was a large series of storms in the southeastern US (see [June 12-13, 2013 derecho series](https://en.wikipedia.org/wiki/June_12%E2%80%9313,_2013_derecho_series))


```r
library(viridis)
#> Loading required package: viridisLite
flights %>%
  filter(year == 2013, month == 6, day == 13) %>%
  group_by(dest) %>%
  summarise(delay = mean(arr_delay, na.rm = TRUE)) %>%
  na.omit() %>%
  inner_join(airports, by = c("dest" = "faa")) %>%
  ggplot(aes(y = lat, x = lon, size = delay, colour = delay)) +
  borders("state") +
  geom_point() +
  coord_quickmap() +
  scale_colour_viridis()
```

<img src="04-relationaldata_files/figure-html/unnamed-chunk-19-1.png" width="70%" style="display: block; margin: auto;" />

## Exercises 13.5.1 

What does it mean for a flight to have a missing tailnum? What do the tail numbers that don’t have a matching record in planes have in common? (Hint: one variable explains ~90% of the problems.)


```r
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
# Flights with tailnum that are not in the planes table
flights %>%
  anti_join(planes, by = "tailnum") %>%
  count(carrier, sort = TRUE)
#> # A tibble: 10 x 2
#>   carrier     n
#>   <chr>   <int>
#> 1 MQ      25397
#> 2 AA      22558
#> 3 UA       1693
#> 4 9E       1044
#> 5 B6        830
#> 6 US        699
#> # ... with 4 more rows

# tailnum in flight but not in planes
planes %>% filter(tailnum == "N3ALAA")
#> # A tibble: 0 x 9
#> # ... with 9 variables: tailnum <chr>, year <int>, type <chr>,
#> #   manufacturer <chr>, model <chr>, engines <int>, seats <int>,
#> #   speed <int>, engine <chr>
```

Filter flights to only show flights with planes that have flown at least 100 flights.



```r
planes_gt_100 <- flights %>%
  count(tailnum) %>%
  filter(n > 99)

flights %>%
  semi_join(planes_gt_100, by = "tailnum")
#> # A tibble: 230,902 x 19
#>    year month   day dep_time sched_dep_time dep_delay arr_time
#>   <int> <int> <int>    <int>          <int>     <dbl>    <int>
#> 1  2013     1     1      517            515         2      830
#> 2  2013     1     1      533            529         4      850
#> 3  2013     1     1      544            545        -1     1004
#> 4  2013     1     1      554            558        -4      740
#> 5  2013     1     1      555            600        -5      913
#> 6  2013     1     1      557            600        -3      709
#> # ... with 2.309e+05 more rows, and 12 more variables:
#> #   sched_arr_time <int>, arr_delay <dbl>, carrier <chr>, flight <int>,
#> #   tailnum <chr>, origin <chr>, dest <chr>, air_time <dbl>,
#> #   distance <dbl>, hour <dbl>, minute <dbl>, time_hour <dttm>
```

Combine fueleconomy::vehicles and fueleconomy::common to find only the records for the most common models.


```r
glimpse(fueleconomy::vehicles)
#> Observations: 33,442
#> Variables: 12
#> $ id    <int> 27550, 28426, 27549, 28425, 1032, 1033, 3347, 13309, 133...
#> $ make  <chr> "AM General", "AM General", "AM General", "AM General", ...
#> $ model <chr> "DJ Po Vehicle 2WD", "DJ Po Vehicle 2WD", "FJ8c Post Off...
#> $ year  <int> 1984, 1984, 1984, 1984, 1985, 1985, 1987, 1997, 1997, 19...
#> $ class <chr> "Special Purpose Vehicle 2WD", "Special Purpose Vehicle ...
#> $ trans <chr> "Automatic 3-spd", "Automatic 3-spd", "Automatic 3-spd",...
#> $ drive <chr> "2-Wheel Drive", "2-Wheel Drive", "2-Wheel Drive", "2-Wh...
#> $ cyl   <int> 4, 4, 6, 6, 4, 6, 6, 4, 4, 6, 4, 4, 6, 4, 4, 6, 5, 5, 6,...
#> $ displ <dbl> 2.5, 2.5, 4.2, 4.2, 2.5, 4.2, 3.8, 2.2, 2.2, 3.0, 2.3, 2...
#> $ fuel  <chr> "Regular", "Regular", "Regular", "Regular", "Regular", "...
#> $ hwy   <int> 17, 17, 13, 13, 17, 13, 21, 26, 28, 26, 27, 29, 26, 27, ...
#> $ cty   <int> 18, 18, 13, 13, 16, 13, 14, 20, 22, 18, 19, 21, 17, 20, ...
glimpse(fueleconomy::common)
#> Observations: 347
#> Variables: 4
#> $ make  <chr> "Acura", "Acura", "Acura", "Acura", "Acura", "Audi", "Au...
#> $ model <chr> "Integra", "Legend", "MDX 4WD", "NSX", "TSX", "A4", "A4 ...
#> $ n     <int> 42, 28, 12, 28, 27, 49, 49, 66, 20, 12, 46, 20, 30, 29, ...
#> $ years <int> 16, 10, 12, 14, 11, 19, 15, 19, 19, 12, 20, 15, 16, 16, ...
```


```r
fueleconomy::vehicles %>%
  semi_join(fueleconomy::common, by = c("make", "model"))
#> # A tibble: 14,531 x 12
#>      id make  model   year class trans drive   cyl displ fuel    hwy   cty
#>   <int> <chr> <chr>  <int> <chr> <chr> <chr> <int> <dbl> <chr> <int> <int>
#> 1  1833 Acura Integ…  1986 Subc… Auto… Fron…     4   1.6 Regu…    28    22
#> 2  1834 Acura Integ…  1986 Subc… Manu… Fron…     4   1.6 Regu…    28    23
#> 3  3037 Acura Integ…  1987 Subc… Auto… Fron…     4   1.6 Regu…    28    22
#> 4  3038 Acura Integ…  1987 Subc… Manu… Fron…     4   1.6 Regu…    28    23
#> 5  4183 Acura Integ…  1988 Subc… Auto… Fron…     4   1.6 Regu…    27    22
#> 6  4184 Acura Integ…  1988 Subc… Manu… Fron…     4   1.6 Regu…    28    23
#> # ... with 1.452e+04 more rows
```


Find the 48 hours (over the course of the whole year) that have the worst delays. Cross-reference it with the weather data. Can you see any patterns?


```r
flights %>%
  group_by(year, month, day) %>%
  summarise(total_24 = sum(dep_delay, na.rm = TRUE)+ sum(arr_delay, na.rm = TRUE)) %>%
  mutate(total_48 = total_24 + lag(total_24)) %>%
  arrange(desc(total_48))
#> # A tibble: 365 x 5
#> # Groups:   year, month [12]
#>    year month   day total_24 total_48
#>   <int> <int> <int>    <dbl>    <dbl>
#> 1  2013     7    23    80641   175419
#> 2  2013     3     8   135264   167530
#> 3  2013     6    25    80434   166649
#> 4  2013     8     9    72866   165287
#> 5  2013     6    28    81389   157910
#> 6  2013     7    10    97120   157396
#> # ... with 359 more rows
```
What does anti_join(flights, airports, by = c("dest" = "faa")) tell you? What does anti_join(airports, flights, by = c("faa" = "dest")) tell you?


Return all rows from flights where destination (`dest`)  is not matched in airports.  This only keeps columns from flights.

```r
# Flights with dest that are not in the airports table
anti_join(flights, airports, by = c("dest" = "faa"))
#> # A tibble: 7,602 x 19
#>    year month   day dep_time sched_dep_time dep_delay arr_time
#>   <int> <int> <int>    <int>          <int>     <dbl>    <int>
#> 1  2013     1     1      544            545        -1     1004
#> 2  2013     1     1      615            615         0     1039
#> 3  2013     1     1      628            630        -2     1137
#> 4  2013     1     1      701            700         1     1123
#> 5  2013     1     1      711            715        -4     1151
#> 6  2013     1     1      820            820         0     1254
#> # ... with 7,596 more rows, and 12 more variables: sched_arr_time <int>,
#> #   arr_delay <dbl>, carrier <chr>, flight <int>, tailnum <chr>,
#> #   origin <chr>, dest <chr>, air_time <dbl>, distance <dbl>, hour <dbl>,
#> #   minute <dbl>, time_hour <dttm>


# dest (`faa`) in flight but not in airports
airports %>% filter(faa == "BQN")
#> # A tibble: 0 x 8
#> # ... with 8 variables: faa <chr>, name <chr>, lat <dbl>, lon <dbl>,
#> #   alt <int>, tz <dbl>, dst <chr>, tzone <chr>
```


```r
# Airports that are not in the flights table for dest
anti_join(airports, flights, by = c("faa" = "dest"))
#> # A tibble: 1,357 x 8
#>   faa   name                      lat   lon   alt    tz dst   tzone       
#>   <chr> <chr>                   <dbl> <dbl> <int> <dbl> <chr> <chr>       
#> 1 04G   Lansdowne Airport        41.1 -80.6  1044    -5 A     America/New…
#> 2 06A   Moton Field Municipal …  32.5 -85.7   264    -6 A     America/Chi…
#> 3 06C   Schaumburg Regional      42.0 -88.1   801    -6 A     America/Chi…
#> 4 06N   Randall Airport          41.4 -74.4   523    -5 A     America/New…
#> 5 09J   Jekyll Island Airport    31.1 -81.4    11    -5 A     America/New…
#> 6 0A9   Elizabethton Municipal…  36.4 -82.2  1593    -5 A     America/New…
#> # ... with 1,351 more rows

# airport not in dest (`faa`) in flight 
flights %>% filter(dest == "04G")
#> # A tibble: 0 x 19
#> # ... with 19 variables: year <int>, month <int>, day <int>,
#> #   dep_time <int>, sched_dep_time <int>, dep_delay <dbl>, arr_time <int>,
#> #   sched_arr_time <int>, arr_delay <dbl>, carrier <chr>, flight <int>,
#> #   tailnum <chr>, origin <chr>, dest <chr>, air_time <dbl>,
#> #   distance <dbl>, hour <dbl>, minute <dbl>, time_hour <dttm>
```

You might expect that there’s an implicit relationship between plane and airline, because each plane is flown by a single airline. Confirm or reject this hypothesis using the tools you’ve learned above.


```r
flights %>% group_by(tailnum, carrier)
#> # A tibble: 336,776 x 19
#> # Groups:   tailnum, carrier [4,067]
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
```


