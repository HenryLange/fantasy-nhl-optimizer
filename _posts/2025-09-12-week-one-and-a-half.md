---
title: "Week 1.5: exploring the data and visuals
date: 2025-09-11
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

# Visual Time

This process was very interesting and important actually. While making my first visual, I found that the numbers in columns weren't actually numeric values amd were discrete values, so making visuals wasn't even possible to begin with. I fixed this as well as many other problems that I ran into along the way that will be shown

my first wisual was a distribution of skaters' points. It was in this step that I got an error and realized my values weren't numeric. however once I fixed that, the visual worked!

![Points Distribution]({{ site.baseurl }}/assets/images/dist_of_skater_points.png)

test for visual format
