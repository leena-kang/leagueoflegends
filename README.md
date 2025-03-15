# League of Legends Mid Lane Performance Analysis

Author: Leena Kang 

# Introduction
League of Legends (LoL) is one of the most popular team-based multiplayer online battle arena (MOBA) game, with millions of players and a thriving professional scene. As a 5v5 strategy game, each player's role and performance contribute to the team's overall success. Specifically, the mid-laner, being primarily involved in the middle lane, can greatly influence team coordination, and thus can play a central role in shaping the flow of the game. We will investigate further what drive's a team's success in a League of Legends match, with a primary focus on mid-laners. 

Understanding the key drivers of victory in LoL is significant not only for competitive players and analysts but also for amateur players and esports fans. Identifying and quantifying these winning conditions can offer players at all skill levels the insights needed to improve their own gameplay and strategy. Additionally, being able to predict the likelihood of a team’s victory (_especially based on early-game performance_) provides viewers with a deeper understanding of match dynamics, enhancing their experience and enjoyment of the game. 

The primary goal of this project is to answer the following central question: **Amongst mid-laners, what are the most significant factors that contribute to a team's victory in a League of Legends match?** We will leverage in-game metrics to perform data and statistical analyses, and design, train, and evaluate predictive models to investigate if these measures hold predictive power (but more on that later! 😊)

## The Dataset 
The dataset was adopted from [Oracle's Elixir](https://oracleselixir.com/tools/downloads), which provides various game statistics of individual players, as well as a summary of each team for each game. The files are accessed through a google drive, and for this project, we used the matches from 2022. This dataset contains _148992_ rows and _161_ columns, but we only saved the columns that could be relevant for this project. Below are brief descriptions about each column:

### Unfamiliar with LoL? Here are some basic terminology to better understand the data!
> **Creep Score (CS)**: The number of enemy minions (small AI-controlled units) a player has defeated. More CS means more resources for the player.

> **Vision Score**: A measure of how well a player provides vision for their team by placing wards (which reveal areas of the map) and removing enemy wards.

> **Gold**: The in-game currency earned by defeating enemies, minions, and objectives. It’s used to buy powerful items that improve a player’s strength.

> **XP (Experience Points)**: Gained by being near defeated enemies, allowing a player to level up and unlock stronger abilities.

> **Towers**: Defensive structures that attack enemies and protect key areas of the map. Destroying them helps a team advance toward victory.

> **Heralds (Rift Herald)**: A powerful neutral monster that, when defeated, helps a team push down towers by charging into them.

> **First Blood**: The first kill of the game, which grants bonus gold to the player who secures it, giving an early advantage.

### Column Descriptions
- `gameid`: Unique ID for a specific game.

  >  This **not** unique across all rows, but instead there 12 rows per `gameid`: 10 rows are for the 10 total players in a match, and last 2 contain summary data for the blue and red team. 

- `participantid`: The player number in a game. 
  > Note this is **not** a unique key for each player. Possible values are 1-10, 100, or 200. Note that 100 and 200 represents the whole team, and 1-10 represents each player. 

- `result`: Outcome of a match. 1 indicates that the team/player won this match, and 0 if they lost.

- `side`: Indicates whether they a player/team is associated with the `blue` team or `red` team. 

- `position`: The position/role each of player. If the value is "team", the row corresponds to the team summary data. 

- `gamelength`: The total duration of the game, in seconds. 

- `kills`: Number of enemy champions killed during the game. 

- `deaths`: Number of times the player has died during the game.

- `assists`: The number of assists, that is, the number of kills that the player contributed to, during the game. 

- `firstblood`: Indicates if the player/team got first blood. 1 indicates they got the first blood, 0 otherwise. 

- `firstherald`: Indicates if the player/team got first the first Rift Herald. 1 indicates they got the first blood, 0 otherwise. 

- `team kpm`: The team's kills per minute. 

- `ckpm`: The averaged combined kills per minute (team kills + opponents). 

- `vspm`: The vision score per minute for each player/team.

- `visionscore`: The total vision score for each player/team by the end of the match. 

- `cspm`: The creep score per minute for each player/team. 

- `total cs`: The total creep score for each player/team by the end of the match. 

- `earned gpm`: Gold per minue earned for each player/team. 

- `dpm`: Damage dealt to enemy champions for each player/team. 

- `damagetochampions`: The total damage dealt to enemy champions for each player/team. 

- `totalgold`: The total amount of gold for each player/team by the end of the match. 

- `split`: The segment of a competitive season. 

- `firstmidtower`: 1 if the team destroyed the first middle tower, 0 if they did not. 

- `goldat15`: The amount of gold accumulated after 15 minutes into the match. 

- `xpat15`: The amount of XP each player has 15 minutes into the match. 

- `csat15`: The creep score of each player 15 minutes into the match. 

- `opp_goldat15`: The player's opponent (_opponent corresponds to the player with the same position on the opposing side_) accumulated gold 15 minutes into the match.

- `opp_xpat15`: The player's opponent accumulated XP score 15 minutes into the match.

- `opp_csat15`: The player's opponent accumulated creep score 15 minutes into the match.

- `golddiffat15`: The difference between a player and their opponents gold at 15 minutes ('goldat15' - 'opp_goldat15'). 

- `xpdiffat15`: The difference between a player and their opponents XP score at 15 minutes ('xpat15' - 'opp_xpat15'). 

- `csdiffat15`: The difference between a player and their opponents creep score at 15 minutes ('csat15' - 'opp_csat15'). 

- `killsat15`: The number of kills the player has 15 minutes into the match. 

- `assistsat15`: The number of assists the player has 15 minutes into the match. 

- `deathsat15`: The number of deaths the player has 15 minutes into the match. 

- `opp_killsat15`: The number of kills the player's opponent has 15 minutes into the match. 

- `opp_assistsat15`: The number of assists player's opponent has 15 minutes into the match. 

- `opp_deathsat15`: The number of deaths player's opponent has 15 minutes into the match. 


# Data Cleaning and Exploratory Data Analysis

## Data Cleaning 
Below are the main steps taken in the data cleaning process:  

### Naming conventions
There are some inconsistencies with the column names-- some has a space in between words, while some are separated by an underscore ('\_'). Here we changed all column names such that **any space is replaced with '_'**. 

There are also inconsistencies with the capitalization of String values on certain columns. For all String values, we ensured all letters are **lowercased**. 

Ensuring uniformity across all column names and values will allow for a more convenient analysis to be performed movign forward, especially with datasets with an extensive amount of columns!

### Adding/Scaling Columns 
`gamelength` originally came as seconds, but for readability, we converted the columns to minutes. 

We added a new column, `kppm`, which represents the **kill participation per minute**. Often times a player's contribution/performance is evaluated based on both assists and kills, so `kppm` combines the two 
- In short, `kppm` = ((kills + assists) / gamelength)

### Handling Null Values 
Some columns, in the way they are constructed, are null for rows that represent the entire team and vice versa. However, some are null when they can intuitively computed. 

For example, `total_cs` is null for all team-based rows (_i.e., `position == 'team'`_), however this _could_ be imputed as the sum of all creep scores of each player in the team. So, we **imputed** these null values with the **accumlated creep score** within each corresponding team. 

All columns related to data **15 minutes** into the game contains several null values. However, since we have an adequate amount of rows in our dataset, we decided to drop  the games where they contain null values in any of the 15 minute related columns. 

---

After running this procedure, we get a dataframe with *127872* rows and *39* columns. 
##### Here first few rows and columns of our cleaned, Dataframe!


| gameid                |   result | side   | position   |   gamelength |   team_kpm |   ckpm |
|:----------------------|---------:|:-------|:-----------|-------------:|-----------:|-------:|
| ESPORTSTMNT01_2690210 |        0 | blue   | top        |        28.55 |     0.3152 | 0.9807 |
| ESPORTSTMNT01_2690210 |        0 | blue   | jng        |        28.55 |     0.3152 | 0.9807 |
| ESPORTSTMNT01_2690210 |        0 | blue   | mid        |        28.55 |     0.3152 | 0.9807 |
| ESPORTSTMNT01_2690210 |        0 | blue   | bot        |        28.55 |     0.3152 | 0.9807 |
| ESPORTSTMNT01_2690210 |        0 | blue   | sup        |        28.55 |     0.3152 | 0.9807 |

## Univariate Analysis

### Team-wide Distributions 
Here, we will visualize the distribution of certain game-metrics to get a better sense of how the data is spread (team-wide). 

<iframe
  src="assets/univariate_team.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

It appears that Game Length, Total Gold, Vision Score, Total Creep Score, Damage to Champions, and Kill Participation all resemble a bell-curve, where some have a slight skewness to the right. 
> Note that the data here are only correspond to the rows that are team summaries (_i.e., `position == 'team'`). 

### Mid lane Distributions 
Below are histograms of the same metrics from before, except for **mid lane players only***. 

<iframe
  src="assets/univariate_mid.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

We see that the distributions here are very similar to the entire team-wide distributions!


## Bivariate Analysis 

### Comparing Winning and Losing Mid Lane players 
Here we will continue to look investgate further in mid laners only. Below are boxplots of several game metrics of mid lane players, split between players that won and lost their respective match. 

<iframe
  src="assets/bivariate_boxplot.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

Amongst mid lane players, it appears that on average, mid laners from a winning team have higher metrics compared to its losing counterpart on **all** features, the greatest deviation being Earned Gold/min (`earned_gpm`) and Kill Participation/min (`kppm`)

### Identifying Correlated Columns 
Many columns are intuitively dependent on each other, but here will identify which columns are more correlated to each than others by visualizing their correlations through a **heatmap**.  

<iframe
  src="assets/bivariate_heatmap.html"
  width="800"
  height="800"
  frameborder="0"
></iframe>

Notice that `kppm` + `team_kppm` and `goldat15` + `earned_gpm` essentially collect the same information, which is why it have a higher correlation. We'll plot the most highly correlated and meaningful (both negatively and positively) below: 
1. `earned_gpm` and `team_kpm`
2. `dpm` and `team_kpm`
3. `dpm` and `kppm`
4. `ckpm` and `csat15`

### Graphing Noticeable Correlated Features

<iframe
  src="assets/bivariate_scatter.html"
  width="900"
  height="700"
  frameborder="0"
></iframe>

What is worth noticing here is that for the **positively correlated** scatter plots, majority of **teams that lost are on the bottom of the graph**, and though that is intuitive, it reaffirms that these features can hold valuable relationship that can help distinguish winning/losing teams (_this will be helpful when we eventually build our model_). On the contrary, winning and losing teams appear to be a more blended together on the `ckpm` vs `csat15` scatterplot.

## Interesting Aggregates 

Below is a Dataframe that is grouped by positions and result (whether that player won or lost the match), aggregated by their **means**. Here we only used the quantitative columns that are scaled by per minute, so that we can make comparable observations (_if not, game length could heavily influence these metrics, for as the game goes longer, these metrics are more likely to increase._)


|             |     kppm |     vspm |     cspm |      dpm |   earned_gpm |
|:------------|---------:|---------:|---------:|---------:|-------------:|
| ('bot', 0)  | 0.18464  | 1.04975  |  8.49251 |  488.064 |     255.182  |
| ('bot', 1)  | 0.431592 | 1.2628   |  8.9602  |  628.831 |     339.583  |
| ('jng', 0)  | 0.200993 | 1.25639  |  5.42796 |  287.381 |     179.756  |
| ('jng', 1)  | 0.438285 | 1.41496  |  5.92972 |  371.505 |     246.701  |
| ('mid', 0)  | 0.18201  | 0.965557 |  8.04911 |  488.917 |     232.016  |
| ('mid', 1)  | 0.416533 | 1.12314  |  8.48864 |  608.303 |     303.021  |
| ('sup', 0)  | 0.196244 | 2.34955  |  1.11018 |  160.518 |      87.4243 |
| ('sup', 1)  | 0.452321 | 2.57972  |  1.14713 |  187.935 |     133.692  |
| ('team', 0) | 0.913517 | 6.52634  | 30.6425  | 1862.21  |     976.804  |
| ('team', 1) | 2.08624  | 7.40393  | 32.586   | 2334.62  |    1313.68   |
| ('top', 0)  | 0.149631 | 0.905099 |  7.5627  |  437.332 |     222.426  |
| ('top', 1)  | 0.347508 | 1.02331  |  8.06037 |  538.045 |     290.688  |


Its worth noting here that for each position, all of the **winning teams** tend to have a **higher average** in **all columns**. 
> We will be investigating later if these differences are statistically significant, so that we can make stronger conclusions about the differences in performance between winning and losing players!

# Assessment of Missingness 

## NMAR Analysis 
`firstdragon`, `firstherald`, `firstbaron`
- These values are binary, so if the value is `NaN`, it might be because these objectives were not taken at all during the game. Though this is unlikely, as dragons, heralds, and barons are highly beneficial to win, which is especially true during late game (i.e., after the 25 minute mark)

`firsttower`, `firstmidtower`, `firsttothreetowers`
- The same logic applies here, however, these are more likely to have ever occured in the game. `NaN` values on these columns could indicate that the losing team (`result` == 0) **surrendered** before any towers (mid tower, or first to three towers) were ever destroyed on either side. Note that is it not possible for a full (non-surrendered) game to be played AND for `firsttower` and `firsttothreetowers`to not have occured, as you must defeat at least 3 towers in order to reach the nexus.
- We will investigate  further whether the missingness of `firstmidtower` could be dependent on the other columns. 

## Missingness Dependency 

### Setup and Motivation: `firstmidtower` and `split`

Observe the Dataframe below, which displays the proportion of `firstmidtower` missing and not missing for each category in `split`: 

| split   |   fmt_missing = false |   fmt_missing = true |
|:--------|----------------------:|---------------------:|
| 2022    |            0.00187705 |           0.00187684 |
| Regular |            0.0035664  |           0.003566   |
| Pro-Am  |            0.00675739 |           0.00675663 |
| Closing |            0.00844674 |           0.00844579 |
| Opening |            0.00844674 |           0.00844579 |
| Champ 1 |            0.00938527 |           0.00938421 |
| Fall    |            0.0107931  |           0.0109044  |
| Champ 2 |            0.0119193  |           0.0119179  |
| Winter  |            0.0191459  |           0.0191438  |
| Split 2 |            0.0396058  |           0.0396014  |
| Split 1 |            0.0565931  |           0.0565868  |
| Spring  |            0.245894   |           0.245866   |
| Summer  |            0.2626     |           0.26257    |
| missing |            0.314969   |           0.314934   |

#### Lets visualize this! 

<iframe
  src="assets/missingness_split_dist.html"
  width="700"
  height="500"
  frameborder="0"
></iframe>

> ** Are these observed differences between `fmt_missing = false` and `fmt_missing = true` statistically significant?**

### Permutation Test: `firstmidtower` and `split` 

**Null Hypothesis**: The distribution of `split` for when `firstmidtower` is missing (`NaN`) is the same as the distribution of `split` for when `firstmidtower` is _not_ missing. Any variation between the two is due to random chance. 

**Alternative Hypothesis**: The distribution of `split` for when `firstmidtower` is missing (`NaN`) is the **NOT** same as the distribution of `split` for when `firstmidtower` is _not_ missing.

- **Test Statistic**: Total Variation Distance (TVD) 
- **Significance Level**: 5%

### Results and Conclusion
After simulating TVD's under the null hypothesis, we have found that the P-value is 1.0. Below is the empirical distribution of our simulated TVD's:  

<iframe
  src="assets/missingness_emp_dist.html"
  width="900"
  height="700"
  frameborder="0"
></iframe>

Since our **P-value > 0.05**, we **fail to reject** the null hypothesis. We have sufficient evidence to conclude that the distributions of `split` between missingness and non-missingness of `firstmidtower` came from the same distribution, i.e., any variation is most likey due to chance. Thus, this suggests that the **missingness of `firstmidtower` does not depend on `split`.** 

### Setup and Motivation: `damagetochampions` and `firstmidtower` 

Here are 2 histograms of `damagetochampions`-- one is the distribution of `damagetochampions` with their `firstmidtower` missing, and the latter not missing: 

<iframe
  src="assets/missingness_dmg_dist.html"
  width="900"
  height="700"
  frameborder="0"
></iframe>


Simular to the previous permutation test, we want to investigate if the missingness of `fisrtmidtower` depends on `damagetochampions`. 

### Permutation Test: `damagetochampion` and `firstmidtower` 

**Null Hypothesis**: The distribution of `damagetochampions` for when `firstmidtower` is missing (`NaN`) is the same as the distribution of `damagetochampions` for when `firstmidtower` is _not_ missing. Any variation between the two is due to random chance. 

**Alternative Hypothesis**: The distribution of `damagetochamptions` for when `firstmidtower` is missing (`NaN`) is the **NOT** same as the distribution of `split` for when `firstmidtower` is _not_ missing.

- **Test Statistic**: Kolmogorov-Smirnov (KS) statistic
  > Note that since the distributions of `damagetochampions` for null and non-null values of `firstmidtower` have different shapes (`fmt_missing == True` is more right skewed, whereas `fmt_missing == False` resembles more of a bell shape). Thus, we will use the KS Statistic as opposed to difference in group means for this test.

- **Significance Level**: 5%

### Results and Conclusion 
After simulating TVD's under the null hypothesis, we have found that the P-value is 0.0. Below is the empirical distribution of our simulated KS statistics: 

<iframe
  src="assets/missingness_ks_dist.html"
  width="900"
  height="700"
  frameborder="0"
></iframe>

> Note that the observed ks-statistic (0.94) **far to the right** of this distribution-- feel free to use the pan feature on this graph and find it! 

Since the p-value < 0.05, we **reject the null hypothesis**. We have sufficient evidence to conclude that the missingness of `firstmidtower` depends on the `damagetochampions` column. This suggests that the **missingness of `firstmidtower` does depend on `damagetochamptions`.

# Hypothesis Testing 
In this hypothesis test, we want to see if the game metric differences between mid laners from winning and losing teams (_explored in our bivariate analysis!_) are significant. This will allow us to further investigate the significant differences between mid laners that win or lose, and thus have a better understanding on the main drivers for winning teams! 

First, let's take a deeper dive in a mid laner's **kill participation per minute** (KPPM)

<iframe
  src="assets/hyp_kppm_dist.html"
  width="900"
  height="700"
  frameborder="0"
></iframe>

### Evaluating KPPM means amongst Mid-laners from Winning/Losing Teams
We will conduct the following permutation test: 

**Null Hypothesis**: Kill Participation per minute (KPPM) of mid-laners from winning and losing teams have the same distribution, thus any observed difference from our sample is due to random chance. p
**Alternative Hypothesis**: Winning mid-laners have a higher KPPM on average compared to mid-laners from a losing team. The observed difference in our sample cannot be explained by random choice alone. 

- **Test Statistic**: Since both distributions resemble a similar bell shape, we will use *__difference in group means__* (Winning - losing)
- **Significance Level**: 5%


### Results and Conclusion




# Framing a Prediction Problem 
- Can a mid-laner's performance within the first 15 miniutes reliably predict game outcomes?

## Problem Identification 




# Baseline Model 




# Final Model 
- **CONFUSION MATRIX** 





# Fairness Analysis
- **EMP DIST**




