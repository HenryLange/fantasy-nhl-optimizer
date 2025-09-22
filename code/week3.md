---
title: "Week 3 Code"
layout: page
permalink: /code/week3/
---

```r
#######################
###WEEK 3 NEW TABLES###
#######################

# adding projected fantasy points per game for skaters
player_stats_2425 <- player_stats_2425 %>%
  mutate(FP_per_game = fantasy_points / GP,
         Proj_fantasy_points = FP_per_game * 82)

# I'm not gonna do the same For goalies since they don't play 82 games. It's better to use their points from last year as a projection
goalie_stats_2425 <- goalie_stats_2425 %>% 
  mutate(Proj_fantasy_points = fantasy_points)

# finding best players based on projected points for skaters by position
top_players_by_position <- player_stats_2425 %>%
  group_by(position) %>%                    
  arrange(desc(Proj_fantasy_points)) %>%    
  slice_head(n = 10) %>%                   
  select(player, position, Proj_fantasy_points) %>%
  ungroup()

# View the table
top_players_by_position

# I think I'll actually make separate tables for each position

# for Centers
centers <- player_stats_2425 %>%
  filter(position == "C") %>%
  arrange(desc(Proj_fantasy_points)) %>%
  select(player, Proj_fantasy_points)

# for Left Wingers
left_wings <- player_stats_2425 %>%
  filter(position == "LW") %>%
  arrange(desc(Proj_fantasy_points)) %>%
  select(player, Proj_fantasy_points)

# for Right Wingers
right_wings <- player_stats_2425 %>%
  filter(position == "RW") %>%
  arrange(desc(Proj_fantasy_points)) %>%
  select(player, Proj_fantasy_points)

# for defensemen
defense <- player_stats_2425 %>%
  filter(position == "D") %>%
  arrange(desc(Proj_fantasy_points)) %>%
  select(player, Proj_fantasy_points)

# Now I have a separate table for each position ranking players based on projected points

# making a table ranking by projected points, keeping all stats in this table
top_players_by_proj_points <- player_stats_2425 %>% 
  arrange(desc(Proj_fantasy_points))

top_players_by_proj_points

# I've spotted a bit of an issue. My top player has only one game. I should make a filter for games played

top_players_by_proj_points <- top_players_by_proj_points %>% 
  filter(GP >= 30)

top_players_by_proj_points

#same for goalies

# making a table ranking by projected points, keeping all stats in this table
top_goalies_by_proj_points <- goalie_stats_2425 %>% 
  arrange(desc(Proj_fantasy_points)) %>% 
  filter(GS >= 10)

top_goalies_by_proj_points

# im also gonna add that filter to my other tables

# for Centers
centers <- player_stats_2425 %>%
  filter(position == "C" & GP >= 30) %>%
  arrange(desc(Proj_fantasy_points)) %>%
  select(player, Proj_fantasy_points)

# for Left Wingers
left_wings <- player_stats_2425 %>%
  filter(position == "LW" & GP >= 30) %>%
  arrange(desc(Proj_fantasy_points)) %>%
  select(player, Proj_fantasy_points)

# for Right Wingers
right_wings <- player_stats_2425 %>%
  filter(position == "RW" & GP >= 30) %>%
  arrange(desc(Proj_fantasy_points)) %>%
  select(player, Proj_fantasy_points)

# for defensemen
defense <- player_stats_2425 %>%
  filter(position == "D" & GP >= 30) %>%
  arrange(desc(Proj_fantasy_points)) %>%
  select(player, Proj_fantasy_points)

# i just remembered I also forgot to do it for the position "F" and "W"

# for just "Wingers"
wingers <- player_stats_2425 %>%
  filter(position == "W" & GP >= 30) %>%
  arrange(desc(Proj_fantasy_points)) %>%
  select(player, Proj_fantasy_points)

# for just "forwards"
forwards <- player_stats_2425 %>%
  filter(position == "F" & GP >= 30) %>%
  arrange(desc(Proj_fantasy_points)) %>%
  select(player, Proj_fantasy_points)

# visualization for players
top10_players_for_visual <- top_players_by_proj_points %>% 
  slice(1:10)

top10_projected_skaters_real = ggplot(top10_players_for_visual, aes(x = reorder(player, Proj_fantasy_points), y = Proj_fantasy_points)) +
  geom_bar(stat = "identity", fill = "blue") +
  coord_flip() +
  labs(title = "Top 10 Projected Skaters",
       x = "Player", y = "Projected Points") +
  theme_minimal()

top10_projected_skaters_real

# save for blog
ggsave("assets/images/top10_projected_skaters_real.png", plot = top10_projected_skaters_real, width = 8, height = 6, dpi = 300)

# I actually really like how that visual turned out. Ill do the same for goalies
# Get top 10 goalies
top10_goalies <- top_goalies_by_proj_points %>%
  slice(1:10)

# Create bar chart
top10_projected_goalies = ggplot(top10_goalies, aes(x = reorder(player, Proj_fantasy_points), y = Proj_fantasy_points)) +
  geom_bar(stat = "identity", fill = "aquamarine2") +
  coord_flip() + 
  labs(title = "Top 10 Projected Goalies",
       x = "Goalie", y = "Projected Fantasy Points") +
  theme_minimal()

top10_projected_goalies

# save for blog
ggsave("assets/images/top10_projected_goalies.png", plot = top10_projected_goalies, width = 8, height = 6, dpi = 300)
```

