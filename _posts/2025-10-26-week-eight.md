---
title: "Week 8: My Team and Cleaning Up"
date: 2025-10-26
---

Last week, I made a ton of changes to my first draft of a tool on ShinyApps. I removed some features, like the “run optimizer” button, and made additions like making the main table reactive and adding a filter option to only see certain positions. I really only had **one more idea** for this week, and that is exactly what I implemented. However, it did take much longer than I anticipated.

---

### This Week’s major change!

The idea I had thought of last week to add this week was a **“My Team”** tab on the site, where a user could select any player from the main table to add to their team, and it would be added to their own table in this new tab. In this new tab, the user could look at his or her individual team, get a **summary of the team**, and have **available visuals** for their team. I thought this would be very useful if my tool is being used during a real draft, so when a user drafts the player on his or her real fantasy team, they can add the player to their team on the site and get even more info.

---

### How did I do it?

Adding this tab to my site was much harder than I anticipated. I’ll go through some of the steps I took and how I learned what to do or how to get the result I wanted. The first thing I had to do was actually **create the two separate tabs**. This wasn’t too hard, and the code for this is shown here:

```
# optimizer tab
  tabPanel("Optimizer",
           fluidPage(…
			     …)

# second tab for "My Team" section
tabPanel("My Team",
         fluidPage(…
			   …)
```

All it really took was adding these **tabpanel** lines before my original code. Next, I had to add the **“Add”** button and make this button work as intended. Below, I will show the code used to make the button function. 

```
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
```
If you read the code, you can follow some of the comments, but to summarize what is going on, the first thing that’s happening is, the code is ensuring the team size **doesn’t exceed 20**. I added this after experimenting with the add button and seeing how bad my visuals looked after adding over 50 players to my team, something that is unrealistic and unneeded. I limited it to a reasonable 20. It even **returns a notification** if the player tries to exceed this. Then, the following code is what actually moves the selected player to the user’s team table. I made a comment that says preventing duplicates is no longer needed. This is because I added code that eliminates the selected player from the main table when they are moved into the user table.

Other than this, there were many other smaller edits I needed to make for this tab to work. For example, the code below makes sure the “Add” button **stays assigned to the correct player** as the main reactive table changes.

```
# when table redrawn this makes sure add keeps working
  observeEvent(input$table_redrawn, {
    session$sendCustomMessage("bindAddButtons", list())
  })
  
  # makes sure add buttons work from the second the site loads
  observe({
    session$sendCustomMessage("bindAddButtons", list())
  })
```

The last addition I will show in this post is, I made many additions that make the “My Team” page look clean when no players have been selected. Here is the code for that:

```
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
```
With this if statement, the code is displaying an empty data frame with the text **“No players selected yet”** if the player has not selected any players from the main table. Before I added this, the table was completely empty and looked **very bland.** I did this with other parts of the “My Team” tab, too, so it didn’t have a weird-looking space and instead just really reinforced that the user needs to select any player before going to that tab. 

There were a lot of other minor changes I made to the code that I won’t show the code for, but are available to see in my full week’s code section (link found at the bottom of this post). I’ll briefly explain some of the other changes or additions I made. In the “My Team” tab, I added **2 visuals.** One of them compares players by projected points and colors each position a different color, so users can also compare positions among their selected players. The other shows projected points by each position. I also added a **reset team button** so users could reset their team back to empty if they made a mistake or are doing another draft. Really, any situation that requires a reset, this button is useful for. Lastly, I bolded some titles and words to make the site appear better, and made more color choices in my new tab to match the Rangers theme. Last week, I said I was going to add more visual or decorative elements to the site, but I didn’t get to do much of that this week due to how long it took to learn some of the code for this week’s changes, specifically making sure the **“Add” button stays assigned to the right player.**

---

### What's Next?

I’m really happy with how this week turned out. Was it challenging to do this? Yes, without a doubt. But now, I am almost 100% done. The only thing I am going to do between now and next week is figure out some **visual elements** to add to the site to make it look cooler and cleaner. I am completely done with the features I want to add. I am really satisfied with how the final tool works functionally, and now all that’s left is to finish it visually. 

You can see my code for this week in the [code](https://henrylange.github.io/fantasy-nhl-optimizer/code/) section.

Like I said above, next week I will make a post on the final decorative touches I make to the site, but that’s all it will be. Then, I think I will do a **personal reflection** that I’ll post in that same post or a separate one, but it will also be posted next week. Now, the thing I am super pumped for that I am doing next. 

Last week, I mentioned moving on to a **new project** that I had been thinking about. I am still brainstorming all the details, but something that I think would be really cool would be to follow a similar structure that I have already been following, posting once every week. But as I have stated, I am getting really into finance. I think it would be awesome to, every week or possibly two weeks if it takes longer, **analyze a new company** financially, possibly analyzing things like **EPS** and **stock performance** compared to the industry or other financial data to analyze them as a company, and almost give a report. I think it would be super cool to build up a bunch of snapshots/reports of companies at different times as I keep analyzing, and I think it would definitely help me learn a lot when it comes to analysis and research. I don’t have this all figured out yet, but I will keep thinking on it and give an update next week on any new ideas. Until then, it is adding the final touches to this site, all shown next week. 
