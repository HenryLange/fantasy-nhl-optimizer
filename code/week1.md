---
title: "Week 1 Code"
layout: page
permalink: /code/week1/
---

```r
# setting wd
setwd("~/NHL Optimizer")

# install lpsolve, I already have others
install.packages("lpSolve")

# loading packages
library(tidyverse)
library(lpSolve)

#load in data
player_stats_2425 = read_csv("player_stats.csv")

# changing column names
player_stats_2425 = player_stats_2425 %>% 
  rename(
    rank = ...1,
    player = ...2,
    age = ...3,
    team = ...4,
    position = ...5,
    GP = ...6,
    G = Scoring...7,
    A = Scoring...8,
    PTS = Scoring...9,
    plusMinus = ...10,
    PIM = ...11,
    evenStrengthGoals = Goals...12,
    powerPlayGoals = Goals...13,
    shortHandedGoals = Goals...14,
    gameWinningGoals = Goals...15,
    evenStrengthAssists = Assists...16,
    powerPlayAssists = Assists...17, 
    shorthandedAssists = Assists...18,
    SOG = Shots...19,
    shootingPercentage = Shots...20,
    totalShotsAttempted = Shots...21,
    timeOnIce = `Ice Time...22`,
    averageTimeOnIce = `Ice Time...23`,
    faceoffWins = Faceoffs...24,
    faceoffloses = Faceoffs...25,
    faceoffPercentage = Faceoffs...26,
    BLK = ...27,
    Hits = ...28,
    takeaways = ...29,
    giveaways = ...30,
    awards = ...31
  )

#eliminating row with names of columns
player_stats_2425 = player_stats_2425 %>% 
  slice(-1)

#getting rid of duplicate players
player_stats_2425 <- player_stats_2425 %>%
  group_by(player) %>%
  filter(n_distinct(team) == 1 | grepl("TM", team)) %>%
  ungroup()

#read in goalie data
goalie_stats_2425 = read_csv("goalie_stats.csv")

# changing column names for goalie stats
goalie_stats_2425 = goalie_stats_2425 %>% 
  rename(
    rank = ...1,
    player = ...2,
    age = ...3,
    team = ...4,
    position = ...5,
    GP = `Goalie Stats...6`,
    GS = `Goalie Stats...7`,
    W = `Goalie Stats...8`,
    L = `Goalie Stats...9`,
    tiesAndOTLosses = `Goalie Stats...10`,
    GA = `Goalie Stats...11`,
    shots = `Goalie Stats...12`,
    saves = `Goalie Stats...13`,
    savePercentage = `Goalie Stats...14`,
    GAA = `Goalie Stats...15`,
    shutouts = `Goalie Stats...16`,
    minutes = `Goalie Stats...17`, 
    qualityStarts = `Goalie Stats...18`,
    qualityStartsPercentage = `Goalie Stats...19`,
    reallyBadStarts = `Goalie Stats...20`,
    goalsAgainstPercentage = `Goalie Stats...21`,
    goalsSavedAboveAverage = `Goalie Stats...22`,
    adjustedGoalsAgainstAverage = `Goalie Stats...23`,
    goaliePointsShare = `Goalie Stats...24`,
    G = Scoring...25,
    A = Scoring...26,
    PTS = Scoring...27,
    PIM = ...28,
    awards = ...29
  )

#eliminating the row with names of columns
goalie_stats_2425 = goalie_stats_2425 %>% 
  slice(-1)

#getting rid of duplicate goalies
goalie_stats_2425 <- goalie_stats_2425 %>%
  group_by(player) %>%
  filter(n_distinct(team) == 1 | grepl("TM", team)) %>%
  ungroup()

# check for N/A
sum(is.na(player_stats_2425))
sum(is.na(goalie_stats_2425))

# Remove the Awards column
player_stats_2425 <- player_stats_2425 %>%
  select(-awards) 
goalie_stats_2425 <- goalie_stats_2425 %>%
  select(-awards)

# check for N/A
sum(is.na(player_stats_2425))
colSums(is.na(player_stats_2425))
rowSums(is.na(player_stats_2425))

# Replace NA in faceoff percentage with 0 for non-centers
player_stats_2425 <- player_stats_2425 %>%
  mutate(faceoffPercentage = ifelse(position != "C" & is.na(faceoffPercentage), 0, faceoffPercentage))

# Replace NA in shooting percentage with 0 for guys who never scored
player_stats_2425 <- player_stats_2425 %>%
  mutate(shootingPercentage = ifelse(is.na(shootingPercentage), 0, shootingPercentage))

#get rid of league average row
player_stats_2425 <- player_stats_2425 %>%
  slice(1:(n() - 1))

# check for N/A
sum(is.na(player_stats_2425))
colSums(is.na(player_stats_2425))
rowSums(is.na(player_stats_2425))

# adding age for Sam Morton
player_stats_2425$age[player_stats_2425$player == "Sam Morton"] <- 25

# adding age for Jack Williams
player_stats_2425$age[player_stats_2425$player == "Jack Williams"] <- 22

# Replace NA in faceoff percentage with 0 for centers (yes there are centers with NA faceoff percentages, I just learned this too)
player_stats_2425 <- player_stats_2425 %>%
  mutate(faceoffPercentage = ifelse(is.na(faceoffPercentage), 0, faceoffPercentage))

# check for N/A
sum(is.na(player_stats_2425))
colSums(is.na(player_stats_2425))
rowSums(is.na(player_stats_2425))

# now there are no NAs in the player stats for 2024-2025

# check for NAs but now for goalies
sum(is.na(goalie_stats_2425))
colSums(is.na(goalie_stats_2425))
rowSums(is.na(goalie_stats_2425))

#get rid of league average row
goalie_stats_2425 <- goalie_stats_2425 %>%
  slice(1:(n() - 1))

# Replace NA in quality starts percentage with 0 for goalies with no starts
goalie_stats_2425 <- goalie_stats_2425 %>%
  mutate(qualityStartsPercentage = ifelse(is.na(qualityStartsPercentage), 0, qualityStartsPercentage))

# check for NAs again
sum(is.na(goalie_stats_2425))
colSums(is.na(goalie_stats_2425))
rowSums(is.na(goalie_stats_2425))

# now there are no more NAs in the goalie stats for 2024-2025

```
