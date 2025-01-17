---
name: Coupled hover events with Shiny and Plotly
permalink: r/shiny-coupled-hover-events/
description: Coupled events with Shiny and Plotly
layout: base
language: r
page_type: example_index
output:
  html_document:
    keep_md: true
---



### New to Plotly?

Plotly's R library is free and open source!<br>
[Get started](https://plot.ly/r/getting-started/) by downloading the client and [reading the primer](https://plot.ly/r/getting-started/).<br>
You can set up Plotly to work in [online](https://plot.ly/r/getting-started/#hosting-graphs-in-your-online-plotly-account) or [offline](https://plot.ly/r/offline/) mode.<br>
We also have a quick-reference [cheatsheet](https://images.plot.ly/plotly-documentation/images/r_cheat_sheet.pdf) (new!) to help you get started!

### Version Check

Version 4 of Plotly's R package is now [available](https://plot.ly/r/getting-started/#installation)!<br>
Check out [this post](http://moderndata.plot.ly/upgrading-to-plotly-4-0-and-above/) for more information on breaking changes and new features available in this version.

```r
library(plotly)
packageVersion('plotly')
```

```
## [1] '4.5.6.9000'
```

### Coupled Hover Events in Plotly using Shiny
Apart from **click** and **selection** events, Plotly supports **hover** events as well.

- `plotly_hover` can be used to return information about data points on mouse hover.

Below is an example on how to couple a hover event on one chart to trigger a computation on another chart. For an example showcasing **click** and **selection** events see [here](/r/shiny-coupled-events/)

### UI


```r
library(shiny)
library(plotly)
library(shinythemes)
library(dplyr)

ui <- fluidPage(
  # Set theme
  theme = shinytheme("spacelab"),

  # Some help text
  h2("Coupled hover-events in plotly charts using Shiny"),
  h4("This Shiny app showcases coupled hover-events using Plotly's ", tags$code("event_data()"), " function."),

  # Vertical space
  tags$hr(),

  # Window length selector
  selectInput("window", label = "Select Window Length", choices = c(10, 20, 30, 60, 90), selected = 10),

  # Plotly Chart Area
  fluidRow(
    column(6, plotlyOutput(outputId = "timeseries", height = "600px")),
    column(6, plotlyOutput(outputId = "correlation", height = "600px"))),

  tags$hr(),
  tags$blockquote("Hover over time series chart to fix a specific date. Correlation chart will update with historical
                  correlations (time span will be hover date +/- selected window length)")
  )
```

### Server


```r
server <- function(input, output){

  # Read data
  stockdata <- read.csv("https://cdn.rawgit.com/plotly/datasets/master/stockdata.csv")

  # Create dates
  stockdata$Date <- as.Date(stockdata$Date)

  # Reshape
  ds <- reshape2::melt(stockdata, id = "Date")
  ds <- filter(ds, variable != "GSPC")

  # Set some colors
  plotcolor <- "#F5F1DA"
  papercolor <- "#E3DFC8"

  # Plot time series chart
  output$timeseries <- renderPlotly({
    p <- plot_ly(source = "source") %>%
      add_lines(data = ds, x = ~Date, y = ~value, color = ~variable, mode = "lines", line = list(width = 3))

    # Add SP500
    p <- p %>%
      add_lines(data = stockdata, x = ~Date, y = ~GSPC, mode = "lines", yaxis = "y2", name = "SP500", opacity = 0.3,
                   line = list(width = 5)) %>%
      layout(title = "Stock prices for different stocks overlaid with SP500",
             xaxis = list(title = "Dates", gridcolor = "#bfbfbf", domain = c(0, 0.98)),
             yaxis = list(title = "Stock Price", gridcolor = "#bfbfbf"),
             plot_bgcolor = plotcolor,
             paper_bgcolor = papercolor,
             yaxis2 = list(title = "SP500", side = "right", overlaying = "y"))
    p
  })

  # Coupled hover event
  output$correlation <- renderPlotly({

    # Read in hover data
    eventdata <- event_data("plotly_hover", source = "source")
    validate(need(!is.null(eventdata), "Hover over the time series chart to populate this heatmap"))

    # Get point number
    datapoint <- as.numeric(eventdata$pointNumber)[1]

    # Get window length
    window <- as.numeric(input$window)

    # Show correlation heatmap
    rng <- (datapoint - window):(datapoint + window)
    cormat <- round(cor(stockdata[rng, 1:5]),2)

    plot_ly(x = rownames(cormat), y = colnames(cormat), z = cormat, type = "heatmap",
            colors = colorRamp(c('#e3dfc8', '#808c6c')))%>%
      layout(title = "Correlation heatmap",
             xaxis = list(title = ""),
             yaxis = list(title = ""))

  })

}
```

### Shiny app

<iframe src="https://plotly.shinyapps.io/ShinyCoupledHoverEvents/" width="100%" height= "800"  scrolling="no" seamless="seamless" style="border: none; position: relative"></iframe>


#Reference

See [https://plot.ly/r/reference](https://plot.ly/r/reference) for more information and chart attribute options!
