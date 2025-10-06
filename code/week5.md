---
title: "Week 5 Code"
layout: page
permalink: /code/week5/
---

```r
##############
### WEEK 5 ###
##############
  
simulate_draft_advanced <- function(all_players, remove_top_n = 7, total_players = 16) {
  # copy the player pool so we don’t overwrite/mess with original player pool
  pool <- all_players
    
  # stores/tracks team that we are selecting
  drafted_team <- data.frame()
    
  # keeps track of how many of each position we have
  counts <- list(C=0, LW=0, RW=0, D=0, G=0, Bench=0)
    
  # roster requirements
  req_counts <- c(C=2, LW=2, RW=2, D=4, G=2, Bench=4)
    
  while(nrow(drafted_team) < total_players & nrow(pool) > 0) {
    # picks the best available player
    best_pick <- pool %>% arrange(desc(Proj_fantasy_points)) %>% slice(1)
    
    #gets position of that player  
    pos <- best_pick$position
      
    # check if pick satisfies position requirements or if he can go to the bench
    if (pos %in% names(counts) && counts[[pos]] < req_counts[[pos]]) {
      drafted_team <- bind_rows(drafted_team, best_pick)
      counts[[pos]] <- counts[[pos]] + 1
    } else if (counts$Bench < req_counts["Bench"]) {
      drafted_team <- bind_rows(drafted_team, best_pick)
      counts$Bench <- counts$Bench + 1
    }
      
    # remove the drafted player and top n others that have been taken
    pool <- pool %>% arrange(desc(Proj_fantasy_points)) %>% 
      slice(-(1:(remove_top_n+1)))
  }
  
  # Return the team!!!!!  
  return(drafted_team)
}
  
# test run
advanced_team_1 <- simulate_draft_advanced(all_players, remove_top_n = 7, total_players = 16)

advanced_team_1

# LET'S GO!!!! probably the longest/hardest thing I've had to learn and I got it after a while

# will test but instead do higher or lower for removing top n, to show how the amount of players in a league effects total expected points of a team

advanced_team_2 <- simulate_draft_advanced(all_players, remove_top_n = 9, total_players = 16)

advanced_team_2

advanced_team_3 <- simulate_draft_advanced(all_players, remove_top_n = 11, total_players = 16)

advanced_team_3


advanced_team_proj_totals <- data.frame(
  advanced_team = c("Team 1", "Team 2", "Team 3", "Team 4"),
  total_proj_points = c(
    sum(advanced_team_1$Proj_fantasy_points),
    sum(advanced_team_2$Proj_fantasy_points),
    sum(advanced_team_3$Proj_fantasy_points),
    sum(advanced_team_4$Proj_fantasy_points)
  )
)

# compare with visual 
advanced_team_proj_totals_plot = ggplot(advanced_team_proj_totals, aes(x = advanced_team, y = total_proj_points, fill = advanced_team)) +
  geom_col(width = 0.6) +
  labs(title = "Total Projected Fantasy Points of Accurately Simulated Teams",
       x = "Team",
       y = "Total Projected Fantasy Points") +
  theme_minimal() + 
  theme(legend.position = "none")

advanced_team_proj_totals_plot

# I want to show this even more so I'm gonna make one more team and add it
# with 19 other people in the league
advanced_team_4 <- simulate_draft_advanced(all_players, remove_top_n = 19, total_players = 16)

advanced_team_4

# re ran the visual and I think it shows well!

# save for blog
ggsave("assets/images/advanced_team_proj_totals_plot.png", plot = advanced_team_proj_totals_plot, width = 8, height = 6, dpi = 300)


# now I think I am ready to make the tool. I have done the optimization with lp solve, advanced optimization with if else statements in attempt to simulate a real draft, and set up many tables. Now I will make a function so people can enter their leagues weighting on stats, and it will change the optimization

# I need these tables to have all the stats so when people enter weights, it will calculate the projected fantasy points under those conditions
draftable_players_FOR_TOOL = player_stats_2425 %>% 
  mutate(
    is_C  = if_else(position == "C", 1, 0),
    is_LW = if_else(position == "LW", 1, 0),
    is_RW = if_else(position == "RW", 1, 0),
    is_D  = if_else(position == "D", 1, 0),
    is_G  = if_else(position == "G", 1, 0),
    is_F  = if_else(position %in% c("C","LW","RW"), 1, 0), 
    is_W  = if_else(position %in% c("LW","RW"), 1, 0)) %>% 
  filter(GP >= 30) %>% 
  select(-c(fantasy_points, Proj_fantasy_points))
  
draftable_goalies_FOR_TOOL = goalie_stats_2425 %>% 
mutate(
  is_C  = if_else(position == "C", 1, 0),
  is_LW = if_else(position == "LW", 1, 0),
  is_RW = if_else(position == "RW", 1, 0),
  is_D  = if_else(position == "D", 1, 0),
  is_G  = if_else(position == "G", 1, 0),
  is_F  = if_else(position %in% c("C","LW","RW"), 1, 0), 
  is_W  = if_else(position %in% c("LW","RW"), 1, 0)) %>% 
  filter(GS >= 15) %>% 
  select(-c(fantasy_points, Proj_fantasy_points, avg_fantasy_points))
  

# funny how I say I'm ready and have created all the tables I need, but first thing I do after is create new tables


# creating a function to calculate the players/goalies points based on weights entered 
project_skaters_points_FOR_TOOL <- function(draftable_players_FOR_TOOL, weight_G = 3, weight_A = 2, weight_SOG = 0.5, weight_BLK = 0.5, weight_PPG = 0.5, weight_PPA = 0.5, weight_SHG = 0.5, weight_SHA = 0.5) {
  draftable_players_FOR_TOOL %>% 
    mutate(
      projected_fantasy_points_FOR_TOOL = 
        (weight_G  * (ifelse(!is.na(G), G, 0))) +
        (weight_A  * (ifelse(!is.na(A), A, 0))) +
        (weight_SOG * (ifelse(!is.na(SOG), SOG, 0))) +
        (weight_BLK * (ifelse(!is.na(BLK), BLK, 0))) +
        (weight_PPG * (ifelse(!is.na(powerPlayGoals), powerPlayGoals, 0))) +
        (weight_PPA * (ifelse(!is.na(powerPlayAssists), powerPlayAssists, 0))) +
        (weight_SHG * (ifelse(!is.na(shortHandedGoals), shortHandedGoals, 0))) +
        (weight_SHA * (ifelse(!is.na(shorthandedAssists), shorthandedAssists, 0))) # example
    ) 
}

project_goalies_points_FOR_TOOL <- function(draftable_goalies_FOR_TOOL, weight_W = 3, weight_GA = -1, weight_saves = 0.2, weight_shutouts = 2) {
  draftable_goalies_FOR_TOOL %>% 
    mutate(
      projected_fantasy_points_FOR_TOOL = 
        (weight_W  * (ifelse(!is.na(W), W, 0))) +
        (weight_GA  * (ifelse(!is.na(GA), GA, 0))) +
        (weight_saves * (ifelse(!is.na(saves), saves, 0))) +
        (weight_shutouts * (ifelse(!is.na(shutouts), shutouts, 0)))
     ) 
}

# test goalie function with some very pumped up weights
test_goalie_projection = project_goalies_points_FOR_TOOL(draftable_goalies_FOR_TOOL, weight_W = 5, weight_GA = -2, weight_saves = 1, weight_shutouts = 5)

# organize a little
test_goalie_projection <- test_goalie_projection %>%
  arrange(desc(projected_fantasy_points_FOR_TOOL)) %>%
  slice(1:20) %>%
  select(player, projected_fantasy_points_FOR_TOOL)

# get these projected values for blog
knitr::kable(test_goalie_projection)

# test player function with some very small weights
test_skater_projection = project_skaters_points_FOR_TOOL(draftable_players_FOR_TOOL, weight_G = 1, weight_A = 0.5, weight_SOG = 0.1, weight_BLK = 0.05, weight_PPG = 0.25, weight_PPA = 0.25, weight_SHG = 0.067, weight_SHA = 0.41)

# organize a little
test_skater_projection <- test_skater_projection %>%
  arrange(desc(projected_fantasy_points_FOR_TOOL)) %>%
  slice(1:20) %>%
  select(player, projected_fantasy_points_FOR_TOOL)

# get these projected values for blog
knitr::kable(test_skater_projection)
```
