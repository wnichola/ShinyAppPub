---
title       : Coursera - John Hopkins Data Science Specialisation
subtitle    : Developing Data Products - Shiny Project Presentation
author      : Nicholas Wee
job         : Signature Course Participant
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : [bootstrap]            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
knit        : slidify::knit2slides
---
## Shiny Project: 
## Weather Impact on Population Health and Economy
### Background
Using US National Weather Services data from 1950 to 2011, an exploratory analysis tells us that Tonardo alone, has resulted in over 83,058 injuries and fatalities, and over US$40,972,120,087 of losses in Property and Crop Damages.  

This ShinyApp allows us to interactively explore this data further, after some level of cleaning, and removing earlier data from 1950 to 2000 as it contains a lot of missing data.

The ShinyApp provides a slider input that allows the user to define the data to extract from a given start and end years, and to select the top impactful events to Population Health and the Economy  

The ShinyApp can be found here: https://wnichola.shinyapps.io/WeatherImpact

--- .class #id 

### ShinyUI

A tab based menu with a sidebar and mainpanel is used to layout the application.  The main tab presents the application, and a "help" tab provides usage information.

On the sidebar, the sliderinput that limits the range between 2001 and 2011, and numeric input that is limited between 1 to 10 top events,  are defined by the following codes:

```r
            sliderInput("timeRange", "Date Range (Years):",
                min = 2001, max = 2011, sep = "", value = c(2006, 2011)
            ),
            numericInput(
                'top_n', 'Select the top', 5, min = 1, max = 10, step = 1
            )
```

--- .class #id 

### ShinyServer
On the server, the cleaned data is loaded, manipulated, and computed.  The following shows  the manipulation of Population Health data using the function input.


```r
load(file="./data/storm_data_clean.RData")
storm_range <- filter(storm_data_clean, BGN_DATE_YR >= input$timeRange[1],
                      BGN_DATE_YR <= input$timeRange[2])
storm_summary0 <- storm_range %>% group_by(EVTYPE) %>% 
        summarise(Total_Fatalities = sum(FATALITIES), Total_Injuries = sum(INJURIES),
                  Total_Hlth_Impact = sum(FATALITIES + INJURIES),
                  Total_PropDmg = sum(PROPDMG), Total_CropDmg = sum(CROPDMG),
                  Total_Econ_Impact = sum(PROPDMG + CROPDMG))
storm_sum_health <- arrange(storm_summary0, desc(Total_Hlth_Impact))
storm_sum_health <- storm_sum_health[1:top_n, ] 
```
Then the data is sent to the UI, using the function's output.

```r
output$healthTable <- renderDataTable({displayTopHealth(storm_sum_health, input$top_n)})
output$health_plot <- renderPlot({print(plot_health(storm_sum_health)})
```

--- .class #id 

### Datatable & Plots
The table is created using a datatable, and the chart by ggplot:

```r
healthSum <- select(dt, EVTYPE, Total_Injuries, Total_Fatalities, Total_Hlth_Impact)
result <- datatable(healthSum, options = list(iDisplayLength = top_n))
```
The stacked bar chart is then assigned to the UI, and displayed using plotOutput('health_plot'):

![plot of chunk unnamed-chunk-6](assets/fig/unnamed-chunk-6-1.png) 

--- .class #id 
### Reactivity
The previous slides simply used the inputs from the UI.  However, these inputs are reactive.  That is, these react to the action of the users, and needs to be handled differently from normal variables as follows:

```r
storm_range <- reactive({
    filter(storm_data_clean, BGN_DATE_YR >= input$timeRange[1], BGN_DATE_YR <= input$timeRange[2])
})
```
The reactive function wraps a normal expression to create a reactive expression. Conceptually, a reactive expression is a expression whose result will change over time depending on user actions.  

In addition, all data return from a reactive expression needs to be accessed differently from normal. For example, storm_range in the sample above should be access as follows:

```r
dt <- group_by_evtype(storm_range())
```
Note the brackets behind the variable.
