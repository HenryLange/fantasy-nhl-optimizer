---
title: "Week 7: Removing and Adding Tool Features"
date: 2025-10-19
---

Last week, I used **Shinyapps** to create the first real version of the tool that this project sought to create. It took a while to learn how Shinyapps works and the different code for the user interface and the server. I got the tool to a place where users could have success with it, but by no means did it feel complete. This week, I added a lot of **features and changes** to get this much closer to a final product. 

---

### What did I change?

The first thing I did was actually **remove** something. Last week, I thought it would be a cool idea to have a small red boxplot under each player’s projected points to visualize how their score compared to other players. In theory, this was pretty cool, but it didn’t add new information and only made it **harder to read** the player’s projected points with the red and blue colors on top of each other. Removing this made the table easier to read and didn’t remove any information. 

The only other thing I removed was the **“run optimizer”** button. This was for two reasons. Basically, in my first version of the tool, no table was automatically generated when the website was loaded. This made for a big gap or empty space until the user clicked the “run optimizer” button. Also, when the user would change the sliders or remove players, to see any changes in the table, they would need to **click this button again**. Therefore, I removed this button and changed how the table works, which required the additions that I am talking about next.

---

### Additions
So if I removed the “run optimizer” button, then how would the table change when the sliders or players did? For this, I made the table and the optimization that goes into it **reactive.** Being reactive means that whenever a slider is changed or a player is removed, the user automatically sees this change happen in their table. They don’t have to click any buttons. This also fixed my problem of the table not immediately loading, because now, as soon as the app loaded, the table with default sliders and all players appeared. This addition looked like this within the code:
```
optimizer_results <- reactive({
…
…
}
```

I will not show all the code within the brackets, as it is very long and much of it is repeated code from last week, but by changing the code to reactive and putting what I had before inside, I fixed a few problems and made the tool easier to use.

Another major element of the tool and optimization in general is the **position of players** we are selecting. Over the past few weeks, I have done a lot of work optimizing within league position restrictions. I noticed that this wasn’t part of my first version of the tool, so I added it, like this:

```
# in the UI
 selectInput(
                 inputId = "position_filter",
                 label = HTML("<b>Filter by Position</b>"),
                 choices = c("All", "C", "LW", "RW", "D", "G"),
                 selected = "All"
               ), 
# in the server
# Apply position filter
    if (input$position_filter != "All") {
      available <- subset(available, position == input$position_filter)
    }
```

In the UI, this gives the users a place to select which position they want to choose from, with “All” being the default option, and all other positions available. Then, in the server, there is an if statement that basically says, if they have not selected “All”, then show the one they selected. If it is “All,” show all of course. This accounts for the position factor in a fantasy, as now users can see the best players of each position.

Lastly, I added a feature I thought was pretty fun. It isn’t absolutely necessary for the optimizer, but it can help when comparing players for different league weightings. That’s exactly what I added. **An area to compare both players and goalies.** The code for players is shown here:

```
# in the UI
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
# in the server
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
```

I won’t show the code for goalies because it is very similar. What this does is that it essentially takes in 2 players from the user. Then, in the server, it makes a side-by-side comparison for these players on a plot, so it is very easy to compare their stats. Of course, this is also in Rangers colors.

I also made some smaller, general changes that went with the others, like changing the description at the top of my site and also changing the color of the “reset weights” button.

---

### What's Next?

I think I made great progress on the tool this week. This is **very close to finished** in my eyes, and I can only currently think of one more thing I definitely want to add next week. Next week, I will make that addition and possibly decorate/visually enhance both my tool and this website.

You can see my code for this week in the [code](https://henrylange.github.io/fantasy-nhl-optimizer/code/) section.

Unless I uncover some big problems or new ideas next week, week 8 will likely be my last on this specific project. I’ve had my fun with this, and now I’m excited to move on to another project, which I’ve already started thinking about. I will share more details next week.
