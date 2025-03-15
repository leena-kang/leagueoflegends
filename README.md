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
- There are some inconsistencies with the column names-- some has a space in between words, while some are separated by an underscore ('\_'). Here we changed all column names such that **any space is replaced with '_'**. 

- There are also inconsistencies with the capitalization of String values on certain columns. For all String values, we ensured all letters are **lowercased**. 

Ensuring uniformity across all column names and values will allow for a more convenient analysis to be performed movign forward, especially with datasets with an extensive amount of columns!

### Adding/Scaling Columns 
- `gamelength` originally came as seconds, but for readability, we converted the columns to minutes. 

- We added a new column, `kppm`, which represents the **kill participation per minute**. Often times a player's contribution/performance is evaluated based on both assists and kills, so `kppm` combines the two 
  - In short, `kppm` = ((kills + assists) / gamelength)

### Handling Null Values 
- Some columns, in the way they are constructed, are null for rows that represent the entire team and vice versa. However, some are null when they can intuitively computed. 

  - For example, `total_cs` is null for all team-based rows (_i.e., `position == 'team'`_), however this _could_ be imputed as the sum of all creep scores of each player in the team. So, we **imputed** these null values with the **accumlated creep score** within each corresponding team. 

- All columns related to data **15 minutes** into the game contains several null values. However, since we have an adequate amount of rows in our dataset, we decided to drop the games where they contain null values in any of the 15 minute related columns. 

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

---

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
  height="450"
  frameborder="0"
></iframe>

**Are these observed differences between `fmt_missing = false` and `fmt_missing = true` statistically significant?**

### Permutation Test: `firstmidtower` and `split` 

**Null Hypothesis**: The distribution of `split` for when `firstmidtower` is missing (`NaN`) is the same as the distribution of `split` for when `firstmidtower` is _not_ missing. Any variation between the two is due to random chance. 

**Alternative Hypothesis**: The distribution of `split` for when `firstmidtower` is missing (`NaN`) is the **NOT** same as the distribution of `split` for when `firstmidtower` is _not_ missing.

- **Test Statistic**: Total Variation Distance (TVD) 
- **Significance Level**: 5%

### Results and Conclusion
After simulating TVD's under the null hypothesis, we have found that the p-value is 1.0. Below is the empirical distribution of our simulated TVD's:  

<iframe
  src="assets/missingness_emp_dist.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>

Since our **p-value > 0.05**, we **fail to reject** the null hypothesis. We have sufficient evidence to conclude that the distributions of `split` between missingness and non-missingness of `firstmidtower` came from the same distribution, i.e., any variation is most likey due to chance. Thus, this suggests that the **missingness of `firstmidtower` does not depend on `split`.** 

### Setup and Motivation: `damagetochampions` and `firstmidtower` 

Here are 2 histograms of `damagetochampions`-- one is the distribution of `damagetochampions` with their `firstmidtower` missing, and the latter not missing: 

<iframe
  src="assets/missingness_dmg_dist.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>


Similar to the previous permutation test, we want to investigate if the missingness of `fisrtmidtower` depends on `damagetochampions`. 

### Permutation Test: `damagetochampion` and `firstmidtower` 

**Null Hypothesis**: The distribution of `damagetochampions` for when `firstmidtower` is missing (`NaN`) is the same as the distribution of `damagetochampions` for when `firstmidtower` is _not_ missing. Any variation between the two is due to random chance. 

**Alternative Hypothesis**: The distribution of `damagetochamptions` for when `firstmidtower` is missing (`NaN`) is the **NOT** same as the distribution of `split` for when `firstmidtower` is _not_ missing.

- **Test Statistic**: Kolmogorov-Smirnov (KS) statistic
  > Note that since the distributions of `damagetochampions` for null and non-null values of `firstmidtower` have different shapes (`fmt_missing == True` is more right skewed, whereas `fmt_missing == False` resembles more of a bell shape). Thus, we will use the KS Statistic as opposed to difference in group means for this test.

- **Significance Level**: 5%

### Results and Conclusion 
After simulating TVD's under the null hypothesis, we have found that the p-value is 0.0. Below is the empirical distribution of our simulated KS statistics: 

<iframe
  src="assets/missingness_ks_dist.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>

> Note that the observed ks-statistic (0.94) **far to the right** of this distribution-- feel free to use the pan feature on this graph and find it! 

Since the p-value < 0.05, we **reject the null hypothesis**. We have sufficient evidence to conclude that the missingness of `firstmidtower` depends on the `damagetochampions` column. This suggests that the **missingness of `firstmidtower` does depend on `damagetochamptions`.**

---

# Hypothesis Testing 
In this hypothesis test, we want to see if the game metric differences between mid laners from winning and losing teams (_explored in our bivariate analysis!_) are significant. This will allow us to further investigate the significant differences between mid laners that win or lose, and thus have a better understanding on the main drivers for winning teams! 

First, let's take a deeper dive in a mid laner's **kill participation per minute** (KPPM)

<iframe
  src="assets/hyp_kppm_dist.html"
  width="850"
  height="450"
  frameborder="0"
></iframe>

### Evaluating KPPM means amongst Mid-laners from Winning/Losing Teams
We will conduct the following permutation test: 

**Null Hypothesis**: Kill Participation per minute (KPPM) of mid-laners from winning and losing teams have the same distribution, thus any observed difference from our sample is due to random chance. 

**Alternative Hypothesis**: Winning mid-laners have a **higher KPPM** on average compared to mid-laners from a losing team. The observed difference in our sample cannot be explained by random choice alone. 

- **Test Statistic**: Difference in group means (Winning - Losing)
> Since both distributions resemble a similar bell shape, we will use difference in group means

- **Significance Level**: 5%

### Results and Conclusion
After simulating these KPPM group means, we get a **p-value** or **0.0**. The empirical distribution of the mean differences of KPPM of Mid Laners is shown below: 

<iframe
  src="assets/hyp_kppm.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>

Since the p-value < 0.05, we **reject the null hypothesis**. We have sufficient evidence to conclude that mid-laners from winning teams has a higher average KPPM than mid-laners from losing teams.


### Testing the remaining aggregate group means amongst mid laners
Let's apply the same hypothesis test as above, but with `vspm`, `cspm`, `dpm`, `earned_gpm`

Below is a Dataframe with the feature of interest, and the resulting p-value, using the same test as above: 

|        |   kppm |   vspm |   cspm |   dpm |   earned_gpm |
|:-------|-------:|-------:|-------:|------:|-------------:|
| pvalue |      0 |      0 |      0 |     0 |            0 |


We observe that all p-values are close to 0, so we reject the null for every test. Thus, we have sufficient evidence to conclude that **mid laners from winning teams, on average, have a higher `kppm`, `vspm`, `cspm`, `dpm`, and `earned_gpm` than mid laners from losing teams.** 


---


# Framing a Prediction Problem 
Knowing that there are significant game-metric differences between mid laners from winning and losing teams, we propose the following prediction problem: **Can a mid laner's performance within the first 15 miniutes reliably predict game outcomes?**

To answer this question, we will design, train, and evaluate a **binary classification model** that predicts if the player won or lost their LoL match, provided only with their in-game statistics 15 minutes into the game.

## Setup 
- **Model**: Binary classification

- **Target/Response Variable**: `result` (1 will indicate that the player won, and 0 will indicate that the player lost.)
> We chose `result` as this column directly answers what we are trying to predict, i.e., the game outcome. 

- **Performance Metric**: Accuracy and F1-Score 
  > **Why F1-Score?**
    > Predicting falsely whether a team won or not is of equal importance. Thus, we will F1 score as a performance evaluation method, since we care equally on the model's precision and recall score. 

  > **Why Accuracy?**
    > We will see later that our dataset is well balanced between each class (win or lose). Also, since misclassification is equally costly for each class, accuracy provides a reliable overall measure of our model performance.

- **What kind of features can we use to train our model?**: Since we are trying to predict the game outcome within 15 minutes of a mid laner's performance, **we can only use metrics that are measured within those 15 minutes and amongst mid laners only**. In this case, these are the columns in our Dataframe that has "at15" in it's name, and any "event" that is highly probably to have occured within 15 minutes (_but more on that later!_)


# Baseline Model 

For our baseline model, we split our data into a 80/20 train/test split (i.e., 20% of the data is reserved to test/evaluate our model, and we use the remaining 80% of our data to train/fit our Random Forest Classifier.) We then implemented a Random Forest Classifer using the following 3 features: `goldat15`, `xpat15`, `csat15`. All three of these features are **quantitative features**, and to make them more comparable/to scale, we also used a `StandardScaler()` to standardize them. Our target, `result`, already are 1's and 0's, so encoding is not necessary here. 


## Baseline Model Performance 
- **Accuracy (train data):** 1.0
- **Accuracy (test data):** 0.588
- **F1-Score:** 0.582

Though our accuracy score on the training data is 1.0 (_i.e., the fitted model predicted all values on the training data perfectly_), this is **very** misleading as our accuracy score on the test data is approximately **0.58**, meaning  that our model only predicted around 58% of the test data correctly-- this model has a lot of room for improvement!

The discrepancy of accuracy scores between the training and test data indicate that the model is **overfitting** the training data, and thus **does not generalize well** with new observations.


# Final Model 
Below are the ways we have improved our model:
> Note that this took a lot of trial and error, and multiple models were tested/compared in the process!

### 0. Addressing Column Dependency 

One of the biggest problems in our columns is that many of these columns are dependent on each other.

For example, in the game, you obtain more gold by killing more monsters/minions (increasing your creep score), and killing more enemy champions. Thus, `goldat15` is a mixed bag of all sorts of metrics that already exist in our columns. 

However, we want to have more control with exactly what features are being trained into the model, and since gold is a byproduct/combination of a lot of other metrics, it may be a better idea to exclude `goldat15`, or any feature that is related to gold. The same logic applies for `xpat15` (or any feature that is related to XP). 

### 1. Feature Selection

#### Quantitative Features
To mitigate this column dependency issue, we will use the features `csdiffat15` and `kpdiffat15` (a new column added), where
  - `csdiffat15` is the difference in creep score between the player and their opponent in 15 minutes
  - `kpdiffat15` is the difference in kill participation between the player and their opponent in 15 minutes

We selected these 2 columns for 2 main reasons: 
  1. Kill participation and creep score has no direct association with each other in the game: 1 focuses on enemy kills, while the other focuses on minion kills. 
  > Though there **could** have some association between the two (_e.g. if you have more kills, it probably means you are at the advantage, and thus it is easier for you to kill minions_), but unlike gold and XP, killing more minions does not directly translate to increasing kill participation.

  2. Using the **difference** of metrics between player and opponent better captures how the player is doing **compared to their opponent**, which ultimately determines the chances on whether they win the game or not. 


Since we are using the differences, then we know that for every row, **_there exists another row from the opposing side with the same metrics, but with the signs flipped_**. Since they essentially capture the same information (and thus each pair are dependent on each other), we filtered our dataset where all **players are from the `blue` team** (this choice was arbitary).

Note that even though `kpdiffat15` and `csdiffat15` have pretty different scales, Random Forest classifiers typically does not require feature scaling because is splits data based on feature values rather than magnitude. **So, we will not standardize those columns in our final model.**

Additionally, we converted all quantitative features from floats to **integers** (using `FunctionTransformer()`). This is because in theory, no value should have decimal, but only should be whole numbers. 



#### Categorical Features 
In our final model, we added the following 3 binary features: `firstblood`,`firstherald`, and `firstmidtower`. 

##### Why these features specifically? 
According to several sources, `firstblood` typically occurs within 5-10 minues of a match starting, so it is plausible to assume that first blood has already occured within 15 minutes. This is useful information as getting the first blood typiclaly indicates that the player is at an early advantage. 
Similarly, it is most plausible that the `firstherald` and `firstmidtower` are taken/destroyed during early game (though this is not guranteed). Using the herald and destroying the opponent's middle lane tower are good indicators of early game leads, as they suggest that the team is winning in objectives. However, it is worth noting that if the player who got the first herald (_this is typically the jungler_) drops it on the middle lane, these two features become dependent on each other, as the herald immensely helps with destroying plates/towers. However, we do not have any information on where the herald is dropped. 

All of these are encoded already, and null values were already handled from the cleaning procedure. 

#### ⚠️IMPORTANT NOTES AND ASSUMPTIONS⚠️
-  `firstherald` and `firstmidtower` indicates whether the **team** got the first herald, as opposed to if the individual mid laner got it
- This time, **we will assume that all 3 features happened within 15 minutes of a game**, which is plausible, but still an important assumption (and limitation!) made. 


> In summary, we added 5 new features: `csdiffat15`, `kpdiffat15`,`firstblood`,`firstherald`, and `firstmidtower`, with transformations/filtering described above 

Below are the first few rows of our final Dataframe that will be used to fit and train our model: 

|   result |   kpdiffat15 |   csdiffat15 |   firstblood |   firstherald |   firstmidtower |
|---------:|-------------:|-------------:|-------------:|--------------:|----------------:|
|        0 |           -3 |            1 |            0 |             1 |               1 |
|        0 |           -2 |            0 |            0 |             1 |               0 |
|        1 |            2 |            9 |            0 |             0 |               1 |
|        0 |            3 |            7 |            1 |             1 |               1 |
|        1 |            3 |           -9 |            0 |             1 |               1 |


### 2. Hyperparameter Tuning 
Here, we used **K-Fold cross validation** (K=5), and leveraged `GridSearchCV` to optimize the following parameters:

- `criterion`: 'gini' or 'entropy' (the "splitting method")
- `max_depth`: maximum of splits in a tree (this prevents potential overfitting)
- `min_samples_leaf`: minimum number of samples required at each each leaf node
- `n_estimators`: number of decision trees

After fitting our `GridSearchCV()` with our training data, we get the following best set of hyperparameters:

- `criterion`: 'gini' 
- `max_depth`: 5
- `min_samples_leaf`: 5
- `n_estimators`: 200

## Final Model Performance 
With our final features-- filtered and transformed accordingly-- and with our new hyperparameters, we implemented another Random Forest Classifier and got the following scores: 

- **Accuracy (train data):** 0.734
- **Accuracy (test data):** 0.723
- **F1-Score:** 0.748

We can see that implementing these changes does in fact improve our model! Our final model has an **accuracy score of 0.72** and an **F1 Score of 0.75**, which is a huge improvement from our base model scores (accuracy on test data: 0.58, F1 score: 0.57). In addition, the train and test accuracy scores are very similar, indicating that the final model generalizes better to new unseen data. However, there still is room for a lot more improvement in our final model, but we may need more richer data in order to add more uncorrelated features. 

This also could suggest the difficulty in predicting team-based game outcomes solely from **just one player**. Though the performance of each player in their respective position is crucial, League of Legends demands a lot of teamwork, and it is only until you see how the entire team is doing to determine how well they are progressing in the game. 


# Fairness Analysis
With our final model, we are going to assess whether are model is fair across based on the player's creep score difference. In other words, we will investigate the following question: **Does our model perform worse for players with a negative `csdiffat15` score than those with non-negative `csdiffat15` scores?** 

## Setup 

**Group `X`**: Players with `csdiffat15` greater than or equal to 0 

**Group `Y`**: Players with `csdifffat15` less than 0 

**Evaluation metric**: Accuracy

**Significance Level**: 5%

Below are the observed accuracy scores when we making predictions with our final model, separated by each group defined above: 


| group   |   accuracy |
|:--------|-----------:|
| X       |   0.738679 |
| Y       |   0.707669 |


**We will test if this difference is significant!**

## Permutation Test 

**Null Hypothesis**: The classifer's accuracy is the same for both Group `X` (non-negative `csdiffat15`) and Group `Y` (negative `csdiffat15`), and any differences between the two are due to random chance. 

**Alternative Hypothesis**: The classifier's accuracy is higher for Group `X`

**Test Statistic**: Difference in accuracy (Group `X` minus Group `Y`)

## Results and Conclusion 
After simulating these accuracy differences under the null hypothesis, we get a **p-value of 0.084**. Below is the empirical distribution these simulated differences: 

<iframe
  src="assets/fairness.html"
  width="900"
  height="700"
  frameborder="0"
></iframe>


Since our p-value (0.084) is greater than 0.05, we fail to reject the null hypothesis. Thus, have sufficient evidence to conclude that any differences in accuracy scores between players with a non-negative and negatve `csdiffat15` values are most likely due to random chance. Consequently, the classifer does not seem to demonstrate any unfairness within these groups. 


