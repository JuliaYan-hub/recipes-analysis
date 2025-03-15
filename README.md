# Investigation on the Relationship Between Cooking Time and Average Recipe Ratings

by Xingyue Yan (x8yan@ucsd.edu)

## Overview

This is a project for DSC 80 at UCSD, which examine the relationship between the cooking time and the average rating of a recipe.


## Introduction

Food plays an important role in our daily lives, not only fulfilling basic needs but also bringing joy and fulfillment through both cooking and eating and cooking process to many. Nevertheless, in today’s fast-paced world, the time required to prepare a meal became a critical factors influencing people’s choices of recipe. According to the U.S. Bureau of Labor Statistics, 57.2 percent of people 15 and older in the United States spent time preparing food and drink on an average day in 2022, and the amount of time they spent on average is 53 minutes. Considering the limited time we have in a day, people who have tight schedules might just wan to seek quick and easy recipes, and others may spend more time crafting elaborate dishes. Besides, longer cooking times can raise our expectations for the food’s quality. Such  raises an intriguing question: **Does the preparation time of a recipe affect how people rate it?**

In this project, I investigated the relationship between **recipe ratings** and **cooking time**. To be more specific, I want to see whether recipes with shorter preparation times tend to receive higher ratings, reflecting a preference for convenience, or whether longer preparation times are associated with higher ratings, implying a preference for more elaborate dishes. To explore this question, I analyze two datasets from [food.com](https://www.food.com/), consisting of recipes and user ratings posted since 2008. These datasets were originally used in the research paper [Generating Personalized Recipes from Historical User Preferences](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf) by Majumder et al.

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

   - This allowed us to combine recipe information with user ratings and reviews.

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

   - I removed rows where the minutes column had a value of 0 to ensure that only valid cooking times were included. The upper bound using the IQR method is 132.5 minutes and 7991 rows will be removed if we use this method. This might lead to some bias in the analysis as there are many recipes that involve baking and boiling meat, which may take longer time and are still valuable to our analysis. Therefore, instead of using the IQR, I defined an upper bound of 360 minutes (6 hours) to filter out extreme outliers. I use 6 hours as it is typically the longest time when baking or boiling meat. This ensures that recipes with unrealistically long cooking times are excluded from the analysis. Before any deletion, there are 83782 rows. Since 1 value of 0 and 2911 outliers in the 'minutes' were removed, the number of rows after removing outliers is 80870.

1. Create dummy variables for tags

   - I first flattened the tags column into a single list and counted the frequency of each tag to identify the most common tags. Then I created dummy variables related to our analysis: 'easy', '60-minutes-or-less', '30-minutes-or-less', and '15-minutes-or-less'. For example, 'is_easy' is a boolean column that checks if the tags of recipes have the tag 'easy'. Recipes tagged as 'easy' are typically simpler and quicker to prepare, requiring fewer steps or less expertise. Such provides us with another way to examine the relationship between the ratings of recipes and cooking time based on their perceived difficulty. Note that there are some Recipes with minutes less than 30 but are not tagged with '30-minutes-or-less' in the tags column, but we generate new variables like 'is_30_minutes_or_less' based on tags column, not the minutes column. The reason we chose the tags column is that people might be able to filter recipes through tags when searching without looking at the actual cooking time of a recipe. Nevertheless, all recipes with the 'is_30_minutes_or_less' tag have a cooking time of less than 30 minutes. 


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

| name                                 |     id |   minutes |   contributor_id |   n_steps |   n_ingredients |   average_rating |   calories |   sugar | is_easy   | is_60_minutes_or_less   | is_30_minutes_or_less   |
|:-------------------------------------|-------:|----------:|-----------------:|----------:|----------------:|-----------------:|-----------:|--------:|:----------|:------------------------|:------------------------|
| 1 brownies in the world    best ever | 333281 |        40 |           985201 |        10 |               9 |                4 |      138.4 |      50 | False     | True                    | False                   |
| 1 in canada chocolate chip cookies   | 453467 |        45 |          1848091 |        12 |              11 |                5 |      595.1 |     211 | False     | True                    | False                   |
| 412 broccoli casserole               | 306168 |        40 |            50969 |         6 |               9 |                5 |      194.8 |       6 | True      | True                    | False                   |
| millionaire pound cake               | 286009 |       120 |           461724 |         7 |               7 |                5 |      878.3 |     326 | False     | False                   | False                   |
| 2000 meatloaf                        | 475785 |        90 |          2202916 |        17 |              13 |                5 |      267   |      12 | False     | False                   | False                   |

### Univariate Analysis

#### Distribution of Cooking Time

I first plotted the distribution of cooking times for recipes in the dataset using a histogram. As shown below, most recipes take between 0 and 60 minutes to prepare. The peak of around 30 minutes. The median cooking time is **35 minutes**, as indicated by the red dashed line. The distribution is right-skewed, which aligns with our expectations.  

<iframe
  src="assets/cooking_time_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Distribution of Average Ratings

I then analyzed the distribution of average ratings for recipes with another histogram. Most recipes have ratings between 4 and 5. Such indicates that users generally rate recipes highly. The mean average rating is **4.63**, as indicated by the green dashed line. The overall distribution is left-skewed.

<iframe
  src="assets/average_rating_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

The heatmap below shows the density of recipes across cooking time and average rating. Darker colors indicate a lower number of recipes, while lighter colors indicate fewer recipes. The graph shows that the lightest color appears for recipes with a cooking time of 0–30 minutes and an average rating of 5, implying that quick, highly-rated recipes are popular. As cooking time increases, the colors seem to darken for each rating, meaning fewer recipes. Such implies that users may prefer shorter preparation times.

<iframe
  src="assets/heatmap_minutes_vs_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

To further evaluate the relationship between average rating and cooking time, I plotted a horizontal grouped bar plot, comparing the distribution of average ratings for recipes that take 30 minutes or less (yellow) and those that take more than 30 minutes using `'minutes'`, not `'is_30_minutes_or_less'` tag. Each bar represents the proportion of recipes with a specific average rating in each group. Recipes with a cooking time of less than 30 minutes tend to have a higher proportion of 5-star ratings. Such further indicates that users might prefer quick recipes and are more likely to rate them highly. Nevertheless, further analysis will be conducted to check if the difference in the proportion is significant or not.

<iframe
  src="recipes-analysis/assets/rating_distribution_cooking_time_30_real.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

As this project focuses on the relationship between average rating and cooking time, it is intriguing to see how cooking time correlated with other variables relevant to preparation, such as the number of ingredients and steps to cook. Therefore, to investigate the relationship between cooking time, number of steps, and number of ingredients, I first  create a copy of our cleaned dataset and removed futher remove outliers from the `minutes` column of the copied dataframe using the IQR method to analyze more regular cooking time. I then grouped the copied data by cooking time bins and calculated the mean number of steps and mean number of ingredients for each bin. The table is shown below. 

| minutes_bin   |   mean_n_steps |   mean_n_ingredients |
|:--------------|---------------:|---------------------:|
| 0-30          |        7.64206 |              7.80665 |
| 30-60         |       11.4932  |             10.0906  |
| 60-120        |       13.1389  |             10.9732  |

The bar plot below shows the relationship between cooking time and the mean number of steps. In our dataset, the mean number of steps is highest for cooking time from 60-120 minutes, and there seems to be a positive correlation between these two variables, implying that longer recipes are more complex and are likely to involve more preparation steps.

<iframe
  src="assets/mean_steps_by_cooking_time.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The bar plot below shows the relationship between cooking time and the mean number of ingredients. Longer recipes tend to require more ingredients, which implies they are likely to involve more components.

<iframe
  src="assets/mean_ingredients_by_cooking_time.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Assessment of Missingness

In the cleaned dataset, columns of 'average_rating' and 'description' have a lot missing values. The following assessment dive into the type of missingness and the its dependency.

### NMAR Analysis

I believe that the missingness of the `average_rating` column is **NMAR (Not Missing at Random)**. The reason behind my interpretation is that recipes with no ratings are likely to be less popular or less frequently interacted with by users. In other words, recipes that are more popular or have been tried by more users are more likely to receive ratings, while less popular recipes may remain unrated. And recipes with no ratings means the lack of `average_rating`.  Such introduces a bias in the missingness of the `average_rating` column, making it NMAR.

### Missingness Dependency

I analyzed the missingness of the `average_rating` column by testing its dependency on two other columns: `minutes` (cooking time) and `n_steps` (number of steps).

> Cooking Time (`minutes`) and Average Rating

**Null Hypothesis**: The missingness of average ratings does not depend on the cooking time of the recipe.

**Alternate Hypothesis**: The missingness of average ratings does depend on the cooking time of the recipe.

**Test Statistic**: The absolute difference in the mean cooking time between recipes with missing average ratings and recipes with non-missing average ratings.

**Significance Level**: 0.05

The plot below compares the distribution of cooking times (`minutes`) for recipes where the `average_rating` is missing and where it is not missing. This helps us understand whether the missingness of `average_rating` is related to cooking time. The curves are clearly distinct when cooking time is below around 50 cooking minutes,and it became overlaped when cooking time increases. It suggests that the missingness of `average_rating` may depend on cooking time. 

<iframe
  src="assets/kde_minutes_by_average_rating_missingness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

I conducted a permutation test through randomly shuffling the missingness of average rating for 1000 times to determine whether the missingness of `average_rating` depends on cooking time (`minutes`). 

<iframe
  src="assets/permutation_test_distribution_cooking_time.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The observed test statistic is **9.61**, which means there is a substantial difference in the mean cooking time between recipes with missing and non-missing `average_rating`. The p-value of **0.0** suggests that the probability of observing such a large difference by random chance is extremely low. Such provides us a strong evidence to **reject the null hypothesis**. In other words, the missingness of `average_rating` is very likely to  **dependent on cooking time**. Nevertheless, it is important to keep in mind that we are using the `minutes` from the cleaned data, where extremely large minutes, those greater than 360, were removed. 

> Sodium and Average Rating

**Null Hypothesis:** The missingness of ratings does not depend on the sodium content of the recipe.

**Alternate Hypothesis:** The missingness of ratings does depend on the sodium content of the recipe.

**Test Statistic:** The absolute difference of mean in sodium content (PDV) of the distribution of the group with missing ratings and the distribution of the group without missing ratings.

**Significance Level:** 0.05

Since I did not remove outliers in sodium, it is hard to identify the shapes of the two distributions due to outliers. I updated the upper bound using IQR method to take a closer look.

<iframe
  src="assets/kde_sodium_by_average_rating_missingness_zoomed.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Through similar shuffling process, I collect 1000 simulated mean differences in the two distributions as described in the test statistic.

<iframe
  src="assets/permutation_test_sodium.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The **observed statistic** is  **1.3882** , shown by the red vertical line on the graph. Since the **p-value** that we found **(1.39)** is **greater than** 0.05, we **fail to reject** the null hypothesis. The missingness of ratings is likely to **not depend** on the sodium content of the recipe.


## Hypothesis Testing

I am curious about whether recipes with longer cooking times (above 30 minutes) are rated differently compared to recipes with shorter cooking times (30 minutes or less). I take 30 minutes as an standard because it is around median of cooking time, and according to the bivariate analysis we conducted before, recipes with a cooking time of ≤30 minutes tend to have a higher proportion of 5-star ratings. To investigate this, I performed a permutation test with the following hypotheses, test statistic, and significance level.

**Null Hypothesis:** Recipes with cooking times above 30 minutes are rated the same as recipes with cooking times equal to or below 30 minutes.

**Alternative Hypothesis:** Recipes with cooking times above 30 minutes are rated lower than recipes with cooking times equal to or below 30 minutes.

**Test Statistic:** The difference in mean rating between recipes with cooking times equal to or below 30 minutes and recipes with cooking times above 30 minutes.

**Significance Level:** 0.05

I chose a permutation test because we do not have information about the underlying population distribution, and I want to test whether the two groups of recipes (above 30 minutes and below/equal to 30 minutes) come from the same distribution. The test statistic, the difference in mean ratings, allows us to directly compare the average ratings of the two groups with direction.

The **observed test statistic** is **0.0311**, as how red vertical line on the graph shows. Since the **p-value** that we found **(0.0)** is **less than** 0.05, we **reject** the null hypothesis. Such suggests that recipes with cooking times above 30 minutes are rated significantly differently compared to recipes with cooking times equal to or below 30 minutes. To be more specific,, recipes with shorter cooking times, equal or less than 30 minutes, **tend to have higher average ratings** than recipes with longer cooking times, greater than 30 minutes, but it is not certain.

<iframe
  src="assets/permutation_test_minutes_vs_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Framing a Prediction Problem

With previous analysis, it would be valuable to form a model to predict the **average rating** of a recipe based on its features. Such would allow users to gauge the overall quality and popularity of a recipe in a short time without needing to scroll through individual reviews. In other words, the average rating serves as a summary of user satisfaction and can help others decide whether to try a recipe. Additionally, this prediction can provide insights for creators into how to improve recipes by identifying which features, such as cooking time and number of ingredients, are associated with higher or lower ratings.

We will use **regression** because while rating is a categorical value, our response variable (`average_rating`) is a continuous numeric value ranging from 1 to 5. ( We would remove NA values for effective and unbiased analysis). With average rating as our response variable, I planned to take `minutes`(the time required to prepare the recipe),`n_steps`(The number of steps in the recipe), `is_easy`(Whether the recipe is easy) as features used to predict `average_rating`. I planned to include features because they directly or indirectly influence user experience and satisfaction.

Since the average rating is a continuous numerical variable with a highly skewed distribution,  I will use **Mean Absolute Error (MAE)** to evaluate the accuracy of the model, as it measures the average absolute difference between the predicted and observed values. Such would provide us with a clear interpretation of the average error in the same units as the response variable (`average_rating`). Also, it is less sensitive to outliers and skewed data because it does not square errors, considering that most average ratings in our dataset are concentrated in the 4-5 range. I chose this instead of **F1-score** is because **F1-score** is a metric for classification tasks, and our problem is a regression problem.

## Baseline Model

For the baseline model, I trained a **Linear Regression model** to predict the average rating of recipes. The dataset was randomly split into training (80%) and test (20%) sets to evaluate the model. The two key features in this baseline model are `real_30_minutes_or_less`, a binary nominal feature indicating whether the recipe takes 30 minutes or less to prepare based on `minutes` (`1` for `True` and `0` for `False`), and `'is_easy'`, a binary nominal feature indicating whether the recipe is tagged as "easy" (`1` for `True` and `0` for `False`). 

The baseline model has an MAE of **0.4617**. This means that the predictions of the baseline model are, on average, off by about **0.4617** points from the observed average ratings. Given that `average_rating` ranges from 1 to 5, we can say that the model's performance is acceptable. Nevertheless, further analysis of additional features like `n_steps`, `n_ingredients`, and `calories` and more advanced models to improve performance and better capture the factors influencing recipe ratings are needed. 

## Final Model

For the final model, I extended the baseline model by adding new features and using linear regression to predict the average rating of recipes. The dataset was split as the one split in the baseline model for fair comparison. 

I used the following features to predict the average rating of a recipe:

`'steps_per_ingredient'`

This feature represents the ratio of the number of steps to the number of ingredients in a recipe. I chose this feature because it captures the complexity of the recipe. To be more specific, recipes with a higher `steps_per_ingredient` might be perceived as more difficult, potentially leading to lower ratings. By contrast, recipes with fewer steps per ingredient might seem to be easier to users. I generated it  through dividing `'n_steps'` by `'n_ingredients'`.


`''prop_sugar''`

This feature means the proportion of sugar calories relative to the total calories in a recipe. When approaching the data, I found that recipes with a higher proportion of sugar tend to receive lower ratings. It is in my expectation as nowadays people might prefer less sugar for health. I took several steps to generate this, given that the `'sugar'` column represents sugar content as a percentage of the daily value (PDV), I divided values in `'sugar'` by 100 and multiplied them by 25 to convert this to grams, since 100% PDV equals 25 grams on food.com. Considering that each gram of sugar contains 4 calories, I then multiplied the grams by 4 to get the total sugar calories. Finally, I divide it by the recipe’s `'calories'` to find the proportion of sugar-derived calories.  I want to capture the relationship between sugar content and user satisfaction through this feature.

`'real_30_minutes_or_less'`

It is a binary feature that indicates whether a recipe can be prepared in 30 minutes or less based on `'calories'` not `'real_30_minutes_or_less'`. As shown by our previous analysis, more recipes with shorter preparation times seem to receive a higher average rating. Such might be caused by the limited time people have. It would be hard for a full-time employee or student to spend a huge amount of time on cooking. I generated this feature from the 'minutes' column and one-hot encoded for use in the model, and I want to capture the relationship between preparation time and user satisfaction through this feature. 

`'is_easy'`

It is a binary feature that shows whether a recipe is tagged as "easy" in its `'tags'` column. I chose this feature because recipes labeled as "easy" are likely to appeal to users seeking quick and simple meals, which could lead to higher ratings. People who are busy might just filter "easy" recipes. Also, it is reasonable to lower expectations and therefore more likely to be fulfilled by the easy recipes given that it is easy to relate easy with fast food and fine dining with long cooking time. This feature was created by checking if the `'tags'` column contains the word "easy." By including this feature, we aim to capture the relationship between recipe simplicity and user satisfaction, as users are more likely to rate recipes higher if they are easy to prepare and require minimal effort.

### Modeling Algorithm

 I used **Linear Regression** as my final modeling algorithm. It is is a simple yet effective model for regression tasks, and it allows us to interpret the relationship between the features and the target variable (`average_rating`) directly through the coefficients. 

### Hyperparameter Tuning

I focused on feature engineering and preprocessing to improve the model's performance, considering that linear regression does not have hyperparameters to tune. To make variables more suitable for the model, I standardized the numerical features using `StandardScaler`  and one-hot encoded the categorical features on `real_30_minutes_or_less` and `is_easy`.

I also experimented with **Lasso Regression** for comparison and performed hyperparameter tuning using `GridSearchCV` to find the best regularization strength (`alpha`). The best hyperparameters for Lasso Regression were:'model__alpha': 0.01

### Final Model Performance and Improvement

As mentioned before, the baseline model resulted a **Mean Absolute Error (MAE)** of **0.4617** and an **R-squared** value of **0.0005**. Our final Linear Regression model which adds more features has a slightly better **MAE** of **0.4613** and an **R-squared** value of **0.0010**. Although the improvement in MAE is minimal, the final model's R-squared value is higher, indicating that the model could explain a slightly larger portion of the variance in the data compared to the baseline model.

Such improvement can be attributed to the inclusion of additional features and more sophisticated feature engineering. For example, by adding features like `'steps_per_ingredient'` and `'prop_sugar'`, the final model was able to capture more nuanced relationships between recipe characteristics and user ratings. 
 

## Fairness Analysis

To assess the fairness of the final model, I evaluated if it performs differently for two distinct groups of recipes: **high-calorie recipes** and **low-calorie recipes**. These groups were defined based on the distribution of the `'calories'` column in the dataset:

- **Group X (High-Calorie Recipes)**: Recipes with calories greater than or equal to the **75th percentile** of the `'calories'` distribution.
- **Group Y (Low-Calorie Recipes)**: Recipes with calories less than or equal to the **25th percentile** of the `'calories'` distribution.

I chose these thresholds to ensure that the groups represent distinct subsets of the data. That is, high-calorie recipes should be significantly more calorie-dense than low-calorie recipes.

I used **Mean Absolute Error (MAE)** as the evaluation metric, which is the same metric we use for modeling. This metric is suitable because it would provide a straightforward interpretation of the model's prediction errors in the same units as the response variable.

- **Null Hypothesis**: The model is fair. The MAE for high-calorie recipes and low-calorie recipes is roughly the same, and any differences are due to random chance.

- **Alternative Hypothesis**: The model is unfair. The MAE for high-calorie recipes is significantly different from the MAE for low-calorie recipes.

- **Test Statistic**:  **absolute difference in MAE** 

- **Significance Level**: **0.05** 

### Permutation Test Results
- **Observed Difference in MAE**: **0.0070**
- **p-value**: **0.108**

Since the p-value (**0.108**) is greater than the significance level (**0.05**), we **fail to reject the null hypothesis**. This indicates that the observed difference in MAE between high-calorie and low-calorie recipes is not statistically significant. Therefore, we could say that the model is fair with respect to these two groups.

### Visualization

Below is the distribution of the absolute differences in MAE from the permutation test, and the observed difference marked by a red dashed line.

<iframe
  src="assets/fairness_analysis_plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The observed difference falls within the bulk of the distribution, which confirms that the difference is not statistically significant. This supports our conclusion that the model is fair with respect to high-calorie and low-calorie recipes.

