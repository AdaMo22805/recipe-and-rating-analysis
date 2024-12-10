# Does a recipe's level of difficulty affect user ratings?
by Ada Mo

## Introduction
Cooking is a part of many people's daily lives but some dishes are easier to make while others are more difficult. Often times, the difficulty of a dish is associated with the time and effort put into the making of the dish, meaning a dish is more difficult to make when more time, ingredients, and skills are required. I am curious **how the level of difficulty can affect one's rating of the dish**. To investigate in the relationship, I will be analyzing from 2 datasets containing recipes and ratings since 2008 by [food.com](https://www.food.com).

The first dataset, `recipe`, contains 83782 rows of unique recipes and 12 columns listed below:

| Columns | Description |
| ----------- | ----------- |
| `'name'` | Recipe name |
| `'id'` | Recipe ID |
| `'minutes'` | Minutes to prepare recipe |
| `'contributor_id'` | User ID who submitted this recipe |
| `'submitted'` | Date recipe was submitted |
| `'tags'` | Food.com tags for recipe |
| `'nutrition'` | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| `'n_steps'` | Number of steps in recipe |
| `'steps'` | Text for recipe steps, in order |
| `'description'` | User-provided description |
| `'ingredients'` | Text for recipe ingredients |
| `'n_ingredients'` | Number of ingredients |


The second dataset, `interactions`, contains 731927 rows of unique user reviews and 5 columns listed below:

| Columns | Description |
| ----------- | ----------- |
| `'user_id'` | User ID |
| `'recipe_id'` | Recipe ID |
| `'date'` | Date of interaction |
| `'rating'` | Rating given |
| `'review'` | Review text |

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning
To make use of the `recipe` and `interactions` datasets to answer our question, we will perform the following steps to clean our data

1. Left merge the recipes and interactions datasets together.
> Allows for each review to be matched with the recipe it is interacting with.
2. Replace all `0`s in the `'rating'` column with `np.nan`.
> A rating of 0 does not exist in the original website our data was collected from. All ratings should be from a scale of 1-5, with 1 being the lowest and 5 being the highest. A rating of 0 in the dataset indicates a missing rating in that specific interaction with a recipe, thus, it is more appropriate to replace all 0s with np.nan, else a numerical representation of missing values may introduce bias later on in our analysis.
3. Computed and included the average rating per recipe into our merged dataset.
> Each recipe can have multiple ratings from many different users so to have a better understanding of a recipe's overall rating, we will compute the average rating for each recipe.
4. Create `'easy_tag'` column which takes on the value of `1` if the recipe contains a `'easy'` or `'beginner-cook'` in the `'tag'` column, else `0`.
> If a dish has the `'easy'` or `'beginner-cook'` tag, the author of the recipe indicates it is an easy dish to make.
5. Converted minutes to hours and stored in a new column `'hours'`
> A recipe's difficulty does not vary much with an additional minute but it for sure will with an additional hour, therefore, it is more appropriate to store the time it takes to make a recipe in units of hours.
6. Create `'easy_hour'` column which takes on the value `1` if `'hours' <= 1` hour, else `0`
> If a dish can be made within 1 hour, then we will classify it as an easy dish, thus label it with `1` in the new column.
7. After merging and performing some cleaning on our dataset, some columns are no longer needed so we will select and keep necessary columns.
> - `'id'` is redundant with `'recipe_id'` so we will drop `'id'`.
> - time is now stored in `'hours'` so we will drop `'minutes'`.
8. Sort through each column and remove outliers.
> - There is a recipe with a oddly high time requirement to make the dish. It turns out the name of the recipe is *'how to preserve a husband'*, and includes steps on how to preserve a husband 💀. This obviously isn't a food dish so we will remove it from our dataset.
> - There is also another recipe whose `'name'` is `np.nan` and `'rating'` is `np.nan` even before step 2. This is quite odd since all other `rating` without values used `0` as a placeholder. Therefore, to prevent any future bias/confusion, we will remove this unnamed recipe from our dataset as well.
> - There is a recipe requiring 0 minutes to make but in the `'steps'` column, `['grind the almonds into a fine powder using a coffee , nut , or spice grinder']` suggests the recipe will take longer than 0 minutes to make.
9. Convert dates into `datetime` objects and extract the year.
> Converting strings of dates into `datetime` objects will allow us to extract meaning and perform calculations on the dates (if needed later on).

Our dataset `recipe_ratings_clean` is now ready for use with 234423 rows of recipe reviews and 19 columns.

| Columns | Description |
| ----------- | ----------- |
| `'name'` | Recipe name |
| `'contributor_id'` | id of author |
| `'submitted'` | year of recipe submission |
| `'tags'` | Food.com tags for recipe |
| `'nutrition'` | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| `'n_steps'` | Number of steps in recipe |
| `'steps'` | Text for recipe steps, in order |
| `'description'` | User-provided description |
| `'ingredients'` | Text for recipe ingredients |
| `'n_ingredients'` | Number of ingredients |
| `'user_id'` | User ID |
| `'recipe_id'` | Recipe ID |
| `'rating_date'`| Year of rating interaction |
| `'rating'` | Rating given |
| `'review'` | Review text |
| `'avg_rating'` | Average rating of a recipe |
| `'easy_tag'` | Binary difficulty label based on recipe tags |
| `'hours'` | Hours needed to prepare recipe |
| `'easy_hour'` | Binary difficulty label based on recipe hours |



Here is a sneak peek of the dataset with only a sub-set of the columns.

| name                                      |   submitted |   n_steps |   n_ingredients |   easy_tag |   easy_hour |   rating |   avg_rating | ... |
|:------------------------------------------|------------:|----------:|----------------:|-----------:|------------:|---------:|-------------:|----:|
| easy milk chocolate frosting for brownies |        2008 |         4 |               5 |          1 |           1 |        5 |      4.60714 | ... |
| won ton soup                              |        2008 |        11 |              16 |          0 |           1 |        5 |      5       | ... |
| kung pao rice with edamame                |        2008 |        10 |              17 |          0 |           1 |        4 |      4.66667 | ... |
| apple and cinnamon sponge pudding         |        2009 |         9 |               8 |          0 |           0 |        5 |      4.90909 | ... |
| indian style lentils                      |        2011 |         5 |              11 |          1 |           0 |        4 |      4       | ... |
| ...                                       |         ... |       ... |             ... |        ... |         ... |      ... |      ...     | ... |



### Univariate Analysis

Let's explore the distribution of hours used in each recipe. 

<iframe
  src="assets/hours.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This graph tells us that only very few recipes take over 500 hours to make. This isn't very informative about our distribution, so let's zoom in.

<iframe
  src="assets/hours_zoomed.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We can now see that our distribution is skewed right with a majority of our recipes requiring less than 1 hour to make.

Now, let's explore the distribution of the number of ingredients used in each recipe. Below, we can see an almost-normal distribution but slightly skewed right. The mean number of ingredients across all recipes is approximately 9.07.

<iframe
  src="assets/n_ingredients.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


### Bivariate Analysis
Now let's explore the relationship between 2 variables in our data.

Here is a scatter plot showing the relationship between the number of steps and the number of ingredients in a recipe.

<iframe
  src="assets/steps_vs_ingredients.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

It is actually interesting to see that recipes seem to have a general range of number of ingredients that is unaffected by the number of steps.


### Interesting Aggregates

From the 2 grouped tables below, we can see that the recipes with an easy tag (`'easy_hour' == 1` or `'easy_tag' == 1`) tend to have a higher rating while requiring less steps and less ingredients


|   easy_hour |   rating |   n_steps |   n_ingredients |
|------------:|---------:|----------:|----------------:|
|           0 | 4.658025 | 12.781188 |       10.480886 |
|           1 | 4.686616 |  9.137786 |        8.622740 |  

  

|   easy_tag |   rating |   n_steps |   n_ingredients |
|-----------:|---------:|----------:|----------------:|
|          0 | 4.673912 | 12.605641 |       10.795289 |
|          1 | 4.684104 |  8.159377 |        7.833623 |

Combining the 2 tables above, we can see more interesting patterns as seen below.
- Recipes that are marked easy (`'easy_tag' == 1`) and can be quickly made (`'easy_hours' == 1`) have the highest average ratings and are the quickest to make, on average.
- Recipes that are falsely marked as easy (`'easy_tag' == 1`) but take over 1 hour to make (`'easy_hours' == 0`) have the lowest average ratings and requires the most time to make, on average.
- Recipes that are marked as hard (`'easy_tag' == 0`) but can be quickly made (`'easy_hours' == 1`) have a higher average rating than recipes marked as hard (`'easy_tag' == 0`) and take a long time to make (`'easy_hour' == 0`).

|    |   easy_tag |   easy_hour |   rating |   n_steps | n_ingredients |    hours |
|---:|-----------:|------------:|---------:|----------:|--------------:|---------:|
|  0 |          0 |           0 | 4.660219 | 14.926542 |     11.759875 | 3.740897 |
|  1 |          0 |           1 | 4.679985 | 11.547602 |     10.355560 | 0.587478 |
|  2 |          1 |           0 | 4.655434 | 10.244364 |      8.968515 | 7.026235 |
|  3 |          1 |           1 | 4.690641 |  7.669675 |      7.567070 | 0.412299 |

These tables present us with a possible claim that easier recipes may tend to earn higher ratings than hard recipes. We will test this later.

## Assessment of Missingness
There are 3 columns (`'description'`, `'rating'`, and `'reviews'`) in the original merged dataset that contains a significant amount of missing values. Let's investigate the missingness of these columns.

### NMAR Analysis
The missingness of the `'review'` column is possibly `NMAR` because reviews may generally take a longer amount of time to write in comparison to clicking a button to rate a recipe. If we were to collect data on how much time someone had on their hands to rate/review a recipe, we may find that people tighter on time 
may choose to not include a review and just rate the recipe, thus making the missingness of the review column `MAR`.

### Missingness Dependency
To test the missingness in the `'rating'` column, let's perform some permutation test (on the original merged DataFrame) to decide whether it is `MCAR` or `MAR`.

To test the dependency of missingness of `'rating'` on `'minutes'`:
> - H0: There *IS NO* difference between the distribution of minutes when `'rating'` is missing/not missing.
> - H1: There *IS A* difference between the distribution of minutes when `'rating'` is missing/not missing.
> - Test Statistic: absolute difference in mean minutes between the missing group and the not missing group
> - Significance Level: 0.05

<iframe
  src="assets/MCAR_minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

In our permutation test, we obtained a **p-value of 0.232**. Since our p-value of 0.232 is greater than our significance level of 0.05, we **fail to reject the null**. This indicates that the missingness of the `'rating'` column is most likely not dependent on the `'minutes'` column, meaning it is `MCAR`.

Next, to test the dependency of missingness of `'rating'` on `'n_ingredients'`:
> - H0: There *IS NO* difference between the distribution of the number of ingredients when `'rating'` is missing/not missing.
> - H1: There *IS A* difference between the distribution of the number of ingredients when `'rating'` is missing/not missing.
> - Test Statistic: absolute difference in mean number of ingredients between the missing group and the not missing group
> - Significance Level: 0.05

<iframe
  src="assets/MAR_n_ingredients.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

In this permutation test, we obtained a **p-value of 0.0**. Our p-value of 0.0 is less than our significance level of 0.05 so we will **reject the null**. It is likely that the missingness of the `'rating'` column is likely dependent on the `'n_ingredients'` column, meaning it is `MAR`.

## Hypothesis Testing
To answer our driving question `'Does a recipe's level of difficulty affect user ratings?'`, we can observe the difference in ratings between easy and hard recipes. To determine whether a recipe is easy or not, time is one indication. In our `'easy_hour'` column, we assigned each recipe `1` in the `'easy_hour'` column if it could be completed within 1 hour, else `0`. In the grouped table above, we can see that there is a difference in the average ratings of recipes in the `'easy_hour' == 0` and `'easy_hour' == 1` groups. How significant is this difference? Was it by chance?
-   Average rating of `'easy_hour' == 0` = 4.658025
-   Average rating of `'easy_hour' == 1` = 4.686616

Let's perform a permutation test to answer our questions.
> - H0: There is no difference between the average rating when `'easy_hour' == 0` and when `'easy_hour' == 1`.
> - H1: The average rating of recipes when `'easy_hour' == 1` is greater than the average rating of recipes when `'easy_hour' == 0`.
> - Test Statistic: average rating of 'easy' recipes `-` average rating of 'non-easy' recipes
> - Observed Statistic: 4.686616 - 4.658025 = `0.028591`
> - Significance Level: 0.05

<iframe
  src="assets/diff_avg_ratings_easy_hard.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

In the permutation above, we obtained a **p-value of 0.00**, meaning the probability of observing a test statistic at least as extreme as our observed statistic is 0.00. Since the p-value is less than the significance level of 0.05, we would **reject the null**. This means the difference in average ratings between easy and non-easy recipes were most likely not by chance. The alternative hypothesis proposes that one possible explanation of the difference is because recipes easier to make tend to earn a higher rating than non-easy recipes since people are more likely to be successful in its production.

## Framing a Prediction Problem
Let's predict the average rating of a recipe and treat this as a regression problem since average ratings are continuous. 

> For our prediction model:
> - Response variable: `'avg_rating'`
> - Features: will be chosen from the information we have about recipes (will not choose from variables in interactions dataset)
> - Metric: R²

`'avg_rating'` is chosen as the response variable because it is a good overall representation of the ratings of a recipe. Also, I want to build a prediction model that can allow the author of a recipe to estimate the average rating of their about-to-be-posted recipes so they can get a glimpse of how favorable their recipe would be, and whether anything needs to be modified, before posting it, in order to receive higher ratings. For the purpose of the model, I will choose features only within the original `'recipe'` dataset since all the information `'recipe'` should be known at the time of prediction. Therefore, columns like `'rating'` and `'review` would not be included in our model since this information would be unknown. Since this is a regression model, the metric I will use to evaluate my model performance is R² because it will calculate the proportion of variance in the prediction variable explained by the model.

## Baseline Model
Let's create a simple baseline model with `'hours'`, `'n_steps'`, and `'n_ingredients'` as our features, and using a `DecisionTreeRegressor` as our estimator. Since all 3 features are quantitative, we do not need to perform any transformations and we can just leave them as is. 

The **R² score** of the model on the test set is about **0.040418**. This means our model performs extremely poor in predicting our response variable `'avg_rating'` since our R² is very close to 0. This is likely because there is a lot of clustering in our data. Most recipes can be made under 1 hour, as seen in our univariate analysis above, which is normal since you wouldn't expect most dishes to take too long to prepare in reality. Also from the bivariate analysis, we can see that a high variance exists for the number of ingredients, which introduces uncertainty into our model during the training process.


## Final Model
In the final model, I included 5 more features, `'name'`, `'tags'`, `'steps'`, `'easy_tag'`, `'easy_hour'`. 

`'name'`

Certain foods may be more popular so ratings might be higher for those recipes (Ex: lots of people like chocolate so if 'chocolate' is an important word in a recipe's name, ratings may perhaps be higher). Therefore, I transformed the column using `TfidfVectorizer` to calculate the importance of each word in the recipe name.

`'tags'` & `'steps'`

These columns may contain keywords that may affect user liking of a recipe (Ex: there are certain steps in cooking that are harder so if a recipe contains harder steps, the rating of the recipe may be lower). Therefore, I performed bagging on the tags using `CountVectorizer`.

`'easy_tag'` & `'easy_hour'`

These binary values can indicate whether a recipe is easy or not. As we saw earlier in our grouped tables, the mean rating of `'easy_tag' == 1` and `'easy_hour' == 1` were both higher than when the columns took on the value 0. We also ran a permutation test and found a p-value of **0.00** (shown above), indicating that it is highly likely that easy recipes recieve higher average ratings. Therefore, I used `OneHotEncoder` with the `drop = 'first'` parameter to transform the columns and avoid multicollinearity.

> After transforming our features, I performed a `GridSearchCV` to tune the hyperparameters. Because we are using a `DecisionTreeRegressor` as our prediction model, to prevent overfitting the training data and allow generalization, we will tune the `'max_depth'` and `'min_samples_split'` hyperparameters so we ensure that when the model is fitting to our training set, it will not continue splitting until all leaf nodes are pure. This prevents our model from overfitting to our training set. The best-performing hyperparameters were `'max_depth' = 22` and `'min_samples_split' = 835`. After fitting our tuned model on the training set, we now obtain a **prediction score of 0.049802** on the test set (same test set as baseline model). Although this is still poor performance, there is approximately a **0.009384 increase in R²**.

## Fairness Assessment
Let's test the fairness of our model on easy and hard recipes. Easy recipes are defined as recipes with `'easy_tag' == 1`, while hard recipes are recipes with `'easy_tag' == 0`. I want to see if our model can predict fairly on these 2 groups and we will do so by evaluating the absolute difference in R².

> - H0: Our model is fair. The R² for hard and easy recipes are roughly the same, and any differences are due to random chance.
> - H1: Our model is not fair. Its R² for hard recipes is different from the R² of easy recipes.
> - Test Statistic: Absolute difference in R² (|new - old|)
> - Significance Level: 0.05

<iframe
  src="assets/fairness_analysis.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We obtained a **p-value of 0.057** which is greater than the significance level at 0.05 so we would **fail to reject** the null. It is likely that the difference in R² between the 2 groups was caused by chance. We should conclude that the model performs fairly for both easy and hard recipes.