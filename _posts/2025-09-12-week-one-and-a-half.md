---
title: "Week 1.5: exploring the data and visuals"
date: 2025-09-12
---

So, in the last post, I showed a lot about what I did with the data to clean it up to a state where working with it will be smooth. However, something the last week’s post lacked is what was left over in the data after removing all the problems in it. For example, questions like **“what stats do you even have in player data? or for goalie data?”** were left unanswered unless you dove deep into my code. That’s what this week is supposed to cover. What exactly I have in my table of thousands of NHL players.

---

### Skater Data Overview
For skaters, the dataset includes about 30 statistics. Some of the most important ones are:  

- Goals, assists, and total points
- Shots and shooting percentage  
- Time on ice (both total and per game)  
- Faceoff percentage (for centers)  
- Penalty minutes  
- Plus/minus rating  

These stats will form the foundation for any fantasy analysis, since categories like goals, assists, and shots are almost always part of league scoring systems. Some stats like **giveaways** or **takeaways** won’t be as crucial, but it’s still nice to have them for visuals, as well as the fact that some leagues do use very niche stats for scoring.

---

### Goalie Data Overview
The goalie dataset obviously looks a little different, but it is just as important for fantasy hockey. There are still about 30 stats, and some of the key stats include:  

- Games played, games started, and minutes on ice  
- Wins and losses  
- Goals against average (GAA)  
- Save percentage (SV%)  
- Shutouts  

I speak from experience when I say that, despite only one goalie being on the ice for a team during games, goalies can make or break games. To truly have an effective team for fantasy,** A goalie is just as important as any other player** (very unbiased opinion).

---

### Useless Stats?
It’s hard to say that some stats are useless, and I think it is very rare for them to be, but one of the stats that really meant nothing for my project is the **awards** stat. Only a very select few goalies and skaters had awards, so this column just caused a lot of problems in the table. I removed the awards since it would be really hard to make visualizations or involve this in my optimization

---

### Advanced Stats
Some stats from Hockey Reference are very complex. For example, there’s a goalie state called **“goalie points share,”** which is an estimate of the number of points contributed by a goalie due to his play in goal. Since goalies rarely shoot or get assists, this stat is predicting how many points (meaning goals or assists) the goalie indirectly caused in his play. This stat doesn’t go towards any fantasy scoring system and is pretty complex, so it’s likely that this stat won’t get touched on much. This advanced stat and others won’t be fully explained unless they become very involved in the process later on

---

# The Visuals

This process was very interesting and important, actually. While making my first visual, I found that the numbers in columns weren't actually numeric values and were discrete values, so making visuals wasn't even possible to begin with. I fixed this, as well as many other problems that I ran into along the way, and they will be shown later on.

My first visual was a **distribution of skaters' points.** It was in this step that I got an error and realized my values weren't numeric. However, once I fixed that, the visual worked!

![Points Distribution]({{ site.baseurl }}/assets/images/Dist_of_skater_points.png)

Once I fixed that problem, the distribution worked perfectly. This distribution displays that there are a lot of players who get no points all year, and as the amount of points increases, the number of players at that point value decreases. We can also see top players like **Kucherov** and **Draisaitl** at the very end with a tiny bar.

---

### Age vs Points
![Age vs Points]({{ site.baseurl }}/assets/images/age_vs_PTS.png)

I was also interested in how age related to performance. I wondered if older players lost some ability as they aged or if their experience made them perform better. I plotted age and points and, this was the result. It turns out that the oldest and youngest players don't do the best, but it's the players aged in the middle of the pack who get the most points.

---

### Points by Position
![Points by Position]({{ site.baseurl }}/assets/images/PTS_by_position_plot.png)

I also wanted to find out if certain positions did better than others when it came to getting points. Obviously, offensive players are likely to get more goals than defensive players, but a lot of defensemen like **Lane Hutson** get a lot of assists. To visualize this, I made box plots for points and each position. Through this step, I found that there are the main 5 positions, but there is also a position **"W"** for wing, and **"F"** for forward. I was unaware of these positions before I made this plot. The plot for W looks very above average, while for F it looks very below average

---

### Time on Ice vs Points (All Players)
![TOI vs Points]({{ site.baseurl }}/assets/images/TOI_vs_Points.png)

Lastly, for players, I wanted to see if there was a correlation between the time on ice a player gets and the points they have. The first challenge I ran into, was the **"time on ice"** stat was listed as some number like **"1641:21,"** and this colon didn't allow for a visual right away. I had to solve this by separating this column into two, one called mins and one called sec. I then did a little math with that to get a new column that was some amount of minutes like **"1641.34567"** that could be used. I did that, then I made the plot seen above. However, as seen with the line of points at the bottom of the plot, there were some players that had over 3000 minutes and a very small amount of points. How is this possible? It turns out that goalies were actually included in my dataset. I fixed this, as shown below.

---

### Time on Ice vs Points (No Goalies)
![TOI vs Points No Goalies]({{ site.baseurl }}/assets/images/TOI_vs_Points_no_goalies.png)

I removed all goalies from the player dataset and made this new plot. It looks much better after this change. And there is actually a noticeable correlation between time on ice and points, which makes sense because a player who gets more points will likely be played more. **Now onto the goalie visuals!**

---

### Goalie Save Percentage vs Wins
![Save Percentage vs Wins]({{ site.baseurl }}/assets/images/SV_vs_wins.png)

For goalies, the first thing I wanted to do was analyze or visualize **save percentage** since I know how much this stat is valued for a goalie. I plotted save percentage vs wins to see if a goalie's save percentage was correlated to more wins for his team. From the graph, we see that while there are some outlier goalies with very low save percentage, a lot of goalies fall between 0.85 and 0.95, and within this range there are goalies with very few wins but also some with a very high amount of wins. A correlation is hard to get from this plot.

---

### Goalie GAA vs Minutes Played
![GAA vs Minutes Played]({{ site.baseurl }}/assets/images/GAA_vs_mins_plot.png)

I also wanted to test **Goals against average (GAA)** and see how time played by a goalie correlated to their GAA. Does more time correlate to the goalie being better and having a lower GAA, or does more time leave goalies with more opportunities to be scored on and correlate to a lower GAA? This visual shows that many goalies fall around **2.5-3 GAA**, and some goalies have a ton of time while some have very little. Again, correlation is hard to get from this plot.

---

### Save Percentage Histogram
![Save Percentage Histogram]({{ site.baseurl }}/assets/images/save_percentage_histogram.png)

Lastly, I wanted to look at save percentage league-wide. I made a histogram for save percentage and the results aren't too surprising unless you have limited knowledge about goalies in hockey. A lot of goalies fall between 0.85 and 0.95 save percentage, with some goalies on the lower side and higher side. 

---

### In conclusion

I'm glad I made all the visuals I did. I found a lot of small errors in my dataset that I was able to fix, like the time with a colon in it, all numbers being discrete values, and goalies being in the player dataset. I learned a lot, like the "W" and "F" positions and about how different stats correlate. I think I am now in a great position to get into the optimization process next

Additionally, this post was supposed to be released yesterday, however, some of the problems I ran into caused me to have to research for a long time, and I wasn't able to get all my code as well as this post out on the exact time. Nevertheless, I will continue to follow my schedule/structure that I planned at the start of this project.

---

### What's next?

Now I am all set and ready to get into the optimization. I have already learned a bunch so far, but I think it is this step that I will learn the most and be able to share a lot of that. I'm excited to get that started!
