# Investigation on the Relationship Between Cooking Time and Average Recipe Ratings

by Xingyue Yan (x8yan@ucsd.edu)

## Overview

This is a project for DSC 80 at UCSD, exploring the relationship between the cooking time and the average rating of a recipe.


## Introduction

Food plays an important role in our daily lives, not only fulfilling basic needs but also bringing joy and fulfillment through both cooking and eating and cooking process to many. Nevertheless, in today’s fast-paced world, the time required to prepare a meal is one of the most critical factors influencing people’s recipe choices.  According to the U.S. Bureau of Labor Statistics, 57.2 percent of people 15 and older in the United States spent time preparing food and drink on an average day in 2022, and the amount of time they spent on average is around an hour per day (53 minutes). Considering the limited time we have in a day, while people who have tight schedules might seek quick and easy recipes, others may spend more time in the kitchen crafting elaborate dishes. Besides, longer cooking times can raise expectations for the food’s quality. This raises an intriguing question: **Does the preparation time of a recipe affect how people rate it?**

In this project, I investigate the relationship between **recipe ratings** and **cooking time**. Specifically, I want to determine whether recipes with shorter preparation times tend to receive higher ratings, reflecting a preference for convenience, or whether longer preparation times are associated with higher ratings, implying a preference for more elaborate dishes. To explore this question, I analyze two datasets from [food.com](https://www.food.com/), consisting of recipes and user ratings posted since 2008. These datasets were originally used in the research paper [Generating Personalized Recipes from Historical User Preferences](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf) by Majumder et al.

There are 83782 rows, each indicating a unique recipe, and 10 columns in the first dataset, `recipe`. A description of each column is given below:

| Column             | Description                                                                                                                                                                                       |
| :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `'name'`           | Recipe name                                                                                                                                                                                       |
| `'id'`             | Recipe ID                                                                                                                                                                                         |
| `'minutes'`        | Minutes to prepare recipe                                                                                                                                                                         |
| `'contributor_id'` | User ID who submitted this recipe                                                                                                                                                                 |
| `'submitted'`      | Date recipe was submitted                                                                                                                                                                         |
| `'tags'`           | Food.com tags for recipe                                                                                                                                                                          |
| `'nutrition'`      | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| `'n_steps'`        | Number of steps in recipe                                                                                                                                                                         |
| `'steps'`          | Text for recipe steps, in order                                                                                                                                                                   |
| `'description'`    | User-provided description                                                                                                                                                                         |
| `'ingredients'`    | Text for recipe ingredients                                                                                                                                                                       |
| `'n_ingredients'`  | Number of ingredients in recipe                                                                                                                                                                   |

There are 731927 rows, each showing a review from the user on a certain recipe, and 5 columns in the second dataset, `interactions`. A description of each column is given below:

| Column        | Description         |
| :------------ | :------------------ |
| `'user_id'`   | User ID             |
| `'recipe_id'` | Recipe ID           |
| `'date'`      | Date of interaction |
| `'rating'`    | Rating given        |
| `'review'`    | Review text         |

## Data Cleaning and Exploratory Data Analysis

To make sure the dataset is suitable for an effective analysis, I cleaned the data through the steps below. 

1. Merge the recipes and interactions datasets on id and recipe_id with a left join.

   - This allowed me to combine recipe information with user ratings and reviews.

1. Check the data types of all the columns.

   - I inspected the data types to identify any necessary conversions or cleaning steps.
   - | Column             | Description |
     | :----------------- | :---------- |
     | `'name'`           | object      |
     | `'id'`             | int64       |
     | `'minutes'`        | int64       |
     | `'contributor_id'` | int64       |
     | `'submitted'`      | object      |
     | `'tags'`           | object      |
     | `'nutrition'`      | object      |
     | `'n_steps'`        | int64       |
     | `'steps'`          | object      |
     | `'description'`    | object      |
     | `'ingredients'`    | object      |
     | `'n_ingredients'`  | int64       |
     | `'user_id'`        | float64     |
     | `'recipe_id'`      | float64     |
     | `'date'`           | object      |
     | `'rating'`         | float64     |
     | `'review'`         | object      |
     
1. Fill all ratings of 0 with np.nan.

   - Ratings are typically on a scale of 1 to 5, where 1 is the lowest and 5 is the highest. Therefore, a rating of 0 may indicate missing data. To avoid potential bias, I replaced all 0 ratings with np.nan

1. Add column `'average_rating'` containing average rating per recipe.

   - I calculated the average rating for each recipe, as a recipe can have multiple ratings from different users. Such provides a more comprehensive understanding of a recipe's overall rating.

1. Split values in the `'nutrition'` column into individual columns of floats.
   - The nutrition column stores various nutritional information as an object that looks like a list. I split this column into separate columns for each nutritional component (e.g., calories, total fat, sugar, etc.) and converted them to floats for numerical analysis.

1. Remove 0 values and values above the upper bound in minutes

   - I removed rows where the minutes column had a value of 0 to ensure that only valid cooking times were included. The upper bound using the IQR method is 132.5 minutes and 7991 rows will be removed. This might lead to some bias in the analysis as there are many recipes that involve baking and boiling meat, which may take longer time and are still valuable to our analysis. Therefore, instead of using the IQR, I defined an upper bound of 360 minutes (6 hours) to filter out extreme outliers. This ensures that recipes with unrealistically long cooking times are excluded from the analysis. Before any deletion, there are 83782 rows. Since 1 value of 0 and 2911 outliers in the 'minutes' were removed, the number of rows after removing outliers is 80870.

1. Create dummy variables for tags
   - I first flattened the tags column into a single list and counted the frequency of each tag to identify the most common tags. We created dummy variables for the following tags: 'easy', '60-minutes-or-less', '30-minutes-or-less', and '15-minutes-or-less'. For example, 'is_easy' is a boolean column that checks if the tags of recipes contain the word 'easy'. Recipes tagged as 'easy' are typically simpler and quicker to prepare, requiring fewer steps or less expertise. This step separates the recipes into two groups: those that are easy to prepare and those that are not. This provides us with another way to examine the relationship between the ratings of recipes and cooking time based on their perceived difficulty. Overall, these tags are particularly relevant to our analysis of cooking time and ratings. 


#### Result
Here are the types of all the columns after data cleaning.

| Column                   | Description    |
| :----------------------  | :------------- |
| `'name'`                 | object         |
| `'id'`                   | int64          |
| `'minutes'`              | int64          |
| `'contributor_id'`       | int64          |
| `'submitted'`            | datetime64[ns] |
| `'tags'`                 | object         |
| `'nutrition'`            | object         |
| `'n_steps'`              | int64          |
| `'steps'`                | object         |
| `'description'`          | object         |
| `'ingredients'`          | object         |
| `'n_ingredients'`        | int64          |
| `'average_rating'`       | float64        |
| `'calories'`             | float64        |
| `'total_fat'`            | float64        |
| `'sugar'`                | float64        |
| `'sodium'`               | float64        |
| `'protein'`              | float64        |
| `'saturated fat'`        | float64        |
| `'carbohydrates'`        | float64        |
| `'is_easy'`              | bool           |
| `'is_60_minutes_or_less'`| bool           |
| `'is_30_minutes_or_less'`| bool           |
| `'is_15_minutes_or_less'`| bool           |


The cleaned dataframe has 80870 rows and 24 columns. The first 5 rows of the cleaned dataframe are shown below, with columns most related to the project topic selected.

| name                                 |     id |   minutes |   contributor_id |   n_steps |   n_ingredients |   average_rating | is_easy   | is_60_minutes_or_less   | is_30_minutes_or_less   | is_15_minutes_or_less   |
|:-------------------------------------|-------:|----------:|-----------------:|----------:|----------------:|-----------------:|:----------|:------------------------|:------------------------|:------------------------|
| 1 brownies in the world    best ever | 333281 |        40 |           985201 |        10 |               9 |                4 | False     | True                    | False                   | False                   |
| 1 in canada chocolate chip cookies   | 453467 |        45 |          1848091 |        12 |              11 |                5 | False     | True                    | False                   | False                   |
| 412 broccoli casserole               | 306168 |        40 |            50969 |         6 |               9 |                5 | True      | True                    | False                   | False                   |
| millionaire pound cake               | 286009 |       120 |           461724 |         7 |               7 |                5 | False     | False                   | False                   | False                   |
| 2000 meatloaf                        | 475785 |        90 |          2202916 |        17 |              13 |                5 | False     | False                   | False                   | False                   |

### Univariate Analysis

#### Distribution of Cooking Time

I first plotted the distribution of cooking times for recipes in the dataset using a histogram. As shown below, most recipes take between 0 and 60 minutes to prepare, with a peak of around 30 minutes. The median cooking time is **35 minutes**, as indicated by the red dashed line. The distribution is right-skewed, which aligns with expectations.  

<iframe
  src="recipes-analysis/assets/cooking_time_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Distribution of Average Ratings

I then analyze the distribution of average ratings for recipes with another histogram. Most recipes have ratings between 4 and 5, indicating that users generally rate recipes highly. The mean average rating is **4.63**, as indicated by the green dashed line. The overall distribution is left-skewed.

<iframe
  src="recipes-analysis/assets/average_rating_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

The heatmap below shows the density of recipes across cooking time and average rating. Darker colors indicate a lower number of recipes, while lighter colors indicate fewer recipes. The graph shows that the lightest color appears for recipes with a cooking time of 0–30 minutes and an average rating of 5, implying that quick, highly-rated recipes are popular. As cooking time increases, the colors seem to darken for each rating, indicating fewer recipes, suggesting that users may prefer shorter preparation times.

<iframe
  src="recipes-analysis/assets/heatmap_minutes_vs_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

To further evaluate the relationship between average rating and cooking time, I plotted a horizontal grouped bar plot, comparing the distribution of average ratings for recipes that take 30 minutes or less (yellow) and those that take more than 30 minutes*. Each bar represents the proportion of recipes with a specific average rating in each group. Recipes with a cooking time of ≤30 minutes tend to have a higher proportion of 5-star ratings. Such further indicates that users might prefer quick recipes and are more likely to rate them highly. Nevertheless, further analysis will be conducted to check if the difference in the proportion is significant or not.

<iframe
  src="recipes-analysis/assets/rating_distribution_cooking_time_30.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates
As this project focuses on the relationship between average rating and cooking time, it is intriguing to see how cooking time correlated with other variables relevant to preparation, such as the number of ingredients and 
steps to cook. Therefore, to investigate the relationship between cooking time, number of steps, and number of ingredients, I first removed outliers from the `minutes` column using the IQR method. I then grouped the data by cooking time bins and calculated the mean number of steps and mean number of ingredients for each bin. The table is shown below. 

| minutes_bin   |   mean_n_steps |   mean_n_ingredients |
|:--------------|---------------:|---------------------:|
| 0-30          |        7.64206 |              7.80665 |
| 30-60         |       11.4932  |             10.0906  |
| 60-120        |       13.1389  |             10.9732  |

The bar plot below shows the relationship between cooking time and the mean number of steps. In our dataset, the mean number of steps is highest for cooking time from 60-120 minutes. There seems to be a positive correlation between these two variables, indicating that longer recipes are more complex and are likely to involve more preparation steps.

<iframe
  src="assets/mean_steps_by_cooking_time.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The bar plot below shows the relationship between cooking time and the mean number of ingredients. Longer recipes tend to require more ingredients, suggesting that they are likely to involve a wider variety of components.

<iframe
  src="assets/mean_ingredients_by_cooking_time.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>



## Assessment of Missingness




## Hypothesis Testing


