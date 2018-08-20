Visualisation in a Shiny App
============================

Supposing you have data you want to present in a way which will be intuitive easy to interpret. Perhaps you want to present it to colleagues, or perhaps as outreach to the public. What's the best way to achieve that?

Effective visualisation is clearly the key to conveying your point. An interactive plot in a website could really make it happen. But how do you go about doing that?

Fortunately, there are a number of popular visualisation tools around designed to do this. In this post, we're going to use Shiny, an R package that allows you to make interactive plot apps really easily. We'll make an app which allows you to generate an interactive contour map of heatwave data.

Let's Get Started
-----------------

Before you can create a Shiny app, you first have to install RStudio. (See https://www.rstudio.com/ for installation and other basics about using RStudio.)

Once you've done that, you'll want to install some packages that we'll be using in the Console window:
```
install.packages("shiny", "shinydashboard", "ggplot2", "dplyr", "ncdf4", "itsmr", "colourpicker")
```
Create a new R script. We will start by importing the libraries our app will need:
```
## Basic Heatwave App ##

library(shiny)
library(shinydashboard)
library(ggplot2)
library(dplyr)
library(ncdf4)
library(itsmr)
library(colourpicker)
```

The basic compnents of a Shiny app are the following 3 commands:
```
ui <- fluidPage()

server <- function(input, output) { }

#run/call the shiny app
shinyApp(ui, server)
```

The first command, `ui <- fluidPage()` tells Shiny what the UI - the user interface - will do. In other words, all your inputs which will be used for interactivity - such as slider inputs and select inputs - will be coded here.

The second command, `server <- function(input, output) { }` is the code for the server or the "back-end". The server does the heavy lifting in terms of processing and will generate output which can be displayed on the screen.

The third command, `shinyApp(ui, server)` tells R to run the Shiny app.

Click `Run App` and it should bring up for you an empty browser window, indicating that your Shiny app works, but doesn't yet do anything.

Providing Data For The App
--------------------------

To correct this situation, let's get our app to read in some data. Beforehand we downloaded some heatwave data which we will be using. Above our `ui <- fluidPage()` command, let's add in the following:
```
ncname <- "hw_ANN_CAM5-1-2degree_All-Hist_run001_1959-2012.nc"
ncin <- nc_open(ncname)
```
Now we'll extract some basic variables from the files - `lat`, `lon` - with which we'll calculate the dimensions for our contour map, as well as the names of the other variables in the data:
```
lat <- ncvar_get(ncin, "lat")
lon <- ncvar_get(ncin, "lon")
rows <- dim(lat)
cols <- dim(lon)
area <- matrix(0, nrow=rows, ncol=cols)

varNames <- attributes(ncin$var)$names[4:23]
```
Implementing the User Interface (UI)
------------------------------------

We're now going to expand our `ui <- fluidPage()` to provide the controllers that will allow us to manipulate the plot. We will put our controllers in a sidebar panel on the left and there we will put 6 controllers which will:
1. Select the variable from the data to plot.
2. Select the statistic we wish to plot - mean, variance or standard deviation.
3. 
```
ui <- fluidPage(
  titlePanel("The Facts About Heatwaves"),
  
  # Code for the Side bar
  sidebarLayout(
    sidebarPanel(
      
      selectInput("var", "Variable:", choices=varNames),
      selectInput("stat", "Statistic:",
                  choices=c("Mean", "Variance", "Standard Deviation")),
      sliderInput("dateRange", "Date Range", min=1959, max=2012,
                  value=c(1959,2012), dragRange = TRUE, sep=""),
      colourInput("colMax", "Maximum Colour", "Red", showColour = "background"),
      colourInput("colMid", "Median Colour", "White", showColour = "background"),
      colourInput("colMin", "Minimum Colour", "Blue", showColour = "background")
    ),
    
    mainPanel(
      plotOutput(outputId = "contourMap")
    )
  )
)
```
