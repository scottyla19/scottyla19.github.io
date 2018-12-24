---
layout: post
title:  NBA Shot Range Analysis
date:   2018-12-24 13:38:01 -0600
author: Scott LaForest
categories: nylon

---

NBA shot range analysis
=======================

## Raw data

To begin, I retrieved raw data for game logs for each team from <https://stats.nba.com/stats/teamgamelog/?TeamID=&Season=2018-19&SeasonType=Regular%20Season> as well as shot chart data for each player from <https://stats.nba.com/stats/shotchartdetail?Period=0&VsConference=&LeagueID=00&LastNGames=0&TeamID=0&Position=&Location=&Outcome=&ContextMeasure=FGA&DateFrom=&StartPeriod=&DateTo=&OpponentTeamID=0&ContextFilter=&RangeType=&Season=2018-19&AheadBehind=&PlayerID=%7B%7D&EndRange=&VsDivision=&PointDiff=&RookieYear=&GameSegment=&Month=0&ClutchTime=&StartRange=&EndPeriod=&SeasonType=Regular+Season&SeasonSegment=&GameID=&PlayerPosition=>. In the URLs from above we must a TeamID from <https://github.com/seemethere/nba_py/wiki/stats.nba.com-Endpoint-Documentation#current-teams> or any PlayerID from the league to get the raw JSON data. I saved the data in a csv to be able to analyze later.

### Raw shot chart data

``` r
shots <- read.csv("shotcharts-2018-19-Dec-10-18.csv")

                         
allShotShort <- shots %>% select(GAME_ID, TEAM_ID,TEAM_NAME,PERIOD, ACTION_TYPE,MINUTES_REMAINING, SECONDS_REMAINING, PLAYER_ID, PLAYER_NAME, EVENT_TYPE, SHOT_TYPE, SHOT_ZONE_BASIC, SHOT_ZONE_AREA, SHOT_ZONE_RANGE,
                                 SHOT_ATTEMPTED_FLAG, SHOT_MADE_FLAG,SHOT_DISTANCE, LOC_X, LOC_Y, HTM, VTM)

head(allShotShort)
```

    ##    GAME_ID    TEAM_ID      TEAM_NAME PERIOD                    ACTION_TYPE
    ## 1 21800001 1610612738 Boston Celtics      1             Driving Layup Shot
    ## 2 21800001 1610612738 Boston Celtics      2           Alley Oop Layup shot
    ## 3 21800001 1610612738 Boston Celtics      2 Driving Finger Roll Layup Shot
    ## 4 21800001 1610612738 Boston Celtics      3                      Jump Shot
    ## 5 21800001 1610612738 Boston Celtics      4             Fadeaway Jump Shot
    ## 6 21800001 1610612738 Boston Celtics      4                      Jump Shot
    ##   MINUTES_REMAINING SECONDS_REMAINING PLAYER_ID PLAYER_NAME  EVENT_TYPE
    ## 1                 3                47    201143  Al Horford   Made Shot
    ## 2                 5                 7    201143  Al Horford   Made Shot
    ## 3                 2                24    201143  Al Horford   Made Shot
    ## 4                11                 5    201143  Al Horford   Made Shot
    ## 5                 7                24    201143  Al Horford Missed Shot
    ## 6                 4                31    201143  Al Horford Missed Shot
    ##        SHOT_TYPE       SHOT_ZONE_BASIC SHOT_ZONE_AREA SHOT_ZONE_RANGE
    ## 1 2PT Field Goal       Restricted Area      Center(C) Less Than 8 ft.
    ## 2 2PT Field Goal       Restricted Area      Center(C) Less Than 8 ft.
    ## 3 2PT Field Goal       Restricted Area      Center(C) Less Than 8 ft.
    ## 4 2PT Field Goal             Mid-Range   Left Side(L)        8-16 ft.
    ## 5 2PT Field Goal In The Paint (Non-RA)      Center(C) Less Than 8 ft.
    ## 6 3PT Field Goal        Right Corner 3  Right Side(R)         24+ ft.
    ##   SHOT_ATTEMPTED_FLAG SHOT_MADE_FLAG SHOT_DISTANCE LOC_X LOC_Y HTM VTM
    ## 1                   1              1             2    24    -1 BOS PHI
    ## 2                   1              1             1    -2    11 BOS PHI
    ## 3                   1              1             1    -9     7 BOS PHI
    ## 4                   1              1            12  -114    53 BOS PHI
    ## 5                   1              0             6    40    57 BOS PHI
    ## 6                   1              0            24   236    53 BOS PHI

### Raw game log data

``` r
logs <- read.csv('all-game-logs.csv')
head(logs)
```

    ##   X    Team_ID  Game_ID    GAME_DATE     MATCHUP WL W  L W_PCT MIN FGM FGA
    ## 1 0 1610612737 21800412 DEC 12, 2018   ATL @ DAL  L 6 21 0.222 240  43  93
    ## 2 1 1610612737 21800380 DEC 08, 2018 ATL vs. DEN  W 6 20 0.231 240  40  93
    ## 3 2 1610612737 21800357 DEC 05, 2018 ATL vs. WAS  L 5 20 0.200 240  41  92
    ## 4 3 1610612737 21800344 DEC 03, 2018 ATL vs. GSW  L 5 19 0.208 240  44  90
    ## 5 4 1610612737 21800325 NOV 30, 2018   ATL @ OKC  L 5 18 0.217 240  43  96
    ## 6 5 1610612737 21800306 NOV 28, 2018   ATL @ CHA  L 5 17 0.227 240  32  93
    ##   FG_PCT FG3M FG3A FG3_PCT FTM FTA FT_PCT OREB DREB REB AST STL BLK TOV PF
    ## 1  0.462   11   33   0.333  10  14  0.714   16   28  44  21   7   2  21 30
    ## 2  0.430   15   42   0.357  11  16  0.688    9   42  51  33   9  11  18 23
    ## 3  0.446   12   38   0.316  23  31  0.742    8   35  43  28  13   4  17 27
    ## 4  0.489    5   26   0.192  18  24  0.750    9   26  35  23  10   4  18 21
    ## 5  0.448   11   36   0.306  12  20  0.600   14   28  42  26  15   3  21 12
    ## 6  0.344   11   44   0.250  19  22  0.864   16   38  54  20  11   8  19 25
    ##   PTS
    ## 1 107
    ## 2 106
    ## 3 117
    ## 4 111
    ## 5 109
    ## 6  94

Data Wrangling
--------------

There was a log of data wrangling and summarizing involved. To start I need to merge the shot chart data with the game log data and select the necessary columns.

``` r
alllogs <- merge(allShotShort,logs,by.x= c("GAME_ID", "TEAM_ID"),by.y=c("Game_ID","Team_ID"))


allLogShort <- alllogs %>% select(GAME_ID, TEAM_ID,TEAM_NAME,PERIOD, ACTION_TYPE,MINUTES_REMAINING, SECONDS_REMAINING, PLAYER_ID, PLAYER_NAME,   
                                  EVENT_TYPE, SHOT_TYPE, SHOT_ZONE_BASIC, SHOT_ZONE_AREA, SHOT_ZONE_RANGE, 
                                  SHOT_ATTEMPTED_FLAG, SHOT_MADE_FLAG,SHOT_DISTANCE, LOC_X, LOC_Y, HTM, VTM,GAME_DATE, MATCHUP, WL)
```

Next there were some initial variables that I wanted so I created a variable for location (home or away), the opponent, a concatenation of date and opponent, team abbreviations, and finally the shot range.

``` r
allLogShort$HOMEAWAY <- "Away"
allLogShort$HOMEAWAY[str_detect(allLogShort$MATCHUP, "vs") ]<- "Home"

allLogShort$OPPONENT <-str_sub(allLogShort$MATCHUP,-3)

allLogShort$DATE_OPPONENT <- paste(allLogShort$GAME_DATE, str_sub(allLogShort$MATCHUP,5), sep = " ")

allLogShort$TEAM_ABRV <- substr(allLogShort$MATCHUP, 0, 3)

allLogShort <- allLogShort %>% mutate(RANGE = ifelse(SHOT_DISTANCE < 4, "RIM",
                                                     ifelse(SHOT_DISTANCE >= 5 & SHOT_DISTANCE < 14, "SMR", 
                                                            ifelse(SHOT_DISTANCE > 14 & SHOT_TYPE == "2PT Field Goal", "LMR", 
                                                            ifelse(SHOT_DISTANCE >= 22 & SHOT_TYPE == "3PT Field Goal", "3PT", "LMR")))))
allLogShort <- allLogShort %>% mutate(SHOT_VALUE = ifelse(SHOT_TYPE == "3PT Field Goal", 3,2))

head(allLogShort)
```

    ##    GAME_ID    TEAM_ID      TEAM_NAME PERIOD        ACTION_TYPE
    ## 1 21800001 1610612738 Boston Celtics      2   Pullup Jump shot
    ## 2 21800001 1610612738 Boston Celtics      4          Jump Shot
    ## 3 21800001 1610612738 Boston Celtics      1   Pullup Jump shot
    ## 4 21800001 1610612738 Boston Celtics      1   Pullup Jump shot
    ## 5 21800001 1610612738 Boston Celtics      2          Jump Shot
    ## 6 21800001 1610612738 Boston Celtics      2 Running Layup Shot
    ##   MINUTES_REMAINING SECONDS_REMAINING PLAYER_ID   PLAYER_NAME  EVENT_TYPE
    ## 1                 8                49   1626179  Terry Rozier   Made Shot
    ## 2                 7                 3    202694 Marcus Morris   Made Shot
    ## 3                 2                11   1626179  Terry Rozier Missed Shot
    ## 4                 9                19   1628369  Jayson Tatum   Made Shot
    ## 5                 7                28   1628369  Jayson Tatum Missed Shot
    ## 6                 9                42   1628369  Jayson Tatum   Made Shot
    ##        SHOT_TYPE       SHOT_ZONE_BASIC        SHOT_ZONE_AREA
    ## 1 2PT Field Goal             Mid-Range  Left Side Center(LC)
    ## 2 3PT Field Goal     Above the Break 3 Right Side Center(RC)
    ## 3 2PT Field Goal             Mid-Range  Left Side Center(LC)
    ## 4 2PT Field Goal In The Paint (Non-RA)             Center(C)
    ## 5 3PT Field Goal        Right Corner 3         Right Side(R)
    ## 6 2PT Field Goal       Restricted Area             Center(C)
    ##   SHOT_ZONE_RANGE SHOT_ATTEMPTED_FLAG SHOT_MADE_FLAG SHOT_DISTANCE LOC_X
    ## 1       16-24 ft.                   1              1            19   -67
    ## 2         24+ ft.                   1              1            26   160
    ## 3       16-24 ft.                   1              0            19  -111
    ## 4        8-16 ft.                   1              1            15   -63
    ## 5         24+ ft.                   1              0            22   227
    ## 6 Less Than 8 ft.                   1              1             1   -18
    ##   LOC_Y HTM VTM    GAME_DATE     MATCHUP WL HOMEAWAY OPPONENT
    ## 1   181 BOS PHI OCT 16, 2018 BOS vs. PHI  W     Home      PHI
    ## 2   209 BOS PHI OCT 16, 2018 BOS vs. PHI  W     Home      PHI
    ## 3   159 BOS PHI OCT 16, 2018 BOS vs. PHI  W     Home      PHI
    ## 4   138 BOS PHI OCT 16, 2018 BOS vs. PHI  W     Home      PHI
    ## 5    29 BOS PHI OCT 16, 2018 BOS vs. PHI  W     Home      PHI
    ## 6    -3 BOS PHI OCT 16, 2018 BOS vs. PHI  W     Home      PHI
    ##          DATE_OPPONENT TEAM_ABRV RANGE SHOT_VALUE
    ## 1 OCT 16, 2018 vs. PHI       BOS   LMR          2
    ## 2 OCT 16, 2018 vs. PHI       BOS   3PT          3
    ## 3 OCT 16, 2018 vs. PHI       BOS   LMR          2
    ## 4 OCT 16, 2018 vs. PHI       BOS   LMR          2
    ## 5 OCT 16, 2018 vs. PHI       BOS   3PT          3
    ## 6 OCT 16, 2018 vs. PHI       BOS   RIM          2

Next, I needed a bunch of summary statistics for my league and team analysis. I needed team and player totals but also team and player totals by shot range. Both are shown below.

``` r
playerStatsByRange <- allLogShort %>% group_by(PLAYER_NAME, RANGE) %>% summarise(FGM = sum(SHOT_MADE_FLAG),FGA = sum(SHOT_ATTEMPTED_FLAG),
                                                                         FG_PERCENTAGE = FGM/FGA, PPS =sum(SHOT_MADE_FLAG * SHOT_VALUE/FGA),  PTS  = sum(SHOT_MADE_FLAG * SHOT_VALUE))
playerStats <- allLogShort %>% group_by(PLAYER_NAME) %>% summarise(FGM = sum(SHOT_MADE_FLAG),FGA = sum(SHOT_ATTEMPTED_FLAG),
                                                           PTS  = sum(SHOT_MADE_FLAG * SHOT_VALUE), EFG = PTS/(FGA*2), PPS = PTS/FGA)

teamStatsByRange <- allLogShort %>% group_by(TEAM_ABRV, RANGE) %>% summarise(FGM = sum(SHOT_MADE_FLAG),FGA = sum(SHOT_ATTEMPTED_FLAG),
                                                                     FG_PERCENTAGE = FGM/FGA, PPS =sum(SHOT_MADE_FLAG * SHOT_VALUE/FGA), PTS  = sum(SHOT_MADE_FLAG * SHOT_VALUE))
teamStats <- allLogShort %>% group_by(TEAM_ABRV) %>% summarise(FGM = sum(SHOT_MADE_FLAG),FGA = sum(SHOT_ATTEMPTED_FLAG),
                                                       PTS  = sum(SHOT_MADE_FLAG * SHOT_VALUE), EFG = PTS/(FGA*2), PPS = PTS/FGA)
```

Merging the range and total data sets for both players and teams we get the following data.

``` r
playerStatsByRange <- playerStatsByRange %>% merge(playerStats, by = 'PLAYER_NAME', suffixes = c("_range","_total"))
head(playerStatsByRange)
```

    ##     PLAYER_NAME RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range
    ## 1  Aaron Gordon   3PT        43       113     0.3805310 1.1415929
    ## 2  Aaron Gordon   LMR        14        54     0.2592593 0.5185185
    ## 3  Aaron Gordon   RIM        84       124     0.6774194 1.3548387
    ## 4  Aaron Gordon   SMR        18        50     0.3600000 0.7200000
    ## 5 Aaron Holiday   3PT        11        43     0.2558140 0.7674419
    ## 6 Aaron Holiday   LMR         8        15     0.5333333 1.0666667
    ##   PTS_range FGM_total FGA_total PTS_total       EFG PPS_total
    ## 1       129       159       341       361 0.5293255 1.0586510
    ## 2        28       159       341       361 0.5293255 1.0586510
    ## 3       168       159       341       361 0.5293255 1.0586510
    ## 4        36       159       341       361 0.5293255 1.0586510
    ## 5        33        37        89        85 0.4775281 0.9550562
    ## 6        16        37        89        85 0.4775281 0.9550562

``` r
teamStatsByRange <- teamStatsByRange %>% merge(teamStats, by = 'TEAM_ABRV', suffixes = c("_range","_total"))
head(teamStatsByRange)
```

    ##   TEAM_ABRV RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range PTS_range
    ## 1       ATL   3PT       303       954     0.3176101 0.9528302       909
    ## 2       ATL   LMR       104       250     0.4160000 0.8360000       209
    ## 3       ATL   RIM       556       910     0.6109890 1.2219780      1112
    ## 4       ATL   SMR       102       292     0.3493151 0.6986301       204
    ## 5       BKN   3PT       348       983     0.3540183 1.0620549      1044
    ## 6       BKN   LMR       107       295     0.3627119 0.7288136       215
    ##   FGM_total FGA_total PTS_total       EFG PPS_total
    ## 1      1065      2406      2434 0.5058188  1.011638
    ## 2      1065      2406      2434 0.5058188  1.011638
    ## 3      1065      2406      2434 0.5058188  1.011638
    ## 4      1065      2406      2434 0.5058188  1.011638
    ## 5      1140      2566      2629 0.5122759  1.024552
    ## 6      1140      2566      2629 0.5122759  1.024552

Calculate shotRangeRatio (% of shots at each range)

``` r
playerStatsByRange$shotRangeRatio <- playerStatsByRange$FGA_range/playerStatsByRange$FGA_total
teamStatsByRange$shotRangeRatio <- teamStatsByRange$FGA_range/teamStatsByRange$FGA_total
```

Analysis
--------

### Players by range sorted on shotRangeRatio (% of shots at each range)

``` r
rimPlayer <- playerStatsByRange %>% filter(RANGE == "RIM" & FGA_range >20) %>% arrange(desc(shotRangeRatio))

smrPlayer <- playerStatsByRange %>% filter(RANGE == "SMR" & FGA_range >20) %>% arrange(desc(shotRangeRatio))

lmrPlayer <- playerStatsByRange %>% filter(RANGE == "LMR" & FGA_range >20) %>% arrange(desc(shotRangeRatio))

threePlayer <- playerStatsByRange %>% filter(RANGE == "3PT" & FGA_range >20) %>% arrange(desc(shotRangeRatio))

head(rimPlayer, n=10L)
```

    ##           PLAYER_NAME RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range
    ## 1      Tyson Chandler   RIM        30        48     0.6250000  1.250000
    ## 2   Mitchell Robinson   RIM        51        79     0.6455696  1.291139
    ## 3            Ed Davis   RIM        63       101     0.6237624  1.247525
    ## 4      DeAndre Jordan   RIM        98       151     0.6490066  1.298013
    ## 5         Rudy Gobert   RIM       145       203     0.7142857  1.428571
    ## 6        Clint Capela   RIM       182       262     0.6946565  1.389313
    ## 7        Damian Jones   RIM        48        62     0.7741935  1.548387
    ## 8         Joakim Noah   RIM        11        22     0.5000000  1.000000
    ## 9    Montrezl Harrell   RIM       141       194     0.7268041  1.453608
    ## 10 Isaiah Hartenstein   RIM        13        25     0.5200000  1.040000
    ##    PTS_range FGM_total FGA_total PTS_total       EFG PPS_total
    ## 1         60        30        50        60 0.6000000 1.2000000
    ## 2        102        55        86       110 0.6395349 1.2790698
    ## 3        126        66       113       132 0.5840708 1.1681416
    ## 4        196       104       171       208 0.6081871 1.2163743
    ## 5        290       158       231       316 0.6839827 1.3679654
    ## 6        364       200       307       400 0.6514658 1.3029316
    ## 7         96        52        73       104 0.7123288 1.4246575
    ## 8         22        12        26        24 0.4615385 0.9230769
    ## 9        282       157       245       314 0.6408163 1.2816327
    ## 10        26        15        32        32 0.5000000 1.0000000
    ##    shotRangeRatio
    ## 1       0.9600000
    ## 2       0.9186047
    ## 3       0.8938053
    ## 4       0.8830409
    ## 5       0.8787879
    ## 6       0.8534202
    ## 7       0.8493151
    ## 8       0.8461538
    ## 9       0.7918367
    ## 10      0.7812500

``` r
head(smrPlayer, n=10L)
```

    ##          PLAYER_NAME RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range
    ## 1       Kosta Koufos   SMR        11        25     0.4400000 0.8800000
    ## 2   Shaun Livingston   SMR        15        39     0.3846154 0.7692308
    ## 3        Robin Lopez   SMR        22        41     0.5365854 1.0731707
    ## 4     T.J. McConnell   SMR        23        41     0.5609756 1.1219512
    ## 5    Harry Giles III   SMR        16        36     0.4444444 0.8888889
    ## 6      Marcin Gortat   SMR        13        29     0.4482759 0.8965517
    ## 7        Jeff Teague   SMR        29        68     0.4264706 0.8529412
    ## 8      DeMar DeRozan   SMR        81       172     0.4709302 0.9418605
    ## 9  Jonas Valanciunas   SMR        43        84     0.5119048 1.0238095
    ## 10  Tristan Thompson   SMR        43        83     0.5180723 1.0361446
    ##    PTS_range FGM_total FGA_total PTS_total       EFG PPS_total
    ## 1         22        24        50        48 0.4800000 0.9600000
    ## 2         30        35        78        70 0.4487179 0.8974359
    ## 3         44        50        96       102 0.5312500 1.0625000
    ## 4         46        61       104       127 0.6105769 1.2211538
    ## 5         32        43        93        86 0.4623656 0.9247312
    ## 6         26        46        84        92 0.5476190 1.0952381
    ## 7         58        80       203       179 0.4408867 0.8817734
    ## 8        162       254       528       514 0.4867424 0.9734848
    ## 9         86       149       258       307 0.5949612 1.1899225
    ## 10        86       141       255       282 0.5529412 1.1058824
    ##    shotRangeRatio
    ## 1       0.5000000
    ## 2       0.5000000
    ## 3       0.4270833
    ## 4       0.3942308
    ## 5       0.3870968
    ## 6       0.3452381
    ## 7       0.3349754
    ## 8       0.3257576
    ## 9       0.3255814
    ## 10      0.3254902

``` r
head(lmrPlayer, n=10L)
```

    ##          PLAYER_NAME RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range
    ## 1      Collin Sexton   LMR        70       168     0.4166667 0.8333333
    ## 2       Gorgui Dieng   LMR        19        47     0.4042553 0.8085106
    ## 3      Terrence Ross   LMR        62       122     0.5081967 1.0163934
    ## 4      DeMar DeRozan   LMR        81       194     0.4175258 0.8350515
    ## 5       Myles Turner   LMR        50       101     0.4950495 0.9900990
    ## 6  LaMarcus Aldridge   LMR        59       157     0.3757962 0.7515924
    ## 7      Klay Thompson   LMR        95       198     0.4797980 0.9595960
    ## 8    Darren Collison   LMR        33        76     0.4342105 0.8684211
    ## 9      Avery Bradley   LMR        24        59     0.4067797 0.8135593
    ## 10          JR Smith   LMR        10        26     0.3846154 0.7692308
    ##    PTS_range FGM_total FGA_total PTS_total       EFG PPS_total
    ## 1        140       176       387       376 0.4857881 0.9715762
    ## 2         38        52       116       108 0.4655172 0.9310345
    ## 3        124       144       317       347 0.5473186 1.0946372
    ## 4        162       254       528       514 0.4867424 0.9734848
    ## 5        100       137       280       289 0.5160714 1.0321429
    ## 6        118       202       439       404 0.4601367 0.9202733
    ## 7        190       256       567       590 0.5202822 1.0405644
    ## 8         66        96       222       215 0.4842342 0.9684685
    ## 9         48        65       176       146 0.4147727 0.8295455
    ## 10        20        27        78        66 0.4230769 0.8461538
    ##    shotRangeRatio
    ## 1       0.4341085
    ## 2       0.4051724
    ## 3       0.3848580
    ## 4       0.3674242
    ## 5       0.3607143
    ## 6       0.3576310
    ## 7       0.3492063
    ## 8       0.3423423
    ## 9       0.3352273
    ## 10      0.3333333

``` r
head(threePlayer, n=10L)
```

    ##         PLAYER_NAME RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range
    ## 1  Anthony Tolliver   3PT        22        54     0.4074074 1.2222222
    ## 2        Gary Clark   3PT        20        75     0.2666667 0.8000000
    ## 3   Wayne Ellington   3PT        45       122     0.3688525 1.1065574
    ## 4     Darius Miller   3PT        38        99     0.3838384 1.1515152
    ## 5      Alex Abrines   3PT        32       102     0.3137255 0.9411765
    ## 6      Troy Daniels   3PT        23        55     0.4181818 1.2545455
    ## 7      Gerald Green   3PT        35       108     0.3240741 0.9722222
    ## 8      Mike Muscala   3PT        36       101     0.3564356 1.0693069
    ## 9     Landry Shamet   3PT        51       132     0.3863636 1.1590909
    ## 10    Davis Bertans   3PT        39        88     0.4431818 1.3295455
    ##    PTS_range FGM_total FGA_total PTS_total       EFG PPS_total
    ## 1         66        26        59        74 0.6271186 1.2542373
    ## 2         60        24        82        68 0.4146341 0.8292683
    ## 3        135        53       146       151 0.5171233 1.0342466
    ## 4        114        51       122       140 0.5737705 1.1475410
    ## 5         96        42       126       116 0.4603175 0.9206349
    ## 6         69        31        70        85 0.6071429 1.2142857
    ## 7        105        58       142       151 0.5316901 1.0633803
    ## 8        108        53       133       142 0.5338346 1.0676692
    ## 9        153        77       180       205 0.5694444 1.1388889
    ## 10       117        58       122       155 0.6352459 1.2704918
    ##    shotRangeRatio
    ## 1       0.9152542
    ## 2       0.9146341
    ## 3       0.8356164
    ## 4       0.8114754
    ## 5       0.8095238
    ## 6       0.7857143
    ## 7       0.7605634
    ## 8       0.7593985
    ## 9       0.7333333
    ## 10      0.7213115

### Players by PPS

``` r
rimPlayer <- playerStatsByRange %>% filter(RANGE == "RIM" & FGA_range >20) %>% arrange(desc(PPS_range))

smrPlayer <- playerStatsByRange %>% filter(RANGE == "SMR" & FGA_range >20) %>% arrange(desc(PPS_range))

lmrPlayer <- playerStatsByRange %>% filter(RANGE == "LMR" & FGA_range >20) %>% arrange(desc(PPS_range))

threePlayer <- playerStatsByRange %>% filter(RANGE == "3PT" & FGA_range >20) %>% arrange(desc(PPS_range))


head(rimPlayer, n=10L)
```

    ##              PLAYER_NAME RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range
    ## 1          Dwight Powell   RIM        61        75     0.8133333  1.626667
    ## 2            Omri Casspi   RIM        29        36     0.8055556  1.611111
    ## 3         Andre Iguodala   RIM        26        33     0.7878788  1.575758
    ## 4             Al Horford   RIM        49        63     0.7777778  1.555556
    ## 5           Damian Jones   RIM        48        62     0.7741935  1.548387
    ## 6  Giannis Antetokounmpo   RIM       203       263     0.7718631  1.543726
    ## 7            Serge Ibaka   RIM        83       108     0.7685185  1.537037
    ## 8          Pascal Siakam   RIM       115       150     0.7666667  1.533333
    ## 9          Klay Thompson   RIM        46        60     0.7666667  1.533333
    ## 10          Kevin Durant   RIM        82       107     0.7663551  1.532710
    ##    PTS_range FGM_total FGA_total PTS_total       EFG PPS_total
    ## 1        122        72       122       151 0.6188525  1.237705
    ## 2         58        39        72        85 0.5902778  1.180556
    ## 3         52        45        99       104 0.5252525  1.050505
    ## 4         98       112       227       252 0.5550661  1.110132
    ## 5         96        52        73       104 0.7123288  1.424658
    ## 6        406       245       425       498 0.5858824  1.171765
    ## 7        166       200       365       422 0.5780822  1.156164
    ## 8        230       173       284       365 0.6426056  1.285211
    ## 9         92       256       567       590 0.5202822  1.040564
    ## 10       164       293       572       634 0.5541958  1.108392
    ##    shotRangeRatio
    ## 1       0.6147541
    ## 2       0.5000000
    ## 3       0.3333333
    ## 4       0.2775330
    ## 5       0.8493151
    ## 6       0.6188235
    ## 7       0.2958904
    ## 8       0.5281690
    ## 9       0.1058201
    ## 10      0.1870629

``` r
head(smrPlayer, n=10L)
```

    ##         PLAYER_NAME RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range
    ## 1   Otto Porter Jr.   SMR        21        36     0.5833333  1.166667
    ## 2     Jalen Brunson   SMR        16        28     0.5714286  1.142857
    ## 3  Tomas Satoransky   SMR        12        21     0.5714286  1.142857
    ## 4    T.J. McConnell   SMR        23        41     0.5609756  1.121951
    ## 5      Kyrie Irving   SMR        42        76     0.5526316  1.105263
    ## 6    Justin Jackson   SMR        16        29     0.5517241  1.103448
    ## 7       Robin Lopez   SMR        22        41     0.5365854  1.073171
    ## 8     Jarrett Allen   SMR        15        28     0.5357143  1.071429
    ## 9       T.J. Warren   SMR        41        77     0.5324675  1.064935
    ## 10    E'Twaun Moore   SMR        47        89     0.5280899  1.056180
    ##    PTS_range FGM_total FGA_total PTS_total       EFG PPS_total
    ## 1         42       116       237       267 0.5632911  1.126582
    ## 2         32        50       105       109 0.5190476  1.038095
    ## 3         24        54       112       120 0.5357143  1.071429
    ## 4         46        61       104       127 0.6105769  1.221154
    ## 5         84       214       443       494 0.5575621  1.115124
    ## 6         32        64       142       154 0.5422535  1.084507
    ## 7         44        50        96       102 0.5312500  1.062500
    ## 8         30       124       211       251 0.5947867  1.189573
    ## 9         82       148       289       332 0.5743945  1.148789
    ## 10        94       155       304       349 0.5740132  1.148026
    ##    shotRangeRatio
    ## 1       0.1518987
    ## 2       0.2666667
    ## 3       0.1875000
    ## 4       0.3942308
    ## 5       0.1715576
    ## 6       0.2042254
    ## 7       0.4270833
    ## 8       0.1327014
    ## 9       0.2664360
    ## 10      0.2927632

``` r
head(lmrPlayer, n=10L)
```

    ##          PLAYER_NAME RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range
    ## 1        Serge Ibaka   LMR        53        83     0.6385542  1.277108
    ## 2         Jeremy Lin   LMR        15        24     0.6250000  1.250000
    ## 3  Jonas Valanciunas   LMR        16        27     0.5925926  1.185185
    ## 4         Quinn Cook   LMR        32        55     0.5818182  1.163636
    ## 5    Emmanuel Mudiay   LMR        19        34     0.5588235  1.147059
    ## 6       Kemba Walker   LMR        45        79     0.5696203  1.139241
    ## 7         Chris Paul   LMR        21        39     0.5384615  1.128205
    ## 8          JJ Redick   LMR        63       115     0.5478261  1.095652
    ## 9       Kevin Durant   LMR        93       171     0.5438596  1.087719
    ## 10       CJ McCollum   LMR        61       113     0.5398230  1.079646
    ##    PTS_range FGM_total FGA_total PTS_total       EFG PPS_total
    ## 1        106       200       365       422 0.5780822 1.1561644
    ## 2         30        84       163       194 0.5950920 1.1901840
    ## 3         32       149       258       307 0.5949612 1.1899225
    ## 4         64        91       191       217 0.5680628 1.1361257
    ## 5         39        99       222       221 0.4977477 0.9954955
    ## 6         90       231       537       550 0.5121043 1.0242086
    ## 7         44       119       285       279 0.4894737 0.9789474
    ## 8        126       179       410       437 0.5329268 1.0658537
    ## 9        186       293       572       634 0.5541958 1.1083916
    ## 10       122       234       495       525 0.5303030 1.0606061
    ##    shotRangeRatio
    ## 1       0.2273973
    ## 2       0.1472393
    ## 3       0.1046512
    ## 4       0.2879581
    ## 5       0.1531532
    ## 6       0.1471136
    ## 7       0.1368421
    ## 8       0.2804878
    ## 9       0.2989510
    ## 10      0.2282828

``` r
head(threePlayer, n=10L)
```

    ##         PLAYER_NAME RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range
    ## 1  Bojan Bogdanovic   3PT        60       121     0.4958678  1.487603
    ## 2  Dante Cunningham   3PT        23        47     0.4893617  1.468085
    ## 3     Stephen Curry   3PT        92       189     0.4867725  1.460317
    ## 4          Rudy Gay   3PT        31        64     0.4843750  1.453125
    ## 5      Derrick Rose   3PT        40        83     0.4819277  1.445783
    ## 6    Meyers Leonard   3PT        21        45     0.4666667  1.400000
    ## 7        Seth Curry   3PT        21        45     0.4666667  1.400000
    ## 8  Alfonzo McKinnie   3PT        20        43     0.4651163  1.395349
    ## 9   Malcolm Brogdon   3PT        48       104     0.4615385  1.384615
    ## 10       Joe Harris   3PT        59       129     0.4573643  1.372093
    ##    PTS_range FGM_total FGA_total PTS_total       EFG PPS_total
    ## 1        180       162       309       384 0.6213592 1.2427184
    ## 2         69        47       100       117 0.5850000 1.1700000
    ## 3        276       174       346       440 0.6358382 1.2716763
    ## 4         93       130       255       291 0.5705882 1.1411765
    ## 5        120       178       364       396 0.5439560 1.0879121
    ## 6         63        45        92       111 0.6032609 1.2065217
    ## 7         63        33        90        87 0.4833333 0.9666667
    ## 8         60        50        99       120 0.6060606 1.2121212
    ## 9        144       159       309       366 0.5922330 1.1844660
    ## 10       177       120       247       299 0.6052632 1.2105263
    ##    shotRangeRatio
    ## 1       0.3915858
    ## 2       0.4700000
    ## 3       0.5462428
    ## 4       0.2509804
    ## 5       0.2280220
    ## 6       0.4891304
    ## 7       0.5000000
    ## 8       0.4343434
    ## 9       0.3365696
    ## 10      0.5222672

### Team by shotRangeRatio for each range

``` r
rimTeam <- teamStatsByRange %>% filter(RANGE == "RIM") %>% arrange(desc(shotRangeRatio))
smrTeam <- teamStatsByRange %>% filter(RANGE == "SMR") %>% arrange(desc(shotRangeRatio))
lmrTeam <- teamStatsByRange %>% filter(RANGE == "LMR") %>%  arrange(desc(shotRangeRatio))
threeTeam <- teamStatsByRange %>% filter(RANGE == "3PT") %>% arrange(desc(shotRangeRatio))

head(rimTeam)
```

    ##   TEAM_ABRV RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range PTS_range
    ## 1       LAL   RIM       634       984     0.6443089  1.288618      1268
    ## 2       MIL   RIM       652       957     0.6812957  1.362591      1304
    ## 3       ATL   RIM       556       910     0.6109890  1.221978      1112
    ## 4       UTA   RIM       572       868     0.6589862  1.317972      1144
    ## 5       OKC   RIM       560       888     0.6306306  1.261261      1120
    ## 6       NOP   RIM       622       947     0.6568110  1.313622      1244
    ##   FGM_total FGA_total PTS_total       EFG PPS_total shotRangeRatio
    ## 1      1160      2444      2608 0.5335516  1.067103      0.4026187
    ## 2      1163      2449      2703 0.5518579  1.103716      0.3907717
    ## 3      1065      2406      2434 0.5058188  1.011638      0.3782211
    ## 4      1083      2331      2457 0.5270270  1.054054      0.3723724
    ## 5      1086      2391      2435 0.5092012  1.018402      0.3713927
    ## 6      1259      2617      2808 0.5364922  1.072984      0.3618647

``` r
head(smrTeam)
```

    ##   TEAM_ABRV RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range PTS_range
    ## 1       SAS   SMR       251       543     0.4622468 0.9244936       502
    ## 2       MEM   SMR       175       453     0.3863135 0.7726269       350
    ## 3       MIN   SMR       198       475     0.4168421 0.8336842       396
    ## 4       MIA   SMR       160       469     0.3411514 0.6823028       320
    ## 5       NOP   SMR       204       490     0.4163265 0.8326531       408
    ## 6       SAC   SMR       183       454     0.4030837 0.8061674       366
    ##   FGM_total FGA_total PTS_total       EFG PPS_total shotRangeRatio
    ## 1      1156      2493      2571 0.5156438 1.0312876      0.2178099
    ## 2      1011      2226      2285 0.5132525 1.0265049      0.2035040
    ## 3      1082      2416      2449 0.5068295 1.0136589      0.1966060
    ## 4      1039      2405      2403 0.4995842 0.9991684      0.1950104
    ## 5      1259      2617      2808 0.5364922 1.0729843      0.1872373
    ## 6      1180      2454      2661 0.5421760 1.0843521      0.1850041

``` r
head(lmrTeam)
```

    ##   TEAM_ABRV RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range PTS_range
    ## 1       SAS   LMR       251       649     0.3867488 0.7734977       502
    ## 2       GSW   LMR       297       598     0.4966555 0.9933110       594
    ## 3       CLE   LMR       228       563     0.4049734 0.8099467       456
    ## 4       IND   LMR       212       506     0.4189723 0.8379447       424
    ## 5       ORL   LMR       194       484     0.4008264 0.8037190       389
    ## 6       NYK   LMR       218       506     0.4308300 0.8636364       437
    ##   FGM_total FGA_total PTS_total       EFG PPS_total shotRangeRatio
    ## 1      1156      2493      2571 0.5156438 1.0312876      0.2603289
    ## 2      1241      2526      2822 0.5585907 1.1171813      0.2367379
    ## 3      1088      2439      2419 0.4959000 0.9917999      0.2308323
    ## 4      1138      2384      2537 0.5320889 1.0641779      0.2122483
    ## 5      1073      2388      2448 0.5125628 1.0251256      0.2026801
    ## 6      1152      2620      2598 0.4958015 0.9916031      0.1931298

``` r
head(threeTeam)
```

    ##   TEAM_ABRV RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range PTS_range
    ## 1       HOU   3PT       362      1074     0.3370577 1.0111732      1086
    ## 2       MIL   3PT       377      1080     0.3490741 1.0472222      1131
    ## 3       BOS   3PT       355       983     0.3611394 1.0834181      1065
    ## 4       ATL   3PT       303       954     0.3176101 0.9528302       909
    ## 5       DAL   3PT       308       865     0.3560694 1.0682081       924
    ## 6       BKN   3PT       348       983     0.3540183 1.0620549      1044
    ##   FGM_total FGA_total PTS_total       EFG PPS_total shotRangeRatio
    ## 1       993      2212      2350 0.5311935  1.062387      0.4855335
    ## 2      1163      2449      2703 0.5518579  1.103716      0.4409963
    ## 3      1109      2441      2573 0.5270381  1.054076      0.4027038
    ## 4      1065      2406      2434 0.5058188  1.011638      0.3965087
    ## 5      1015      2210      2338 0.5289593  1.057919      0.3914027
    ## 6      1140      2566      2629 0.5122759  1.024552      0.3830865

### Team by PPS for each range

``` r
rimTeam <- teamStatsByRange %>% filter(RANGE == "RIM") %>% arrange(desc(PPS_range))#desc(shotRangeRatio), desc(EFG))
smrTeam <- teamStatsByRange %>% filter(RANGE == "SMR") %>% arrange(desc(PPS_range))#desc(shotRangeRatio), desc(EFG))
lmrTeam <- teamStatsByRange %>% filter(RANGE == "LMR") %>% arrange(desc(PPS_range))#shotRangeRatio), desc(EFG))
threeTeam <- teamStatsByRange %>% filter(RANGE == "3PT") %>% arrange(desc(PPS_range))#shotRangeRatio), desc(EFG))

head(rimTeam)
```

    ##   TEAM_ABRV RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range PTS_range
    ## 1       MIL   RIM       652       957     0.6812957  1.362591      1304
    ## 2       GSW   RIM       428       637     0.6718995  1.343799       856
    ## 3       TOR   RIM       570       852     0.6690141  1.338028      1140
    ## 4       UTA   RIM       572       868     0.6589862  1.317972      1144
    ## 5       PHI   RIM       511       776     0.6585052  1.317010      1022
    ## 6       NOP   RIM       622       947     0.6568110  1.313622      1244
    ##   FGM_total FGA_total PTS_total       EFG PPS_total shotRangeRatio
    ## 1      1163      2449      2703 0.5518579  1.103716      0.3907717
    ## 2      1241      2526      2822 0.5585907  1.117181      0.2521774
    ## 3      1300      2682      2946 0.5492170  1.098434      0.3176734
    ## 4      1083      2331      2457 0.5270270  1.054054      0.3723724
    ## 5      1176      2522      2674 0.5301348  1.060270      0.3076923
    ## 6      1259      2617      2808 0.5364922  1.072984      0.3618647

``` r
head(smrTeam)
```

    ##   TEAM_ABRV RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range PTS_range
    ## 1       SAS   SMR       251       543     0.4622468 0.9244936       502
    ## 2       TOR   SMR       205       457     0.4485777 0.8971554       410
    ## 3       BOS   SMR       162       372     0.4354839 0.8709677       324
    ## 4       IND   SMR       154       360     0.4277778 0.8555556       308
    ## 5       GSW   SMR       176       414     0.4251208 0.8502415       352
    ## 6       LAC   SMR       185       436     0.4243119 0.8486239       370
    ##   FGM_total FGA_total PTS_total       EFG PPS_total shotRangeRatio
    ## 1      1156      2493      2571 0.5156438  1.031288      0.2178099
    ## 2      1300      2682      2946 0.5492170  1.098434      0.1703952
    ## 3      1109      2441      2573 0.5270381  1.054076      0.1523966
    ## 4      1138      2384      2537 0.5320889  1.064178      0.1510067
    ## 5      1241      2526      2822 0.5585907  1.117181      0.1638955
    ## 6      1105      2358      2455 0.5205683  1.041137      0.1849025

``` r
head(lmrTeam)
```

    ##   TEAM_ABRV RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range PTS_range
    ## 1       GSW   LMR       297       598     0.4966555 0.9933110       594
    ## 2       TOR   LMR       179       372     0.4811828 0.9623656       358
    ## 3       NYK   LMR       218       506     0.4308300 0.8636364       437
    ## 4       PHI   LMR       174       408     0.4264706 0.8529412       348
    ## 5       UTA   LMR       106       250     0.4240000 0.8480000       212
    ## 6       DEN   LMR       137       326     0.4202454 0.8435583       275
    ##   FGM_total FGA_total PTS_total       EFG PPS_total shotRangeRatio
    ## 1      1241      2526      2822 0.5585907 1.1171813      0.2367379
    ## 2      1300      2682      2946 0.5492170 1.0984340      0.1387025
    ## 3      1152      2620      2598 0.4958015 0.9916031      0.1931298
    ## 4      1176      2522      2674 0.5301348 1.0602696      0.1617764
    ## 5      1083      2331      2457 0.5270270 1.0540541      0.1072501
    ## 6      1114      2409      2504 0.5197177 1.0394355      0.1353259

``` r
head(threeTeam)
```

    ##   TEAM_ABRV RANGE FGM_range FGA_range FG_PERCENTAGE PPS_range PTS_range
    ## 1       SAC   3PT       301       768     0.3919271  1.175781       903
    ## 2       GSW   3PT       340       877     0.3876853  1.163056      1020
    ## 3       SAS   3PT       259       671     0.3859911  1.157973       777
    ## 4       IND   3PT       261       698     0.3739255  1.121777       783
    ## 5       MIN   3PT       285       770     0.3701299  1.110390       855
    ## 6       LAC   3PT       245       666     0.3678679  1.103604       735
    ##   FGM_total FGA_total PTS_total       EFG PPS_total shotRangeRatio
    ## 1      1180      2454      2661 0.5421760  1.084352      0.3129584
    ## 2      1241      2526      2822 0.5585907  1.117181      0.3471892
    ## 3      1156      2493      2571 0.5156438  1.031288      0.2691536
    ## 4      1138      2384      2537 0.5320889  1.064178      0.2927852
    ## 5      1082      2416      2449 0.5068295  1.013659      0.3187086
    ## 6      1105      2358      2455 0.5205683  1.041137      0.2824427

NBA Summary stats
-----------------

Look at teh PPS\_range and shotRangeRatio to see averages in the NBA for each range.

``` r
allNBA <- teamStatsByRange %>% group_by(RANGE) %>% summarise(TEAM_ABRV = "All" ,FGA_range = sum(FGA_range), FGM_range = sum(FGM_range), 
                                                             FG_PERCENTAGE = FGM_range/FGA_range,PTS_range = sum(PTS_range), EFG = mean(EFG),
                                                             FGM_total = sum(.$FGM_range), FGA_total = sum(.$FGA_range), PTS_total= sum(.$PTS_range),
                                                             PPS_range = PTS_range/FGA_range,shotRangeRatio = mean(shotRangeRatio), 
                                                             PPS_total = sum(.$PTS_range)/sum(.$FGA_range))
head(allNBA)
```

    ## # A tibble: 4 x 13
    ##   RANGE TEAM_ABRV FGA_range FGM_range FG_PERCENTAGE PTS_range   EFG
    ##   <chr> <chr>         <int>     <int>         <dbl>     <dbl> <dbl>
    ## 1 3PT   All           25613      8988         0.351     26964 0.520
    ## 2 LMR   All           11276      4558         0.404      9125 0.520
    ## 3 RIM   All           24167     15176         0.628     30352 0.520
    ## 4 SMR   All           11811      4716         0.399      9432 0.520
    ## # ... with 6 more variables: FGM_total <int>, FGA_total <int>,
    ## #   PTS_total <dbl>, PPS_range <dbl>, shotRangeRatio <dbl>,
    ## #   PPS_total <dbl>

Wrap up and export to csv for Shiny app use
-------------------------------------------

``` r
teamStatsByRange <- rbind(teamStatsByRange, allNBA)
write_csv(teamStatsByRange,"longTeamByRange.csv")
write_csv(allLogShort,"all-log-short.csv")
```
