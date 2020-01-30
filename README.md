README
================

<!-- README.md is generated from README.Rmd. Please edit that file -->

# Status

[![Build
Status](https://travis-ci.org/RobWHickman/ggparliament.png)](https://travis-ci.org/RobWHickman/ggparliament)
![CRAN Status](https://www.r-pkg.org/badges/version/ggparliament)
![Downloads](https://cranlogs.r-pkg.org/badges/grand-total/ggparliament)
[![DOI](https://joss.theoj.org/papers/10.21105/joss.01313/status.svg)](https://doi.org/10.21105/joss.01313)
<img src = "man/figures/HexSticker.png" align = "right" width = "200"/>

# `ggparliament`: Parliament plots

This package attempts to implement “parliament plots” - visual
representations of the composition of legislatures that display seats
colour-coded by party. The input is a data frame containing one row per
party, with columns representing party name/label and number of seats,
respectively.

This `R` package is a `ggplot2` extension and is now on CRAN. Please
install the stable version in `R` by running:

``` r
install.packages("ggparliament")
```

To install the package from source:

``` r
devtools::install_github("robwhickman/ggparliament")
```

Inspiration from this package comes from:
[parliamentdiagram](https://github.com/slashme/parliamentdiagram), which
is used on Wikipedia,
[parliament-svg](https://github.com/juliuste/parliament-svg), which is a
javascript clone, and [a discussion on
StackOverflow](http://stackoverflow.com/questions/42729174/creating-a-half-donut-or-parliamentary-seating-chart),
which provided some of the code for part for the “arc” representations
used in this package.

If you have any issues, please note the problem and inform us\!

## Election data

`ggparliament` provides election data from the following countries.

``` r
election_data %>% 
  distinct(year, country, house) %>% 
  arrange(country, year)
```

    ##    year   country           house
    ## 1  2010 Australia Representatives
    ## 2  2010 Australia          Senate
    ## 3  2013 Australia Representatives
    ## 4  2013 Australia          Senate
    ## 5  2016 Australia Representatives
    ## 6  2016 Australia          Senate
    ## 7  2009     Chile       Diputados
    ## 8  2009     Chile       Senadores
    ## 9  2013     Chile       Diputados
    ## 10 2013     Chile       Senadores
    ## 11 2017     Chile       Diputados
    ## 12 2017     Chile       Senadores
    ## 13 1990   Germany       Bundestag
    ## 14 1994   Germany       Bundestag
    ## 15 1998   Germany       Bundestag
    ## 16 2002   Germany       Bundestag
    ## 17 2005   Germany       Bundestag
    ## 18 2009   Germany       Bundestag
    ## 19 2013   Germany       Bundestag
    ## 20 2017   Germany       Bundestag
    ## 21 2007    Russia            Duma
    ## 22 2011    Russia            Duma
    ## 23 2016    Russia            Duma
    ## 24 2010        UK         Commons
    ## 25 2015        UK         Commons
    ## 26 2017        UK         Commons
    ## 27 2012       USA          Senate
    ## 28 2012       USA Representatives
    ## 29 2014       USA          Senate
    ## 30 2014       USA Representatives
    ## 31 2016       USA          Senate
    ## 32 2016       USA Representatives

We also provide the following vignettes for further explanation:

1.  Basic parliament plots
2.  Labelling parties
3.  Drawing the majority threshold line
4.  Highlighting parties in power
5.  Faceting legislatures
6.  Emphasizing certain seats
7.  Visualizaing overhang seats in MMP electoral systems
8.  Arranging seat order in ggparliament plots.

Quick `ggparliament` examples can be viewed
below.

## Semicircle parliament

### EU, France, United States, and so on…

### Plot of US House of Representatives

``` r
#filter the election data for the most recent US House of Representatives
us_house <- election_data %>%
  filter(country == "USA" &
    year == 2016 &
    house == "Representatives")

us_house <- parliament_data(election_data = us_house,
  type = "semicircle",
  parl_rows = 10,
  party_seats = us_house$seats)

us_senate <- election_data %>%
  filter(country == "USA" &
    year == 2016 &
    house == "Senate")

us_senate <- parliament_data(
  election_data = us_senate,
  type = "semicircle",
  parl_rows = 4,
  party_seats = us_senate$seats)
```

``` r
representatives <- ggplot(us_house, aes(x, y, colour = party_short)) +
  geom_parliament_seats() + 
  #highlight the party in control of the House with a black line
  geom_highlight_government(government == 1) +
  #draw majority threshold
  draw_majoritythreshold(n = 218, label = TRUE, type = 'semicircle')+
  #set theme_ggparliament
  theme_ggparliament() +
  #other aesthetics
  labs(colour = NULL, 
       title = "United States House of Representatives",
       subtitle = "Party that controls the House highlighted.") +
  scale_colour_manual(values = us_house$colour, 
                      limits = us_house$party_short) 

representatives
```

![](README_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

### Plot of US Senate

``` r
senate <- ggplot(us_senate, aes(x, y, colour = party_long)) +
  geom_parliament_seats() + 
  geom_highlight_government(government == 1) +
  # add bar showing proportion of seats by party in legislature
  geom_parliament_bar(colour = colour, party = party_long) + 
  theme_ggparliament(legend = FALSE) +
  labs(colour = NULL, 
       title = "United States Senate",
       subtitle = "The party that has control of the Senate is encircled in black.") +
  scale_colour_manual(values = us_senate$colour,
                      limits = us_senate$party_long)
senate 
```

![](README_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

### Plot of German Bundestag

``` r
germany <- election_data %>%
  filter(year == 2017 & 
           country == "Germany") 

germany <- parliament_data(election_data = germany, 
                           parl_rows = 10,
                           type = 'semicircle',
                           party_seats = germany$seats)

bundestag <- ggplot(germany, aes(x, y, colour = party_short)) +
  geom_parliament_seats(size = 3) +
  labs(colour="Party") +  
  theme_ggparliament(legend = TRUE) +
  scale_colour_manual(values = germany$colour, 
                      limits = germany$party_short) 

bundestag
```

![](README_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

## Opposing Benches Parliament

### United Kingdom

``` r
#data preparation
uk_17 <- election_data %>% 
  filter(country == "UK" & 
           year == "2017") %>% 
  parliament_data(election_data = .,
                  party_seats = .$seats,
                  parl_rows = 12,
                  type = "opposing_benches",
                  group = .$government)


commons <- ggplot(uk_17, aes(x, y, colour = party_short)) +
  geom_parliament_seats(size = 3) + 
  theme_ggparliament() + 
  coord_flip() + 
  labs(colour = NULL, 
       title = "UK parliament in 2017") +
  scale_colour_manual(values = uk_17$colour, 
                      limits = uk_17$party_short)

commons
```

![](README_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

## Horseshoe parliament

### Australia, New Zealand

``` r
australia <- election_data %>%
  filter(country == "Australia" &
    house == "Representatives" &
    year == 2016) %>% 
  parliament_data(election_data = .,
    party_seats = .$seats,
    parl_rows = 4,
    type = "horseshoe")
```

### Plot of Australian parliament

``` r
au_rep <-ggplot(australia, aes(x, y, colour = party_short)) +
  geom_parliament_seats(size = 3.5) + 
  geom_highlight_government(government == 1, colour = "pink", size = 4) + 
  draw_majoritythreshold(n = 76, 
                         label = TRUE, 
                         linesize = 0.5,
                         type = 'horseshoe') + 
  theme_ggparliament() +
  theme(legend.position = 'bottom') + 
  labs(colour = NULL,
       title = "Australian Parliament",
       subtitle = "Government circled in pink.") +
  scale_colour_manual(values = australia$colour, 
                      limits = australia$party_short) 

au_rep
```

![](README_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->
