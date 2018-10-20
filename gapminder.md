---
title: "STAT545 Homework 05 Factor and figure management"
output:
  html_document:
    keep_md: true
---



## Overview

This Rmarkdown file has several goals as listed below. It will be used as a cheatsheet for future data frame reshaping and data wrangling.  
* Reorder a factor 
* Write and read data using R
* Improve a figure
* Make a plotly visual
* Implement visualization design principles


## Import data frame and the tidyverse pacakge

Gapminder data will be used in this homework, and the dataset will be explored using the "tidyverse" package. Figures will be plotted using the "ggplot2" package.


```r
library(gapminder)
library(tidyverse)
```

```
## ── Attaching packages ────────────────────────────────────────────────────────────────────────────── tidyverse 1.2.1 ──
```

```
## ✔ ggplot2 3.0.0     ✔ purrr   0.2.4
## ✔ tibble  1.4.1     ✔ dplyr   0.7.4
## ✔ tidyr   0.7.2     ✔ stringr 1.2.0
## ✔ readr   1.1.1     ✔ forcats 0.2.0
```

```
## ── Conflicts ───────────────────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
```

```r
library(ggplot2)
# use suppressMessages(library(tidyverse)) to generate a pdf file
```

## Part 1 Factor management
### Overview

This two parts has two goals. Dropping factor/levels, and reordering levels based on understanding of the dataset.

### Drop Oceania
First, let's figure out the structure of the Gapminder dataset before we drop anything out.


```r
str(gapminder)
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	1704 obs. of  6 variables:
##  $ country  : Factor w/ 142 levels "Afghanistan",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ continent: Factor w/ 5 levels "Africa","Americas",..: 3 3 3 3 3 3 3 3 3 3 ...
##  $ year     : int  1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
##  $ lifeExp  : num  28.8 30.3 32 34 36.1 ...
##  $ pop      : int  8425333 9240934 10267083 11537966 13079460 14880372 12881816 13867957 16317921 22227415 ...
##  $ gdpPercap: num  779 821 853 836 740 ...
```

It can be noticed that there are 142 levels in country, and 5 levels in continent. Now let's drop the continent "Oceania" out.


```r
gap_no_oce <- gapminder %>% 
  filter(continent != "Oceania")
```


```r
str(gap_no_oce)
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	1680 obs. of  6 variables:
##  $ country  : Factor w/ 142 levels "Afghanistan",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ continent: Factor w/ 5 levels "Africa","Americas",..: 3 3 3 3 3 3 3 3 3 3 ...
##  $ year     : int  1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
##  $ lifeExp  : num  28.8 30.3 32 34 36.1 ...
##  $ pop      : int  8425333 9240934 10267083 11537966 13079460 14880372 12881816 13867957 16317921 22227415 ...
##  $ gdpPercap: num  779 821 853 836 740 ...
```

After the Oceania has been dropped, the levels of country and continent remained unchanges. Now we want to remove all the unused levels. We could do that using either droplevels() or forcats::fct_drop(). 

Let's try droplevels() first.


```r
gap_no_oce %>% 
  droplevels() %>% 
  str()
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	1680 obs. of  6 variables:
##  $ country  : Factor w/ 140 levels "Afghanistan",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ continent: Factor w/ 4 levels "Africa","Americas",..: 3 3 3 3 3 3 3 3 3 3 ...
##  $ year     : int  1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
##  $ lifeExp  : num  28.8 30.3 32 34 36.1 ...
##  $ pop      : int  8425333 9240934 10267083 11537966 13079460 14880372 12881816 13867957 16317921 22227415 ...
##  $ gdpPercap: num  779 821 853 836 740 ...
```

It can be noticed that droplevels() removed unused levels in both country and continent. Oringinally, country has 142 levels and continent has 5 levels. The chunk above removed 2 levels of country and 1 level of continent, resulting in 140 levels in country and 4 levels in continent. 


```r
gap_no_oce %>% 
  mutate(continent = fct_drop(continent)) %>% 
  str()
```

```
## Classes 'tbl_df', 'tbl' and 'data.frame':	1680 obs. of  6 variables:
##  $ country  : Factor w/ 142 levels "Afghanistan",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ continent: Factor w/ 4 levels "Africa","Americas",..: 3 3 3 3 3 3 3 3 3 3 ...
##  $ year     : int  1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
##  $ lifeExp  : num  28.8 30.3 32 34 36.1 ...
##  $ pop      : int  8425333 9240934 10267083 11537966 13079460 14880372 12881816 13867957 16317921 22227415 ...
##  $ gdpPercap: num  779 821 853 836 740 ...
```

The function fct_drop() can specify levels of which factor to remove. Here I only removed unused levels in continent. Oringinally, country has 142 levels and continent has 5 levels. The chunk above only  1 level of continent, resulting in  4 levels in continent, and the levels in country remains unchanged.

### Reorder the levels of continent.

First, let's get a sub-dataset that only has data from the year 2002 to work with. 

```r
gap_2002 <- gapminder %>% 
  filter(year == 2002)
```

Let's plot the gdp per capital of different continent using a boxplot, and the mean value of the gdp per capital is indicated as a statistics summary. I found this [webpage](https://stackoverflow.com/questions/50112033/how-do-i-plot-the-mean-instead-of-the-median-with-geom-boxplot) useful. 


```r
gap_2002 %>% 
  ggplot(aes(x = continent, y = gdpPercap, fill=continent))+
  geom_boxplot(fatten = NULL) +
  stat_summary(fun.y = mean, geom = "errorbar", aes(ymax = ..y.., ymin = ..y..),
               width = 0.75, size = 1, linetype = "solid")
```

```
## Warning: Removed 1 rows containing missing values (geom_segment).

## Warning: Removed 1 rows containing missing values (geom_segment).

## Warning: Removed 1 rows containing missing values (geom_segment).

## Warning: Removed 1 rows containing missing values (geom_segment).

## Warning: Removed 1 rows containing missing values (geom_segment).
```

![](gapminder_files/figure-html/unnamed-chunk-8-1.png)<!-- -->


To reorder the levels of continent based on the gdpPercap in decreasing. Let's try arrange() first.  

```r
gap_2002 %>% 
  arrange(desc(gdpPercap)) %>% 
  ggplot(aes(x = continent, y = gdpPercap, fill=continent))+
  geom_boxplot(fatten = NULL) +
  stat_summary(fun.y = mean, geom = "errorbar", aes(ymax = ..y.., ymin = ..y..),
               width = 0.75, size = 1, linetype = "solid")
```

```
## Warning: Removed 1 rows containing missing values (geom_segment).

## Warning: Removed 1 rows containing missing values (geom_segment).

## Warning: Removed 1 rows containing missing values (geom_segment).

## Warning: Removed 1 rows containing missing values (geom_segment).

## Warning: Removed 1 rows containing missing values (geom_segment).
```

![](gapminder_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

It shows that arrange() won't change the order of the continent in the figure. Now let's try fct_reorder().


```r
gap_2002 %>% 
  mutate(continent = fct_reorder(continent, gdpPercap, fun = mean, .desc = TRUE)) %>% 
  ggplot(aes(x = continent, y = gdpPercap, fill=continent))+
  geom_boxplot(fatten = NULL) +
  stat_summary(fun.y = mean, geom = "errorbar", aes(ymax = ..y.., ymin = ..y..),
               width = 0.75, size = 1, linetype = "solid")
```

```
## Warning: Removed 1 rows containing missing values (geom_segment).

## Warning: Removed 1 rows containing missing values (geom_segment).

## Warning: Removed 1 rows containing missing values (geom_segment).

## Warning: Removed 1 rows containing missing values (geom_segment).

## Warning: Removed 1 rows containing missing values (geom_segment).
```

![](gapminder_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

It shows fct_reorder() can reorder the continent by gdpPercap in decreasing order. How about using factor reordering coupled with arrange()?


```r
gap_2002 %>% 
  arrange(desc(gdpPercap)) %>% 
  mutate(continent = factor(continent, unique(continent))) %>% 
  ggplot(aes(x = continent, y = gdpPercap, fill=continent))+
  geom_boxplot(fatten = NULL) +
  stat_summary(fun.y = mean, geom = "errorbar", aes(ymax = ..y.., ymin = ..y..),
               width = 0.75, size = 1, linetype = "solid")
```

```
## Warning: Removed 1 rows containing missing values (geom_segment).

## Warning: Removed 1 rows containing missing values (geom_segment).

## Warning: Removed 1 rows containing missing values (geom_segment).

## Warning: Removed 1 rows containing missing values (geom_segment).

## Warning: Removed 1 rows containing missing values (geom_segment).
```

![](gapminder_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

The above chunk also help us to reorder levels in a factor using arrange() and factor(). However, it only can reorder based on the largest value of each continent rather than the mean value. I would prefer not using this one because it is hard for us to specify the criteria of reordering. 

## Part 2 File I/O
### Overview
This part will write a sub-dataset of Gapminder to a .csv file and then read it back in.

First, let write gap_2002 onto a .csv. The gap_2002 dataset will be reordered by gdpPercap to make sure it is not ordered alphabetically. 


```r
gap_2002 %>% 
  arrange(desc(gdpPercap)) %>% 
  write_csv("gap_2002.csv")
```

Then, read the gap_2002.csv back in R.


```r
gap_2002_new <- read_csv("gap_2002.csv")
```

```
## Parsed with column specification:
## cols(
##   country = col_character(),
##   continent = col_character(),
##   year = col_integer(),
##   lifeExp = col_double(),
##   pop = col_integer(),
##   gdpPercap = col_double()
## )
```

```r
head(gap_2002_new)
```

```
## # A tibble: 6 x 6
##   country       continent  year lifeExp       pop gdpPercap
##   <chr>         <chr>     <int>   <dbl>     <int>     <dbl>
## 1 Norway        Europe     2002    79.0   4535591     44684
## 2 United States Americas   2002    77.3 287675526     39097
## 3 Singapore     Asia       2002    78.8   4197776     36023
## 4 Kuwait        Asia       2002    76.9   2111561     35110
## 5 Switzerland   Europe     2002    80.6   7361757     34481
## 6 Ireland       Europe     2002    77.8   3879155     34077
```

It can be notice that the dataframe that is read from the .csv file has the same order with the arranged gap_2002 dataframe, which is ordering by decreasing of gdpPercap.

## Part 3 Visualization design.
In this part, several graphs will be ploted and compared with each other. Scale of one axis will be manipulated and the font size of axis will be changed using theme(). Some of the graphs will be plotted using ggplot2, and some will be plotted using ploty.

Using the Gapminder dataset, and plot the trend of gdpPercap of a few countries, and some of them have a large gdpPercap value that is greater than 40000 in the year 2007, and the others have a small gdpPercap value that is smaller thatn 1000 in the year 2007. Using the country color sheme. 

First, let's get two groups of countries.


```r
country_large_gdp <- gapminder %>% 
  filter(year == "2007", gdpPercap >= 40000) %>% 
  select(country) %>% 
  droplevels()
country_small_gdp <- gapminder %>% 
  filter(year == "2007", gdpPercap <= 500) %>% 
  select(country) %>% 
  droplevels()
```

Then let's make the plot.


```r
dat1 <- gapminder %>% 
  filter(country %in% country_large_gdp$country)
dat2 <- gapminder %>% 
  filter(country %in% country_small_gdp$country)
dat_all <- bind_rows(dat1, dat2)
p_country_col <- dat_all %>% 
  ggplot(aes(year, gdpPercap, col = country))+
  scale_colour_manual(values = country_colors) +
  geom_line()
p_country_col
```

![](gapminder_files/figure-html/unnamed-chunk-15-1.png)<!-- -->


In the first attempt, I found that the trend of gdpPercap of countries that has low gdpPercap is not illustrative, and I think the reason is the gap bwteen the gdpPercap values of two groups is large. So let's try to log the y axis.


```r
p_country_col_log <- p_country_col + 
  scale_y_log10()
p_country_col_log
```

![](gapminder_files/figure-html/unnamed-chunk-16-1.png)<!-- -->

Compare this figure to the first one, it can be noticed that the trend of both groups are clearly shown. The graph itself is more illustrative. However, the order of the legend does not correspond the order of the lines. Let's make the legend is in order of the last gdpPercap.


```r
p_reorder <- dat_all %>% 
  mutate(country=fct_reorder2(country, year, gdpPercap)) %>% 
  ggplot(aes(year, gdpPercap, col = country))+
  scale_colour_manual(values = country_colors) +
  geom_line() +
  scale_y_log10()
p_reorder
```

![](gapminder_files/figure-html/unnamed-chunk-17-1.png)<!-- -->

It can be noticed that the order of the legend matches with the order of gdpPercap of 2007. Now let's make the text of axis x and y bigger, and give the graph a title.


```r
p_title <- p_reorder + 
  labs(x = "Year",
  y = "GDP Per Capita",
  title = "Change of GDP Per Capita over time") +
  theme_bw() +
  theme(axis.text = element_text(size = 16))
p_title
```

![](gapminder_files/figure-html/unnamed-chunk-18-1.png)<!-- -->

It can be notice that the labels in this plot is clearer than the previous ones, and the title of the graph makes it easy to understand.


To conver the graphs to a 'plotly' plot.


```r
library(plotly)
```

```
## 
## Attaching package: 'plotly'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     last_plot
```

```
## The following object is masked from 'package:stats':
## 
##     filter
```

```
## The following object is masked from 'package:graphics':
## 
##     layout
```

```r
ggplotly(p_title)
```

<!--html_preserve--><div id="htmlwidget-2c2476c00dd2de1d8df1" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-2c2476c00dd2de1d8df1">{"x":{"data":[{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[4.00412446561185,4.06647400906966,4.12873524872317,4.21383310947184,4.27795411820419,4.36756741361793,4.41993321264512,4.49887511143166,4.53104007265859,4.61577297830881,4.6501518025891,4.69335042800253],"text":["year: 1952<br />gdpPercap:  10095.4217<br />country: Norway","year: 1957<br />gdpPercap:  11653.9730<br />country: Norway","year: 1962<br />gdpPercap:  13450.4015<br />country: Norway","year: 1967<br />gdpPercap:  16361.8765<br />country: Norway","year: 1972<br />gdpPercap:  18965.0555<br />country: Norway","year: 1977<br />gdpPercap:  23311.3494<br />country: Norway","year: 1982<br />gdpPercap:  26298.6353<br />country: Norway","year: 1987<br />gdpPercap:  31540.9748<br />country: Norway","year: 1992<br />gdpPercap:  33965.6611<br />country: Norway","year: 1997<br />gdpPercap:  41283.1643<br />country: Norway","year: 2002<br />gdpPercap:  44683.9753<br />country: Norway","year: 2007<br />gdpPercap:  49357.1902<br />country: Norway"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(185,225,136,1)","dash":"solid"},"hoveron":"points","name":"Norway","legendgroup":"Norway","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[5.03495857498782,5.05508436784766,4.97981283941297,4.90792105261074,5.0388103158678,4.7728017852095,4.49629344891365,4.4489910677682,4.54323488420159,4.60531172711223,4.54543213642878,4.67492531406996],"text":["year: 1952<br />gdpPercap: 108382.3529<br />country: Kuwait","year: 1957<br />gdpPercap: 113523.1329<br />country: Kuwait","year: 1962<br />gdpPercap:  95458.1118<br />country: Kuwait","year: 1967<br />gdpPercap:  80894.8833<br />country: Kuwait","year: 1972<br />gdpPercap: 109347.8670<br />country: Kuwait","year: 1977<br />gdpPercap:  59265.4771<br />country: Kuwait","year: 1982<br />gdpPercap:  31354.0357<br />country: Kuwait","year: 1987<br />gdpPercap:  28118.4300<br />country: Kuwait","year: 1992<br />gdpPercap:  34932.9196<br />country: Kuwait","year: 1997<br />gdpPercap:  40300.6200<br />country: Kuwait","year: 2002<br />gdpPercap:  35110.1057<br />country: Kuwait","year: 2007<br />gdpPercap:  47306.9898<br />country: Kuwait"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(226,206,228,1)","dash":"solid"},"hoveron":"points","name":"Kuwait","legendgroup":"Kuwait","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[3.3645769259938,3.45379280883569,3.5652260934194,3.6970041611318,3.93438512609999,4.04960907918865,4.18096156423903,4.27557693735977,4.39392409898215,4.52529722858871,4.5565811487931,4.67341887079622],"text":["year: 1952<br />gdpPercap:   2315.1382<br />country: Singapore","year: 1957<br />gdpPercap:   2843.1044<br />country: Singapore","year: 1962<br />gdpPercap:   3674.7356<br />country: Singapore","year: 1967<br />gdpPercap:   4977.4185<br />country: Singapore","year: 1972<br />gdpPercap:   8597.7562<br />country: Singapore","year: 1977<br />gdpPercap:  11210.0895<br />country: Singapore","year: 1982<br />gdpPercap:  15169.1611<br />country: Singapore","year: 1987<br />gdpPercap:  18861.5308<br />country: Singapore","year: 1992<br />gdpPercap:  24769.8912<br />country: Singapore","year: 1997<br />gdpPercap:  33519.4766<br />country: Singapore","year: 2002<br />gdpPercap:  36023.1054<br />country: Singapore","year: 2007<br />gdpPercap:  47143.1796<br />country: Singapore"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(203,176,213,1)","dash":"solid"},"hoveron":"points","name":"Singapore","legendgroup":"Singapore","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[4.14583267954377,4.17164242694385,4.20879450331745,4.29071037250177,4.33857672354045,4.38152357914239,4.3981060362026,4.47544382010215,4.5052033422329,4.55348777266477,4.59214454010295,4.632979883279],"text":["year: 1952<br />gdpPercap:  13990.4821<br />country: United States","year: 1957<br />gdpPercap:  14847.1271<br />country: United States","year: 1962<br />gdpPercap:  16173.1459<br />country: United States","year: 1967<br />gdpPercap:  19530.3656<br />country: United States","year: 1972<br />gdpPercap:  21806.0359<br />country: United States","year: 1977<br />gdpPercap:  24072.6321<br />country: United States","year: 1982<br />gdpPercap:  25009.5591<br />country: United States","year: 1987<br />gdpPercap:  29884.3504<br />country: United States","year: 1992<br />gdpPercap:  32003.9322<br />country: United States","year: 1997<br />gdpPercap:  35767.4330<br />country: United States","year: 2002<br />gdpPercap:  39097.0995<br />country: United States","year: 2007<br />gdpPercap:  42951.6531<br />country: United States"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(165,0,38,1)","dash":"solid"},"hoveron":"points","name":"United States","legendgroup":"United States","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[3.71686109021475,3.74811650770658,3.82161814697266,3.88397747306288,3.97912812108504,4.04731308089082,4.10100158542625,4.14216620766251,4.24449521672678,4.38955495174527,4.53246198371447,4.60933819969144],"text":["year: 1952<br />gdpPercap:   5210.2803<br />country: Ireland","year: 1957<br />gdpPercap:   5599.0779<br />country: Ireland","year: 1962<br />gdpPercap:   6631.5973<br />country: Ireland","year: 1967<br />gdpPercap:   7655.5690<br />country: Ireland","year: 1972<br />gdpPercap:   9530.7729<br />country: Ireland","year: 1977<br />gdpPercap:  11150.9811<br />country: Ireland","year: 1982<br />gdpPercap:  12618.3214<br />country: Ireland","year: 1987<br />gdpPercap:  13872.8665<br />country: Ireland","year: 1992<br />gdpPercap:  17558.8155<br />country: Ireland","year: 1997<br />gdpPercap:  24521.9471<br />country: Ireland","year: 2002<br />gdpPercap:  34077.0494<br />country: Ireland","year: 2007<br />gdpPercap:  40675.9964<br />country: Ireland"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(204,233,167,1)","dash":"solid"},"hoveron":"points","name":"Ireland","legendgroup":"Ireland","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[2.6094707348536,2.71497005472619,2.72203485911935,2.75571868821708,2.90274359447453,2.83606300605042,2.89699720519387,2.84890145668142,2.84099685568797,2.89897184798542,2.82739423308876,2.67182915731664],"text":["year: 1952<br />gdpPercap:    406.8841<br />country: Zimbabwe","year: 1957<br />gdpPercap:    518.7643<br />country: Zimbabwe","year: 1962<br />gdpPercap:    527.2722<br />country: Zimbabwe","year: 1967<br />gdpPercap:    569.7951<br />country: Zimbabwe","year: 1972<br />gdpPercap:    799.3622<br />country: Zimbabwe","year: 1977<br />gdpPercap:    685.5877<br />country: Zimbabwe","year: 1982<br />gdpPercap:    788.8550<br />country: Zimbabwe","year: 1987<br />gdpPercap:    706.1573<br />country: Zimbabwe","year: 1992<br />gdpPercap:    693.4208<br />country: Zimbabwe","year: 1997<br />gdpPercap:    792.4500<br />country: Zimbabwe","year: 2002<br />gdpPercap:    672.0386<br />country: Zimbabwe","year: 2007<br />gdpPercap:    469.7093<br />country: Zimbabwe"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(204,111,13,1)","dash":"solid"},"hoveron":"points","name":"Zimbabwe","legendgroup":"Zimbabwe","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[2.5305793268568,2.57928573342662,2.55047690303059,2.61592640520288,2.66661110417941,2.74515544499733,2.74788021332769,2.79366386151668,2.80051079288172,2.66568898634557,2.64972760326105,2.63353984732021],"text":["year: 1952<br />gdpPercap:    339.2965<br />country: Burundi","year: 1957<br />gdpPercap:    379.5646<br />country: Burundi","year: 1962<br />gdpPercap:    355.2032<br />country: Burundi","year: 1967<br />gdpPercap:    412.9775<br />country: Burundi","year: 1972<br />gdpPercap:    464.0995<br />country: Burundi","year: 1977<br />gdpPercap:    556.1033<br />country: Burundi","year: 1982<br />gdpPercap:    559.6032<br />country: Burundi","year: 1987<br />gdpPercap:    621.8188<br />country: Burundi","year: 1992<br />gdpPercap:    631.6999<br />country: Burundi","year: 1997<br />gdpPercap:    463.1151<br />country: Burundi","year: 2002<br />gdpPercap:    446.4035<br />country: Burundi","year: 2007<br />gdpPercap:    430.0707<br />country: Burundi"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(231,144,41,1)","dash":"solid"},"hoveron":"points","name":"Burundi","legendgroup":"Burundi","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[2.76010041014319,2.79307061233617,2.80222292500175,2.8534570617051,2.90471849473935,2.8063987207202,2.75754752664772,2.7042482283709,2.80388226962819,2.78474132397179,2.72548886123795,2.61753222691212],"text":["year: 1952<br />gdpPercap:    575.5730<br />country: Liberia","year: 1957<br />gdpPercap:    620.9700<br />country: Liberia","year: 1962<br />gdpPercap:    634.1952<br />country: Liberia","year: 1967<br />gdpPercap:    713.6036<br />country: Liberia","year: 1972<br />gdpPercap:    803.0055<br />country: Liberia","year: 1977<br />gdpPercap:    640.3224<br />country: Liberia","year: 1982<br />gdpPercap:    572.1996<br />country: Liberia","year: 1987<br />gdpPercap:    506.1139<br />country: Liberia","year: 1992<br />gdpPercap:    636.6229<br />country: Liberia","year: 1997<br />gdpPercap:    609.1740<br />country: Liberia","year: 2002<br />gdpPercap:    531.4824<br />country: Liberia","year: 2007<br />gdpPercap:    414.5073<br />country: Liberia"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(252,182,97,1)","dash":"solid"},"hoveron":"points","name":"Liberia","legendgroup":"Liberia","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[2.89239645809614,2.95706119338084,2.95246048690214,2.93530228406906,2.95659870133898,2.90078062155373,2.82849737175522,2.82786972355356,2.66059911189776,2.49441679392038,2.38231585779586,2.44334414004637],"text":["year: 1952<br />gdpPercap:    780.5423<br />country: Congo, Dem. Rep.","year: 1957<br />gdpPercap:    905.8602<br />country: Congo, Dem. Rep.","year: 1962<br />gdpPercap:    896.3146<br />country: Congo, Dem. Rep.","year: 1967<br />gdpPercap:    861.5932<br />country: Congo, Dem. Rep.","year: 1972<br />gdpPercap:    904.8961<br />country: Congo, Dem. Rep.","year: 1977<br />gdpPercap:    795.7573<br />country: Congo, Dem. Rep.","year: 1982<br />gdpPercap:    673.7478<br />country: Congo, Dem. Rep.","year: 1987<br />gdpPercap:    672.7748<br />country: Congo, Dem. Rep.","year: 1992<br />gdpPercap:    457.7192<br />country: Congo, Dem. Rep.","year: 1997<br />gdpPercap:    312.1884<br />country: Congo, Dem. Rep.","year: 2002<br />gdpPercap:    241.1659<br />country: Congo, Dem. Rep.","year: 2007<br />gdpPercap:    277.5519<br />country: Congo, Dem. Rep."],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(139,65,7,1)","dash":"solid"},"hoveron":"points","name":"Congo, Dem. Rep.","legendgroup":"Congo, Dem. Rep.","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":43.7625570776256,"r":7.30593607305936,"b":49.7467828974678,"l":78.7048567870486},"plot_bgcolor":"rgba(255,255,255,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"title":"Change of GDP Per Capita over time","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":17.5342465753425},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[1949.25,2009.75],"tickmode":"array","ticktext":["1950","1960","1970","1980","1990","2000"],"tickvals":[1950,1960,1970,1980,1990,2000],"categoryorder":"array","categoryarray":["1950","1960","1970","1980","1990","2000"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":21.2536322125363},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"y","title":"Year","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[2.24867743229327,5.18872279335025],"tickmode":"array","ticktext":["1e+03","1e+04","1e+05"],"tickvals":[3,4,5],"categoryorder":"array","categoryarray":["1e+03","1e+04","1e+05"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":21.2536322125363},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(235,235,235,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"x","title":"GDP Per Capita","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":"transparent","line":{"color":"rgba(51,51,51,1)","width":0.66417600664176,"linetype":"solid"},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":true,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895},"y":0.913385826771654},"annotations":[{"text":"country","x":1.02,"y":1,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"left","yanchor":"bottom","legendTitle":true}],"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","modeBarButtonsToAdd":[{"name":"Collaborate","icon":{"width":1000,"ascent":500,"descent":-50,"path":"M487 375c7-10 9-23 5-36l-79-259c-3-12-11-23-22-31-11-8-22-12-35-12l-263 0c-15 0-29 5-43 15-13 10-23 23-28 37-5 13-5 25-1 37 0 0 0 3 1 7 1 5 1 8 1 11 0 2 0 4-1 6 0 3-1 5-1 6 1 2 2 4 3 6 1 2 2 4 4 6 2 3 4 5 5 7 5 7 9 16 13 26 4 10 7 19 9 26 0 2 0 5 0 9-1 4-1 6 0 8 0 2 2 5 4 8 3 3 5 5 5 7 4 6 8 15 12 26 4 11 7 19 7 26 1 1 0 4 0 9-1 4-1 7 0 8 1 2 3 5 6 8 4 4 6 6 6 7 4 5 8 13 13 24 4 11 7 20 7 28 1 1 0 4 0 7-1 3-1 6-1 7 0 2 1 4 3 6 1 1 3 4 5 6 2 3 3 5 5 6 1 2 3 5 4 9 2 3 3 7 5 10 1 3 2 6 4 10 2 4 4 7 6 9 2 3 4 5 7 7 3 2 7 3 11 3 3 0 8 0 13-1l0-1c7 2 12 2 14 2l218 0c14 0 25-5 32-16 8-10 10-23 6-37l-79-259c-7-22-13-37-20-43-7-7-19-10-37-10l-248 0c-5 0-9-2-11-5-2-3-2-7 0-12 4-13 18-20 41-20l264 0c5 0 10 2 16 5 5 3 8 6 10 11l85 282c2 5 2 10 2 17 7-3 13-7 17-13z m-304 0c-1-3-1-5 0-7 1-1 3-2 6-2l174 0c2 0 4 1 7 2 2 2 4 4 5 7l6 18c0 3 0 5-1 7-1 1-3 2-6 2l-173 0c-3 0-5-1-8-2-2-2-4-4-4-7z m-24-73c-1-3-1-5 0-7 2-2 3-2 6-2l174 0c2 0 5 0 7 2 3 2 4 4 5 7l6 18c1 2 0 5-1 6-1 2-3 3-5 3l-174 0c-3 0-5-1-7-3-3-1-4-4-5-6z"},"click":"function(gd) { \n        // is this being viewed in RStudio?\n        if (location.search == '?viewer_pane=1') {\n          alert('To learn about plotly for collaboration, visit:\\n https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html');\n        } else {\n          window.open('https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html', '_blank');\n        }\n      }"}],"cloud":false},"source":"A","attrs":{"2130dc2de59":{"x":{},"y":{},"colour":{},"type":"scatter"}},"cur_data":"2130dc2de59","visdat":{"2130dc2de59":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"base_url":"https://plot.ly"},"evals":["config.modeBarButtonsToAdd.0.click"],"jsHooks":[]}</script><!--/html_preserve-->


```r
ggplotly(p_reorder)
```

<!--html_preserve--><div id="htmlwidget-742f7bebc2a774a9ea25" style="width:672px;height:480px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-742f7bebc2a774a9ea25">{"x":{"data":[{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[4.00412446561185,4.06647400906966,4.12873524872317,4.21383310947184,4.27795411820419,4.36756741361793,4.41993321264512,4.49887511143166,4.53104007265859,4.61577297830881,4.6501518025891,4.69335042800253],"text":["year: 1952<br />gdpPercap:  10095.4217<br />country: Norway","year: 1957<br />gdpPercap:  11653.9730<br />country: Norway","year: 1962<br />gdpPercap:  13450.4015<br />country: Norway","year: 1967<br />gdpPercap:  16361.8765<br />country: Norway","year: 1972<br />gdpPercap:  18965.0555<br />country: Norway","year: 1977<br />gdpPercap:  23311.3494<br />country: Norway","year: 1982<br />gdpPercap:  26298.6353<br />country: Norway","year: 1987<br />gdpPercap:  31540.9748<br />country: Norway","year: 1992<br />gdpPercap:  33965.6611<br />country: Norway","year: 1997<br />gdpPercap:  41283.1643<br />country: Norway","year: 2002<br />gdpPercap:  44683.9753<br />country: Norway","year: 2007<br />gdpPercap:  49357.1902<br />country: Norway"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(185,225,136,1)","dash":"solid"},"hoveron":"points","name":"Norway","legendgroup":"Norway","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[5.03495857498782,5.05508436784766,4.97981283941297,4.90792105261074,5.0388103158678,4.7728017852095,4.49629344891365,4.4489910677682,4.54323488420159,4.60531172711223,4.54543213642878,4.67492531406996],"text":["year: 1952<br />gdpPercap: 108382.3529<br />country: Kuwait","year: 1957<br />gdpPercap: 113523.1329<br />country: Kuwait","year: 1962<br />gdpPercap:  95458.1118<br />country: Kuwait","year: 1967<br />gdpPercap:  80894.8833<br />country: Kuwait","year: 1972<br />gdpPercap: 109347.8670<br />country: Kuwait","year: 1977<br />gdpPercap:  59265.4771<br />country: Kuwait","year: 1982<br />gdpPercap:  31354.0357<br />country: Kuwait","year: 1987<br />gdpPercap:  28118.4300<br />country: Kuwait","year: 1992<br />gdpPercap:  34932.9196<br />country: Kuwait","year: 1997<br />gdpPercap:  40300.6200<br />country: Kuwait","year: 2002<br />gdpPercap:  35110.1057<br />country: Kuwait","year: 2007<br />gdpPercap:  47306.9898<br />country: Kuwait"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(226,206,228,1)","dash":"solid"},"hoveron":"points","name":"Kuwait","legendgroup":"Kuwait","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[3.3645769259938,3.45379280883569,3.5652260934194,3.6970041611318,3.93438512609999,4.04960907918865,4.18096156423903,4.27557693735977,4.39392409898215,4.52529722858871,4.5565811487931,4.67341887079622],"text":["year: 1952<br />gdpPercap:   2315.1382<br />country: Singapore","year: 1957<br />gdpPercap:   2843.1044<br />country: Singapore","year: 1962<br />gdpPercap:   3674.7356<br />country: Singapore","year: 1967<br />gdpPercap:   4977.4185<br />country: Singapore","year: 1972<br />gdpPercap:   8597.7562<br />country: Singapore","year: 1977<br />gdpPercap:  11210.0895<br />country: Singapore","year: 1982<br />gdpPercap:  15169.1611<br />country: Singapore","year: 1987<br />gdpPercap:  18861.5308<br />country: Singapore","year: 1992<br />gdpPercap:  24769.8912<br />country: Singapore","year: 1997<br />gdpPercap:  33519.4766<br />country: Singapore","year: 2002<br />gdpPercap:  36023.1054<br />country: Singapore","year: 2007<br />gdpPercap:  47143.1796<br />country: Singapore"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(203,176,213,1)","dash":"solid"},"hoveron":"points","name":"Singapore","legendgroup":"Singapore","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[4.14583267954377,4.17164242694385,4.20879450331745,4.29071037250177,4.33857672354045,4.38152357914239,4.3981060362026,4.47544382010215,4.5052033422329,4.55348777266477,4.59214454010295,4.632979883279],"text":["year: 1952<br />gdpPercap:  13990.4821<br />country: United States","year: 1957<br />gdpPercap:  14847.1271<br />country: United States","year: 1962<br />gdpPercap:  16173.1459<br />country: United States","year: 1967<br />gdpPercap:  19530.3656<br />country: United States","year: 1972<br />gdpPercap:  21806.0359<br />country: United States","year: 1977<br />gdpPercap:  24072.6321<br />country: United States","year: 1982<br />gdpPercap:  25009.5591<br />country: United States","year: 1987<br />gdpPercap:  29884.3504<br />country: United States","year: 1992<br />gdpPercap:  32003.9322<br />country: United States","year: 1997<br />gdpPercap:  35767.4330<br />country: United States","year: 2002<br />gdpPercap:  39097.0995<br />country: United States","year: 2007<br />gdpPercap:  42951.6531<br />country: United States"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(165,0,38,1)","dash":"solid"},"hoveron":"points","name":"United States","legendgroup":"United States","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[3.71686109021475,3.74811650770658,3.82161814697266,3.88397747306288,3.97912812108504,4.04731308089082,4.10100158542625,4.14216620766251,4.24449521672678,4.38955495174527,4.53246198371447,4.60933819969144],"text":["year: 1952<br />gdpPercap:   5210.2803<br />country: Ireland","year: 1957<br />gdpPercap:   5599.0779<br />country: Ireland","year: 1962<br />gdpPercap:   6631.5973<br />country: Ireland","year: 1967<br />gdpPercap:   7655.5690<br />country: Ireland","year: 1972<br />gdpPercap:   9530.7729<br />country: Ireland","year: 1977<br />gdpPercap:  11150.9811<br />country: Ireland","year: 1982<br />gdpPercap:  12618.3214<br />country: Ireland","year: 1987<br />gdpPercap:  13872.8665<br />country: Ireland","year: 1992<br />gdpPercap:  17558.8155<br />country: Ireland","year: 1997<br />gdpPercap:  24521.9471<br />country: Ireland","year: 2002<br />gdpPercap:  34077.0494<br />country: Ireland","year: 2007<br />gdpPercap:  40675.9964<br />country: Ireland"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(204,233,167,1)","dash":"solid"},"hoveron":"points","name":"Ireland","legendgroup":"Ireland","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[2.6094707348536,2.71497005472619,2.72203485911935,2.75571868821708,2.90274359447453,2.83606300605042,2.89699720519387,2.84890145668142,2.84099685568797,2.89897184798542,2.82739423308876,2.67182915731664],"text":["year: 1952<br />gdpPercap:    406.8841<br />country: Zimbabwe","year: 1957<br />gdpPercap:    518.7643<br />country: Zimbabwe","year: 1962<br />gdpPercap:    527.2722<br />country: Zimbabwe","year: 1967<br />gdpPercap:    569.7951<br />country: Zimbabwe","year: 1972<br />gdpPercap:    799.3622<br />country: Zimbabwe","year: 1977<br />gdpPercap:    685.5877<br />country: Zimbabwe","year: 1982<br />gdpPercap:    788.8550<br />country: Zimbabwe","year: 1987<br />gdpPercap:    706.1573<br />country: Zimbabwe","year: 1992<br />gdpPercap:    693.4208<br />country: Zimbabwe","year: 1997<br />gdpPercap:    792.4500<br />country: Zimbabwe","year: 2002<br />gdpPercap:    672.0386<br />country: Zimbabwe","year: 2007<br />gdpPercap:    469.7093<br />country: Zimbabwe"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(204,111,13,1)","dash":"solid"},"hoveron":"points","name":"Zimbabwe","legendgroup":"Zimbabwe","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[2.5305793268568,2.57928573342662,2.55047690303059,2.61592640520288,2.66661110417941,2.74515544499733,2.74788021332769,2.79366386151668,2.80051079288172,2.66568898634557,2.64972760326105,2.63353984732021],"text":["year: 1952<br />gdpPercap:    339.2965<br />country: Burundi","year: 1957<br />gdpPercap:    379.5646<br />country: Burundi","year: 1962<br />gdpPercap:    355.2032<br />country: Burundi","year: 1967<br />gdpPercap:    412.9775<br />country: Burundi","year: 1972<br />gdpPercap:    464.0995<br />country: Burundi","year: 1977<br />gdpPercap:    556.1033<br />country: Burundi","year: 1982<br />gdpPercap:    559.6032<br />country: Burundi","year: 1987<br />gdpPercap:    621.8188<br />country: Burundi","year: 1992<br />gdpPercap:    631.6999<br />country: Burundi","year: 1997<br />gdpPercap:    463.1151<br />country: Burundi","year: 2002<br />gdpPercap:    446.4035<br />country: Burundi","year: 2007<br />gdpPercap:    430.0707<br />country: Burundi"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(231,144,41,1)","dash":"solid"},"hoveron":"points","name":"Burundi","legendgroup":"Burundi","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[2.76010041014319,2.79307061233617,2.80222292500175,2.8534570617051,2.90471849473935,2.8063987207202,2.75754752664772,2.7042482283709,2.80388226962819,2.78474132397179,2.72548886123795,2.61753222691212],"text":["year: 1952<br />gdpPercap:    575.5730<br />country: Liberia","year: 1957<br />gdpPercap:    620.9700<br />country: Liberia","year: 1962<br />gdpPercap:    634.1952<br />country: Liberia","year: 1967<br />gdpPercap:    713.6036<br />country: Liberia","year: 1972<br />gdpPercap:    803.0055<br />country: Liberia","year: 1977<br />gdpPercap:    640.3224<br />country: Liberia","year: 1982<br />gdpPercap:    572.1996<br />country: Liberia","year: 1987<br />gdpPercap:    506.1139<br />country: Liberia","year: 1992<br />gdpPercap:    636.6229<br />country: Liberia","year: 1997<br />gdpPercap:    609.1740<br />country: Liberia","year: 2002<br />gdpPercap:    531.4824<br />country: Liberia","year: 2007<br />gdpPercap:    414.5073<br />country: Liberia"],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(252,182,97,1)","dash":"solid"},"hoveron":"points","name":"Liberia","legendgroup":"Liberia","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null},{"x":[1952,1957,1962,1967,1972,1977,1982,1987,1992,1997,2002,2007],"y":[2.89239645809614,2.95706119338084,2.95246048690214,2.93530228406906,2.95659870133898,2.90078062155373,2.82849737175522,2.82786972355356,2.66059911189776,2.49441679392038,2.38231585779586,2.44334414004637],"text":["year: 1952<br />gdpPercap:    780.5423<br />country: Congo, Dem. Rep.","year: 1957<br />gdpPercap:    905.8602<br />country: Congo, Dem. Rep.","year: 1962<br />gdpPercap:    896.3146<br />country: Congo, Dem. Rep.","year: 1967<br />gdpPercap:    861.5932<br />country: Congo, Dem. Rep.","year: 1972<br />gdpPercap:    904.8961<br />country: Congo, Dem. Rep.","year: 1977<br />gdpPercap:    795.7573<br />country: Congo, Dem. Rep.","year: 1982<br />gdpPercap:    673.7478<br />country: Congo, Dem. Rep.","year: 1987<br />gdpPercap:    672.7748<br />country: Congo, Dem. Rep.","year: 1992<br />gdpPercap:    457.7192<br />country: Congo, Dem. Rep.","year: 1997<br />gdpPercap:    312.1884<br />country: Congo, Dem. Rep.","year: 2002<br />gdpPercap:    241.1659<br />country: Congo, Dem. Rep.","year: 2007<br />gdpPercap:    277.5519<br />country: Congo, Dem. Rep."],"type":"scatter","mode":"lines","line":{"width":1.88976377952756,"color":"rgba(139,65,7,1)","dash":"solid"},"hoveron":"points","name":"Congo, Dem. Rep.","legendgroup":"Congo, Dem. Rep.","showlegend":true,"xaxis":"x","yaxis":"y","hoverinfo":"text","frame":null}],"layout":{"margin":{"t":26.2283105022831,"r":7.30593607305936,"b":40.1826484018265,"l":54.7945205479452},"plot_bgcolor":"rgba(235,235,235,1)","paper_bgcolor":"rgba(255,255,255,1)","font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[1949.25,2009.75],"tickmode":"array","ticktext":["1950","1960","1970","1980","1990","2000"],"tickvals":[1950,1960,1970,1980,1990,2000],"categoryorder":"array","categoryarray":["1950","1960","1970","1980","1990","2000"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(255,255,255,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"y","title":"year","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"yaxis":{"domain":[0,1],"automargin":true,"type":"linear","autorange":false,"range":[2.24867743229327,5.18872279335025],"tickmode":"array","ticktext":["1e+03","1e+04","1e+05"],"tickvals":[3,4,5],"categoryorder":"array","categoryarray":["1e+03","1e+04","1e+05"],"nticks":null,"ticks":"outside","tickcolor":"rgba(51,51,51,1)","ticklen":3.65296803652968,"tickwidth":0.66417600664176,"showticklabels":true,"tickfont":{"color":"rgba(77,77,77,1)","family":"","size":11.689497716895},"tickangle":-0,"showline":false,"linecolor":null,"linewidth":0,"showgrid":true,"gridcolor":"rgba(255,255,255,1)","gridwidth":0.66417600664176,"zeroline":false,"anchor":"x","title":"gdpPercap","titlefont":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"hoverformat":".2f"},"shapes":[{"type":"rect","fillcolor":null,"line":{"color":null,"width":0,"linetype":[]},"yref":"paper","xref":"paper","x0":0,"x1":1,"y0":0,"y1":1}],"showlegend":true,"legend":{"bgcolor":"rgba(255,255,255,1)","bordercolor":"transparent","borderwidth":1.88976377952756,"font":{"color":"rgba(0,0,0,1)","family":"","size":11.689497716895},"y":0.913385826771654},"annotations":[{"text":"country","x":1.02,"y":1,"showarrow":false,"ax":0,"ay":0,"font":{"color":"rgba(0,0,0,1)","family":"","size":14.6118721461187},"xref":"paper","yref":"paper","textangle":-0,"xanchor":"left","yanchor":"bottom","legendTitle":true}],"hovermode":"closest","barmode":"relative"},"config":{"doubleClick":"reset","modeBarButtonsToAdd":[{"name":"Collaborate","icon":{"width":1000,"ascent":500,"descent":-50,"path":"M487 375c7-10 9-23 5-36l-79-259c-3-12-11-23-22-31-11-8-22-12-35-12l-263 0c-15 0-29 5-43 15-13 10-23 23-28 37-5 13-5 25-1 37 0 0 0 3 1 7 1 5 1 8 1 11 0 2 0 4-1 6 0 3-1 5-1 6 1 2 2 4 3 6 1 2 2 4 4 6 2 3 4 5 5 7 5 7 9 16 13 26 4 10 7 19 9 26 0 2 0 5 0 9-1 4-1 6 0 8 0 2 2 5 4 8 3 3 5 5 5 7 4 6 8 15 12 26 4 11 7 19 7 26 1 1 0 4 0 9-1 4-1 7 0 8 1 2 3 5 6 8 4 4 6 6 6 7 4 5 8 13 13 24 4 11 7 20 7 28 1 1 0 4 0 7-1 3-1 6-1 7 0 2 1 4 3 6 1 1 3 4 5 6 2 3 3 5 5 6 1 2 3 5 4 9 2 3 3 7 5 10 1 3 2 6 4 10 2 4 4 7 6 9 2 3 4 5 7 7 3 2 7 3 11 3 3 0 8 0 13-1l0-1c7 2 12 2 14 2l218 0c14 0 25-5 32-16 8-10 10-23 6-37l-79-259c-7-22-13-37-20-43-7-7-19-10-37-10l-248 0c-5 0-9-2-11-5-2-3-2-7 0-12 4-13 18-20 41-20l264 0c5 0 10 2 16 5 5 3 8 6 10 11l85 282c2 5 2 10 2 17 7-3 13-7 17-13z m-304 0c-1-3-1-5 0-7 1-1 3-2 6-2l174 0c2 0 4 1 7 2 2 2 4 4 5 7l6 18c0 3 0 5-1 7-1 1-3 2-6 2l-173 0c-3 0-5-1-8-2-2-2-4-4-4-7z m-24-73c-1-3-1-5 0-7 2-2 3-2 6-2l174 0c2 0 5 0 7 2 3 2 4 4 5 7l6 18c1 2 0 5-1 6-1 2-3 3-5 3l-174 0c-3 0-5-1-7-3-3-1-4-4-5-6z"},"click":"function(gd) { \n        // is this being viewed in RStudio?\n        if (location.search == '?viewer_pane=1') {\n          alert('To learn about plotly for collaboration, visit:\\n https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html');\n        } else {\n          window.open('https://cpsievert.github.io/plotly_book/plot-ly-for-collaboration.html', '_blank');\n        }\n      }"}],"cloud":false},"source":"A","attrs":{"21302223b47c":{"x":{},"y":{},"colour":{},"type":"scatter"}},"cur_data":"21302223b47c","visdat":{"21302223b47c":["function (y) ","x"]},"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.2,"selected":{"opacity":1},"debounce":0},"base_url":"https://plot.ly"},"evals":["config.modeBarButtonsToAdd.0.click"],"jsHooks":[]}</script><!--/html_preserve-->

"plotly" plot has many advantages over the "ggplot2" plot. There are a number of functions such as downloading plot, zooming in and out, reset axes, autoscale and etc. at the top right corner of the plot. Also, it will show you the value of y and x axis when you put your cursor on the line plotted in the graph. These are two features that ggplot2 does not have. Also, though it is not shown here, "plotly" can plot a graph that is 3-D.

## Part 4 Writing figures to file.
To save the graphs in part 3 to my local disk using ggsave(). One graph will be saved in vector format, e.g. pdf, and another one will be saved in raster format, e.g. png.

First, let's save the p_title as pdf.

```r
ggsave("~/Downloads/STAT545/hw05-liuyadong08/gdpPercap_title.pdf", plot = p_title, width = 20, height = 10)
```

Now let's try to save p_reorder as png without specifying the plot. 


```r
ggsave("~/Downloads/STAT545/hw05-liuyadong08/gdpPercap_reorder_wrong.png", width = 20, height = 10)
```

The chunk above FAILED to save the p_reorder graph since the plot argument is not specified here. It will save the graph that is in the memory of R. Let's specify the plot as p_reorder and try again.



```r
ggsave("~/Downloads/STAT545/hw05-liuyadong08/gdpPercap_reorder_correct.png", plot = p_reorder, width = 20, height = 10)
```

Last but not least, load those graphs back to the report. 

![graph1](gdpPercap_title.pdf)

![graph2](gdpPercap_reorder_wrong.png)

![graph3](gdpPercap_reorder_correct.png)




