```python
import sqlalchemy
sqlalchemy.create_engine('mysql+mysqlconnector://root:secret@localhost:3306/lahmansbaseballdb')
```


```python
%reload_ext sql
```


```python
%sql mysql+mysqlconnector://root:secret@localhost:3306/lahmansbaseballdb
```

# Lahman's Baseball db SQL practice
I wanted to brush off some of the rust for my SQL skills and I figured the Lahman Baseball database offers a decent database to practice. I downloaded the install script from here and found some practice exercises here. The rest of this post are my attempts to solve the problems. ~~Play Ball!~~ Query data!

## Question 1: 
#### What range of years does the provided database cover?


```sql
%%sql 
SELECT 
    MIN(yearID) AS first_year, MAX(yearID) AS last_year
FROM
    batting;
```

     * mysql+mysqlconnector://root:***@localhost:3306/lahmansbaseballdb
    1 rows affected.





<table>
    <tr>
        <th>first_year</th>
        <th>last_year</th>
    </tr>
    <tr>
        <td>1871</td>
        <td>2019</td>
    </tr>
</table>



## Question 2: 
#### Find the name and height of the shortest player in the database. How many games did he play in? What is the name of the team for which he played?


```sql
%%sql
SELECT 
    p1.playerID,
    p1.nameFirst,
    p1.nameLast,
    p1.height,
    SUM(apps.G_all) AS games_played,
    t.name
FROM
    people p1
        INNER JOIN
    (SELECT 
        MIN(people.height) AS min_height
    FROM
        people) p2 ON p1.height = p2.min_height
        INNER JOIN
    appearances AS apps ON p1.playerID = apps.playerID
        INNER JOIN
    teams t ON t.teamID = apps.teamID
GROUP BY p1.playerID , p1.nameFirst , p1.nameLast , p1.height , t.name;

```

     * mysql+mysqlconnector://root:***@localhost:3306/lahmansbaseballdb
    1 rows affected.





<table>
    <tr>
        <th>playerID</th>
        <th>nameFirst</th>
        <th>nameLast</th>
        <th>height</th>
        <th>games_played</th>
        <th>name</th>
    </tr>
    <tr>
        <td>gaedeed01</td>
        <td>Eddie</td>
        <td>Gaedel</td>
        <td>43</td>
        <td>52</td>
        <td>St. Louis Browns</td>
    </tr>
</table>



# Question 3: 
#### Find all players in the database who played at Vanderbilt University. Create a list showing each player’s first and last names as well as the total salary they earned in the major leagues. Sort this list in descending order by the total salary earned. Which Vanderbilt player earned the most money in the majors?


```sql
%%sql
SELECT 
    p.nameFirst, p.nameLast, SUM(s.salary) total_salary
FROM
    people p
        INNER JOIN
    salaries s ON s.playerID = p.playerID
        INNER JOIN
    collegeplaying cp ON p.playerID = cp.playerID
        INNER JOIN
    schools sc ON sc.schoolID = cp.schoolID
WHERE
    sc.name_full = 'Vanderbilt University'
GROUP BY p.nameFirst , p.nameLast
ORDER BY total_salary DESC;
```

     * mysql+mysqlconnector://root:***@localhost:3306/lahmansbaseballdb
    15 rows affected.





<table>
    <tr>
        <th>nameFirst</th>
        <th>nameLast</th>
        <th>total_salary</th>
    </tr>
    <tr>
        <td>David</td>
        <td>Price</td>
        <td>245553888.0</td>
    </tr>
    <tr>
        <td>Pedro</td>
        <td>Alvarez</td>
        <td>62045112.0</td>
    </tr>
    <tr>
        <td>Scott</td>
        <td>Sanderson</td>
        <td>21500000.0</td>
    </tr>
    <tr>
        <td>Mike</td>
        <td>Minor</td>
        <td>20512500.0</td>
    </tr>
    <tr>
        <td>Joey</td>
        <td>Cora</td>
        <td>16867500.0</td>
    </tr>
    <tr>
        <td>Mark</td>
        <td>Prior</td>
        <td>12800000.0</td>
    </tr>
    <tr>
        <td>Ryan</td>
        <td>Flaherty</td>
        <td>12183000.0</td>
    </tr>
    <tr>
        <td>Josh</td>
        <td>Paul</td>
        <td>7920000.0</td>
    </tr>
    <tr>
        <td>Sonny</td>
        <td>Gray</td>
        <td>4627500.0</td>
    </tr>
    <tr>
        <td>Mike</td>
        <td>Baxter</td>
        <td>4188836.0</td>
    </tr>
    <tr>
        <td>Jensen</td>
        <td>Lewis</td>
        <td>3702000.0</td>
    </tr>
    <tr>
        <td>Matt</td>
        <td>Kata</td>
        <td>3180000.0</td>
    </tr>
    <tr>
        <td>Nick</td>
        <td>Christiani</td>
        <td>2000000.0</td>
    </tr>
    <tr>
        <td>Jeremy</td>
        <td>Sowers</td>
        <td>1154400.0</td>
    </tr>
    <tr>
        <td>Scotti</td>
        <td>Madison</td>
        <td>540000.0</td>
    </tr>
</table>



# Question 4:
#### Using the fielding table, group players into three groups based on their position: label players with position OF as "Outfield", those with position "SS", "1B", "2B", and "3B" as "Infield", and those with position "P" or "C" as "Battery". Determine the number of putouts made by each of these three groups in 2016.


```sql
%%sql
SELECT 
    CASE
        WHEN POS = 'OF' THEN 'Outfield'
        WHEN POS IN ('P' , 'C') THEN 'Battery'
        ELSE 'Infield'
    END AS POS_Group,
    SUM(PO) AS 'Putouts'
FROM
    fielding
WHERE
    yearID = 2016
GROUP BY POS_Group;

```

     * mysql+mysqlconnector://root:***@localhost:3306/lahmansbaseballdb
    3 rows affected.





<table>
    <tr>
        <th>POS_Group</th>
        <th>Putouts</th>
    </tr>
    <tr>
        <td>Battery</td>
        <td>41424</td>
    </tr>
    <tr>
        <td>Infield</td>
        <td>58935</td>
    </tr>
    <tr>
        <td>Outfield</td>
        <td>29560</td>
    </tr>
</table>



## Question 5:
#### Find the average number of strikeouts per game by decade since 1920. Round the numbers you report to 2 decimal places. Do the same for home runs per game. Do you see any trends?


```sql
%%sql
SELECT 
    FLOOR(yearID / 10) * 10 AS decade,
    AVG(SO / G) 'K per Game',
    AVG(HR / G) 'HR per Game'
FROM
    teams
WHERE
    yearID >= 1920
GROUP BY decade;
```

     * mysql+mysqlconnector://root:***@localhost:3306/lahmansbaseballdb
    10 rows affected.





<table>
    <tr>
        <th>decade</th>
        <th>K per Game</th>
        <th>HR per Game</th>
    </tr>
    <tr>
        <td>1920</td>
        <td>2.81490875</td>
        <td>0.40173063</td>
    </tr>
    <tr>
        <td>1930</td>
        <td>3.31660500</td>
        <td>0.54575125</td>
    </tr>
    <tr>
        <td>1940</td>
        <td>3.54989625</td>
        <td>0.52308313</td>
    </tr>
    <tr>
        <td>1950</td>
        <td>4.39932063</td>
        <td>0.84299375</td>
    </tr>
    <tr>
        <td>1960</td>
        <td>5.71242879</td>
        <td>0.82020455</td>
    </tr>
    <tr>
        <td>1970</td>
        <td>5.14521423</td>
        <td>0.74564065</td>
    </tr>
    <tr>
        <td>1980</td>
        <td>5.34239077</td>
        <td>0.80403538</td>
    </tr>
    <tr>
        <td>1990</td>
        <td>6.15079065</td>
        <td>0.96031367</td>
    </tr>
    <tr>
        <td>2000</td>
        <td>6.56109333</td>
        <td>1.07335433</td>
    </tr>
    <tr>
        <td>2010</td>
        <td>7.81854300</td>
        <td>1.06862133</td>
    </tr>
</table>



## Question 6:
#### Find the player who had the most success stealing bases in 2016, where success is measured as the percentage of stolen base attempts which are successful. (A stolen base attempt results either in a stolen base or being caught stealing.) Consider only players who attempted at least 20 stolen bases.


```sql
%%sql
SELECT 
    p.nameFirst,
    p.nameLast,
    ROUND(b.SB / (b.SB + b.CS), 2) AS sb_perc
FROM
    batting b
        INNER JOIN
    people p ON p.playerID = b.playerID
WHERE
    b.SB + b.CS >= 20 AND b.yearID = 2016
ORDER BY sb_perc DESC;
```

     * mysql+mysqlconnector://root:***@localhost:3306/lahmansbaseballdb
    47 rows affected.





<table>
    <tr>
        <th>nameFirst</th>
        <th>nameLast</th>
        <th>sb_perc</th>
    </tr>
    <tr>
        <td>Chris</td>
        <td>Owings</td>
        <td>0.91</td>
    </tr>
    <tr>
        <td>Brian</td>
        <td>Dozier</td>
        <td>0.90</td>
    </tr>
    <tr>
        <td>Rajai</td>
        <td>Davis</td>
        <td>0.88</td>
    </tr>
    <tr>
        <td>Billy</td>
        <td>Hamilton</td>
        <td>0.88</td>
    </tr>
    <tr>
        <td>Kevin</td>
        <td>Kiermaier</td>
        <td>0.88</td>
    </tr>
    <tr>
        <td>Mookie</td>
        <td>Betts</td>
        <td>0.87</td>
    </tr>
    <tr>
        <td>Paul</td>
        <td>Goldschmidt</td>
        <td>0.86</td>
    </tr>
    <tr>
        <td>Trea</td>
        <td>Turner</td>
        <td>0.85</td>
    </tr>
    <tr>
        <td>Keon</td>
        <td>Broxton</td>
        <td>0.85</td>
    </tr>
    <tr>
        <td>Hernan</td>
        <td>Perez</td>
        <td>0.83</td>
    </tr>
    <tr>
        <td>Josh</td>
        <td>Harrison</td>
        <td>0.83</td>
    </tr>
    <tr>
        <td>Eduardo</td>
        <td>Nunez</td>
        <td>0.82</td>
    </tr>
    <tr>
        <td>Wil</td>
        <td>Myers</td>
        <td>0.82</td>
    </tr>
    <tr>
        <td>Jarrod</td>
        <td>Dyson</td>
        <td>0.81</td>
    </tr>
    <tr>
        <td>Mike</td>
        <td>Trout</td>
        <td>0.81</td>
    </tr>
    <tr>
        <td>Alcides</td>
        <td>Escobar</td>
        <td>0.81</td>
    </tr>
    <tr>
        <td>Dee</td>
        <td>Gordon</td>
        <td>0.81</td>
    </tr>
    <tr>
        <td>B. J.</td>
        <td>Upton</td>
        <td>0.80</td>
    </tr>
    <tr>
        <td>Brett</td>
        <td>Gardner</td>
        <td>0.80</td>
    </tr>
    <tr>
        <td>Leonys</td>
        <td>Martin</td>
        <td>0.80</td>
    </tr>
    <tr>
        <td>Starling</td>
        <td>Marte</td>
        <td>0.80</td>
    </tr>
    <tr>
        <td>Francisco</td>
        <td>Lindor</td>
        <td>0.79</td>
    </tr>
    <tr>
        <td>Jonathan</td>
        <td>Villar</td>
        <td>0.78</td>
    </tr>
    <tr>
        <td>Ian</td>
        <td>Desmond</td>
        <td>0.78</td>
    </tr>
    <tr>
        <td>Odubel</td>
        <td>Herrera</td>
        <td>0.78</td>
    </tr>
    <tr>
        <td>Jean</td>
        <td>Segura</td>
        <td>0.77</td>
    </tr>
    <tr>
        <td>Ryan</td>
        <td>Braun</td>
        <td>0.76</td>
    </tr>
    <tr>
        <td>Jose</td>
        <td>Ramirez</td>
        <td>0.76</td>
    </tr>
    <tr>
        <td>Jose</td>
        <td>Altuve</td>
        <td>0.75</td>
    </tr>
    <tr>
        <td>Todd</td>
        <td>Frazier</td>
        <td>0.75</td>
    </tr>
    <tr>
        <td>Elvis</td>
        <td>Andrus</td>
        <td>0.75</td>
    </tr>
    <tr>
        <td>Gregory</td>
        <td>Polanco</td>
        <td>0.74</td>
    </tr>
    <tr>
        <td>Freddy</td>
        <td>Galvis</td>
        <td>0.74</td>
    </tr>
    <tr>
        <td>Cameron</td>
        <td>Maybin</td>
        <td>0.71</td>
    </tr>
    <tr>
        <td>Jacoby</td>
        <td>Ellsbury</td>
        <td>0.71</td>
    </tr>
    <tr>
        <td>Travis</td>
        <td>Jankowski</td>
        <td>0.71</td>
    </tr>
    <tr>
        <td>Ian</td>
        <td>Kinsler</td>
        <td>0.70</td>
    </tr>
    <tr>
        <td>Ender</td>
        <td>Inciarte</td>
        <td>0.70</td>
    </tr>
    <tr>
        <td>Kevin</td>
        <td>Pillar</td>
        <td>0.70</td>
    </tr>
    <tr>
        <td>Jose</td>
        <td>Peraza</td>
        <td>0.68</td>
    </tr>
    <tr>
        <td>Bryce</td>
        <td>Harper</td>
        <td>0.68</td>
    </tr>
    <tr>
        <td>Rougned</td>
        <td>Odor</td>
        <td>0.67</td>
    </tr>
    <tr>
        <td>Mallex</td>
        <td>Smith</td>
        <td>0.67</td>
    </tr>
    <tr>
        <td>Charlie</td>
        <td>Blackmon</td>
        <td>0.65</td>
    </tr>
    <tr>
        <td>Brandon</td>
        <td>Phillips</td>
        <td>0.64</td>
    </tr>
    <tr>
        <td>Cesar</td>
        <td>Hernandez</td>
        <td>0.57</td>
    </tr>
    <tr>
        <td>Danny</td>
        <td>Santana</td>
        <td>0.57</td>
    </tr>
</table>



## Question 7:
#### From 1970 – 2019, what is the largest number of wins for a team that did not win the world series? What is the smallest number of wins for a team that did win the world series? Doing this will probably result in an unusually small number of wins for a world series champion – determine why this is the case. Then redo your query, excluding the problem year. How often from 1970 – 2016 was it the case that a team with the most wins also won the world series? What percentage of the time?


```sql
%%sql
SELECT 
    t1.teamID, t1.yearID, t1.W
FROM
    teams t1
        INNER JOIN
    (SELECT 
        t2.W, t2.teamID, t2.yearID
    FROM
        teams t2
    WHERE
        t2.WSWin = 'N' AND t2.yearID >= 1970
    ORDER BY t2.W DESC
    LIMIT 1) Max_w_no_WS ON t1.teamID = Max_w_no_WS.teamID
        AND Max_w_no_WS.yearID = t1.yearID 
UNION ALL SELECT 
    t1.teamID, t1.yearID, t1.W
FROM
    teams t1
        INNER JOIN
    (SELECT 
        t2.W, t2.teamID, t2.yearID
    FROM
        teams t2
    WHERE
        t2.WSWin = 'Y' AND t2.yearID >= 1970
            AND t2.G > 150
    ORDER BY t2.W ASC
    LIMIT 1) Min_w_WS ON t1.teamID = Min_w_WS.teamID
        AND Min_w_WS.yearID = t1.yearID;
```

     * mysql+mysqlconnector://root:***@localhost:3306/lahmansbaseballdb
    2 rows affected.





<table>
    <tr>
        <th>teamID</th>
        <th>yearID</th>
        <th>W</th>
    </tr>
    <tr>
        <td>SEA</td>
        <td>2001</td>
        <td>116</td>
    </tr>
    <tr>
        <td>SLN</td>
        <td>2006</td>
        <td>83</td>
    </tr>
</table>



Part 2: How often from 1970 – 2016 was it the case that a team with the most wins also won the world series? What percentage of the time?


```sql
%%sql
SELECT 
    COUNT(*) / (2019 - 1970) AS perc_most_wins_and_WS_champs
FROM
    teams t1
WHERE
    (t1.yearID , t1.W) IN (SELECT 
            t2.yearID, MAX(t2.W) AS max_wins
        FROM
            teams t2
        GROUP BY t2.yearID)
        AND t1.yearID >= 1970
        AND t1.WSWin = 'Y'
ORDER BY t1.yearID DESC , t1.W DESC;
```

     * mysql+mysqlconnector://root:***@localhost:3306/lahmansbaseballdb
    1 rows affected.





<table>
    <tr>
        <th>perc_most_wins_and_WS_champs</th>
    </tr>
    <tr>
        <td>0.2653</td>
    </tr>
</table>



## Question 8:
#### Using the attendance figures from the homegames table, find the teams and parks which had the top 5 average attendance per game in 2016 (where average attendance is defined as total attendance divided by number of games). Only consider parks where there were at least 10 games played. Report the park name, team name, and average attendance. Repeat for the lowest 5 average attendance.


```sql
%%sql
SELECT 
    parks.parkname,
    teams.name,
    homegames.attendance / homegames.games average_attendance
FROM
    homegames
        INNER JOIN
    parks ON parks.ID = homegames.park_ID
        INNER JOIN
    teams ON teams.ID = homegames.team_ID
WHERE
    games > 10 AND yearkey = 2019
ORDER BY average_attendance DESC;
```

     * mysql+mysqlconnector://root:***@localhost:3306/lahmansbaseballdb
    30 rows affected.





<table>
    <tr>
        <th>parkname</th>
        <th>name</th>
        <th>average_attendance</th>
    </tr>
    <tr>
        <td>Dodger Stadium</td>
        <td>Los Angeles Dodgers</td>
        <td>49065.5432</td>
    </tr>
    <tr>
        <td>Busch Stadium III</td>
        <td>St. Louis Cardinals</td>
        <td>42967.8148</td>
    </tr>
    <tr>
        <td>Yankee Stadium II</td>
        <td>New York Yankees</td>
        <td>40795.1111</td>
    </tr>
    <tr>
        <td>Wrigley Field</td>
        <td>Chicago Cubs</td>
        <td>38208.2099</td>
    </tr>
    <tr>
        <td>Angel Stadium of Anaheim</td>
        <td>Los Angeles Angels of Anaheim</td>
        <td>37812.9241</td>
    </tr>
    <tr>
        <td>Coors Field</td>
        <td>Colorado Rockies</td>
        <td>36953.6296</td>
    </tr>
    <tr>
        <td>Miller Park</td>
        <td>Milwaukee Brewers</td>
        <td>36090.5309</td>
    </tr>
    <tr>
        <td>Fenway Park</td>
        <td>Boston Red Sox</td>
        <td>35402.3291</td>
    </tr>
    <tr>
        <td>Minute Maid Park</td>
        <td>Houston Astros</td>
        <td>35276.1358</td>
    </tr>
    <tr>
        <td>Citizens Bank Park</td>
        <td>Philadelphia Phillies</td>
        <td>33671.8642</td>
    </tr>
    <tr>
        <td>AT&amp;T Park</td>
        <td>San Francisco Giants</td>
        <td>33429.1358</td>
    </tr>
    <tr>
        <td>Suntrust Park</td>
        <td>Atlanta Braves</td>
        <td>32776.7901</td>
    </tr>
    <tr>
        <td>Citi Field</td>
        <td>New York Mets</td>
        <td>30154.7160</td>
    </tr>
    <tr>
        <td>PETCO Park</td>
        <td>San Diego Padres</td>
        <td>29585.1728</td>
    </tr>
    <tr>
        <td>Target Field</td>
        <td>Minnesota Twins</td>
        <td>28435.7901</td>
    </tr>
    <tr>
        <td>Nationals Park</td>
        <td>Washington Nationals</td>
        <td>27898.5309</td>
    </tr>
    <tr>
        <td>Chase Field</td>
        <td>Arizona Diamondbacks</td>
        <td>26364.3210</td>
    </tr>
    <tr>
        <td>Rangers Ballpark in Arlington</td>
        <td>Texas Rangers</td>
        <td>26333.2593</td>
    </tr>
    <tr>
        <td>Great American Ballpark</td>
        <td>Cincinnati Reds</td>
        <td>22473.3671</td>
    </tr>
    <tr>
        <td>Safeco Field</td>
        <td>Seattle Mariners</td>
        <td>22112.4568</td>
    </tr>
    <tr>
        <td>Rogers Centre</td>
        <td>Toronto Blue Jays</td>
        <td>21606.7160</td>
    </tr>
    <tr>
        <td>Progressive Field</td>
        <td>Cleveland Indians</td>
        <td>21464.7160</td>
    </tr>
    <tr>
        <td>Oakland-Alameda County Coliseum</td>
        <td>Oakland Athletics</td>
        <td>20626.3457</td>
    </tr>
    <tr>
        <td>Guaranteed Rate Field</td>
        <td>Chicago White Sox</td>
        <td>20622.1875</td>
    </tr>
    <tr>
        <td>Comerica Park</td>
        <td>Detroit Tigers</td>
        <td>18536.1728</td>
    </tr>
    <tr>
        <td>PNC Park</td>
        <td>Pittsburgh Pirates</td>
        <td>18412.8272</td>
    </tr>
    <tr>
        <td>Kauffman Stadium</td>
        <td>Kansas City Royals</td>
        <td>18177.5625</td>
    </tr>
    <tr>
        <td>Oriole Park at Camden Yards</td>
        <td>Baltimore Orioles</td>
        <td>16145.7654</td>
    </tr>
    <tr>
        <td>Tropicana Field</td>
        <td>Tampa Bay Rays</td>
        <td>14552.2840</td>
    </tr>
    <tr>
        <td>Marlins Park</td>
        <td>Miami Marlins</td>
        <td>10016.0741</td>
    </tr>
</table>



## Question 9:
#### Which managers have won the TSN Manager of the Year award in both the National League (NL) and the American League (AL)? Give their full name and the teams that they were managing when they won the award.


```sql
%%sql
SELECT 
    CONCAT(p.nameFirst, ' ', p.nameLast) Manager,
    am1.awardID award,
    am1.yearID year,
    am1.lgID league,
    m.teamID
FROM
    awardsmanagers am1
        INNER JOIN
    people p ON p.playerID = am1.playerID
        INNER JOIN
    managers m ON m.playerID = am1.playerID
        AND m.yearID = am1.yearID
WHERE
    am1.playerID IN (SELECT 
            am2.playerID
        FROM
            awardsmanagers am2
        WHERE
            am2.awardID = 'TSN Manager of the Year'
                AND am2.lgID = 'NL')
        AND am1.playerID IN (SELECT 
            am2.playerID
        FROM
            awardsmanagers am2
        WHERE
            am2.awardID = 'TSN Manager of the Year'
                AND am2.lgID = 'AL')
        AND am1.awardID = 'TSN Manager of the Year'
```

     * mysql+mysqlconnector://root:***@localhost:3306/lahmansbaseballdb
    6 rows affected.





<table>
    <tr>
        <th>Manager</th>
        <th>award</th>
        <th>year</th>
        <th>league</th>
        <th>teamID</th>
    </tr>
    <tr>
        <td>Jim Leyland</td>
        <td>TSN Manager of the Year</td>
        <td>1988</td>
        <td>NL</td>
        <td>PIT</td>
    </tr>
    <tr>
        <td>Jim Leyland</td>
        <td>TSN Manager of the Year</td>
        <td>1990</td>
        <td>NL</td>
        <td>PIT</td>
    </tr>
    <tr>
        <td>Jim Leyland</td>
        <td>TSN Manager of the Year</td>
        <td>1992</td>
        <td>NL</td>
        <td>PIT</td>
    </tr>
    <tr>
        <td>Jim Leyland</td>
        <td>TSN Manager of the Year</td>
        <td>2006</td>
        <td>AL</td>
        <td>DET</td>
    </tr>
    <tr>
        <td>Davey Johnson</td>
        <td>TSN Manager of the Year</td>
        <td>1997</td>
        <td>AL</td>
        <td>BAL</td>
    </tr>
    <tr>
        <td>Davey Johnson</td>
        <td>TSN Manager of the Year</td>
        <td>2012</td>
        <td>NL</td>
        <td>WAS</td>
    </tr>
</table>


