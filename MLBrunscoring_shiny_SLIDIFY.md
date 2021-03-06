---
title       : MLB Run Scoring 
subtitle    : A Shiny web app
author      : Martin Monkman / 2015-01-14
job         : 
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : []            # {mathjax, quiz, bootstrap, shiny}
mode        : selfcontained # {standalone, draft}
knit        : slidify::knit2slides
---

## The Steps

1. Analysis
2. Storyboard
3. Development
4. Publishing

(repeat steps 2-4 ...)


--- .class #id 

#### Step 1: Analysis

Already undertaken some static analysis ... see my blog

[bayesball.blogspot.com](bayesball.blogspot.com) 
under the label ["run scoring"](http://bayesball.blogspot.ca/search/label/run%20scoring)

Using:
* Lahman database package
* R analytic and graphic packages
  + dplyr
  + ggplot2

--- .class #id 

#### Step 2: Storyboard

[INSERT IMAGE OF PAPER DRAWING]

--- .class #id 

#### Step 3: Development

Incremental! 

One page at a time, and then looping back to add features (I am by no means finished!)

--- .class #id 

#### 3.01 Summarizing the data


Using dplyr to create summary table


```r
MLB_RPG <- Teams %>%
  filter(yearID > 1900, lgID != "FL") %>%
  group_by(yearID) %>%
  summarise(R=sum(R), RA=sum(RA), G=sum(G)) %>%
  mutate(leagueRPG=R/G, leagueRAPG=RA/G)

tail(MLB_RPG)
```

```
## Source: local data frame [6 x 6]
## 
##   yearID     R    RA    G leagueRPG leagueRAPG
## 1   2008 22585 22585 4856  4.650947   4.650947
## 2   2009 22419 22419 4860  4.612963   4.612963
## 3   2010 21308 21308 4860  4.384362   4.384362
## 4   2011 20808 20808 4858  4.283244   4.283244
## 5   2012 21017 21017 4860  4.324486   4.324486
## 6   2013 20255 20255 4862  4.165981   4.165981
```



--- .class #id 

#### 3.02 The first basic chart


```r
MLBRPG <- ggplot(MLB_RPG, aes(x=yearID, y=leagueRPG)) +
  geom_point() + xlim(1901, 2013) + ylim(3, 6) + xlab("year") + ylab("runs per game") +
  ggtitle(paste("Major League Baseball: runs per team per game 1901-2013")) 
MLBRPG 
```

![plot of chunk unnamed-chunk-4](assets/fig/unnamed-chunk-4-1.png) 

--- .class #id 

#### 3.03.01 The structure of a Shiny app

The user interface:  ui.r
* defines page layout
* contains the instructions for the widgets

The server file: server.r
* where the "analysis" is written, e.g. the plot instructions


--- .class #id 

#### 3.03.02a A first example: just the chart

The user interface:  ui.r

```r
library(shiny)

# Define UI layout using "fluidPage"
shinyUI(fluidPage(
  
  # Application title
  titlePanel("MLB run scoring trends"),
  
  # Sidebar with a slider input for the number of bins
  sidebarLayout(
    sidebarPanel(
      h3("this is the sidebar")
    ),
    
    # Show a plot of run scoring trends
    mainPanel(
      plotOutput("MLBRPG")
    )
  )
))
```

--- .class #id 

#### 3.03.02b A first example: just the chart

The server file: server.r

```r
# package load 
library(shiny)
library(dplyr)
library(reshape)
library(ggplot2)
#
library(Lahman)

# load the Lahman data table "Teams", filter
data(Teams)

# select a sub-set of teams from 1901 [the establishment of the American League] forward to most recent year
MLB_RPG <- Teams %>%
  filter(yearID > 1900, lgID != "FL") %>%
  group_by(yearID) %>%
  summarise(R=sum(R), RA=sum(RA), G=sum(G)) %>%
  mutate(leagueRPG=R/G, leagueRAPG=RA/G)
#
shinyServer(function(input, output) {

  output$plot_MLBtrend <- renderPlot({
    
  MLBRPG <- ggplot(MLB_RPG, aes(x=yearID, y=leagueRPG)) +
    geom_point() + xlim(1901, 2013) + ylim(3, 6) + xlab("year") + ylab("runs per game") +
    ggtitle(paste("Major League Baseball: runs per team per game 1901-2013")) 
  MLBRPG 

})

})
```
--- .class #id 

#### 3.03.02c A first example: just the chart - the result


```r
slidifyUI(fluidPage(
  
  # Application title
  titlePanel("MLB run scoring trends"),
  
  # Sidebar with a slider input for the number of bins
  sidebarLayout(
    sidebarPanel(
      h3("this is the sidebar")
    ),
    
    # Show a plot of run scoring trends
    mainPanel(
      plotOutput("MLBRPG")
    )
  )
))
```

```
## Error in eval(expr, envir, enclos): could not find function "slidifyUI"
```

```r
# package load 
library(shiny)
```

```
## Warning: package 'shiny' was built under R version 3.1.2
```

```r
library(dplyr)
library(reshape)
library(ggplot2)
#
library(Lahman)

# load the Lahman data table "Teams", filter
data(Teams)

# select a sub-set of teams from 1901 [the establishment of the American League] forward to most recent year
MLB_RPG <- Teams %>%
  filter(yearID > 1900, lgID != "FL") %>%
  group_by(yearID) %>%
  summarise(R=sum(R), RA=sum(RA), G=sum(G)) %>%
  mutate(leagueRPG=R/G, leagueRAPG=RA/G)
#
shinyServer(function(input, output) {

  output$plot_MLBtrend <- renderPlot({
    
  MLBRPG <- ggplot(MLB_RPG, aes(x=yearID, y=leagueRPG)) +
    geom_point() + xlim(1901, 2013) + ylim(3, 6) + xlab("year") + ylab("runs per game") +
    ggtitle(paste("Major League Baseball: runs per team per game 1901-2013")) 
  MLBRPG 

})

})
```


--- .class #id 

#### 3.04.1 Adding reactive features to the chart

**Reactive values** - passed between the two files ... 

... click a button on the screen, the value is captured in the ui.r file, passed back to the server.r file, where it is used to recalculate the output, which is passed back to the ui.r file and displayed on the screen

**Example: the trend line**

ui.r code -- create a checkbox that will return the value "trendlineselect"

```r
checkboxInput("trendlineselect", label = h4("Add a trend line"), value = FALSE),
```

server.r code -- evaluates "trendlineselect", and if it's true, add the "stat_smooth" (trendline) option to the chart

```r
if (input$trendlineselect == TRUE) {
  MLBRPG <- MLBRPG + 
  stat_smooth(method=loess, 
              span=input$trendline_sen_sel,
              level=as.numeric(input$trendline_conf_sel))
  }
```

--- .class #id 

#### 3.04.2 The resulting chart

![plot of chunk unnamed-chunk-10](assets/fig/unnamed-chunk-10-1.png) 



--- .class #id 

#### 3.05 Adding the summary data table

--- .class #id 

#### 3.06 Page layout & page tabs

--- .class #id 

#### 3.07 Markdown (documentation page)


--- .class #id 

#### Step 4: Publishing

RStudio will host your Shiny app

Here's mine:
[https://monkmanmh.shinyapps.io/MLBrunscoring_shiny/](https://monkmanmh.shinyapps.io/MLBrunscoring_shiny/)

--- .class #id 

#### References and links

[MLB run scoring trends](https://monkmanmh.shinyapps.io/MLBrunscoring_shiny/)

[bayesball.blogspot.com](bayesball.blogspot.com)

[github repository "MLBrunscoring_shiny"](https://github.com/MonkmanMH/MLBrunscoring_shiny)


-30-
