---
title       : Presentation about Shiny application Climate Change
subtitle    : Project from Coursera Developing Data Products Course
author      : AArteaga
job         : 
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : []            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
knit        : slidify::knit2slides
---

# Introduction
- This application plots some actual data and some simple scenarios for CO2 emissions of four important agents: China, USA, Rest of the World and the whole World.
- The base data are obtained from the [International Energy Agency](http://www.iea.org/). Those data include emissions from 1970 uf to 2011.
- The scenarios are build considering a constant emissions increment or decrement rate. 
- The user can choose the emissions change rate for the three main cases (the World data are calculated as the sum of the other three). A Bussiness as Usual case is also defined based on historical data. 

--- .class #id 
# Data Reading and treatment
- The data are read directly from the [International Energy Agency](http://www.iea.org/) web page. The data are in an excel file read with the XLConnect library

```r
library(XLConnect)
iea_url="http://www.iea.org/media/freepublications/2013pubs/CO2HighlightsExceltables.xls"
download.file(iea_url, "CO2HighlightsExceltables.xls", method="wget")
wb <- loadWorkbook("CO2HighlightsExceltables.xls", create = FALSE)
coltypes <- rep("numeric", 42)
coltypes[1] <- "character"
raw_data <- readWorksheet(wb, sheet = "CO2 SA", startRow = 3, colTypes = coltypes)
```
- The obtained data frame needs some trimming as transposing the rows to colums, making the years dates and choosing the data to be used in the application. All the data trimming and getting is done in the file *read_data.R*



--- .class #id 
# User interactive choices.
- Three slider control widgets allow the user to change the predicted scenarios, by chosing the CO2 emissions increment or drecrement rate for China, USA and the rest of the world.
- Base data are the Business As Usual case, calculated as the mean value of last 10 year emision changes.
- The limits for the sliders are calculated depending of the bussiness as ususal values, but roughly range a 10% of the total emissions. Those calculations are done in the files *rate_range.R* and *slider_limits.R*.
- There is a checkcontrol for using the Bussiness as Usual values independently of the slider choices.
- This is a code chunck:

```r
sliderInput('china_rate', 'China',
                  value = base_china_rate, 
                  min = min_china_rate, 
                  max = max_china_rate, 
                  step = step_china_rate)
```

--- .class #id 

# Output plot
- The output plot is a time series ggplot containing 4 set of data for past data and other 4 for prediction scenarios. Each of the sets are for China, USA, World and rest of the World. 
- The data are obtained from the dataframe obtained with data from the [International Energy Agency](http://www.iea.org/) web page.
- The predictions are calculated with the fixed increment rate chosen with the interactive sliders.
The code for the figure and prediction is contained iun the *server.R* file, the main part of figure code is:

```r
ggplot(data, aes(x = year, y = world, colour= "World")) + geom_line() +
      geom_line(mapping = aes(y = usa, colour = "USA") ) +
      geom_line(mapping = aes(y = china, colour = "China")) +
      geom_line(mapping = aes(y = rest_of_world, colour = "Rest of World")) +
      geom_line(mapping = aes(y = predict_world, colour= "World"), lwd = 1.3, lty = "dashed") +
      geom_line(mapping = aes(y = predict_usa, colour= "USA"), lwd = 1.3, lty = "dashed") +
      geom_line(mapping = aes(y = predict_china, colour= "China"), lwd = 1.3, lty = "dashed") +
      geom_line(mapping = aes(y = predict_rest_of_world, colour= "Rest of World"), lwd = 1.3, lty = "dashed") +
      scale_colour_manual("", breaks = c("World", "USA", "China", "Rest of World"),
                         values = c("red", "green", "black", "blue"))
```
