---
layout: post
title:  Simple College Football Rankings
date:   2016-11-02 13:38:01 -0600
categories: CFB
---

# Super Simple College Football Rankings

After the first College Football Playoffs ranking came out there was some controversy over the top four. Nobody was complaining about Alabama as they are in a league of their own, Michigan has been beating teams by a bunch of points, and Clemson has navigated a pretty tough schedule to an undefeated record. However Texas A&M was ahead of Washington, even though UW is undefeated and A&M has one loss to Alabama.

I decided to try seeing what I could do to come up with a simple rating just to see what I could learn. I started by gathering the data and doing a simple comparison between teams and their scoring differential. It's not super accurate or overly complex but it does seem to pass the eye test atleast for the top four. Here's how I did it.

## Load necessary packages and create helper functions.


``` r
library(rvest)
library(dplyr)
library(lubridate)
library(stringr)

#helper function to calculate points diff for a single team
calcTeamTotal <- function(df, team){
  teamWins <- filter(df, df$Winner == team)
  teamLosses <- filter(df, df$Loser == team)
  totalgames <- nrow(teamWins) + nrow(teamLosses)
  teamPlus <- sum(teamWins$WinPts - teamWins$LosePts) - sum(teamLosses$WinPts - teamLosses$LosePts)
  return(c(teamPlus, totalgames))
}

#' Calculate total score differential for a team

#' @param df A data frame
#' @param team A char string
#' @return The sum of a team's scores so far this year

calcOpponentTotals <- function(df, team){
  rows <- rownames(df)[df$Winner == team | df$Loser == team]
  games <- df[rows,]
  opponents <- unique( games$Winner)
  opponents <- append(opponents,unique( games$Loser))
  opponents <- opponents[!grepl(team, opponents)]
  opponents <- opponents[!is.na(opponents)]
  df <- df[! rownames(df) %in% rows, ]
  total <- 0
  numGames <- 0
  for (t in opponents) {
    data <- calcTeamTotal(df, t)
    total <- total + data[1]
    numGames <- numGames + data[2]
     
  }
  return(total/numGames)
}
```

## Fetch and Tidy the Data

``` r
#fetch data from sports-ref site and create df
myHtml <- html("http://www.sports-reference.com/cfb/years/2016-schedule.html")

cfb_df <- myHtml %>%
  html_nodes("table") %>%
  html_table()%>%
  data.frame()

#tidy up the data
#convert date column to date types
cfb_df$Date <- mdy(cfb_df$Date)

#remove non-game rows and rename columns
cfb_df <- filter(cfb_df, Rk != "Rk") %>%
  rename( IsWinnerHome = Var.8 , Winner = Winner.Tie, Loser = Loser.Tie, WinPts = Pts, LosePts = Pts.1)

#convert strings to integers
cfb_df$Rk <- as.integer(cfb_df$Rk)
cfb_df$WinPts <- as.integer(cfb_df$WinPts)
cfb_df$LosePts <- as.integer(cfb_df$LosePts)

tail(cfb_df)
```

    ##      Rk Wk       Date     Time Day              Winner WinPts IsWinnerHome
    ## 824 824 15 2016-12-03 12:00 PM Sat        Kansas State     NA            @
    ## 825 825 15 2016-12-03 12:00 PM Sat Louisiana-Lafayette     NA            @
    ## 826 826 15 2016-12-03 12:00 PM Sat    New Mexico State     NA            @
    ## 827 827 15 2016-12-03 12:00 PM Sat (22) Oklahoma State     NA            @
    ## 828 828 15 2016-12-03 12:00 PM Sat                Troy     NA            @
    ## 829 829 16 2016-12-10  4:00 PM Sat                Navy     NA            @
    ##                Loser LosePts  TV Notes
    ## 824  Texas Christian      NA
    ## 825 Louisiana-Monroe      NA
    ## 826    South Alabama      NA
    ## 827    (12) Oklahoma      NA
    ## 828 Georgia Southern      NA
    ## 829             Army      NA CBS

Notice how any ranked teams show up with a (rank) in front of their names. This will cause a problem later when trying to find unique teams as most everyone's rank will change almost weekly. So I will extract that ranking and place it in a new column.

``` r
#remove rank from team names and move rank value to new column
cfb_df$WinRank <- str_extract(cfb_df$Winner, '[0-9]{1,2}')
cfb_df$Winner <- sub("\\(.+\\).","", cfb_df$Winner)
cfb_df$LoseRank <- str_extract(cfb_df$Loser, '[0-9]{1,2}')
cfb_df$Loser <- sub("\\(.+\\).","", cfb_df$Loser)
tail(cfb_df)
```

    ##      Rk Wk       Date     Time Day              Winner WinPts IsWinnerHome
    ## 824 824 15 2016-12-03 12:00 PM Sat        Kansas State     NA            @
    ## 825 825 15 2016-12-03 12:00 PM Sat Louisiana-Lafayette     NA            @
    ## 826 826 15 2016-12-03 12:00 PM Sat    New Mexico State     NA            @
    ## 827 827 15 2016-12-03 12:00 PM Sat      Oklahoma State     NA            @
    ## 828 828 15 2016-12-03 12:00 PM Sat                Troy     NA            @
    ## 829 829 16 2016-12-10  4:00 PM Sat                Navy     NA            @
    ##                Loser LosePts  TV Notes WinRank LoseRank
    ## 824  Texas Christian      NA              <NA>     <NA>
    ## 825 Louisiana-Monroe      NA              <NA>     <NA>
    ## 826    South Alabama      NA              <NA>     <NA>
    ## 827         Oklahoma      NA                22       12
    ## 828 Georgia Southern      NA              <NA>     <NA>
    ## 829             Army      NA CBS          <NA>     <NA>

``` r
#remove games not yet played
cfb_df <- filter(cfb_df, WinPts != "")
```

## Analyzing the Data


Finally I get the unique teams and calculate their average point differential then calculate the average point differentials of all of their opponents.

``` r
#get all teams
unique_win <- unique(cfb_df$Winner) %>% sort()
unique_lose <- unique(cfb_df$Loser) %>% sort()
uni <- unique(c(unique_lose,unique_win))

team_totals <- data.frame(Team = character(), PtDiff=integer(),stringsAsFactors = FALSE)
for (t in uni) {
  tot <- calcTeamTotal(cfb_df, t)
  vec <- list("Team"= t, "PtDiff" =tot[1]/tot[2])
  team_totals[nrow(team_totals)+1,] <- vec
}

team_totals[order(team_totals$PtDiff, decreasing = TRUE),]
```

    ##                            Team      PtDiff
    ## 208                    Michigan  37.3333333
    ## 212                  Washington  31.3333333
    ## 126                  Ohio State  31.0000000
    ## 83                   Louisville  29.0000000
    ## 201                     Alabama  26.8888889
    ## 214            Western Michigan  25.8888889
    ## 204                     Clemson  22.8888889
    ## 143             San Diego State  20.6666667
    ## 13                       Auburn  18.7777778
    ## 191            Washington State  18.3333333
    ## 178                        Troy  18.1250000
    ## 176                      Toledo  17.5555556
    ## 211                    Richmond  17.0000000
    ## 58                      Houston  16.7777778
    ## 195            Western Kentucky  16.7000000
    ## 32                     Colorado  16.4444444
    ## 170                   Texas A&M  15.7777778
    ## 91                   Miami (FL)  15.0000000
    ## 80               Louisiana Tech  14.8000000
    ## 16                       Baylor  14.7500000
    ## 188               Virginia Tech  14.6666667
    ## 89                      Memphis  14.0000000
    ## 128              Oklahoma State  13.2222222
    ## 151               South Florida  13.2222222
    ## 42                      Florida  12.8750000
    ## 193               West Virginia  12.6250000
    ## 127                    Oklahoma  12.5555556
    ## 18                  Boise State  12.1111111
    ## 164                      Temple  12.1000000
    ## 79              Louisiana State  11.7500000
    ## 180                       Tulsa  11.4444444
    ## 12                         Army  11.2222222
    ## 7             Appalachian State  10.4444444
    ## 95                    Minnesota  10.3333333
    ## 197                   Wisconsin  10.0000000
    ## 26              Central Florida   9.2222222
    ## 132                  Penn State   8.8888889
    ## 2                     Air Force   8.7777778
    ## 155         Southern California   8.4444444
    ## 171             Texas Christian   8.2222222
    ## 113              North Carolina   8.0000000
    ## 94       Middle Tennessee State   7.2222222
    ## 73                 Kansas State   7.1111111
    ## 110                  New Mexico   6.5555556
    ## 182                        Utah   6.5555556
    ## 125                        Ohio   6.3000000
    ## 165                   Tennessee   6.2222222
    ## 199                     Wyoming   6.1111111
    ## 202                      Albany   6.0000000
    ## 105                        Navy   5.8750000
    ## 106                    Nebraska   5.5555556
    ## 129                Old Dominion   5.5555556
    ## 67                         Iowa   5.2222222
    ## 203            Central Arkansas   5.0000000
    ## 210               Northern Iowa   5.0000000
    ## 213            Western Illinois   5.0000000
    ## 46                Florida State   4.7777778
    ## 158        Southern Mississippi   4.7777778
    ## 175           Texas-San Antonio   4.5555556
    ## 173                  Texas Tech   4.4444444
    ## 96                  Mississippi   4.2222222
    ## 21                Brigham Young   4.1111111
    ## 205            Eastern Illinois   4.0000000
    ## 169                       Texas   3.7777778
    ## 33               Colorado State   3.1111111
    ## 206          Eastern Washington   3.0000000
    ## 190                 Wake Forest   2.8888889
    ## 116        North Carolina State   2.5555556
    ## 133                  Pittsburgh   2.5555556
    ## 124                  Notre Dame   2.2222222
    ## 86                     Maryland   2.1111111
    ## 207              Illinois State   2.0000000
    ## 209          North Dakota State   2.0000000
    ## 10                     Arkansas   1.8888889
    ## 99                     Missouri   1.3333333
    ## 160                    Stanford   1.3333333
    ## 52             Georgia Southern   1.1111111
    ## 97            Mississippi State   1.1111111
    ## 122                Northwestern   1.1111111
    ## 54                 Georgia Tech   1.0000000
    ## 65                      Indiana   0.5555556
    ## 181                        UCLA   0.3333333
    ## 37                         Duke   0.2222222
    ## 40             Eastern Michigan  -0.4444444
    ## 9                 Arizona State  -0.8888889
    ## 121           Northern Illinois  -0.8888889
    ## 27             Central Michigan  -0.9000000
    ## 117                North Dakota  -1.0000000
    ## 15                   Ball State  -1.2222222
    ## 11               Arkansas State  -1.2500000
    ## 184                  Vanderbilt  -1.3333333
    ## 108            Nevada-Las Vegas  -1.4444444
    ## 112              Nicholls State  -1.5000000
    ## 147              South Carolina  -2.0000000
    ## 51                      Georgia  -2.2222222
    ## 92                   Miami (OH)  -2.3000000
    ## 146               South Alabama  -2.3333333
    ## 38                East Carolina  -2.5555556
    ## 179                      Tulane  -2.7777778
    ## 23                     Cal Poly  -3.0000000
    ## 102               Montana State  -3.0000000
    ## 75                     Kentucky  -3.1111111
    ## 118                 North Texas  -3.8888889
    ## 74                   Kent State  -4.0000000
    ## 130                      Oregon  -4.3333333
    ## 30                   Cincinnati  -4.4444444
    ## 24                   California  -4.7777778
    ## 183                  Utah State  -4.7777778
    ## 81          Louisiana-Lafayette  -5.0000000
    ## 53                Georgia State  -5.2222222
    ## 107                      Nevada  -5.4444444
    ## 3                         Akron  -5.8000000
    ## 63                     Illinois  -5.8888889
    ## 93               Michigan State  -6.5555556
    ## 186                    Virginia  -6.5555556
    ## 85                     Marshall  -6.6666667
    ## 19               Boston College  -6.7777778
    ## 88                McNeese State  -8.0000000
    ## 156           Southern Illinois  -8.0000000
    ## 131                Oregon State  -8.1111111
    ## 157          Southern Methodist  -8.3333333
    ## 68                   Iowa State  -8.5555556
    ## 61                        Idaho  -9.2222222
    ## 29                    Charlotte  -9.5555556
    ## 87                Massachusetts  -9.8000000
    ## 55              Grambling State -10.0000000
    ## 34                  Connecticut -10.4000000
    ## 163                    Syracuse -10.4444444
    ## 57                       Hawaii -11.4000000
    ## 137                      Purdue -11.6666667
    ## 174               Texas-El Paso -11.8888889
    ## 144              San Jose State -12.2000000
    ## 5                 Alabama State -13.0000000
    ## 45        Florida International -13.3000000
    ## 44             Florida Atlantic -13.8888889
    ## 22                      Buffalo -14.7777778
    ## 8                       Arizona -14.8888889
    ## 49                       Furman -15.0000000
    ## 142                     Samford -15.0000000
    ## 48                 Fresno State -15.6000000
    ## 139                        Rice -15.7777778
    ## 1             Abilene Christian -16.0000000
    ## 50                 Gardner-Webb -16.0000000
    ## 82             Louisiana-Monroe -16.7777778
    ## 35                     Delaware -17.0000000
    ## 114          North Carolina A&T -17.0000000
    ## 168            Tennessee-Martin -17.0000000
    ## 200            Youngstown State -17.0000000
    ## 140                     Rutgers -17.3333333
    ## 172                 Texas State -17.7500000
    ## 150          South Dakota State -18.0000000
    ## 152    Southeast Missouri State -18.0000000
    ## 166             Tennessee State -18.0000000
    ## 78                      Liberty -19.0000000
    ## 111            New Mexico State -19.0000000
    ## 101                    Monmouth -20.0000000
    ## 72                       Kansas -20.6666667
    ## 17              Bethune-Cookman -21.0000000
    ## 70           Jacksonville State -21.0000000
    ## 185                   Villanova -21.0000000
    ## 84                        Maine -22.5000000
    ## 187 Virginia Military Institute -23.0000000
    ## 20          Bowling Green State -23.8888889
    ## 25             California-Davis -24.0000000
    ## 39             Eastern Kentucky -24.0000000
    ## 159               Southern Utah -24.0000000
    ## 90                       Mercer -25.0000000
    ## 198                     Wofford -25.0000000
    ## 31                      Colgate -26.0000000
    ## 120           Northern Colorado -26.0000000
    ## 149                South Dakota -27.0000000
    ## 71                James Madison -28.0000000
    ## 141            Sacramento State -28.0000000
    ## 189                      Wagner -28.0000000
    ## 66                Indiana State -30.0000000
    ## 64               Incarnate Word -31.0000000
    ## 109               New Hampshire -31.0000000
    ## 119            Northern Arizona -31.0000000
    ## 154                    Southern -31.0000000
    ## 41                         Elon -33.0000000
    ## 56                      Hampton -33.0000000
    ## 196              William & Mary -34.0000000
    ## 134              Portland State -34.5000000
    ## 100              Missouri State -35.0000000
    ## 47                      Fordham -36.0000000
    ## 177                      Towson -36.0000000
    ## 162                 Stony Brook -38.0000000
    ## 60                       Howard -38.5000000
    ## 192                 Weber State -39.0000000
    ## 59              Houston Baptist -39.5000000
    ## 62                  Idaho State -39.5000000
    ## 14                  Austin Peay -40.0000000
    ## 6                  Alcorn State -42.0000000
    ## 77                        Lamar -42.0000000
    ## 148        South Carolina State -42.0000000
    ## 28          Charleston Southern -44.0000000
    ## 135            Prairie View A&M -44.0000000
    ## 194            Western Carolina -45.0000000
    ## 115      North Carolina Central -46.0000000
    ## 136                Presbyterian -46.0000000
    ## 98     Mississippi Valley State -47.0000000
    ## 123          Northwestern State -48.0000000
    ## 104                Murray State -49.0000000
    ## 138                Rhode Island -49.0000000
    ## 69                Jackson State -50.0000000
    ## 161           Stephen F. Austin -52.0000000
    ## 153      Southeastern Louisiana -54.0000000
    ## 4                   Alabama A&M -55.0000000
    ## 76                    Lafayette -55.0000000
    ## 145              Savannah State -55.0000000
    ## 167              Tennessee Tech -55.0000000
    ## 103                Morgan State -62.0000000
    ## 43                  Florida A&M -67.0000000
    ## 36               Delaware State -79.0000000

``` r
opp_totals <- list()
for (t  in team_totals$Team) {
  data <- calcOpponentTotals(cfb_df, t)
  opp_totals <- append(opp_totals, as.numeric(data))
}

team_totals$OppDiff <- opp_totals

team_totals$totalDiff <- as.numeric(team_totals$PtDiff) + as.numeric(team_totals$OppDiff)
team_totals[order(team_totals$totalDiff , decreasing = TRUE),]
```

    ##                            Team      PtDiff     OppDiff    totalDiff
    ## 208                    Michigan  37.3333333    6.523077  43.85641026
    ## 201                     Alabama  26.8888889    10.66667  37.55555556
    ## 126                  Ohio State  31.0000000    4.986111  35.98611111
    ## 204                     Clemson  22.8888889    8.846154  31.73504274
    ## 83                   Louisville  29.0000000    2.430556  31.43055556
    ## 212                  Washington  31.3333333   -3.753846  27.57948718
    ## 214            Western Michigan  25.8888889   0.2686567  26.15754561
    ## 13                       Auburn  18.7777778    7.185714  25.96349206
    ## 206          Eastern Washington   3.0000000          21  24.00000000
    ## 79              Louisiana State  11.7500000    12.08929  23.83928571
    ## 32                     Colorado  16.4444444    5.947368  22.39181287
    ## 170                   Texas A&M  15.7777778    5.953125  21.73090278
    ## 132                  Penn State   8.8888889    11.73973  20.62861492
    ## 197                   Wisconsin  10.0000000    10.30556  20.30555556
    ## 91                   Miami (FL)  15.0000000        5.25  20.25000000
    ## 195            Western Kentucky  16.7000000    2.078947  18.77894737
    ## 46                Florida State   4.7777778    13.73438  18.51215278
    ## 58                      Houston  16.7777778    1.492063  18.26984127
    ## 188               Virginia Tech  14.6666667    3.138462  17.80512821
    ## 165                   Tennessee   6.2222222    11.54688  17.76909722
    ## 191            Washington State  18.3333333   -0.703125  17.63020833
    ## 127                    Oklahoma  12.5555556    4.736111  17.29166667
    ## 178                        Troy  18.1250000  -0.8392857  17.28571429
    ## 193               West Virginia  12.6250000    4.357143  16.98214286
    ## 164                      Temple  12.1000000    3.931507  16.03150685
    ## 128              Oklahoma State  13.2222222    2.111111  15.33333333
    ## 96                  Mississippi   4.2222222    11.01587  15.23809524
    ## 143             San Diego State  20.6666667   -5.447761  15.21890547
    ## 180                       Tulsa  11.4444444    3.343284  14.78772803
    ## 18                  Boise State  12.1111111    2.430556  14.54166667
    ## 176                      Toledo  17.5555556   -3.514706  14.04084967
    ## 10                     Arkansas   1.8888889    11.88889  13.77777778
    ## 151               South Florida  13.2222222   0.3846154  13.60683761
    ## 26              Central Florida   9.2222222    4.333333  13.55555556
    ## 42                      Florida  12.8750000 -0.07692308  12.79807692
    ## 89                      Memphis  14.0000000     -1.3125  12.68750000
    ## 105                        Navy   5.8750000    6.614035  12.48903509
    ## 155         Southern California   8.4444444        3.75  12.19444444
    ## 80               Louisiana Tech  14.8000000   -2.649351  12.15064935
    ## 113              North Carolina   8.0000000     3.96875  11.96875000
    ## 211                    Richmond  17.0000000       -5.25  11.75000000
    ## 16                       Baylor  14.7500000   -3.267857  11.48214286
    ## 122                Northwestern   1.1111111    10.29688  11.40798611
    ## 73                 Kansas State   7.1111111    3.904762  11.01587302
    ## 116        North Carolina State   2.5555556    7.984375  10.53993056
    ## 7             Appalachian State  10.4444444  -0.3521127  10.09233177
    ## 171             Texas Christian   8.2222222    1.725806   9.94802867
    ## 173                  Texas Tech   4.4444444       5.125   9.56944444
    ## 169                       Texas   3.7777778    5.563636   9.34141414
    ## 133                  Pittsburgh   2.5555556     6.71875   9.27430556
    ## 160                    Stanford   1.3333333    7.819444   9.15277778
    ## 12                         Army  11.2222222        -2.2   9.02222222
    ## 199                     Wyoming   6.1111111         2.6   8.71111111
    ## 2                     Air Force   8.7777778  -0.5384615   8.23931624
    ## 209          North Dakota State   2.0000000       6.125   8.12500000
    ## 95                    Minnesota  10.3333333    -2.34375   7.98958333
    ## 21                Brigham Young   4.1111111     3.43662   7.54773083
    ## 182                        Utah   6.5555556   0.9384615   7.49401709
    ## 106                    Nebraska   5.5555556    1.863014   7.41856925
    ## 94       Middle Tennessee State   7.2222222  -0.2537313   6.96849088
    ## 33               Colorado State   3.1111111    3.630769   6.74188034
    ## 54                 Georgia Tech   1.0000000    5.609375   6.60937500
    ## 37                         Duke   0.2222222    6.307692   6.52991453
    ## 181                        UCLA   0.3333333    5.708333   6.04166667
    ## 86                     Maryland   2.1111111    3.666667   5.77777778
    ## 129                Old Dominion   5.5555556  -0.1363636   5.41919192
    ## 97            Mississippi State   1.1111111        4.25   5.36111111
    ## 67                         Iowa   5.2222222 -0.01754386   5.20467836
    ## 184                  Vanderbilt  -1.3333333    6.515625   5.18229167
    ## 99                     Missouri   1.3333333    3.704918   5.03825137
    ## 213            Western Illinois   5.0000000      -0.375   4.62500000
    ## 9                 Arizona State  -0.8888889         5.5   4.61111111
    ## 203            Central Arkansas   5.0000000  -0.7142857   4.28571429
    ## 130                      Oregon  -4.3333333    8.061538   3.72820513
    ## 207              Illinois State   2.0000000         1.5   3.50000000
    ## 175           Texas-San Antonio   4.5555556    -1.21875   3.33680556
    ## 27             Central Michigan  -0.9000000    4.178082   3.27808219
    ## 124                  Notre Dame   2.2222222           1   3.22222222
    ## 190                 Wake Forest   2.8888889    0.078125   2.96701389
    ## 121           Northern Illinois  -0.8888889    3.415385   2.52649573
    ## 131                Oregon State  -8.1111111    10.35385   2.24273504
    ## 205            Eastern Illinois   4.0000000   -2.111111   1.88888889
    ## 40             Eastern Michigan  -0.4444444    2.212121   1.76767677
    ## 158        Southern Mississippi   4.7777778   -3.333333   1.44444444
    ## 38                East Carolina  -2.5555556    3.538462   0.98290598
    ## 65                      Indiana   0.5555556   0.4109589   0.96651446
    ## 51                      Georgia  -2.2222222       3.125   0.90277778
    ## 110                  New Mexico   6.5555556   -6.396552   0.15900383
    ## 52             Georgia Southern   1.1111111   -0.952381   0.15873016
    ## 11               Arkansas State  -1.2500000        1.25   0.00000000
    ## 24                   California  -4.7777778    4.692308  -0.08547009
    ## 63                     Illinois  -5.8888889     5.65625  -0.23263889
    ## 183                  Utah State  -4.7777778    4.453125  -0.32465278
    ## 157          Southern Methodist  -8.3333333           8  -0.33333333
    ## 125                        Ohio   6.3000000   -6.638889  -0.33888889
    ## 93               Michigan State  -6.5555556     6.09375  -0.46180556
    ## 75                     Kentucky  -3.1111111    2.571429  -0.53968254
    ## 146               South Alabama  -2.3333333    1.274194  -1.05913978
    ## 179                      Tulane  -2.7777778    1.328125  -1.44965278
    ## 147              South Carolina  -2.0000000   0.2876712  -1.71232877
    ## 118                 North Texas  -3.8888889    1.953125  -1.93576389
    ## 3                         Akron  -5.8000000    3.164384  -2.63561644
    ## 68                   Iowa State  -8.5555556    5.859375  -2.69618056
    ## 74                   Kent State  -4.0000000    1.180328  -2.81967213
    ## 163                    Syracuse -10.4444444    7.369231  -3.07521368
    ## 53                Georgia State  -5.2222222           2  -3.22222222
    ## 30                   Cincinnati  -4.4444444    1.188406  -3.25603865
    ## 92                   Miami (OH)  -2.3000000   -1.368421  -3.66842105
    ## 186                    Virginia  -6.5555556    2.878788  -3.67676768
    ## 19               Boston College  -6.7777778    2.863636  -3.91414141
    ## 210               Northern Iowa   5.0000000          -9  -4.00000000
    ## 15                   Ball State  -1.2222222          -3  -4.22222222
    ## 112              Nicholls State  -1.5000000       -2.75  -4.25000000
    ## 152    Southeast Missouri State -18.0000000        13.5  -4.50000000
    ## 140                     Rutgers -17.3333333    12.53846  -4.79487179
    ## 200            Youngstown State -17.0000000          12  -5.00000000
    ## 61                        Idaho  -9.2222222           4  -5.22222222
    ## 108            Nevada-Las Vegas  -1.4444444   -4.852941  -6.29738562
    ## 85                     Marshall  -6.6666667  -0.7538462  -7.42051282
    ## 1             Abilene Christian -16.0000000       7.875  -8.12500000
    ## 34                  Connecticut -10.4000000    2.136986  -8.26301370
    ## 87                Massachusetts  -9.8000000    0.890411  -8.90958904
    ## 29                    Charlotte  -9.5555556   0.5151515  -9.04040404
    ## 8                       Arizona -14.8888889    5.784615  -9.10427350
    ## 5                 Alabama State -13.0000000         3.5  -9.50000000
    ## 23                     Cal Poly  -3.0000000        -6.5  -9.50000000
    ## 57                       Hawaii -11.4000000    1.786667  -9.61333333
    ## 202                      Albany   6.0000000     -15.875  -9.87500000
    ## 81          Louisiana-Lafayette  -5.0000000   -4.962963  -9.96296296
    ## 70           Jacksonville State -21.0000000    10.42857 -10.57142857
    ## 50                 Gardner-Webb -16.0000000    5.222222 -10.77777778
    ## 150          South Dakota State -18.0000000           7 -11.00000000
    ## 48                 Fresno State -15.6000000       4.375 -11.22500000
    ## 144              San Jose State -12.2000000    0.972973 -11.22702703
    ## 109               New Hampshire -31.0000000      19.375 -11.62500000
    ## 137                      Purdue -11.6666667 -0.09230769 -11.75897436
    ## 139                        Rice -15.7777778    3.454545 -12.32323232
    ## 72                       Kansas -20.6666667    8.238095 -12.42857143
    ## 174               Texas-El Paso -11.8888889  -0.8181818 -12.70707071
    ## 107                      Nevada  -5.4444444   -8.179104 -13.62354892
    ## 102               Montana State  -3.0000000      -10.75 -13.75000000
    ## 45        Florida International -13.3000000   -0.746988 -14.04698795
    ## 88                McNeese State  -8.0000000   -6.857143 -14.85714286
    ## 114          North Carolina A&T -17.0000000      2.0625 -14.93750000
    ## 142                     Samford -15.0000000      -0.625 -15.62500000
    ## 35                     Delaware -17.0000000       1.125 -15.87500000
    ## 44             Florida Atlantic -13.8888889   -3.015152 -16.90404040
    ## 20          Bowling Green State -23.8888889    6.666667 -17.22222222
    ## 22                      Buffalo -14.7777778   -2.818182 -17.59595960
    ## 78                      Liberty -19.0000000      1.1875 -17.81250000
    ## 82             Louisiana-Monroe -16.7777778   -1.873016 -18.65079365
    ## 172                 Texas State -17.7500000   -1.535714 -19.28571429
    ## 159               Southern Utah -24.0000000       4.375 -19.62500000
    ## 111            New Mexico State -19.0000000  -0.8709677 -19.87096774
    ## 185                   Villanova -21.0000000        0.25 -20.75000000
    ## 166             Tennessee State -18.0000000       -3.75 -21.75000000
    ## 84                        Maine -22.5000000   0.5294118 -21.97058824
    ## 66                Indiana State -30.0000000       7.875 -22.12500000
    ## 71                James Madison -28.0000000         5.5 -22.50000000
    ## 149                South Dakota -27.0000000           4 -23.00000000
    ## 198                     Wofford -25.0000000       1.625 -23.37500000
    ## 49                       Furman -15.0000000       -9.25 -24.25000000
    ## 156           Southern Illinois  -8.0000000     -16.625 -24.62500000
    ## 14                  Austin Peay -40.0000000          15 -25.00000000
    ## 177                      Towson -36.0000000      10.375 -25.62500000
    ## 120           Northern Colorado -26.0000000        0.25 -25.75000000
    ## 25             California-Davis -24.0000000          -2 -26.00000000
    ## 90                       Mercer -25.0000000          -2 -27.00000000
    ## 101                    Monmouth -20.0000000          -7 -27.00000000
    ## 168            Tennessee-Martin -17.0000000      -10.08 -27.08000000
    ## 17              Bethune-Cookman -21.0000000          -7 -28.00000000
    ## 55              Grambling State -10.0000000         -18 -28.00000000
    ## 117                North Dakota  -1.0000000         -27 -28.00000000
    ## 77                        Lamar -42.0000000      13.625 -28.37500000
    ## 162                 Stony Brook -38.0000000    9.222222 -28.77777778
    ## 134              Portland State -34.5000000    5.352941 -29.14705882
    ## 148        South Carolina State -42.0000000       12.44 -29.56000000
    ## 56                      Hampton -33.0000000       2.125 -30.87500000
    ## 100              Missouri State -35.0000000       3.625 -31.37500000
    ## 187 Virginia Military Institute -23.0000000          -9 -32.00000000
    ## 39             Eastern Kentucky -24.0000000      -10.25 -34.25000000
    ## 47                      Fordham -36.0000000    1.571429 -34.42857143
    ## 196              William & Mary -34.0000000      -1.375 -35.37500000
    ## 119            Northern Arizona -31.0000000      -4.875 -35.87500000
    ## 115      North Carolina Central -46.0000000      8.9375 -37.06250000
    ## 123          Northwestern State -48.0000000          10 -38.00000000
    ## 62                  Idaho State -39.5000000       -0.25 -39.75000000
    ## 59              Houston Baptist -39.5000000   -1.117647 -40.61764706
    ## 189                      Wagner -28.0000000   -12.64706 -40.64705882
    ## 31                      Colgate -26.0000000         -15 -41.00000000
    ## 28          Charleston Southern -44.0000000      -0.125 -44.12500000
    ## 6                  Alcorn State -42.0000000      -3.125 -45.12500000
    ## 153      Southeastern Louisiana -54.0000000       8.125 -45.87500000
    ## 154                    Southern -31.0000000     -14.875 -45.87500000
    ## 41                         Elon -33.0000000     -14.875 -47.87500000
    ## 141            Sacramento State -28.0000000   -20.44444 -48.44444444
    ## 76                    Lafayette -55.0000000        5.75 -49.25000000
    ## 192                 Weber State -39.0000000      -10.25 -49.25000000
    ## 135            Prairie View A&M -44.0000000        -5.5 -49.50000000
    ## 60                       Howard -38.5000000     -13.375 -51.87500000
    ## 136                Presbyterian -46.0000000   -6.111111 -52.11111111
    ## 98     Mississippi Valley State -47.0000000      -6.375 -53.37500000
    ## 161           Stephen F. Austin -52.0000000        -1.5 -53.50000000
    ## 194            Western Carolina -45.0000000        -8.5 -53.50000000
    ## 4                   Alabama A&M -55.0000000        1.25 -53.75000000
    ## 167              Tennessee Tech -55.0000000       0.125 -54.87500000
    ## 64               Incarnate Word -31.0000000   -24.71429 -55.71428571
    ## 69                Jackson State -50.0000000      -7.875 -57.87500000
    ## 43                  Florida A&M -67.0000000         8.5 -58.50000000
    ## 145              Savannah State -55.0000000     -3.5625 -58.56250000
    ## 104                Murray State -49.0000000      -12.75 -61.75000000
    ## 103                Morgan State -62.0000000      -15.25 -77.25000000
    ## 138                Rhode Island -49.0000000     -29.375 -78.37500000
    ## 36               Delaware State -79.0000000      -8.375 -87.37500000

## Conclusion


There is my top 4 Michigan, Alabama, Ohio State, and Clemson. Looking at the top 6 it seems to be fairly accurate with Michigan (Go Blue!) with the largest total point differential (team Diff + Opponent Diff). Other take aways include Bama with the one of the toughest schedules (high OppDiff) and a very large point differential. Also I see why Washington gets no love as their opponents have an negative average differential.

Some obvious drawbacks of this simple approach is that it benefits teams that run up the score and does not take into account any other factors. Also FCS Eastern Washington snuck in their due to their one win against an apparently decent Pac-12 team in Washington State. I could remove all teams in the FCS by removing teams with less than 3 games.
