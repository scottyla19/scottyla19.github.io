---
layout: post
title:  Tourney Coaches
date:   2018-04-09 13:38:01 -0600
categories: NCAA Tournament
---

<script>
  function code_toggle() {
    if (code_shown){
      $('div.input').hide('500');
      $('#toggleButton').val('Show Code')
    } else {
      $('div.input').show('500');
      $('#toggleButton').val('Hide Code')
    }
    code_shown = !code_shown
  }

  $( document ).ready(function(){
    code_shown=false;
    $('div.input').hide()
  });
</script>
<form action="javascript:code_toggle()"><input type="submit" id="toggleButton" value="Show Code"></form>

# Who's the Best NCAA Tournament Coach?

With the NCAA Men's basketball tournament and my bracket officially busted I want to start gathering information for next year. I unfortunately chose Virginia to win it all over Villanova as I thought they were the best teams all year long. I was right about one team but woefully wrong about the other. Congrats to UMBC but I want to win my bracket challenge next year.

One aspect I have thought to include in my choices next year is the coach of the team. It seems to me that the same coaches are in the sweet sixteen and beyond each year (Bill Self, Coach K, Jim Boeheim, etc). Also it seems certain coaches get their teams to overacheive (Beilein, Brad Stevens, etc.). My goal is to find current coaches that win more than they are supposed to in the tournament.

To do this I have taken the data from the [Kaggle NCAA 2018 ML Contest](https://www.kaggle.com/c/mens-machine-learning-competition-2018) and will analyze it to find the average amount of wins for each seed and compare that to how the coaches do.

## Import, Combine, and Clean the Data

To start I import the data for the conferences, teams, and coaches. Then I merge (AKA SQL join) and sort them by team to get data for each team, their coach, and the current conference on a yearly basis.


```python
%matplotlib inline
import matplotlib as plt
import pandas as pd
import numpy as np
import seaborn as sns
import scipy as spy
from sklearn.model_selection import train_test_split

conf = pd.read_csv("TeamConferences.csv")
teams = pd.read_csv("Teams.csv")
coaches = pd.read_csv("TeamCoaches.csv")

allTeams = coaches.merge(teams[["TeamID", "TeamName"]], on="TeamID").merge(conf, on=["Season", "TeamID"])
allTeams.drop(["FirstDayNum","LastDayNum"], axis=1, inplace=True)
allTeams.drop_duplicates(inplace=True)
allTeams.head(25)

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Season</th>
      <th>TeamID</th>
      <th>CoachName</th>
      <th>TeamName</th>
      <th>ConfAbbrev</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1985</td>
      <td>1102</td>
      <td>reggie_minton</td>
      <td>Air Force</td>
      <td>wac</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1986</td>
      <td>1102</td>
      <td>reggie_minton</td>
      <td>Air Force</td>
      <td>wac</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1987</td>
      <td>1102</td>
      <td>reggie_minton</td>
      <td>Air Force</td>
      <td>wac</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1988</td>
      <td>1102</td>
      <td>reggie_minton</td>
      <td>Air Force</td>
      <td>wac</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1989</td>
      <td>1102</td>
      <td>reggie_minton</td>
      <td>Air Force</td>
      <td>wac</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1990</td>
      <td>1102</td>
      <td>reggie_minton</td>
      <td>Air Force</td>
      <td>wac</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1991</td>
      <td>1102</td>
      <td>reggie_minton</td>
      <td>Air Force</td>
      <td>wac</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1992</td>
      <td>1102</td>
      <td>reggie_minton</td>
      <td>Air Force</td>
      <td>wac</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1993</td>
      <td>1102</td>
      <td>reggie_minton</td>
      <td>Air Force</td>
      <td>wac</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1994</td>
      <td>1102</td>
      <td>reggie_minton</td>
      <td>Air Force</td>
      <td>wac</td>
    </tr>
    <tr>
      <th>10</th>
      <td>1995</td>
      <td>1102</td>
      <td>reggie_minton</td>
      <td>Air Force</td>
      <td>wac</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1996</td>
      <td>1102</td>
      <td>reggie_minton</td>
      <td>Air Force</td>
      <td>wac</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1997</td>
      <td>1102</td>
      <td>reggie_minton</td>
      <td>Air Force</td>
      <td>wac</td>
    </tr>
    <tr>
      <th>13</th>
      <td>1998</td>
      <td>1102</td>
      <td>reggie_minton</td>
      <td>Air Force</td>
      <td>wac</td>
    </tr>
    <tr>
      <th>14</th>
      <td>1999</td>
      <td>1102</td>
      <td>reggie_minton</td>
      <td>Air Force</td>
      <td>wac</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2000</td>
      <td>1102</td>
      <td>reggie_minton</td>
      <td>Air Force</td>
      <td>mwc</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2001</td>
      <td>1102</td>
      <td>joe_scott</td>
      <td>Air Force</td>
      <td>mwc</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2002</td>
      <td>1102</td>
      <td>joe_scott</td>
      <td>Air Force</td>
      <td>mwc</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2003</td>
      <td>1102</td>
      <td>joe_scott</td>
      <td>Air Force</td>
      <td>mwc</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2004</td>
      <td>1102</td>
      <td>joe_scott</td>
      <td>Air Force</td>
      <td>mwc</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2005</td>
      <td>1102</td>
      <td>chris_mooney</td>
      <td>Air Force</td>
      <td>mwc</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2006</td>
      <td>1102</td>
      <td>jeff_bzdelik</td>
      <td>Air Force</td>
      <td>mwc</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2007</td>
      <td>1102</td>
      <td>jeff_bzdelik</td>
      <td>Air Force</td>
      <td>mwc</td>
    </tr>
    <tr>
      <th>23</th>
      <td>2008</td>
      <td>1102</td>
      <td>jeff_reynolds</td>
      <td>Air Force</td>
      <td>mwc</td>
    </tr>
    <tr>
      <th>24</th>
      <td>2009</td>
      <td>1102</td>
      <td>jeff_reynolds</td>
      <td>Air Force</td>
      <td>mwc</td>
    </tr>
  </tbody>
</table>
</div>



Next I will import the compact results as well as the tournament seeds files. Then merge the results with the seeds to create a data frame with all of the results as well as the winning seed and losing seed. I also had to clean the seed values as the seeds included a letter denoting which region the seed was in. All I cared about was the number of the seed, for more explanation and description of the function used see the [Basic Starter Kernel - NCAA Men's Dataset](https://www.kaggle.com/juliaelliott/basic-starter-kernel-ncaa-men-s-dataset).


```python
results = pd.read_csv("NCAATourneyCompactResults.csv")
seeds = pd.read_csv("NCAATourneySeeds.csv")

# convert and remove string characters from seed values
# borrowed from the Kaggle NCAA 2018 Men's competition tutorial notebook.
def seed_to_int(seed):
    #Get just the digits from the seeding. Return as int
    s_int = int(seed[1:3])
    return s_int

seeds['seed_int'] = seeds.Seed.apply(seed_to_int)
seeds.drop(labels=['Seed'], inplace=True, axis=1) # This is the string label

seedResults =  results.merge(seeds, right_on=["TeamID","Season"], left_on= ["WTeamID","Season"]).merge(seeds, right_on=["TeamID","Season"], left_on= ["LTeamID","Season"])
seedResults.drop(["TeamID_x", "TeamID_y"],axis = 1, inplace=True)
seedResults.rename(index=str, columns={"seed_int_x":"WSeed", "seed_int_y":"LSeed"}, inplace=True)
seedResults.head(10)

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Season</th>
      <th>DayNum</th>
      <th>WTeamID</th>
      <th>WScore</th>
      <th>LTeamID</th>
      <th>LScore</th>
      <th>WLoc</th>
      <th>NumOT</th>
      <th>WSeed</th>
      <th>LSeed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1985</td>
      <td>136</td>
      <td>1116</td>
      <td>63</td>
      <td>1234</td>
      <td>54</td>
      <td>N</td>
      <td>0</td>
      <td>9</td>
      <td>8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1985</td>
      <td>136</td>
      <td>1120</td>
      <td>59</td>
      <td>1345</td>
      <td>58</td>
      <td>N</td>
      <td>0</td>
      <td>11</td>
      <td>6</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1985</td>
      <td>138</td>
      <td>1120</td>
      <td>66</td>
      <td>1242</td>
      <td>64</td>
      <td>N</td>
      <td>0</td>
      <td>11</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1985</td>
      <td>136</td>
      <td>1207</td>
      <td>68</td>
      <td>1250</td>
      <td>43</td>
      <td>N</td>
      <td>0</td>
      <td>1</td>
      <td>16</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1985</td>
      <td>138</td>
      <td>1207</td>
      <td>63</td>
      <td>1396</td>
      <td>46</td>
      <td>N</td>
      <td>0</td>
      <td>1</td>
      <td>8</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1985</td>
      <td>143</td>
      <td>1207</td>
      <td>65</td>
      <td>1260</td>
      <td>53</td>
      <td>N</td>
      <td>0</td>
      <td>1</td>
      <td>4</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1985</td>
      <td>145</td>
      <td>1207</td>
      <td>60</td>
      <td>1210</td>
      <td>54</td>
      <td>N</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1985</td>
      <td>152</td>
      <td>1207</td>
      <td>77</td>
      <td>1385</td>
      <td>59</td>
      <td>N</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1985</td>
      <td>136</td>
      <td>1229</td>
      <td>58</td>
      <td>1425</td>
      <td>55</td>
      <td>N</td>
      <td>0</td>
      <td>9</td>
      <td>8</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1985</td>
      <td>136</td>
      <td>1242</td>
      <td>49</td>
      <td>1325</td>
      <td>38</td>
      <td>N</td>
      <td>0</td>
      <td>3</td>
      <td>14</td>
    </tr>
  </tbody>
</table>
</div>



Next I needed to merge the seed results with the team information


```python
allResults =  seedResults.merge(allTeams, right_on=["TeamID","Season"], left_on= ["WTeamID","Season"]).merge(allTeams, right_on=["TeamID","Season"], left_on= ["LTeamID","Season"])

def xyToWL(colNames):
    """
    Convert suffix _x and _y to prefix W and L respectively.

    Parameters
    ----------
    colNames : list
        List of column names to convert

    Returns
    -------
    list
        New column names converted to WColumnName from ColumnName_x

    """
    newCols = []
    for col in colNames:
        if col.endswith("_x"):
            newCol = col.replace("_x", "")
            newCol = "W" + newCol
            newCols.append(newCol)
        elif col.endswith("_y"):
            newCol = col.replace("_y", "")
            newCol = "L" + newCol
            newCols.append(newCol)
        else:
            newCols.append(col)
    return newCols

# drop repeated w team id and l team id
allResults.drop(["TeamID_x", "TeamID_y"], axis=1, inplace=True)
allResults.drop_duplicates(inplace=True)

# Convert suffixed columns
allResults.columns = xyToWL(allResults.columns)
allResults = allResults[~((allResults.WSeed ==16) & (allResults.LSeed == 16))]
allResults.sort_values(["Season","DayNum"], inplace=True)
allResults.head(15)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Season</th>
      <th>DayNum</th>
      <th>WTeamID</th>
      <th>WScore</th>
      <th>LTeamID</th>
      <th>LScore</th>
      <th>WLoc</th>
      <th>NumOT</th>
      <th>WSeed</th>
      <th>LSeed</th>
      <th>WCoachName</th>
      <th>WTeamName</th>
      <th>WConfAbbrev</th>
      <th>LCoachName</th>
      <th>LTeamName</th>
      <th>LConfAbbrev</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1985</td>
      <td>136</td>
      <td>1116</td>
      <td>63</td>
      <td>1234</td>
      <td>54</td>
      <td>N</td>
      <td>0</td>
      <td>9</td>
      <td>8</td>
      <td>eddie_sutton</td>
      <td>Arkansas</td>
      <td>swc</td>
      <td>george_raveling</td>
      <td>Iowa</td>
      <td>big_ten</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1985</td>
      <td>136</td>
      <td>1120</td>
      <td>59</td>
      <td>1345</td>
      <td>58</td>
      <td>N</td>
      <td>0</td>
      <td>11</td>
      <td>6</td>
      <td>sonny_smith</td>
      <td>Auburn</td>
      <td>sec</td>
      <td>gene_keady</td>
      <td>Purdue</td>
      <td>big_ten</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1985</td>
      <td>136</td>
      <td>1207</td>
      <td>68</td>
      <td>1250</td>
      <td>43</td>
      <td>N</td>
      <td>0</td>
      <td>1</td>
      <td>16</td>
      <td>john_thompson_jr</td>
      <td>Georgetown</td>
      <td>big_east</td>
      <td>tom_schneider</td>
      <td>Lehigh</td>
      <td>ecc</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1985</td>
      <td>136</td>
      <td>1229</td>
      <td>58</td>
      <td>1425</td>
      <td>55</td>
      <td>N</td>
      <td>0</td>
      <td>9</td>
      <td>8</td>
      <td>bob_donewald</td>
      <td>Illinois St</td>
      <td>mvc</td>
      <td>stan_morrison</td>
      <td>USC</td>
      <td>pac_ten</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1985</td>
      <td>136</td>
      <td>1242</td>
      <td>49</td>
      <td>1325</td>
      <td>38</td>
      <td>N</td>
      <td>0</td>
      <td>3</td>
      <td>14</td>
      <td>larry_brown</td>
      <td>Kansas</td>
      <td>big_eight</td>
      <td>danny_nee</td>
      <td>Ohio</td>
      <td>mac</td>
    </tr>
    <tr>
      <th>10</th>
      <td>1985</td>
      <td>136</td>
      <td>1246</td>
      <td>66</td>
      <td>1449</td>
      <td>58</td>
      <td>N</td>
      <td>0</td>
      <td>12</td>
      <td>5</td>
      <td>joe_b_hall</td>
      <td>Kentucky</td>
      <td>sec</td>
      <td>marv_harshman</td>
      <td>Washington</td>
      <td>pac_ten</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1985</td>
      <td>136</td>
      <td>1256</td>
      <td>78</td>
      <td>1338</td>
      <td>54</td>
      <td>N</td>
      <td>0</td>
      <td>5</td>
      <td>12</td>
      <td>andy_russo</td>
      <td>Louisiana Tech</td>
      <td>southland</td>
      <td>roy_chipman</td>
      <td>Pittsburgh</td>
      <td>big_east</td>
    </tr>
    <tr>
      <th>14</th>
      <td>1985</td>
      <td>136</td>
      <td>1260</td>
      <td>59</td>
      <td>1233</td>
      <td>58</td>
      <td>N</td>
      <td>0</td>
      <td>4</td>
      <td>13</td>
      <td>gene_sullivan</td>
      <td>Loyola-Chicago</td>
      <td>mw_city</td>
      <td>pat_kennedy</td>
      <td>Iona</td>
      <td>maac</td>
    </tr>
    <tr>
      <th>16</th>
      <td>1985</td>
      <td>136</td>
      <td>1314</td>
      <td>76</td>
      <td>1292</td>
      <td>57</td>
      <td>N</td>
      <td>0</td>
      <td>2</td>
      <td>15</td>
      <td>dean_smith</td>
      <td>North Carolina</td>
      <td>acc</td>
      <td>bruce_stewart</td>
      <td>MTSU</td>
      <td>ovc</td>
    </tr>
    <tr>
      <th>19</th>
      <td>1985</td>
      <td>136</td>
      <td>1323</td>
      <td>79</td>
      <td>1333</td>
      <td>70</td>
      <td>N</td>
      <td>0</td>
      <td>7</td>
      <td>10</td>
      <td>digger_phelps</td>
      <td>Notre Dame</td>
      <td>ind</td>
      <td>ralph_miller</td>
      <td>Oregon St</td>
      <td>pac_ten</td>
    </tr>
    <tr>
      <th>20</th>
      <td>1985</td>
      <td>136</td>
      <td>1326</td>
      <td>75</td>
      <td>1235</td>
      <td>64</td>
      <td>N</td>
      <td>0</td>
      <td>4</td>
      <td>13</td>
      <td>eldon_miller</td>
      <td>Ohio St</td>
      <td>big_ten</td>
      <td>johnny_orr</td>
      <td>Iowa St</td>
      <td>big_eight</td>
    </tr>
    <tr>
      <th>21</th>
      <td>1985</td>
      <td>136</td>
      <td>1328</td>
      <td>96</td>
      <td>1299</td>
      <td>83</td>
      <td>N</td>
      <td>0</td>
      <td>1</td>
      <td>16</td>
      <td>billy_tubbs</td>
      <td>Oklahoma</td>
      <td>big_eight</td>
      <td>don_corbett</td>
      <td>NC A&amp;T</td>
      <td>meac</td>
    </tr>
    <tr>
      <th>24</th>
      <td>1985</td>
      <td>136</td>
      <td>1374</td>
      <td>85</td>
      <td>1330</td>
      <td>68</td>
      <td>N</td>
      <td>0</td>
      <td>5</td>
      <td>12</td>
      <td>dave_bliss</td>
      <td>SMU</td>
      <td>swc</td>
      <td>paul_webb</td>
      <td>Old Dominion</td>
      <td>sun_belt</td>
    </tr>
    <tr>
      <th>25</th>
      <td>1985</td>
      <td>136</td>
      <td>1385</td>
      <td>83</td>
      <td>1380</td>
      <td>59</td>
      <td>N</td>
      <td>0</td>
      <td>1</td>
      <td>16</td>
      <td>lou_carnesecca</td>
      <td>St John's</td>
      <td>big_east</td>
      <td>robert_hopkins</td>
      <td>Southern Univ</td>
      <td>swac</td>
    </tr>
    <tr>
      <th>29</th>
      <td>1985</td>
      <td>136</td>
      <td>1396</td>
      <td>60</td>
      <td>1439</td>
      <td>57</td>
      <td>N</td>
      <td>0</td>
      <td>8</td>
      <td>9</td>
      <td>john_chaney</td>
      <td>Temple</td>
      <td>a_ten</td>
      <td>charles_moir</td>
      <td>Virginia Tech</td>
      <td>metro</td>
    </tr>
  </tbody>
</table>
</div>



Next I needed to calculate the expected wins (EWins) for each seed. To do this I grouped and merged the `allResults` dataframe on the seeds to give the `totalWins`, `totalLosses`, and `WPct` for each seed. I then calculated the `EWins` using the following formula:

$$EWins = \frac{totalWins}{4 \cdot numberOfSeasons}$$

*Note: The 4 in the denominator is due to the fact that each seed has 4 teams per year.*


```python
bySeed = mergeWL(allResults, "Seed")
# EWins is yearly expected wins per team by their seed. --> totalWins / 4 * number of seasons   (4 due to 4 different seeds every year)
bySeed["EWins"] = bySeed.totalWins/(4*seedResults.Season.nunique())
bySeed = bySeed.round(decimals=3)
bySeed.head(16)

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Seed</th>
      <th>totalWins</th>
      <th>totalLosses</th>
      <th>WPct</th>
      <th>EWins</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>446.0</td>
      <td>115</td>
      <td>0.795</td>
      <td>3.379</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>320.0</td>
      <td>130</td>
      <td>0.711</td>
      <td>2.424</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>245.0</td>
      <td>131</td>
      <td>0.652</td>
      <td>1.856</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>207.0</td>
      <td>131</td>
      <td>0.612</td>
      <td>1.568</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>145.0</td>
      <td>133</td>
      <td>0.522</td>
      <td>1.098</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>149.0</td>
      <td>133</td>
      <td>0.528</td>
      <td>1.129</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>123.0</td>
      <td>132</td>
      <td>0.482</td>
      <td>0.932</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>97.0</td>
      <td>132</td>
      <td>0.424</td>
      <td>0.735</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>74.0</td>
      <td>132</td>
      <td>0.359</td>
      <td>0.561</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>83.0</td>
      <td>134</td>
      <td>0.382</td>
      <td>0.629</td>
    </tr>
    <tr>
      <th>10</th>
      <td>11</td>
      <td>89.0</td>
      <td>144</td>
      <td>0.382</td>
      <td>0.674</td>
    </tr>
    <tr>
      <th>11</th>
      <td>12</td>
      <td>71.0</td>
      <td>135</td>
      <td>0.345</td>
      <td>0.538</td>
    </tr>
    <tr>
      <th>12</th>
      <td>13</td>
      <td>33.0</td>
      <td>133</td>
      <td>0.199</td>
      <td>0.250</td>
    </tr>
    <tr>
      <th>13</th>
      <td>14</td>
      <td>24.0</td>
      <td>134</td>
      <td>0.152</td>
      <td>0.182</td>
    </tr>
    <tr>
      <th>14</th>
      <td>15</td>
      <td>9.0</td>
      <td>133</td>
      <td>0.063</td>
      <td>0.068</td>
    </tr>
    <tr>
      <th>15</th>
      <td>16</td>
      <td>0.0</td>
      <td>133</td>
      <td>0.000</td>
      <td>0.000</td>
    </tr>
  </tbody>
</table>
</div>




```python
# helper function mergeWL
def mergeWL(resDF, onCol):
    '''
    Group resDF by onCol and count totalWins, total Losses and WPct

    Parameters
    ----------
    resDF : pandas.dataframe
        Dataframe of results to filter and group wins and losses

    onCol: str
        String of column to group by

    Returns
    -------
    pandas.dataframe
        New dataframe with wins, losses, and WPct calculated.

    '''
    wins = resDF.groupby("W"+onCol, as_index=False)["WTeamID"].agg(['count']).rename(columns={'count': 'totalWins'})
    loss = resDF.groupby("L"+onCol, as_index=False)["LTeamID"].agg(['count']).rename(columns={'count': 'totalLosses'})
    comb = wins.merge(loss, how="right", left_index = True, right_index = True)
    comb.reset_index(inplace=True)
    comb.rename(columns={comb.columns[0]:onCol}, inplace=True)
    comb["WPct"] = comb.totalWins/(comb.totalWins + comb.totalLosses)
    comb.fillna(0, inplace=True)
    return comb
```

Finally on to calculating coach wins, losses, and the average of their wins added based on their seed (`AvgWinsAdded`). To start I grouped and merged `allResults` by `CoachName`.


```python
byCoach = mergeWL(allResults, "CoachName")
byCoach.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CoachName</th>
      <th>totalWins</th>
      <th>totalLosses</th>
      <th>WPct</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>al_brown</td>
      <td>0.0</td>
      <td>1</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>al_skinner</td>
      <td>8.0</td>
      <td>9</td>
      <td>0.470588</td>
    </tr>
    <tr>
      <th>2</th>
      <td>alan_leforce</td>
      <td>1.0</td>
      <td>2</td>
      <td>0.333333</td>
    </tr>
    <tr>
      <th>3</th>
      <td>andrew_toole</td>
      <td>0.0</td>
      <td>1</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>andy_enfield</td>
      <td>4.0</td>
      <td>3</td>
      <td>0.571429</td>
    </tr>
    <tr>
      <th>5</th>
      <td>andy_kennedy</td>
      <td>2.0</td>
      <td>2</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>andy_russo</td>
      <td>2.0</td>
      <td>2</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>andy_stoglin</td>
      <td>0.0</td>
      <td>2</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>8</th>
      <td>anthony_evans</td>
      <td>1.0</td>
      <td>1</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <th>9</th>
      <td>anthony_grant</td>
      <td>1.0</td>
      <td>3</td>
      <td>0.250000</td>
    </tr>
  </tbody>
</table>
</div>



Next I calculated the average seed (`avgSeed`) and number of appearances (`Appearances`) for each coach.


```python
avgSeed = byCoachSeed.groupby("CoachName", as_index=False).agg({'Seed': "mean", 'Season': 'count'})
avgSeed.rename(columns={"Seed":"AvgSeed", "Season":"Appearances"}, inplace=True)
byCoach = byCoach.merge(avgSeed,on="CoachName").round(3)
byCoach["totalWins"] = byCoach["totalWins"].astype(int)
byCoach.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CoachName</th>
      <th>totalWins</th>
      <th>totalLosses</th>
      <th>WPct</th>
      <th>AvgSeed</th>
      <th>Appearances</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>al_brown</td>
      <td>0</td>
      <td>1</td>
      <td>0.000</td>
      <td>14.000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>al_skinner</td>
      <td>8</td>
      <td>9</td>
      <td>0.471</td>
      <td>6.556</td>
      <td>9</td>
    </tr>
    <tr>
      <th>2</th>
      <td>alan_leforce</td>
      <td>1</td>
      <td>2</td>
      <td>0.333</td>
      <td>12.000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>andrew_toole</td>
      <td>0</td>
      <td>1</td>
      <td>0.000</td>
      <td>16.000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>andy_enfield</td>
      <td>4</td>
      <td>3</td>
      <td>0.571</td>
      <td>11.333</td>
      <td>3</td>
    </tr>
    <tr>
      <th>5</th>
      <td>andy_kennedy</td>
      <td>2</td>
      <td>2</td>
      <td>0.500</td>
      <td>11.500</td>
      <td>2</td>
    </tr>
    <tr>
      <th>6</th>
      <td>andy_russo</td>
      <td>2</td>
      <td>2</td>
      <td>0.500</td>
      <td>8.500</td>
      <td>2</td>
    </tr>
    <tr>
      <th>7</th>
      <td>andy_stoglin</td>
      <td>0</td>
      <td>2</td>
      <td>0.000</td>
      <td>16.000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>8</th>
      <td>anthony_evans</td>
      <td>1</td>
      <td>1</td>
      <td>0.500</td>
      <td>15.000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>anthony_grant</td>
      <td>1</td>
      <td>3</td>
      <td>0.250</td>
      <td>10.333</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>



Finally I needed to calculate the number of wins added per year by each coach. To do this I created another dataframe called `byCoachSeed` to hold all of the wins and losses grouped by coach and season. Using this dataframe I was able to count the number of wins each year by each team (coach) to get their wins in the tournament for that year. Also I calculated the difference between average wins by seed and the actual wins that year for that coach(`Diff`).


```python
winCoachSeed = allResults.groupby(["WCoachName", "Season"], as_index=False)["WSeed"].mean()
lossCoachSeed = allResults.groupby(["LCoachName", "Season"], as_index=False)["LSeed"].mean()
winCoachSeed.rename(columns={"WCoachName":"CoachName","WSeed":"Seed"}, inplace=True)
lossCoachSeed.rename(columns={"LCoachName":"CoachName","LSeed":"Seed"}, inplace=True)
byCoachSeed = winCoachSeed.append(lossCoachSeed,ignore_index=True).drop_duplicates().sort_values(["CoachName", "Season"])
byCoachSeed.reset_index(drop=True, inplace=True)

coachWins = allResults.groupby(["WCoachName", "Season"], as_index=False)["WSeed"].count().rename(columns={"WSeed":"TourneyWins","WCoachName":"CoachName"})
byCoachSeed = byCoachSeed.merge(coachWins, how="left",on=["CoachName", "Season"]).fillna(0)
byCoachSeed["TourneyWins"] = byCoachSeed["TourneyWins"].astype(int)

byCoachSeed = byCoachSeed.merge(bySeed[["Seed", "EWins"]], how="left", on="Seed").fillna(0)
byCoachSeed["Diff"] = byCoachSeed.TourneyWins - byCoachSeed.EWins
byCoachSeed.head(15)


```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CoachName</th>
      <th>Season</th>
      <th>Seed</th>
      <th>TourneyWins</th>
      <th>EWins</th>
      <th>Diff</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>al_brown</td>
      <td>1986</td>
      <td>14</td>
      <td>0</td>
      <td>0.182</td>
      <td>-0.182</td>
    </tr>
    <tr>
      <th>1</th>
      <td>al_skinner</td>
      <td>1993</td>
      <td>8</td>
      <td>1</td>
      <td>0.735</td>
      <td>0.265</td>
    </tr>
    <tr>
      <th>2</th>
      <td>al_skinner</td>
      <td>1997</td>
      <td>9</td>
      <td>0</td>
      <td>0.561</td>
      <td>-0.561</td>
    </tr>
    <tr>
      <th>3</th>
      <td>al_skinner</td>
      <td>2001</td>
      <td>3</td>
      <td>1</td>
      <td>1.856</td>
      <td>-0.856</td>
    </tr>
    <tr>
      <th>4</th>
      <td>al_skinner</td>
      <td>2002</td>
      <td>11</td>
      <td>0</td>
      <td>0.674</td>
      <td>-0.674</td>
    </tr>
    <tr>
      <th>5</th>
      <td>al_skinner</td>
      <td>2004</td>
      <td>6</td>
      <td>2</td>
      <td>1.129</td>
      <td>0.871</td>
    </tr>
    <tr>
      <th>6</th>
      <td>al_skinner</td>
      <td>2005</td>
      <td>4</td>
      <td>1</td>
      <td>1.568</td>
      <td>-0.568</td>
    </tr>
    <tr>
      <th>7</th>
      <td>al_skinner</td>
      <td>2006</td>
      <td>4</td>
      <td>2</td>
      <td>1.568</td>
      <td>0.432</td>
    </tr>
    <tr>
      <th>8</th>
      <td>al_skinner</td>
      <td>2007</td>
      <td>7</td>
      <td>1</td>
      <td>0.932</td>
      <td>0.068</td>
    </tr>
    <tr>
      <th>9</th>
      <td>al_skinner</td>
      <td>2009</td>
      <td>7</td>
      <td>0</td>
      <td>0.932</td>
      <td>-0.932</td>
    </tr>
    <tr>
      <th>10</th>
      <td>alan_leforce</td>
      <td>1991</td>
      <td>10</td>
      <td>0</td>
      <td>0.629</td>
      <td>-0.629</td>
    </tr>
    <tr>
      <th>11</th>
      <td>alan_leforce</td>
      <td>1992</td>
      <td>14</td>
      <td>1</td>
      <td>0.182</td>
      <td>0.818</td>
    </tr>
    <tr>
      <th>12</th>
      <td>andrew_toole</td>
      <td>2015</td>
      <td>16</td>
      <td>0</td>
      <td>0.000</td>
      <td>0.000</td>
    </tr>
    <tr>
      <th>13</th>
      <td>andy_enfield</td>
      <td>2013</td>
      <td>15</td>
      <td>2</td>
      <td>0.068</td>
      <td>1.932</td>
    </tr>
    <tr>
      <th>14</th>
      <td>andy_enfield</td>
      <td>2016</td>
      <td>8</td>
      <td>0</td>
      <td>0.735</td>
      <td>-0.735</td>
    </tr>
  </tbody>
</table>
</div>



To get the `AvgWinsAdded` for each coach I was able to group `byCoachSeed` from above on the coaches


```python
coachTotalEWins = byCoachSeed.groupby("CoachName", as_index=False)["Diff"].mean()
byCoach = byCoach.merge(coachTotalEWins, how="left", on="CoachName").rename(columns={"Diff":"AvgWinsAdded"})
byCoach.sort_values("AvgWinsAdded",ascending=False, inplace=True)
byCoach.reset_index(drop=True, inplace=True)
byCoach.head(15)

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CoachName</th>
      <th>totalWins</th>
      <th>totalLosses</th>
      <th>WPct</th>
      <th>AvgSeed</th>
      <th>Appearances</th>
      <th>AvgWinsAdded</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>kevin_ollie</td>
      <td>7</td>
      <td>1</td>
      <td>0.875</td>
      <td>8.0</td>
      <td>2</td>
      <td>2.7535</td>
    </tr>
    <tr>
      <th>1</th>
      <td>john_giannini</td>
      <td>3</td>
      <td>1</td>
      <td>0.750</td>
      <td>13.0</td>
      <td>1</td>
      <td>2.7500</td>
    </tr>
    <tr>
      <th>2</th>
      <td>jim_rosborough</td>
      <td>5</td>
      <td>1</td>
      <td>0.833</td>
      <td>2.0</td>
      <td>1</td>
      <td>2.5760</td>
    </tr>
    <tr>
      <th>3</th>
      <td>kevin_mackey</td>
      <td>2</td>
      <td>1</td>
      <td>0.667</td>
      <td>14.0</td>
      <td>1</td>
      <td>1.8180</td>
    </tr>
    <tr>
      <th>4</th>
      <td>brad_stevens</td>
      <td>12</td>
      <td>5</td>
      <td>0.706</td>
      <td>7.0</td>
      <td>5</td>
      <td>1.5090</td>
    </tr>
    <tr>
      <th>5</th>
      <td>rollie_massimino</td>
      <td>11</td>
      <td>4</td>
      <td>0.733</td>
      <td>9.0</td>
      <td>5</td>
      <td>1.4816</td>
    </tr>
    <tr>
      <th>6</th>
      <td>russ_pennell</td>
      <td>2</td>
      <td>1</td>
      <td>0.667</td>
      <td>12.0</td>
      <td>1</td>
      <td>1.4620</td>
    </tr>
    <tr>
      <th>7</th>
      <td>darrin_horn</td>
      <td>2</td>
      <td>1</td>
      <td>0.667</td>
      <td>12.0</td>
      <td>1</td>
      <td>1.4620</td>
    </tr>
    <tr>
      <th>8</th>
      <td>jim_brandenburg</td>
      <td>2</td>
      <td>1</td>
      <td>0.667</td>
      <td>12.0</td>
      <td>1</td>
      <td>1.4620</td>
    </tr>
    <tr>
      <th>9</th>
      <td>joe_b_hall</td>
      <td>2</td>
      <td>1</td>
      <td>0.667</td>
      <td>12.0</td>
      <td>1</td>
      <td>1.4620</td>
    </tr>
    <tr>
      <th>10</th>
      <td>michael_white</td>
      <td>3</td>
      <td>1</td>
      <td>0.750</td>
      <td>4.0</td>
      <td>1</td>
      <td>1.4320</td>
    </tr>
    <tr>
      <th>11</th>
      <td>craig_esherick</td>
      <td>2</td>
      <td>1</td>
      <td>0.667</td>
      <td>10.0</td>
      <td>1</td>
      <td>1.3710</td>
    </tr>
    <tr>
      <th>12</th>
      <td>johnny_dawkins</td>
      <td>2</td>
      <td>1</td>
      <td>0.667</td>
      <td>10.0</td>
      <td>1</td>
      <td>1.3710</td>
    </tr>
    <tr>
      <th>13</th>
      <td>todd_lickliter</td>
      <td>4</td>
      <td>2</td>
      <td>0.667</td>
      <td>8.5</td>
      <td>2</td>
      <td>1.1820</td>
    </tr>
    <tr>
      <th>14</th>
      <td>greg_gard</td>
      <td>4</td>
      <td>2</td>
      <td>0.667</td>
      <td>7.5</td>
      <td>2</td>
      <td>1.1665</td>
    </tr>
  </tbody>
</table>
</div>



Kevin Ollie leads the pack! He took a 7 seed to the Final Four and won it the next year. However I am more focused on coaches that have more lasting impact. Let's limit it to 3 tournament appearances.


### Top 50 Coaches by Average Wins Above  Seed


```python
byCoach3App = byCoach[byCoach["Appearances"] >= 3].reset_index(drop=True)
byCoach3App.head(50)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CoachName</th>
      <th>totalWins</th>
      <th>totalLosses</th>
      <th>WPct</th>
      <th>AvgSeed</th>
      <th>Appearances</th>
      <th>AvgWinsAdded</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>brad_stevens</td>
      <td>12</td>
      <td>5</td>
      <td>0.706</td>
      <td>7.000</td>
      <td>5</td>
      <td>1.509000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>rollie_massimino</td>
      <td>11</td>
      <td>4</td>
      <td>0.733</td>
      <td>9.000</td>
      <td>5</td>
      <td>1.481600</td>
    </tr>
    <tr>
      <th>2</th>
      <td>richard_williams</td>
      <td>6</td>
      <td>3</td>
      <td>0.667</td>
      <td>5.000</td>
      <td>3</td>
      <td>0.902000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>larry_brown</td>
      <td>13</td>
      <td>4</td>
      <td>0.765</td>
      <td>4.200</td>
      <td>5</td>
      <td>0.881800</td>
    </tr>
    <tr>
      <th>4</th>
      <td>john_groce</td>
      <td>4</td>
      <td>3</td>
      <td>0.571</td>
      <td>11.333</td>
      <td>3</td>
      <td>0.878667</td>
    </tr>
    <tr>
      <th>5</th>
      <td>andy_enfield</td>
      <td>4</td>
      <td>3</td>
      <td>0.571</td>
      <td>11.333</td>
      <td>3</td>
      <td>0.841000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>frank_martin</td>
      <td>10</td>
      <td>5</td>
      <td>0.667</td>
      <td>6.600</td>
      <td>5</td>
      <td>0.827400</td>
    </tr>
    <tr>
      <th>7</th>
      <td>billy_donovan</td>
      <td>35</td>
      <td>12</td>
      <td>0.745</td>
      <td>4.071</td>
      <td>14</td>
      <td>0.733857</td>
    </tr>
    <tr>
      <th>8</th>
      <td>john_beilein</td>
      <td>19</td>
      <td>11</td>
      <td>0.633</td>
      <td>7.818</td>
      <td>11</td>
      <td>0.725182</td>
    </tr>
    <tr>
      <th>9</th>
      <td>paul_westhead</td>
      <td>4</td>
      <td>3</td>
      <td>0.571</td>
      <td>11.000</td>
      <td>3</td>
      <td>0.719667</td>
    </tr>
    <tr>
      <th>10</th>
      <td>sonny_smith</td>
      <td>7</td>
      <td>5</td>
      <td>0.583</td>
      <td>9.400</td>
      <td>5</td>
      <td>0.716600</td>
    </tr>
    <tr>
      <th>11</th>
      <td>tom_izzo</td>
      <td>47</td>
      <td>20</td>
      <td>0.701</td>
      <td>4.950</td>
      <td>20</td>
      <td>0.702250</td>
    </tr>
    <tr>
      <th>12</th>
      <td>bill_guthridge</td>
      <td>8</td>
      <td>3</td>
      <td>0.727</td>
      <td>4.000</td>
      <td>3</td>
      <td>0.676667</td>
    </tr>
    <tr>
      <th>13</th>
      <td>stan_heath</td>
      <td>5</td>
      <td>4</td>
      <td>0.556</td>
      <td>10.500</td>
      <td>4</td>
      <td>0.640000</td>
    </tr>
    <tr>
      <th>14</th>
      <td>rick_pitino</td>
      <td>54</td>
      <td>18</td>
      <td>0.750</td>
      <td>3.550</td>
      <td>20</td>
      <td>0.615500</td>
    </tr>
    <tr>
      <th>15</th>
      <td>john_calipari</td>
      <td>52</td>
      <td>17</td>
      <td>0.754</td>
      <td>2.889</td>
      <td>18</td>
      <td>0.564389</td>
    </tr>
    <tr>
      <th>16</th>
      <td>quin_snyder</td>
      <td>5</td>
      <td>4</td>
      <td>0.556</td>
      <td>9.000</td>
      <td>4</td>
      <td>0.552750</td>
    </tr>
    <tr>
      <th>17</th>
      <td>jerry_tarkanian</td>
      <td>22</td>
      <td>8</td>
      <td>0.733</td>
      <td>4.111</td>
      <td>9</td>
      <td>0.496556</td>
    </tr>
    <tr>
      <th>18</th>
      <td>p_j_carlesimo</td>
      <td>12</td>
      <td>7</td>
      <td>0.632</td>
      <td>5.000</td>
      <td>6</td>
      <td>0.488667</td>
    </tr>
    <tr>
      <th>19</th>
      <td>lefty_driesell</td>
      <td>5</td>
      <td>4</td>
      <td>0.556</td>
      <td>8.750</td>
      <td>4</td>
      <td>0.487000</td>
    </tr>
    <tr>
      <th>20</th>
      <td>dick_tarrant</td>
      <td>3</td>
      <td>4</td>
      <td>0.429</td>
      <td>13.250</td>
      <td>4</td>
      <td>0.456500</td>
    </tr>
    <tr>
      <th>21</th>
      <td>denny_crum</td>
      <td>21</td>
      <td>11</td>
      <td>0.656</td>
      <td>5.583</td>
      <td>12</td>
      <td>0.448917</td>
    </tr>
    <tr>
      <th>22</th>
      <td>archie_miller</td>
      <td>5</td>
      <td>4</td>
      <td>0.556</td>
      <td>9.000</td>
      <td>4</td>
      <td>0.447000</td>
    </tr>
    <tr>
      <th>23</th>
      <td>jim_calhoun</td>
      <td>48</td>
      <td>17</td>
      <td>0.738</td>
      <td>4.500</td>
      <td>20</td>
      <td>0.435650</td>
    </tr>
    <tr>
      <th>24</th>
      <td>roy_williams</td>
      <td>77</td>
      <td>24</td>
      <td>0.762</td>
      <td>2.741</td>
      <td>27</td>
      <td>0.409593</td>
    </tr>
    <tr>
      <th>25</th>
      <td>steve_fisher</td>
      <td>26</td>
      <td>14</td>
      <td>0.650</td>
      <td>6.267</td>
      <td>15</td>
      <td>0.403000</td>
    </tr>
    <tr>
      <th>26</th>
      <td>dean_smith</td>
      <td>37</td>
      <td>13</td>
      <td>0.740</td>
      <td>2.615</td>
      <td>13</td>
      <td>0.393923</td>
    </tr>
    <tr>
      <th>27</th>
      <td>thomas_penders</td>
      <td>12</td>
      <td>11</td>
      <td>0.522</td>
      <td>9.636</td>
      <td>11</td>
      <td>0.382273</td>
    </tr>
    <tr>
      <th>28</th>
      <td>tommy_amaker</td>
      <td>4</td>
      <td>5</td>
      <td>0.444</td>
      <td>12.200</td>
      <td>5</td>
      <td>0.372600</td>
    </tr>
    <tr>
      <th>29</th>
      <td>jim_j_o'brien</td>
      <td>11</td>
      <td>7</td>
      <td>0.611</td>
      <td>5.857</td>
      <td>7</td>
      <td>0.368143</td>
    </tr>
    <tr>
      <th>30</th>
      <td>dan_monson</td>
      <td>3</td>
      <td>3</td>
      <td>0.500</td>
      <td>10.000</td>
      <td>3</td>
      <td>0.366000</td>
    </tr>
    <tr>
      <th>31</th>
      <td>steve_donahue</td>
      <td>2</td>
      <td>3</td>
      <td>0.400</td>
      <td>13.333</td>
      <td>3</td>
      <td>0.366000</td>
    </tr>
    <tr>
      <th>32</th>
      <td>chris_mack</td>
      <td>10</td>
      <td>8</td>
      <td>0.556</td>
      <td>7.571</td>
      <td>7</td>
      <td>0.335429</td>
    </tr>
    <tr>
      <th>33</th>
      <td>bo_ryan</td>
      <td>27</td>
      <td>15</td>
      <td>0.643</td>
      <td>5.200</td>
      <td>15</td>
      <td>0.332867</td>
    </tr>
    <tr>
      <th>34</th>
      <td>clem_haskins</td>
      <td>11</td>
      <td>7</td>
      <td>0.611</td>
      <td>6.714</td>
      <td>7</td>
      <td>0.326714</td>
    </tr>
    <tr>
      <th>35</th>
      <td>sean_miller</td>
      <td>19</td>
      <td>10</td>
      <td>0.655</td>
      <td>5.200</td>
      <td>10</td>
      <td>0.325000</td>
    </tr>
    <tr>
      <th>36</th>
      <td>mike_krzyzewski</td>
      <td>92</td>
      <td>28</td>
      <td>0.767</td>
      <td>2.219</td>
      <td>32</td>
      <td>0.313219</td>
    </tr>
    <tr>
      <th>37</th>
      <td>pete_gillen</td>
      <td>8</td>
      <td>10</td>
      <td>0.444</td>
      <td>10.444</td>
      <td>9</td>
      <td>0.306333</td>
    </tr>
    <tr>
      <th>38</th>
      <td>paul_hewitt</td>
      <td>7</td>
      <td>6</td>
      <td>0.538</td>
      <td>8.167</td>
      <td>6</td>
      <td>0.300500</td>
    </tr>
    <tr>
      <th>39</th>
      <td>bill_frieder</td>
      <td>14</td>
      <td>6</td>
      <td>0.700</td>
      <td>4.429</td>
      <td>7</td>
      <td>0.298714</td>
    </tr>
    <tr>
      <th>40</th>
      <td>gary_williams</td>
      <td>28</td>
      <td>15</td>
      <td>0.651</td>
      <td>5.375</td>
      <td>16</td>
      <td>0.294063</td>
    </tr>
    <tr>
      <th>41</th>
      <td>nolan_richardson_jr</td>
      <td>26</td>
      <td>13</td>
      <td>0.667</td>
      <td>5.500</td>
      <td>14</td>
      <td>0.291714</td>
    </tr>
    <tr>
      <th>42</th>
      <td>ben_jacobson</td>
      <td>4</td>
      <td>4</td>
      <td>0.500</td>
      <td>9.250</td>
      <td>4</td>
      <td>0.282250</td>
    </tr>
    <tr>
      <th>43</th>
      <td>jim_boeheim</td>
      <td>52</td>
      <td>25</td>
      <td>0.675</td>
      <td>4.038</td>
      <td>26</td>
      <td>0.278962</td>
    </tr>
    <tr>
      <th>44</th>
      <td>jim_valvano</td>
      <td>8</td>
      <td>5</td>
      <td>0.615</td>
      <td>5.600</td>
      <td>5</td>
      <td>0.277400</td>
    </tr>
    <tr>
      <th>45</th>
      <td>mike_anderson</td>
      <td>9</td>
      <td>8</td>
      <td>0.529</td>
      <td>8.250</td>
      <td>8</td>
      <td>0.276500</td>
    </tr>
    <tr>
      <th>46</th>
      <td>gary_waters</td>
      <td>2</td>
      <td>3</td>
      <td>0.400</td>
      <td>12.333</td>
      <td>3</td>
      <td>0.275333</td>
    </tr>
    <tr>
      <th>47</th>
      <td>billy_kennedy</td>
      <td>3</td>
      <td>3</td>
      <td>0.500</td>
      <td>10.333</td>
      <td>3</td>
      <td>0.275333</td>
    </tr>
    <tr>
      <th>48</th>
      <td>fang_mitchell</td>
      <td>1</td>
      <td>3</td>
      <td>0.250</td>
      <td>15.000</td>
      <td>3</td>
      <td>0.265333</td>
    </tr>
    <tr>
      <th>49</th>
      <td>wimp_sanderson</td>
      <td>11</td>
      <td>7</td>
      <td>0.611</td>
      <td>5.143</td>
      <td>7</td>
      <td>0.259857</td>
    </tr>
  </tbody>
</table>
</div>



### Bottom 50 Coaches


```python
byCoach3App.tail(50)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>CoachName</th>
      <th>totalWins</th>
      <th>totalLosses</th>
      <th>WPct</th>
      <th>AvgSeed</th>
      <th>Appearances</th>
      <th>AvgWinsAdded</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>198</th>
      <td>scott_nagy</td>
      <td>0</td>
      <td>3</td>
      <td>0.000</td>
      <td>13.000</td>
      <td>3</td>
      <td>-0.323333</td>
    </tr>
    <tr>
      <th>199</th>
      <td>bryce_drew</td>
      <td>0</td>
      <td>3</td>
      <td>0.000</td>
      <td>12.000</td>
      <td>3</td>
      <td>-0.331000</td>
    </tr>
    <tr>
      <th>200</th>
      <td>seth_greenberg</td>
      <td>1</td>
      <td>3</td>
      <td>0.250</td>
      <td>9.667</td>
      <td>3</td>
      <td>-0.340667</td>
    </tr>
    <tr>
      <th>201</th>
      <td>dave_odom</td>
      <td>10</td>
      <td>9</td>
      <td>0.526</td>
      <td>5.222</td>
      <td>9</td>
      <td>-0.341667</td>
    </tr>
    <tr>
      <th>202</th>
      <td>bobby_lutz</td>
      <td>2</td>
      <td>5</td>
      <td>0.286</td>
      <td>7.800</td>
      <td>5</td>
      <td>-0.342600</td>
    </tr>
    <tr>
      <th>203</th>
      <td>mark_fox</td>
      <td>2</td>
      <td>5</td>
      <td>0.286</td>
      <td>8.200</td>
      <td>5</td>
      <td>-0.369800</td>
    </tr>
    <tr>
      <th>204</th>
      <td>mike_montgomery</td>
      <td>18</td>
      <td>16</td>
      <td>0.529</td>
      <td>6.188</td>
      <td>16</td>
      <td>-0.386000</td>
    </tr>
    <tr>
      <th>205</th>
      <td>kevin_stallings</td>
      <td>6</td>
      <td>9</td>
      <td>0.400</td>
      <td>6.778</td>
      <td>9</td>
      <td>-0.388778</td>
    </tr>
    <tr>
      <th>206</th>
      <td>leonard_hamilton</td>
      <td>7</td>
      <td>8</td>
      <td>0.467</td>
      <td>6.125</td>
      <td>8</td>
      <td>-0.403375</td>
    </tr>
    <tr>
      <th>207</th>
      <td>wayne_tinkle</td>
      <td>0</td>
      <td>4</td>
      <td>0.000</td>
      <td>11.750</td>
      <td>4</td>
      <td>-0.403500</td>
    </tr>
    <tr>
      <th>208</th>
      <td>fran_dunphy</td>
      <td>3</td>
      <td>16</td>
      <td>0.158</td>
      <td>10.625</td>
      <td>16</td>
      <td>-0.407125</td>
    </tr>
    <tr>
      <th>209</th>
      <td>tom_asbury</td>
      <td>0</td>
      <td>4</td>
      <td>0.000</td>
      <td>12.250</td>
      <td>4</td>
      <td>-0.416750</td>
    </tr>
    <tr>
      <th>210</th>
      <td>ray_mccallum</td>
      <td>0</td>
      <td>3</td>
      <td>0.000</td>
      <td>12.667</td>
      <td>3</td>
      <td>-0.426667</td>
    </tr>
    <tr>
      <th>211</th>
      <td>jud_heathcote</td>
      <td>7</td>
      <td>7</td>
      <td>0.500</td>
      <td>5.143</td>
      <td>7</td>
      <td>-0.441429</td>
    </tr>
    <tr>
      <th>212</th>
      <td>tad_boyle</td>
      <td>1</td>
      <td>4</td>
      <td>0.200</td>
      <td>9.250</td>
      <td>4</td>
      <td>-0.443250</td>
    </tr>
    <tr>
      <th>213</th>
      <td>greg_mcdermott</td>
      <td>3</td>
      <td>7</td>
      <td>0.300</td>
      <td>8.429</td>
      <td>7</td>
      <td>-0.448143</td>
    </tr>
    <tr>
      <th>214</th>
      <td>speedy_morris</td>
      <td>1</td>
      <td>4</td>
      <td>0.200</td>
      <td>9.500</td>
      <td>4</td>
      <td>-0.450750</td>
    </tr>
    <tr>
      <th>215</th>
      <td>kelvin_sampson</td>
      <td>12</td>
      <td>14</td>
      <td>0.462</td>
      <td>6.429</td>
      <td>14</td>
      <td>-0.454571</td>
    </tr>
    <tr>
      <th>216</th>
      <td>gene_bartow</td>
      <td>2</td>
      <td>5</td>
      <td>0.286</td>
      <td>8.200</td>
      <td>5</td>
      <td>-0.459200</td>
    </tr>
    <tr>
      <th>217</th>
      <td>riley_wallace</td>
      <td>0</td>
      <td>3</td>
      <td>0.000</td>
      <td>11.667</td>
      <td>3</td>
      <td>-0.472333</td>
    </tr>
    <tr>
      <th>218</th>
      <td>larry_eustachy</td>
      <td>4</td>
      <td>5</td>
      <td>0.444</td>
      <td>6.800</td>
      <td>5</td>
      <td>-0.478800</td>
    </tr>
    <tr>
      <th>219</th>
      <td>rick_stansbury</td>
      <td>4</td>
      <td>6</td>
      <td>0.400</td>
      <td>6.667</td>
      <td>6</td>
      <td>-0.487333</td>
    </tr>
    <tr>
      <th>220</th>
      <td>travis_ford</td>
      <td>1</td>
      <td>6</td>
      <td>0.143</td>
      <td>8.833</td>
      <td>6</td>
      <td>-0.492500</td>
    </tr>
    <tr>
      <th>221</th>
      <td>rich_herrin</td>
      <td>0</td>
      <td>3</td>
      <td>0.000</td>
      <td>11.667</td>
      <td>3</td>
      <td>-0.495000</td>
    </tr>
    <tr>
      <th>222</th>
      <td>randy_ayers</td>
      <td>6</td>
      <td>3</td>
      <td>0.667</td>
      <td>3.333</td>
      <td>3</td>
      <td>-0.497667</td>
    </tr>
    <tr>
      <th>223</th>
      <td>jamie_dixon</td>
      <td>12</td>
      <td>12</td>
      <td>0.500</td>
      <td>5.091</td>
      <td>11</td>
      <td>-0.498000</td>
    </tr>
    <tr>
      <th>224</th>
      <td>gene_keady</td>
      <td>18</td>
      <td>15</td>
      <td>0.545</td>
      <td>4.933</td>
      <td>15</td>
      <td>-0.500133</td>
    </tr>
    <tr>
      <th>225</th>
      <td>ed_cooley</td>
      <td>1</td>
      <td>4</td>
      <td>0.200</td>
      <td>9.250</td>
      <td>4</td>
      <td>-0.509500</td>
    </tr>
    <tr>
      <th>226</th>
      <td>bob_wenzel</td>
      <td>0</td>
      <td>3</td>
      <td>0.000</td>
      <td>10.000</td>
      <td>3</td>
      <td>-0.515333</td>
    </tr>
    <tr>
      <th>227</th>
      <td>lou_henson</td>
      <td>9</td>
      <td>11</td>
      <td>0.450</td>
      <td>5.800</td>
      <td>10</td>
      <td>-0.533300</td>
    </tr>
    <tr>
      <th>228</th>
      <td>skip_prosser</td>
      <td>6</td>
      <td>9</td>
      <td>0.400</td>
      <td>7.222</td>
      <td>9</td>
      <td>-0.536111</td>
    </tr>
    <tr>
      <th>229</th>
      <td>steve_cleveland</td>
      <td>0</td>
      <td>3</td>
      <td>0.000</td>
      <td>12.000</td>
      <td>3</td>
      <td>-0.538000</td>
    </tr>
    <tr>
      <th>230</th>
      <td>jerry_green</td>
      <td>3</td>
      <td>5</td>
      <td>0.375</td>
      <td>6.000</td>
      <td>5</td>
      <td>-0.547000</td>
    </tr>
    <tr>
      <th>231</th>
      <td>john_thompson_iii</td>
      <td>9</td>
      <td>10</td>
      <td>0.474</td>
      <td>5.800</td>
      <td>10</td>
      <td>-0.586300</td>
    </tr>
    <tr>
      <th>232</th>
      <td>billy_tubbs</td>
      <td>14</td>
      <td>8</td>
      <td>0.636</td>
      <td>2.875</td>
      <td>8</td>
      <td>-0.609875</td>
    </tr>
    <tr>
      <th>233</th>
      <td>hugh_durham</td>
      <td>1</td>
      <td>4</td>
      <td>0.200</td>
      <td>8.000</td>
      <td>4</td>
      <td>-0.617500</td>
    </tr>
    <tr>
      <th>234</th>
      <td>tony_bennett</td>
      <td>10</td>
      <td>7</td>
      <td>0.588</td>
      <td>3.714</td>
      <td>7</td>
      <td>-0.619000</td>
    </tr>
    <tr>
      <th>235</th>
      <td>pat_foster</td>
      <td>0</td>
      <td>3</td>
      <td>0.000</td>
      <td>10.000</td>
      <td>3</td>
      <td>-0.634000</td>
    </tr>
    <tr>
      <th>236</th>
      <td>rob_evans</td>
      <td>1</td>
      <td>3</td>
      <td>0.250</td>
      <td>7.333</td>
      <td>3</td>
      <td>-0.644000</td>
    </tr>
    <tr>
      <th>237</th>
      <td>norm_stewart</td>
      <td>7</td>
      <td>10</td>
      <td>0.412</td>
      <td>5.900</td>
      <td>10</td>
      <td>-0.665900</td>
    </tr>
    <tr>
      <th>238</th>
      <td>jeff_mullins</td>
      <td>0</td>
      <td>3</td>
      <td>0.000</td>
      <td>9.000</td>
      <td>3</td>
      <td>-0.704667</td>
    </tr>
    <tr>
      <th>239</th>
      <td>tim_welsh</td>
      <td>0</td>
      <td>3</td>
      <td>0.000</td>
      <td>9.000</td>
      <td>3</td>
      <td>-0.755000</td>
    </tr>
    <tr>
      <th>240</th>
      <td>ralph_miller</td>
      <td>0</td>
      <td>3</td>
      <td>0.000</td>
      <td>9.333</td>
      <td>3</td>
      <td>-0.765333</td>
    </tr>
    <tr>
      <th>241</th>
      <td>george_raveling</td>
      <td>1</td>
      <td>4</td>
      <td>0.200</td>
      <td>7.750</td>
      <td>4</td>
      <td>-0.865500</td>
    </tr>
    <tr>
      <th>242</th>
      <td>danny_nee</td>
      <td>0</td>
      <td>6</td>
      <td>0.000</td>
      <td>8.667</td>
      <td>6</td>
      <td>-0.867500</td>
    </tr>
    <tr>
      <th>243</th>
      <td>oliver_purnell</td>
      <td>0</td>
      <td>6</td>
      <td>0.000</td>
      <td>8.167</td>
      <td>6</td>
      <td>-0.878667</td>
    </tr>
    <tr>
      <th>244</th>
      <td>frank_haith</td>
      <td>1</td>
      <td>4</td>
      <td>0.200</td>
      <td>7.250</td>
      <td>4</td>
      <td>-0.897750</td>
    </tr>
    <tr>
      <th>245</th>
      <td>j_d_barnett</td>
      <td>1</td>
      <td>3</td>
      <td>0.250</td>
      <td>7.667</td>
      <td>3</td>
      <td>-0.909000</td>
    </tr>
    <tr>
      <th>246</th>
      <td>eddie_fogler</td>
      <td>2</td>
      <td>6</td>
      <td>0.250</td>
      <td>6.667</td>
      <td>6</td>
      <td>-0.984833</td>
    </tr>
    <tr>
      <th>247</th>
      <td>steve_lappas</td>
      <td>2</td>
      <td>4</td>
      <td>0.333</td>
      <td>4.500</td>
      <td>4</td>
      <td>-1.003750</td>
    </tr>
  </tbody>
</table>
</div>



## Conclusion

Looking at the Top 50 and bottom 50 provide some insights. Tony Bennett is in the bottom 10% (234 out of 247). Maybe defense doesn't win championships? Also if Pitt wasn't happy with Jamie Dixon and his teams' tournament performance, Kevin Stallings might not have been much of an improvement. On the positive side I can see why Brad Stevens has had success in both NCAA and NBA as he leads the group of coaches with 3 or more appearances. Also a lot of the legends (Izzo, Pitino, Calipari) are up towards the top as well.

Some teams to watch next year if they make the tournament could be Indiana (Archie Miller), South Carolina (Frank Martin), and Michigan (John Beilein). It would be nice to add the results from this year's tournament as some of these values might change. Beilein would improve as he took #3 Michigan to the Final and Tony Bennett will fall even further.
