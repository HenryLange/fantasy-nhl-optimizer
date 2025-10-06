---
title: "Week 3: Adding Filters, Projections, and Improving Tables"
date: 2025-09-21
---


Last week, I did some very basic steps that come before the advanced optimization. However, a scoring system is needed for advanced optimization and was a good foundation for what I plan to get into this week and next week. I took a look at the top 20 players and goalies based on total points, but realistically, this isn’t the best measure because some players and goalies play more or fewer games than others. That’s why the first thing I will do this week is look at **fantasy points per game, and also make separate tables for each position.** 


---


### Player Projections 
So as mentioned above, total fantasy points is great to get a good idea about who the better and worse players are to pick. But I can do much better when it comes to picking a team that will perform well if I do one thing. That is projecting points based on the scoring system and total games played in the 2024-2025 season. Here’s a look at what I did:
```
# adding projected fantasy points per game for skaters
player_stats_2425 <- player_stats_2425 %>%
  mutate(FP_per_game = fantasy_points / GP,
         Proj_fantasy_points = FP_per_game * 82)
```
What this step does is take each player and find how many **fantasy points they averaged per game.** Then I scaled that number by the number of games in an NHL season to get each player’s **projected points** assuming they don’t get injured. This is crucial because instead of just saying “this player had the most points in 2024-2025, I now have a number that predicts how many points each player would get if they performed the same as the prior year.


---


### Goalie Projections 
I couldn’t do the same thing for goalies that I did with players, since even the best goalies don’t play 82 games. If I averaged each goalie’s fantasy points and scaled that amount by 82, it wouldn’t be a good predictor of the points they’d get this season. I made **2 measures** for goalies to overcome this. First, I made one I just called a projection, but it was just **the number of fantasy points they got the year prior.** This is probably the closest I would get with a prediction, given the randomness of the position. The other thing I did to measure was just **find the average fantasy points per game for each goalie and not scale it at all.** Theoretically, this would be a good way to find which goalies did the best when it came to getting fantasy points.


---


### new tables using projected points
Now that I had these projections, I was another step closer to truly optimizing the draft, especially for leagues with no restriction on salary cap or the makeup of a roster. With these projections, I then made all new tables that give the best options for players to select based on position. I made a table with the top 10 fantasy point getters of each position, but I also wanted one that showed more players in case someone is in a much larger league. That led me to make a new table for each position “C, RW, LW, W, D, and F” with the greatest point getters on the top and least on the bottom. Here is what that looks like for Centers, Defense, and goalies:

Centers:

|player           | Proj_fantasy_points|
|:----------------|-------------------:|
|Nathan MacKinnon |            515.8734|
|Leon Draisaitl   |            515.0986|
|Auston Matthews  |            475.4776|
|Connor McDavid   |            451.0000|
|Jack Eichel      |            436.6234|
|Jack Hughes      |            426.5323|
|Brayden Point    |            378.5844|
|William Nylander |            378.0000|
|Jake Guentzel    |            377.2000|
|Sam Reinhart     |            376.2658|


---


Defense:

|player           | Proj_fantasy_points|
|:----------------|-------------------:|
|Cale Makar       |            459.2000|
|Zach Werenski    |            432.7778|
|Quinn Hughes     |            378.6471|
|Rasmus Dahlin    |            363.9452|
|Victor Hedman    |            347.7215|
|Evan Bouchard    |            346.5000|
|MacKenzie Weegar |            323.4444|
|Josh Morrissey   |            316.2125|
|Jake Walman      |            315.3846|
|Jake Sanderson   |            314.6750|


---


Goalies:

|player              | Proj_fantasy_points|
|:-------------------|-------------------:|
|Connor Hellebuyck   |               339.8|
|Andrei Vasilevskiy  |               306.0|
|Filip Gustavsson    |               266.2|
|Jake Oettinger      |               257.8|
|Ilya Sorokin        |               247.4|
|Igor Shesterkin     |               242.8|
|Sam Montembeault    |               237.6|
|Mackenzie Blackwood |               237.4|
|Darcy Kuemper       |               236.2|
|Dustin Wolf         |               234.8|


---


### fixing small issues
One thing that I initially ran into when making these tables is that some players had one game played where they did very well, so their fantasy point projection was very high. I looked down my tables for all positions and noticed that there were some players like that in every table, who had 1-3 games played, but an insanely high projection. This obviously wasn’t good because who even knows if those players will be available to draft in the upcoming season? All I did to fix this was add filters for games played being greater than or equal to 30 for players, and Games started being greater than or equal to 10 to 15 for goalies. That solved the problem. Here is an example of that:
```
# for Centers
centers <- player_stats_2425 %>%
  filter(position == "C" & GP >= 30) %>%
  arrange(desc(Proj_fantasy_points)) %>%
  select(player, Proj_fantasy_points)
```
The “GP >= 30” is what I am referring to, and what solved my problems easily.


---


### More visuals!
I just can’t get enough of the data visualization, so I had to do more. This time, I tried to challenge myself with a few arguments in R that I only learned about briefly in reading/at Wharton. 

The visualization below highlights the **10 skaters with the highest projected fantasy points** based on my scoring system. It makes it easy to see which players stand out as elite options. Names like MacKinnon and Draisaitl rise clearly above the rest

![top10_players_by_projected_points]({{ site.baseurl }}/assets/images/top10_projected_skaters_real.png)

Goalies play a very different role in fantasy hockey, but this bar chart shows the **10 most valuable goalies by projected fantasy points.** Again, highly known names like Hellebuyck and Vasilevskiy are at the top.

![top10_goalies_by_projected_points]({{ site.baseurl }}/assets/images/top10_projected_goalies.png)

Something similar between the two visuals is that there are a couple of players who do much better than the rest of the top 10, and those other six to seven players have a very close projected point spread.


---


### What’s next?
This week is much closer to optimization than any of the other weeks. I now have projections for players by position, and one could follow these tables to pick a team with the highest projected points based on which players are available. However, this is still quite simple, and I’m hoping to dive even deeper next week.

You can see my code for this week in the [code](https://henrylange.github.io/fantasy-nhl-optimizer/code/) section.

Next week, I’ll start tackling **more advanced optimization, draft simulations, and advanced scoring features**, adding the real constraints and possibilities that one may run into when drafting a team, depending on his or her league. The real, complex, and fun optimization.


---
