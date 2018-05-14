---
layout: post
title:  Baby Name Analyzer
date:   2018-05-10 13:38:01 -0600
categories: non sports, dash, app
---

# Baby Name App

Are you expecting? If you need help choosing a name for your new baby why not do some baby names research. I created the app and hosted it using heroku.

[Baby Name Analyzer](https://baby-names-analyzer.herokuapp.com/)

The data for this project uses data from the [Kaggle Datasets](https://www.kaggle.com/datagov/usa-names/data). However instead of using Google's Big Query I chose to download the data myself from [ssa.gov](https://www.ssa.gov/OACT/babynames/limits.html). The only problem with the data was it came in .txt files for each year from 1880 to 2016. In order to use this data I started out using R to clean and combine the data.

```R
library(tidyverse)


df <- read.csv("data/yob1880.txt", header = FALSE) %>%
    rename(Name = V1, Gender = V2, n =  V3) %>%
    mutate(Year = 1880)

for(i in 1881:2016) {
  currentFile <- sprintf("data/yob%d.txt", i)
  df2 <- read.csv(currentFile, header = FALSE) %>%
    rename(Name = V1, Gender = V2, n =  V3) %>%
    mutate(Year = i)
  df <- rbind(df, df2)

}
df <- arrange(df, Name, Year)
write.csv(df, file = "data/names-all-years.csv")
```
After cleaning the data with both R and python and prototyping the app in both R shiny and python Dash frameworks I chose Dash. Two reasons include I am more comfortable with python and R Shiny was extremely slow possibly due to the size of the data.

Use the app to find the most popular names by gender and over a specific time range. Alternatively you can research specific names and see their popularity over time. Enjoy and happy baby name choosing.
