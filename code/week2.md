---
title: "Week 2 Code"
layout: page
permalink: /code/week2/
---

```r

####################
###SCORING SYSTEM###
####################

#player scoring system
#I am using point values from fantasydata.com, and I will make a new column to calculate what every players fantasy points would be for this season

player_stats_2425 <- player_stats_2425 %>%
  mutate(fantasy_points = (3 * G) +
      (2 * A) +
      (1 * plusMinus) +
      (0.5 * SOG) +
      (0.5 * BLK) +
      (0.5 * powerPlayGoals) +
      (0.5 * powerPlayAssists) +
      (0.5 * shortHandedGoals) +
      (0.5 * shorthandedAssists)
  )

# same thing but now for goalies and their scoring system

goalie_stats_2425 <- goalie_stats_2425 %>% 
  mutate(fantasy_points = (3 * W) +
      (-1 * GA) + 
      (0.2 * saves) +
      (2 * shutouts)
  )

# create table for players with only their fantasy points
fantasy_points_skaters <- player_stats_2425 %>%
  select(player, position, fantasy_points) %>%
  arrange(desc(fantasy_points))

# do the same for goalies
fantasy_points_goalies <- goalie_stats_2425 %>%
  select(player, fantasy_points) %>%
  arrange(desc(fantasy_points))

# get top players for skaters
top_20_skaters = head(fantasy_points_skaters, 20)

# get this for blog
write.csv(top_20_skaters, "assets/data/top_20_skaters.csv", row.names = FALSE)

#get the top goalies
top_20_goalies = head(fantasy_points_goalies, 20)

# also get this one for blog
write.csv(top_20_goalies, "assets/data/top_20_goalies.csv", row.names = FALSE)

#visualizing fantasy points by position
fantasy_points_by_position <- ggplot(fantasy_points_skaters, aes(x = position, y = fantasy_points, fill = position)) +
geom_boxplot(alpha = 0.67) +
  labs(title = "Distribution of Fantasy Points by Position",
       x = "Position", y = "Total Fantasy Points") +
  theme_minimal() +
  theme(legend.position = "none")

fantasy_points_by_position

# save for blog
ggsave("assets/images/fantast_points_by_position.png", plot = fantasy_points_by_position, width = 8, height = 6, dpi = 300)

```
