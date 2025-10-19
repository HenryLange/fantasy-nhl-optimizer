---
title: "Week 7 Code"
layout: page
permalink: /code/week7/
---

this is the app code after all week 7 changes:
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
    3. Using the 'filter by position' option in the bottom right, choose the position of players you wish to see in the table. <br>
    4. Use <b>Reset Weights</b> to return all sliders to default values if needed. <br>
    5. If you wish to compare 2 players, you can do so at the bottom of the page with both skaters and goalies. <br>
  </p>
"),
  
  # main columns
  div(
    style = "padding-left: 20px; padding-right: 20px;",
    
    fluidRow(
      # LEFT column
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
      
      # CENTER column = table only
      column(width = 6,
             h3(HTML("<b>Best Available Players</b>")),
             DTOutput("player_table")
      ),
      
      # RIGHT column = goalie sliders, drafted players, position filter
      column(width = 3,
             wellPanel(
               h4(HTML("<b>Goalie Scoring</b>")),
               sliderInput("weight_W", "Wins", min = 0, max = 10, value = 3, step = 0.5),
               sliderInput("weight_GA", "Goals Against", min = -10, max = 0, value = -1, step = 0.5),
               sliderInput("weight_Saves", "Saves", min = 0, max = 5, value = 0.2, step = 0.1),
               sliderInput("weight_SO", "Shutouts", min = 0, max = 10, value = 2, step = 0.5),
               hr(),
               selectizeInput(
                 inputId = "taken_players",
                 label = HTML("<b>Players Already Drafted</b>"),
                 choices = draftable_players$player,
                 multiple = TRUE,
                 options = list(placeholder = 'Select drafted players...')
               ),
               selectInput(
                 inputId = "position_filter",
                 label = HTML("<b>Filter by Position</b>"),
                 choices = c("All", "C", "LW", "RW", "D", "G"),
                 selected = "All"
               ),
               hr(),
               actionButton("reset_weights", "Reset Weights", class = "btn-primary")
             )
      )
    ),
    
    hr(),
    
    # New row for player + goalie comparison
    fluidRow(
      h4(HTML("<b>Compare Players & Goalies</b>")),
      p("Select two players and two goalies to compare their stats side-by-side."),
      
      column(width = 6,
             selectizeInput(
               inputId = "compare_players_plot_inputs",
               label = "Select Two Players",
               choices = draftable_players$player[draftable_players$position %in% c("F","W","LW","RW","C","D")],
               multiple = TRUE,
               options = list(maxItems = 2, placeholder = 'Select two players...')
             ),
             plotOutput("compare_players_plot", height = "400px")
      ),
      
      column(width = 6,
             selectizeInput(
               inputId = "compare_goalies_plot_inputs",
               label = "Select Two Goalies",
               choices = draftable_players$player[draftable_players$position == "G"],
               multiple = TRUE,
               options = list(maxItems = 2, placeholder = 'Select two goalies...')
             ),
             plotOutput("compare_goalies_plot", height = "400px")
      )
    )
  )
)

# server part
server <- function(input, output, session) {
  
  # so when app loads a table is automatically there
  optimizer_results <- reactive({
    # ensures table is loaded if no players have been selected and entered yet
    taken <- input$taken_players
    if (is.null(taken)) taken <- character(0)
    
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
    
    # Apply position filter
    if (input$position_filter != "All") {
      available <- subset(available, position == input$position_filter)
    }
    
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
        color = "#0038A8"
      )
  })
  
  # Player comparison plot
  output$compare_players_plot <- renderPlot({
    req(input$compare_players_plot_inputs)
    
    selected <- draftable_players %>%
      filter(player %in% input$compare_players_plot_inputs)
    
    validate(
      need(nrow(selected) == 2, "Please select exactly two players to compare.")
    )
    
    compare_data <- selected %>%
      select(player, G, A, SOG, BLK, powerPlayGoals, powerPlayAssists) %>%
      pivot_longer(cols = -player, names_to = "Category", values_to = "Value")
    
    ggplot(compare_data, aes(x = Category, y = Value, fill = player)) +
      geom_bar(stat = "identity", position = "dodge") +
      theme_minimal(base_size = 14) +
      labs(title = "Player Comparison", x = "", y = "Stat Value") +
      scale_fill_manual(values = c("#0038A8", "#CE1126")) +
      theme(
        plot.title = element_text(face = "bold", color = "#0038A8", hjust = 0.5),
        axis.text.x = element_text(angle = 30, hjust = 1)
      )
  })
  
  # Goalie comparison plot
  output$compare_goalies_plot <- renderPlot({
    req(input$compare_goalies_plot_inputs)
    
    selected <- draftable_players %>%
      filter(player %in% input$compare_goalies_plot_inputs)
    
    validate(
      need(nrow(selected) == 2, "Please select exactly two goalies to compare.")
    )
    
    compare_data <- selected %>%
      select(player, W, GA, saves, shutouts) %>%
      pivot_longer(cols = -player, names_to = "Category", values_to = "Value") %>%
      mutate(
        Value = ifelse(Category == "saves", Value / 10, Value),
        Category = ifelse(Category == "saves", "saves (/10)", Category)
      )
     
    
    ggplot(compare_data, aes(x = Category, y = Value, fill = player)) +
      geom_bar(stat = "identity", position = "dodge") +
      theme_minimal(base_size = 14) +
      labs(title = "Goalie Comparison", x = "", y = "Stat Value") +
      scale_fill_manual(values = c("#0038A8", "#CE1126")) +
      theme(
        plot.title = element_text(face = "bold", color = "#0038A8", hjust = 0.5),
        axis.text.x = element_text(angle = 30, hjust = 1)
      )
  })
}
```

# to run the app!!!
shinyApp(ui = ui, server = server)
