---
title: "Week 6 Code"
layout: page
permalink: /code/week6/
---


Code in regular/original file:
```
##############
### WEEK 6 ###
##############

# installing packages for tool
install.packages("shiny")
install.packages("rsconnect")

# connecting to shinyio account
###### not showing for privacy/security #########

# I just created a test app called "testApp" and now I'm just trying to get the right path to my shinyio app
library(shiny)
library(rsconnect)
rsconnect::deployApp('~/NHL Optimizer/testApp')

# got it! now I will work on my real one

#installing more packages

install.packages("DT")

#loading in app

# combining tables to have one table for app

# adding goalie specific columns as NA
draftable_players_FOR_TOOL <- draftable_players_FOR_TOOL %>%
  mutate(
    W = NA,
    GA = NA,
    saves = NA,
    shutouts = NA
  )
# it doesn't necessarily matter that I'm getting rid of players wins since that isn't used in their projected points calculations

# add skater specific columns as NA
draftable_goalies_FOR_TOOL <- draftable_goalies_FOR_TOOL %>%
  mutate(
    G = NA,
    A = NA,
    SOG = NA,
    BLK = NA,
    powerPlayGoals = NA,
    powerPlayAssists = NA,
    shortHandedGoals = NA,
    shorthandedAssists = NA
  )

draftable_combined_FOR_TOOL <- bind_rows(draftable_players_FOR_TOOL, draftable_goalies_FOR_TOOL)

# making the actual CSV that will go into the app
write.csv(draftable_combined_FOR_TOOL, "draftable_players.csv", row.names = FALSE)

# deploy what I just made
rsconnect::deployApp('~/NHL Optimizer/Fantasy_NHL_Optimizer')
```

code in new app file:
```
#
# This is a Shiny web application. You can run the application by clicking
# the 'Run App' button above.
#
# Find out more about building applications with Shiny here:
#
#    https://shiny.posit.co/
#

library(shiny)
library(DT)
library(tidyverse)

# Load draftable players globally
draftable_players <- read_csv("draftable_players.csv")

ui <- fluidPage(
  
  # making rangers colors, blackhawks wouldn't look good in my opinion. too dark.
  tags$style(HTML("
    body {background-color: #FFFFFF; color: #0038A8; font-family: Arial, sans-serif;}
    
    .well {background-color: #0038A8; color: #FFFFFF; padding: 15px; border-radius: 10px;}
    .btn-primary {background-color: #CE1126; color: #FFFFFF; border-color: #CE1126;}
    
    h3, h4, h5 {color: #CE1126;}
    
    .shiny-input-container {margin-bottom: 15px;}
    
    .dataTables_wrapper .dataTables_filter input {
      border-radius: 5px;
    }
  ")),
  
  # title
  titlePanel(HTML("<b>NHL Fantasy Optimizer</b>")),
  hr(),
  
  # instructions/description
  HTML("
  <p style='font-size:16px; color:#0038A8;'>
    Welcome to my NHL Fantasy Optimizer! <br>
    1. Use the sliders on the left to set skater scoring weights and on the right for goalie weights. <br>
    2. Using the drop down in the bottom right, search and select players that have been taken in your draft so they don't appear as availible in the optimizer. <br>
    3. Click <b>Run Optimizer</b> to see the top available players after you've entered all weights and drafted players in your league. <br>
    4. Use <b>Reset Weights</b> to return all sliders to default values if needed.
  </p>
"),
  
  # main columns
  div(
    style = "padding-left: 20px; padding-right: 20px;",
    fluidRow(
      
      # LEFT column = skater sliders
      column(width = 3,
             wellPanel(
               h4(HTML("<b>Skater Scoring</b>")),
               sliderInput("weight_G", "Goals", min = 0, max = 10, value = 3, step = 0.5),
               sliderInput("weight_A", "Assists", min = 0, max = 10, value = 2, step = 0.5),
               sliderInput("weight_SOG", "Shots on goal", min = 0, max = 10, value = 0.5, step = 0.25),
               sliderInput("weight_BLK", "Blocks", min = 0, max = 10, value = 0.5, step = 0.25),
               sliderInput("weight_PPG", "Power Play Goals", min = 0, max = 10, value = 0.5, step = 0.25),
               sliderInput("weight_PPA", "Power Play Assists", min = 0, max = 10, value = 0.5, step = 0.25),
               sliderInput("weight_SHG", "Short Handed Goals", min = 0, max = 10, value = 0.5, step = 0.25),
               sliderInput("weight_SHA", "Short Handed Assists", min = 0, max = 10, value = 0.5, step = 0.25)
             )
      ),
      
      # CENTER column = table/players displayed
      column(width = 6,
             h3(HTML("<b>Best Available Players</b>")),
             DTOutput("player_table")
      ),
      
      #RIGHT column = goalie sliders, drafted players, and reset sliders
      column(width = 3,
             wellPanel(
               h4(HTML("<b>Goalie Scoring</b>")),
               sliderInput("weight_W", "Wins", min = 0, max = 10, value = 3, step = 0.5),
               sliderInput("weight_GA", "Goals Against", min = -10, max = 0, value = -1, step = 0.5),
               sliderInput("weight_Saves", "Saves", min = 0, max = 5, value = 0.2, step = 0.1),
               sliderInput("weight_SO", "Shutouts", min = 0, max = 10, value = 2, step = 0.5),
               
               hr(),
               
               # dropdown for drafted players
               selectizeInput(
                 inputId = "taken_players",
                 label = HTML("<b>Players Already Drafted</b>"),
                 choices = draftable_players$player,
                 multiple = TRUE,
                 options = list(placeholder = 'Select drafted players...')
               ),
               
               # button to generate table/run optimizer
               actionButton("run_optimizer", "Run Optimizer", class = "btn-primary"),
               
               hr(),
               
               # Reset button
               actionButton("reset_weights", "Reset Weights", class = "btn-secondary")
             )
      )
    ),
    hr()
  )
)

# server part
server <- function(input, output, session) {
  
  # reactive event that calculates projected points when button is pressed
  optimizer_results <- eventReactive(input$run_optimizer, {
  
    # ensures table is loaded if no players have been selected and entered yet
    taken <- input$taken_players
    if (is.null(taken) || length(taken) == 0) {
      taken <- character(0)
    }
    
    # removes selected players
    available <- subset(draftable_players, !(player %in% taken))
    
    # initializing projected points column
    available$ProjectedPoints <- NA
    
    # calculates skater points
    is_skater <- available$position %in% c("F", "W", "LW", "RW", "C", "D")
    available$ProjectedPoints[is_skater] <- with(available[is_skater,],
                                                   input$weight_G * ifelse(!is.na(G), G, 0) +
                                                     input$weight_A * ifelse(!is.na(A), A, 0) +
                                                     input$weight_SOG * ifelse(!is.na(SOG), SOG, 0) +
                                                     input$weight_BLK * ifelse(!is.na(BLK), BLK, 0) +
                                                     input$weight_PPG * ifelse(!is.na(powerPlayGoals), powerPlayGoals, 0) +
                                                     input$weight_PPA * ifelse(!is.na(powerPlayAssists), powerPlayAssists, 0) +
                                                     input$weight_SHG * ifelse(!is.na(shortHandedGoals), shortHandedGoals, 0) +
                                                     input$weight_SHA * ifelse(!is.na(shorthandedAssists), shorthandedAssists, 0)
    )
    
    
    # calculates goalie points
    is_goalie <- available$position == "G"
    available$ProjectedPoints[is_goalie] <- with(available[is_goalie,],
                                                   input$weight_W * ifelse(!is.na(W), W, 0) +
                                                     input$weight_GA * ifelse(!is.na(GA), GA, 0) +
                                                     input$weight_Saves * ifelse(!is.na(saves), saves, 0) +
                                                     input$weight_SO * ifelse(!is.na(shutouts), shutouts, 0)
    )
    
    
    # Sort by projected points
    available <- available[order(-available$ProjectedPoints), ]
    
    # Selecting top rows/players and the desired columns
    available %>%
      select(player, position, ProjectedPoints) %>% 
      head(25)
  })
  
  # making the reset slider values
  # players
  observeEvent(input$reset_weights, {
    updateSliderInput(session, "weight_G", value = 3)
    updateSliderInput(session, "weight_A", value = 2)
    updateSliderInput(session, "weight_SOG", value = 0.5)
    updateSliderInput(session, "weight_BLK", value = 0.5)
    updateSliderInput(session, "weight_PPG", value = 0.5)
    updateSliderInput(session, "weight_PPA", value = 0.5)
    updateSliderInput(session, "weight_SHG", value = 0.5)
    updateSliderInput(session, "weight_SHA", value = 0.5)
    
    #Goalies 
    updateSliderInput(session, "weight_W", value = 3)
    updateSliderInput(session, "weight_GA", value = -1)
    updateSliderInput(session, "weight_Saves", value = 0.2)
    updateSliderInput(session, "weight_SO", value = 2)
  })
  
  # renders/outputs table 
  output$player_table <- renderDT({
    req(optimizer_results())
    
    dat <- optimizer_results()
    
    datatable(
      dat, 
      options = list(pageLength = 25, dom = "t"), 
      colnames = c("Player", "Position", "Projected Points"),
      class = "stripe hover compact",
      rownames = FALSE
    ) %>%
      # adding rangers colors in a color bar in table
      formatStyle(
        "ProjectedPoints",
        fontWeight = "bold",
        color = "#0038A8",
        background = styleColorBar(range(dat$ProjectedPoints, na.rm = TRUE), "#CE1126"),
        backgroundSize = "100% 90%",
        backgroundRepeat = "no-repeat",
        backgroundPosition = "center"
      )
  })
}


# to run the app!!!
shinyApp(ui = ui, server = server)
```

