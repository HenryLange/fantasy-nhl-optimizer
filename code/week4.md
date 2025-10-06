---
title: "Week 4 Code"
layout: page
permalink: /code/week4/
---

```r
##########################
### WEEK 4 OPTIMZATION ###
##########################

#optimization done through lp solve package
library(lpSolve)

# putting all projected players into one table
all_players = bind_rows(top_goalies_by_proj_points,
                        centers,
                        left_wings,
                        right_wings,
                        defense,
                        wingers,
                        forwards) %>% 
  mutate(id = row_number()) %>% 
  select(id, player, position, Proj_fantasy_points)

# the positions you want that will have restrictions for optimization
positions <- c("C","LW","RW","D","G", "F", "W")

# adding binary indicator columns for each position
all_players <- all_players %>% 
  mutate(
    is_C  = if_else(position == "C", 1, 0),
    is_LW = if_else(position == "LW", 1, 0),
    is_RW = if_else(position == "RW", 1, 0),
    is_D  = if_else(position == "D", 1, 0),
    is_G  = if_else(position == "G", 1, 0),
    is_F  = if_else(position %in% c("C","LW","RW"), 1, 0), 
    is_W  = if_else(position %in% c("LW","RW"), 1, 0))

# Define roster requirements
req_counts <- c(
  C = 2, LW = 2, RW = 2, D = 4, G = 2, Bench = 4)

# Number of players
n <- nrow(all_players)

# Objective is to maximize projected fantasy points
objective <- all_players$Proj_fantasy_points

const.mat <- rbind(
  all_players$is_C, all_players$is_LW, all_players$is_RW, all_players$is_D, all_players$is_G)

const.dir <- c(">=", ">=", ">=", ">=", ">=", "==")

# Right-hand side: roster requirements for starters. Bench will just be filled by remaining top points
const.rhs <- c(2, 2, 2, 4, 2)

# Total number of players including bench
total_players <- 16

solution <- lp(
  direction = "max",
  objective.in = all_players$Proj_fantasy_points,
  const.mat = rbind(const.mat, rep(1, nrow(all_players))),
  const.dir = c(const.dir, "=="),
  const.rhs = c(const.rhs, total_players),
  all.bin = TRUE)

optimal_team <- all_players[solution$solution == 1, ]

# View the optimal roster
print(optimal_team)




# Function to run draft simulations
simulate_draft_simple <- function(all_players) {
  
  # Randomly remove 30% of the players
  set.seed(NULL)
  all_players <- all_players %>% 
    slice_sample(prop = 0.7)  
  
  n <- nrow(all_players)
  objective <- all_players$Proj_fantasy_points
  
  # Constraints: at least required starters, 16 total roster
  const.mat <- rbind(
    all_players$is_C,
    all_players$is_LW,
    all_players$is_RW,
    all_players$is_D,
    all_players$is_G,
    rep(1, n)
  )
  
  const.dir <- c(">=", ">=", ">=", ">=", ">=", "==")
  const.rhs <- c(2, 2, 2, 4, 2, 16)
  
solution <- lp(
  direction = "max",
  objective.in = all_players$Proj_fantasy_points,
  const.mat = rbind(const.mat, rep(1, nrow(all_players))),
  const.dir = c(const.dir, "=="),
  const.rhs = c(const.rhs, total_players),
  all.bin = TRUE)
  
  # Return selected players
  optimal_team <- all_players[solution$solution == 1, ]
  return(optimal_team)
}

# Run 5 simulations
set.seed(67)
draft_sims <- lapply(1:5, function(x) simulate_draft_simple(all_players))

# View first simulation
draft_sims[[1]]

#saving teams as variables
team1 = draft_sims[[1]]

team2 = draft_sims[[2]]

team3 = draft_sims[[3]]

team4 = draft_sims[[4]]

team5 = draft_sims[[5]]

# comparing individual's point values on team 1
bar_chart_proj_points_for_team1 = ggplot(team1, aes(x = reorder(player, Proj_fantasy_points), y = Proj_fantasy_points, fill = position)) +
  geom_bar(stat = "identity") +
  coord_flip() +
  labs(title = "Projected Fantasy Points of Team 1", x = "Player", y = "Projected Points") +
  theme_minimal()

bar_chart_proj_points_for_team1

# save for blog
ggsave("assets/images/bar_chart_proj_points_for_team1.png", plot = bar_chart_proj_points_for_team1, width = 8, height = 6, dpi = 300)

#same for team 2
bar_chart_proj_points_for_team2 = ggplot(team2, aes(x = reorder(player, Proj_fantasy_points), y = Proj_fantasy_points, fill = position)) +
  geom_bar(stat = "identity") +
  coord_flip() +
  labs(title = "Projected Fantasy Points of Team 1", x = "Player", y = "Projected Points") +
  theme_minimal()

bar_chart_proj_points_for_team2

# save for blog
ggsave("assets/images/bar_chart_proj_points_for_team2.png", plot = bar_chart_proj_points_for_team2, width = 8, height = 6, dpi = 300)

# and 3
bar_chart_proj_points_for_team4 = ggplot(team4, aes(x = reorder(player, Proj_fantasy_points), y = Proj_fantasy_points, fill = position)) +
  geom_bar(stat = "identity") +
  coord_flip() +
  labs(title = "Projected Fantasy Points of Team 1", x = "Player", y = "Projected Points") +
  theme_minimal()

bar_chart_proj_points_for_team4

# save for blog
ggsave("assets/images/bar_chart_proj_points_for_team4.png", plot = bar_chart_proj_points_for_team4, width = 8, height = 6, dpi = 300)

#get the total points for all the teams I simulated
team_totals <- data.frame(
  team = paste0("Team ", 1:length(draft_sims)),
  total_proj_points = sapply(draft_sims, function(team) sum(team$Proj_fantasy_points)))

# compare totals
compare_total_proj_points = ggplot(team_totals, aes(x = team, y = total_proj_points, fill = team)) +
  geom_col(show.legend = FALSE) +
  labs(title = "Total Projected Fantasy Points by Team",
       x = "Team",
       y = "Total Projected Points") +
  theme_minimal()

# save for blog
ggsave("assets/images/compare_total_proj_points.png", plot = compare_total_proj_points, width = 8, height = 6, dpi = 300)


```
