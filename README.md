# Does a recipe's level of difficulty affect user ratings?
by Ada Mo

## Introduction
Cooking is a part of many people's daily lives but some dishes are easier to make while others are more difficult. Often times, the difficulty of a dish is associated with the time and effort put into the making of the dish, meaning a dish is more difficult to make when more time, ingredients, and skills are required. I am curious **how the level of difficulty can affect one's liking of the dish**. To investigate in the relationship, I will be analyzing from 2 datasets containing recipes and ratings since 2008 by food.com. ==include source==

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
2. Replace all 0s in the `'rating'` column with `np.nan`.
> A rating of 0 does not exist in the original website our data was collected from. All ratings should be from a scale of 1-5, with 1 being the lowest and 5 being the highest. A rating of 0 in the dataset indicates a missing rating in that specific interaction with a recipe, thus, it is more appropriate to replace all 0s with np.nan, else a numerical representation of missing values may introduce bias later on in our analysis.
3. Computed and included the average rating per recipe into our merged dataset.
> Each recipe can have multiple ratings from many different users so to have a better understanding of a recipe's overall rating, we will compute the average rating for each recipe.
4. Converted minutes to hours and stored in a new column `'hours'`
> A recipe's difficulty does not vary much with an additional minute but it for sure will with an additional hour, therefore, it is more appropriate to store the time it takes to make a recipe in units of hours.
5. Create `'easy_tag'` column which takes on the value of 1 if the recipe contains a `'easy'` or `'beginner-cook'` in the `'tag'` column, else 0
> If a dish has the `'easy'` or `'beginner-cook'` tag, the author of the recipe indicates it is an easy dish to make.
6. Create `'easy_hour'` column which takes on the value 1 if `'hours'` is <= 1 hour, else 0
> If a dish can be made within 1 hour, then we will classify it as an easy dish, thus label it with 1 in the new column.
7. Select and keep necessary columns.
> Some columns do not provide valuable information to answer our investigation question and will be meaningless to keep in our dataset. We will only keep the following columns: `'name'`, `'hours'`, `'tags'`, `'n_steps'`, `'steps'`, `'n_ingredients'`, `'easy_tag'`, `'easy_hour'`, `'rating'`, `'avg_rating'`
8. Sort each column and remove outliers.
> - There is a recipe with a oddly high time requirement to make the dish. It turns out the name of the recipe is `'how to preserve a husband'`, and includes steps on how to preserve a husband 💀. This obviously isn't a food dish so we will remove it from our dataset.
> - There is also another recipe whose `'name'` is `np.nan` and `'rating'` is `np.nan` even before step 2. This is quite odd since all other `rating` without values used `0` as a placeholder. Therefore, to prevent any future bias/confusion, we will remove this unnamed recipe from our dataset as well.
> - There is a recipe requiring 0 minutes to make but in the `'steps'` column, `['grind the almonds into a fine powder using a coffee , nut , or spice grinder']` suggests the recipe will take longer than 0 minutes to make.

Our dataset `recipe_ratings_needed` is now ready for use with 234423 rows of recipe reviews and 10 columns.
| Columns | Description |
| ----------- | ----------- |
| `'name'` | Recipe name |
| `'hours'` | Hours needed to prepare recipe |
| `'tags'` | Food.com tags for recipe |
| `'n_steps'` | Number of steps in recipe |
| `'steps'` | Text for recipe steps, in order |
| `'n_ingredients'` | Number of ingredients |
| `'easy_tag'` | Binary difficulty label based on recipe tags |
| `'easy_hour'` | Binary difficulty label based on recipe hours |
| `'rating'` | Rating given |
| `'avg_rating'` | Average rating of a recipe |



Here is a sneak peek of the dataset
|    | name                                 |    hours | tags                                                                                                                                                                                                                        |   n_steps | steps                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |   n_ingredients |   easy_tag |   easy_hour |   rating |   avg_rating |
|---:|:-------------------------------------|---------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------:|-----------:|------------:|---------:|-------------:|
|  0 | 1 brownies in the world    best ever | 0.666667 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'for-large-groups', 'desserts', 'lunch', 'snacks', 'cookies-and-brownies', 'chocolate', 'bar-cookies', 'brownies', 'number-of-servings'] |        10 | ['heat the oven to 350f and arrange the rack in the middle', 'line an 8-by-8-inch glass baking dish with aluminum foil', 'combine chocolate and butter in a medium saucepan and cook over medium-low heat , stirring frequently , until evenly melted', 'remove from heat and let cool to room temperature', 'combine eggs , sugar , cocoa powder , vanilla extract , espresso , and salt in a large bowl and briefly stir until just evenly incorporated', 'add cooled chocolate and mix until uniform in color', 'add flour and stir until just incorporated', 'transfer batter to the prepared baking dish', 'bake until a tester inserted in the center of the brownies comes out clean , about 25 to 30 minutes', 'remove from the oven and cool completely before cutting']                                                  |               9 |          0 |           1 |        4 |            4 |
|  1 | 1 in canada chocolate chip cookies   | 0.75     | ['60-minutes-or-less', 'time-to-make', 'cuisine', 'preparation', 'north-american', 'for-large-groups', 'canadian', 'british-columbian', 'number-of-servings']                                                               |        12 | ['pre-heat oven the 350 degrees f', 'in a mixing bowl , sift together the flours and baking powder', 'set aside', 'in another mixing bowl , blend together the sugars , margarine , and salt until light and fluffy', 'add the eggs , water , and vanilla to the margarine / sugar mixture and mix together until well combined', 'add in the flour mixture to the wet ingredients and blend until combined', 'scrape down the sides of the bowl and add the chocolate chips', 'mix until combined', 'scrape down the sides to the bowl again', 'using an ice cream scoop , scoop evenly rounded balls of dough and place of cookie sheet about 1 - 2 inches apart to allow for spreading during baking', 'bake for 10 - 15 minutes or until golden brown on the outside and soft & chewy in the center', 'serve hot and enjoy !'] |              11 |          0 |           1 |        5 |            5 |
|  2 | 412 broccoli casserole               | 0.666667 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']                                                                        |         6 | ['preheat oven to 350 degrees', 'spray a 2 quart baking dish with cooking spray , set aside', 'in a large bowl mix together broccoli , soup , one cup of cheese , garlic powder , pepper , salt , milk , 1 cup of french onions , and soy sauce', 'pour into baking dish , sprinkle remaining cheese over top', 'bake for 25 minutes or until cheese is lightly browned', 'sprinkle with rest of french fried onions and bake until onions are browned and cheese is bubbly , about 10 more minutes']                                                                                                                                                                                                                                                                                                                              |               9 |          1 |           1 |        5 |            5 |
|  3 | 412 broccoli casserole               | 0.666667 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']                                                                        |         6 | ['preheat oven to 350 degrees', 'spray a 2 quart baking dish with cooking spray , set aside', 'in a large bowl mix together broccoli , soup , one cup of cheese , garlic powder , pepper , salt , milk , 1 cup of french onions , and soy sauce', 'pour into baking dish , sprinkle remaining cheese over top', 'bake for 25 minutes or until cheese is lightly browned', 'sprinkle with rest of french fried onions and bake until onions are browned and cheese is bubbly , about 10 more minutes']                                                                                                                                                                                                                                                                                                                              |               9 |          1 |           1 |        5 |            5 |
|  4 | 412 broccoli casserole               | 0.666667 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']                                                                        |         6 | ['preheat oven to 350 degrees', 'spray a 2 quart baking dish with cooking spray , set aside', 'in a large bowl mix together broccoli , soup , one cup of cheese , garlic powder , pepper , salt , milk , 1 cup of french onions , and soy sauce', 'pour into baking dish , sprinkle remaining cheese over top', 'bake for 25 minutes or until cheese is lightly browned', 'sprinkle with rest of french fried onions and bake until onions are browned and cheese is bubbly , about 10 more minutes']                                                                                                                                                                                                                                                                                                                              |               9 |          1 |           1 |        5 |            5 |



### Univariate Analysis

Let's explore the distribution of hours used in each recipe. This graph tells us that only very few recipes take over 500 hours to make. This isn't very informative about our distribution.
<iframe
  src="assets/hours.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Let's zoom in. We can now see that our distribution is skewed right with a majority of our recipes requiring less than 1 hour to make.

<iframe
  src="assets/hours_zoomed.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


Now, let's explore the distribution of the number of ingredients used in each recipe. Below, we can see an almost-normal distribution but slightly skewed right. The mean number of ingredients across all recipes is approximately 9.21.
<iframe
  src="assets/n-ingredients.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


### Bivariate Analysis
Now let's explore the relationship between 2 variables in our data.

Here is a scatter plot showing the relationship between the number of steps in a recipe and the average 

**'(plotly viz)'**


### Interesting Aggregates

From the 2 grouped tables below, we can see that the recipes with an easy tag (`'easy_hour' == 1` or `'easy_tag' == 1`) tend to have a higher rating while requiring less steps and less ingredients
|   easy_hour |   rating |   n_steps |   n_ingredients |
|------------:|---------:|----------:|----------------:|
|           0 |  4.65803 |  12.7812  |        10.4809  |
|           1 |  4.68662 |   9.13779 |         8.62274 |

|   easy_tag |   rating |   n_steps |   n_ingredients |
|-----------:|---------:|----------:|----------------:|
|          0 |  4.67391 |  12.6056  |        10.7953  |
|          1 |  4.6841  |   8.15938 |         7.83362 |

Combining the 2 tables above, we can see more interesting patterns. 
- Recipes that are marked easy (`'easy_tag' == 1`) and can be quickly made (`'easy_hours' == 1`) have the highest average ratings. 
- Recipes that are falsely marked as easy (`'easy_tag' == 1`) but take over 1 hour to make (`'easy_hours' == 0`) have the lowest average ratings.
- Recipes that are marked as hard (`'easy_tag' == 0`) but can be quickly made (`'easy_hours' == 1`) have a higher average rating than recipes marked as hard (`'easy_tag' == 0`) and take a long time to make (`'easy_hour' == 0`).

|    |   easy_tag |   easy_hour |   rating |   n_steps |   n_ingredients |    hours |
|---:|-----------:|------------:|---------:|----------:|----------------:|---------:|
|  0 |          0 |           0 |  4.66022 |  14.9265  |        11.7599  | 3.7409   |
|  1 |          0 |           1 |  4.67999 |  11.5476  |        10.3556  | 0.587478 |
|  2 |          1 |           0 |  4.65543 |  10.2444  |         8.96852 | 7.02624  |
|  3 |          1 |           1 |  4.69064 |   7.66967 |         7.56707 | 0.412299 |

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

To test the dependency of missingness of `'rating'` on `'n_ingredients'`:
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
-   Average rating of `'easy_hour' == 0` = 4.65803
-   Average rating of `'easy_hour' == 1` = 4.68662

Let's perform a permutation test to answer our questions.
> - H0: There is no difference between the average rating when `'easy_hour' == 0` and when `'easy_hour' == 1`.
> - H1: The average rating of recipes when `'easy_hour' == 1` is greater than the average rating of recipes when `'easy_hour' == 0`.
> - Test Statistic: average rating of 'easy' recipes `-` average rating of 'non-easy' recipes
> - Observed Statistic: 4.68662 - 4.65803 = `0.02859`
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
> - Features: `'name'`, `'hours'`, `'tags'`, `'n_steps'`, `'steps'`, `'n_ingredients'`, `'easy_tag'`, `'easy_hour'`
> - Metric: R-squared

`'avg_rating'` is chosen as the response variable it is a good overall representation of the ratings of a recipe. Alse, I want to build a prediction model that can allow the author of a recipe to estimate the average rating of their about-to-be-posted recipes so they can get a glimpse of how favorable their recipe would be, and whether anything needs to be modified, before posting it. Therefore, I chose features that related to the recipe. The data in our selected features are all ready to be provided once a recipe is done being drafted and ready to be posted, and should be known at the time of prediction. The metric I will use to evaluate my model performance is R-squared because it will 



## Baseline Model

## Final Model

## Fairness Assessment