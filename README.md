# League of Legends Mid Lane Performance Analysis

Author: Leena Kang 

# Introduction
League of Legends (LoL) is one of the most popular team-based multiplayer online battle arena (MOBA) game, with millions of players and a thriving professional scene. As a 5v5 strategy game, each player's role and performance contribute to the team's overall success. Specifically, the mid-laner, being primarily involved in the middle lane, can greatly influence team coordination, and thus can play a central role in shaping the flow of the game. We will investigate further what drive's a team's success in a League of Legends match, with a primary focus on mid-laners. 

Understanding the key drivers of victory in LoL is significant not only for competitive players and analysts but also for amateur players and esports fans. Identifying and quantifying these winning conditions can offer players at all skill levels the insights needed to improve their own gameplay and strategy. Additionally, being able to predict the likelihood of a team’s victory (_especially based on early-game performance_) provides viewers with a deeper understanding of match dynamics, enhancing their experience and enjoyment of the game. 

The primary goal of this project is to answer the following central question: **Amongst mid-laners, what are the most significant factors that contribute to a team's victory in a League of Legends match?** We will leverage in-game metrics to perform data and statistical analyses, and design, train, and evaluate predictive models to investigate if these measures hold predictive power (but more on that later! 😊)

## The Dataset 
- where the dataset is from, what year 
- number of rows, number of columns (and then mention the columns only included)
- columns + description 


# Data Cleaning and Exploratory Data Analysis


## Data Cleaning 
(final)


| gameid                |   result | side   | position   |   gamelength |   team_kpm |   ckpm |
|:----------------------|---------:|:-------|:-----------|-------------:|-----------:|-------:|
| ESPORTSTMNT01_2690210 |        0 | blue   | top        |        28.55 |     0.3152 | 0.9807 |
| ESPORTSTMNT01_2690210 |        0 | blue   | jng        |        28.55 |     0.3152 | 0.9807 |
| ESPORTSTMNT01_2690210 |        0 | blue   | mid        |        28.55 |     0.3152 | 0.9807 |
| ESPORTSTMNT01_2690210 |        0 | blue   | bot        |        28.55 |     0.3152 | 0.9807 |
| ESPORTSTMNT01_2690210 |        0 | blue   | sup        |        28.55 |     0.3152 | 0.9807 |

## Univariate Analysis
<iframe
  src="assets/univariate_team.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

## Bivariate Analysis 

<iframe
  src="assets/bivariate_boxplot.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="assets/bivariate_heatmap.html"
  width="800"
  height="800"
  frameborder="0"
></iframe>

<iframe
  src="assets/bivariate_scatter.html"
  width="900"
  height="700"
  frameborder="0"
></iframe>

## Interesting Aggregates 
(pivot table)


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


# Assessment of Missingness 

## NMAR Analysis 

## Missingness Dependency 
- **COL DIST**
- **EMPIRICAL DIST**


# Hypothesis Testing 
- **EMP DIST**




# Framing a Prediction Problem 
- Can a mid-laner's performance within the first 15 miniutes reliably predict game outcomes?

## Problem Identification 




# Baseline Model 




# Final Model 
- **CONFUSION MATRIX** 





# Fairness Analysis
- **EMP DIST**




