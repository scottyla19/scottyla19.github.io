---
layout: post
title:  ADP Draft Tracker
date:   2016-08-28 11:41:37 -0600
categories: NFL Fantasy
---

# ADP Draft Tracker

Three years ago a bunch of my friends started an online NFL fantasy league. The first year I spent a ton of time researching and working on my draft strategy only to end up with a mediocre team that snuck into the playoffs and exit in the first round. In an attempt to improve and save the embarassment as well as the financial burden ($15 to regular season champ from each player who misses the playoffs) I decided to try something new. I learned about drafting using the average draft position (ADP) to determine who I select and when.

ADP allows for me to save time and make better choices. If you don't believe me check out [Bayes Fantasy Football](http://www.bayesff.com/draft/). I would highly recommend their weekly rankings as well as they helped me out a ton with in season decisions. So last year I used ADP and ended up with a great team that actually ended up winning the league.

ADP is great however I ran into another problem during the draft. I was using [www.myfantasyleague.com](http://www03.myfantasyleague.com/2016/adp) which provides great data to help me make ADP decisions. The problem I had was trying to keep track of which players were already drafted and who was still available. So I decided to try to create my own ADP draft tracker. It is a real basic web app that is available [here](https://scott-laforest.shinyapps.io/adp-tracker/)


If you are curious, it is a [Shiny](http://shiny.rstudio.com/) app. It was all written using R and gathers the data from the My Fantasy League site. If you would like to see the code check it out on [github](https://github.com/scottyla19/draft-tracker)