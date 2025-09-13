---
title: "Week 1.5 Code"
layout: page
permalink: /code/week-one-and-a-half/
---

```r
# all of this code follows the code from week one

###################
###VISUALS TIME####
###################

#tidyverse is already loaded which includes ggplot2

# my numbers were discrete values, so I have to change this
player_stats_2425$PTS <- as.numeric(player_stats_2425$PTS)

Dist_of_skater_points = ggplot(data = player_stats_2425, aes(x = PTS)) +
  geom_histogram(bins = 30, fill = "lightcoral", color = "black") +
  labs(title = "Distribution of Skater Points",
       x = "Total Points", y = "Number of Players") +
  theme_minimal()

Dist_of_skater_points

# Save the plot as a PNG for blog
ggsave("assets/images/Dist_of_skater_points.png", plot = Dist_of_skater_points, width = 8, height = 6, dpi = 300)

# all my other numbers are likely discrete values, so i have to change EVERYTHING

#after a bit of research, I think I can do them all at once

# Converting columns other than time on ice since they have ":" in them
numeric_cols <- c("age", "GP", "G", "A", "plusMinus", "PIM", "evenStrengthGoals", "powerPlayGoals", "shortHandedGoals", "gameWinningGoals", "evenStrengthAssists", "powerPlayAssists", "shorthandedAssists", "SOG", "shootingPercentage", "totalShotsAttempted", "faceoffWins", "faceoffloses", "faceoffPercentage", "BLK", "Hits", "takeaways", "giveaways")

player_stats_2425[numeric_cols] <- lapply(player_stats_2425[numeric_cols], function(x) as.numeric(as.character(x)))

# now trying to rid : so i can use the time
player_stats_2425 <- player_stats_2425 %>%
  separate(timeOnIce, into = c("total_min", "total_sec"), sep = ":", convert = TRUE) %>%
  mutate(
    TOI_total = total_min + total_sec / 60
  )

# same for average time
player_stats_2425 <- player_stats_2425 %>%
  separate(averageTimeOnIce, into = c("average_min", "average_sec"), sep = ":", convert = TRUE) %>%
  mutate(
    TOI_avg = average_min + average_sec / 60
  )

# perfect! now all player values are numbers that we can make visuals
TOI_vs_PTS_plot = ggplot(player_stats_2425, aes(x = TOI_total, y = PTS)) +
  geom_point(color = "forestgreen") +
  labs(title = "Total Time on Ice vs Points",
       x = "Total Time on Ice (minutes)", y = "Total Points") +
  theme_minimal()

TOI_vs_PTS_plot

# Save the plot as a PNG for blog
ggsave("assets/images/TOI_vs_Points.png", plot = TOI_vs_PTS_plot, width = 8, height = 6, dpi = 300)

#this plot just had players with 3000 minutes but no points. I just found out goalies are in my player data
#removing goalies from player data
player_stats_2425 <- player_stats_2425 %>%
  filter(position != "G")

#retry with no goalies
TOI_vs_PTS_plot_no_goalies = ggplot(player_stats_2425, aes(x = TOI_total, y = PTS)) +
  geom_point(color = "forestgreen") +
  labs(title = "Total Time on Ice vs Points",
       x = "Total Time on Ice (minutes)", y = "Total Points") +
  theme_minimal()

TOI_vs_PTS_plot_no_goalies

# Save the plot as a PNG for blog
ggsave("assets/images/TOI_vs_Points_no_goalies.png", plot = TOI_vs_PTS_plot_no_goalies, width = 8, height = 6, dpi = 300)

#much better!!

#new plot
age_vs_PTS_plot <- ggplot(player_stats_2425, aes(x = age, y = PTS)) +
  geom_point(color = "dodgerblue", alpha = 0.67) +
  labs(title = "Player Age vs Total Points",
       x = "Age (years)", y = "Total Points") +
  theme_minimal()

age_vs_PTS_plot

# Save it as PNG
ggsave("assets/images/age_vs_PTS.png", plot = age_vs_PTS_plot, width = 8, height = 6, dpi = 300)

# Boxplot of Points by Position
PTS_by_position_plot <- ggplot(player_stats_2425, aes(x = position, y = PTS, fill = position)) +
  geom_boxplot(alpha = 0.67) +
  labs(title = "Distribution of Points by Position",
       x = "Position", y = "Total Points") +
  theme_minimal() +
  theme(legend.position = "none")

PTS_by_position_plot

# Save it as PNG
ggsave("assets/images/PTS_by_position_plot.png", plot = PTS_by_position_plot, width = 8, height = 6, dpi = 300)

#time to do this whole process again but for goalies!

# Converting columns for goalies other than time on ice since they have ":" in them
numeric_cols_goalies <- c("age", "GP", "GS", "W", "L", "tiesAndOTLosses", "GA", "shots", "saves", "savePercentage", "GAA", "shutouts", "qualityStarts", "qualityStartsPercentage", "reallyBadStarts", "goalsAgainstPercentage", "goalsSavedAboveAverage", "adjustedGoalsAgainstAverage", "goaliePointsShare", "G", "A", "PTS", "PIM")

goalie_stats_2425[numeric_cols_goalies] <- lapply(goalie_stats_2425[numeric_cols_goalies], function(x) as.numeric(as.character(x)))

# now trying to rid : from minutes column so i can use the time
goalie_stats_2425 <- goalie_stats_2425 %>%
  separate(minutes, into = c("mins", "sec"), sep = ":", convert = TRUE) %>%
  mutate(
    total_mins = mins + sec / 60
  )

# Create new scatter
SV_vs_wins_plot <- ggplot(goalie_stats_2425, aes(x = savePercentage, y = W)) +
  geom_point(color = "red", alpha = 0.67, size = 3) +
  labs(title = "Goalie Save Percentage vs Wins",
       x = "Save Percentage (SV%)",
       y = "Wins") +
  theme_minimal()

SV_vs_wins_plot

# Save the plot as PNG for the blog
ggsave("assets/images/SV_vs_wins.png", plot = SV_vs_wins_plot, width = 8, height = 6, dpi = 300)

#create histogram
save_percentage_histogram <- ggplot(goalie_stats_2425, aes(x = savePercentage)) +
  geom_histogram(binwidth = 0.005, fill = "purple", color = "black") +
  labs(title = "Distribution of Goalie Save Percentage",
       x = "Save Percentage (SV%)",
       y = "Number of Goalies") +
  theme_minimal()

save_percentage_histogram

# Save it
ggsave("assets/images/save_percentage_histogram.png", plot = save_percentage_histogram, width = 8, height = 6, dpi = 300)

# Create new scatter for GAA vs mins
GAA_vs_mins_plot <- ggplot(goalie_stats_2425, aes(x = GAA, y = total_mins)) +
  geom_point(color = "gold", alpha = 0.67, size = 2.41) +
  labs(title = "Goals Against Average vs Minutes Played",
       x = "Goals Against Average(GAA)",
       y = "Minutes Played") +
  theme_minimal()

GAA_vs_mins_plot

# Save the plot as PNG for the blog
ggsave("assets/images/GAA_vs_mins_plot.png", plot = GAA_vs_mins_plot, width = 8, height = 6, dpi = 300)

#that's all the visuals I'll do. I fixed/learned a ton about the datasets, so very worth it.

```
