---
title: "Week 8 Code"
layout: page
permalink: /code/week8/
---

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

ui <- navbarPage(
  title = "NHL Fantasy Optimizer",
  id = "main_nav",
  
  # optimizer tab
  tabPanel("Optimizer",
           fluidPage(
  
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
    
        .add_player {
      background-color: #CE1126;
      color: white;
      border: #0038A8;
      border-radius: 6px;
      cursor: pointer;
    }
    .add_player:hover {
      background-color: #b00e20;
    }
    table.dataTable td {
      vertical-align: middle;
    }
  ")),
  
  # title
  titlePanel(HTML("<b>Optimizer</b>")),
  hr(),
  
  # instructions/description
  HTML("
  <p style='font-size:16px; color:#0038A8;'>
    Welcome to my NHL Fantasy Optimizer! <br>
    1. Use the sliders on the left to set skater scoring weights and on the right for goalie weights. <br>
    2. Using the drop down in the bottom right, search and select players that have been taken in your draft so they don't appear as availible in the optimizer. <br>
    3. Using the <b>filter by position</b> option in the bottom right, choose the position of players you wish to see in the table. <br>
    4. Use <b>Reset Weights</b> to return all sliders to default values if needed. <br>
    5. If you wish to compare 2 players, you can do so at the bottom of the page with both skaters and goalies.<br>
    6. You can make your own team by clicking the red <b>Add</b> button next to each player, and you can go to the <b>My Team</b> tab to look at visuals and a summary of your selected team. <br>
  </p>
"),
  
  # main columns
  div(
    style = "padding-left: 20px; padding-right: 20px;",
    
    fluidRow(
      # LEFT column = player sliders
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
  ),
  
  # ensures each add button is reassigned to the right player after an add button is hit
  tags$script(HTML("
               Shiny.addCustomMessageHandler('bindAddButtons', function(message) {
                 $('.add_player').off('click').on('click', function() {
                   var player_id = $(this).attr('id').replace('add_', '');
                   Shiny.setInputValue('add_player', player_id, {priority: 'event'});
                 });
               });
             "))
           )
  ),

# second tab for "My Team" section
tabPanel("My Team",
         fluidPage(
           h3(HTML("<b>My Fantasy Team</b>")),
           DTOutput("team_table"),
           hr(),
           
           actionButton("reset_team", "Reset Team", class = "btn-primary"),
           
           hr(),
           # bolding with HTML didn't work so I used strong here
           h4(strong("Team Summary")),
           verbatimTextOutput("team_summary"),
           
           hr(),
           # same with strong here
           h4(strong("Team Visualizations")),
         # i wanna split up the plots, one on each side
           fluidRow(
             column(6, plotOutput("team_individual_points_plot", height = "400px")),
             column(6, plotOutput("team_points_by_position", height = "400px"))
           )
    )
  )
)



# server part
server <- function(input, output, session) {
  
  # Store players added by the user
  user_team <- reactiveValues(players = data.frame())
  
  # so when app loads a table is automatically there
  optimizer_results <- reactive({
    # ensures table is loaded if no players have been selected and entered yet
    taken <- input$taken_players
    if (is.null(taken)) taken <- character(0)
    
    available <- subset(draftable_players, !(player %in% c(taken, user_team$players$player)))
    
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
    
    # Add the Add button for each player
    dat$Add <- sprintf(
      '<button class="btn btn-primary btn-sm add_player" 
            id="add_%s" 
            style="padding:2px 6px; font-size:12px; border-radius:6px;">Add</button>',
      # substitutes space for _
      gsub(" ", "_", dat$player)
    )
    
    datatable(
      dat,
      escape = FALSE,
      selection = "none",
      rownames = FALSE,
      options = list(
        pageLength = 25,
        dom = "t",
        autoWidth = TRUE,
        columnDefs = list(list(className = "dt-center", targets = "_all")),
        drawCallback = JS("function(settings) { 
        Shiny.setInputValue('table_redrawn', Math.random());
      }")
      ),
      colnames = c("Player", "Position", "Projected Points", "Add to Team")
    ) %>%
      # adding rangers colors in a color bar in table
      formatStyle(
        "ProjectedPoints",
        fontWeight = "bold",
        color = "#0038A8"
      )
  })
  
  # when table redrawn this makes sure add keeps working
  observeEvent(input$table_redrawn, {
    session$sendCustomMessage("bindAddButtons", list())
  })
  
  # makes sure add buttons work from the second the site loads
  observe({
    session$sendCustomMessage("bindAddButtons", list())
  })
  
  observeEvent(input$add_player, {
    req(input$add_player)
    
    # basically this ensures teams don't get to big, messing up tables in the my team tab
    # 20 is a realistic number. If I need to change later I can.
    max_team_size <- 20
    
    # won't add more than 20 players
    if(nrow(user_team$players) >= max_team_size){
      showNotification(
        paste("You can only add up to 20 players!"),
        type = "warning",
        duration = 3
      )
      return()
    }
    
    # replaces _ in name to a space
    player_name <- gsub("_", " ", input$add_player)
    
    dat <- optimizer_results()
    player_row <- dat[dat$player == player_name, ]
    
    if (nrow(player_row) > 0) {
      # Preventing duplicates (this isn't even needed anymore but i took time on it so it'll stay).
      if (!(player_name %in% user_team$players$player)) {
        user_team$players <- rbind(user_team$players, player_row)
      }
    }
  })
  
  output$team_table <- renderDT({
    # I again added something for if no players are selected so it isn't ugly
    if (nrow(user_team$players) == 0) {
      placeholder <- data.frame(
        Player = "No players selected yet",
        Position = "",
        ProjectedPoints = ""
      )
      
      datatable(placeholder,
                options = list(dom = 't', paging = FALSE),
                rownames = FALSE,
                escape = FALSE)
      
    } else {
      # real table when players are selected
      datatable(user_team$players,
                options = list(pageLength = 20),
                rownames = FALSE)
    }
  })
  
  output$team_summary <- renderText({
    # if no team selected this will return text saying no players
    if(nrow(user_team$players) == 0) return("No players added yet.")
    
    # Basic stats about the selected team
    total_players <- nrow(user_team$players)
    total_points <- sum(user_team$players$ProjectedPoints, na.rm = TRUE)
    avg_points <- round(mean(user_team$players$ProjectedPoints, na.rm = TRUE), 2)
    
    # Best and worst players might be intersting
    top_player <- user_team$players %>% slice_max(ProjectedPoints, n = 1)
    bottom_player <- user_team$players %>% slice_min(ProjectedPoints, n = 1)
    
    # Count of players by position
    position_counts <- user_team$players %>%
      count(position) %>%
      arrange(desc(n))
    
    # team summary text/set up
    paste0(
      "Team Size: ", total_players, "\n",
      "Total Projected Points: ", round(total_points, 1), "\n",
      "Average Projected Points: ", avg_points, "\n\n",
      "Top Player: ", top_player$player, " (", round(top_player$ProjectedPoints, 1), " pts)\n",
      "Lowest Player: ", bottom_player$player, " (", round(bottom_player$ProjectedPoints, 1), " pts)\n\n",
      "Roster by Position:\n",
      paste(position_counts$position, ":", position_counts$n, collapse = ", ")
    )
  })
  
  # Player comparison plot
  output$compare_players_plot <- renderPlot({
    req(input$compare_players_plot_inputs)
    
    selected <- draftable_players %>%
      filter(player %in% input$compare_players_plot_inputs)
    
    # 2 players or else error
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
    
    # again needing 2 goalies or else error
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
  
  output$team_individual_points_plot <- renderPlot({
    req(user_team$players)
    
    # when no players were selected, error was ugly so I found a way to show message when no players taken
    if (nrow(user_team$players) == 0) {
      plot.new()
      text(0.5, 0.5, "No players selected yet.\nPlease add players to your team.", 
           cex = 1.2, col = "#0038A8", font = 2, adj = 0.5)
      return()
    }
    
    user_team$players %>%
      arrange(ProjectedPoints) %>%
      ggplot(aes(x = reorder(player, ProjectedPoints),
                 y = ProjectedPoints,
                 fill = position)) +
      geom_bar(stat = "identity") +
      coord_flip() +
      theme_minimal(base_size = 14) +
      labs(title = "Projected Points by Player", x = "", y = "Projected Points") +
      scale_fill_manual(
        values = c(
          "C" = "#0038A8",
          "LW" = "#CE1121",
          "RW" = "#999999", # had to mess around with some non rangers colors. I tried to match.
          "D" = "#0067CF",
          "G" = "#E6B441" 
        ),
        drop = FALSE
      ) +
      theme(plot.title = element_text(face = "bold", color = "#0038A8", hjust = 0.5))
  })
  
  output$team_points_by_position <- renderPlot({
    req(user_team$players)
    
    # same as above with error message
    if (nrow(user_team$players) == 0) {
      plot.new()
      text(0.5, 0.5, "No players selected yet.\nPlease add players to your team.", 
           cex = 1.2, col = "#0038A8", font = 2, adj = 0.5)
      return()
    }
    
    dat <- user_team$players %>%
      group_by(position) %>%
      summarise(TotalPoints = sum(ProjectedPoints, na.rm = TRUE)) %>%
      arrange(desc(TotalPoints))
    
    ggplot(dat, aes(x = reorder(position, TotalPoints), y = TotalPoints, fill = position)) +
      geom_col(width = 0.6) +
      coord_flip() +
      theme_minimal(base_size = 14) +
      labs(title = "Projected Team Points by Position", x = "", y = "Total Projected Points") +
      scale_fill_manual(values = c(
        "C" = "#0038A8",
        "LW" = "#CE1126",
        "RW" = "#999999", # and same with colors here
        "D" = "#0067CF", 
        "G" = "#E6B441"
      ), guide = "none") +
      theme(
        plot.title = element_text(face = "bold", color = "#0038A8", hjust = 0.5),
        axis.text = element_text(color = "black")
      )
  })
  
  # to reset team when button is pressed
  observeEvent(input$reset_team, {
    user_team$players <- data.frame()
  })
  
  
}


# to run the app!!!
shinyApp(ui = ui, server = server)

```

### [return to week 8 post](https://henrylange.github.io/fantasy-nhl-optimizer/2025/10/26/week-eight.html)
