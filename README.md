# [Do Healthier Recipes Take Longer? An Analysis of Fitness-Oriented Nutrition and Preparation Time](https://trstewart-max.github.io/nutrition-prep-time/)

## Introduction

A common belief is that eating healthy meals, especially high-protein dishes, takes more time than cooking regular meals. Because of this perception, many people avoid meal prepping since they believe it is too time-consuming or inconvenient.

This project investigates whether that belief is actually supported by data.

Specifically, we explore whether meals that are higher in protein and more nutrition-focused take longer to prepare than other types of meals. This question matters because time is one of the most frequently cited barriers to eating healthier or maintaining a fitness-oriented diet. If healthier meals do not take substantially longer to prepare, this challenges the idea that meal prepping is too time-intensive and suggests that eating well may be more accessible than people think.

The first dataset, `recipe`, contains **83,782 rows**, representing 83,782 unique recipes, with several columns describing recipe information.

| Column | Description |
|--------|-------------|
| `name` | Recipe name |
| `id` | Recipe ID |
| `minutes` | Minutes required to prepare the recipe |
| `contributor_id` | User ID who submitted the recipe |
| `submitted` | Date the recipe was submitted |
| `tags` | Food.com tags describing the recipe (e.g., dinner, dessert, low-fat, easy) |
| `nutrition` | Nutrition information in the format [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)] |
| `n_steps` | Number of preparation steps |
| `steps` | Text describing the recipe steps |
| `description` | User-provided description |
| `ingredients` | Text describing the ingredients |
| `n_ingredients` | Number of ingredients in the recipe |

The second dataset, `interactions`, contains **731,927 rows**, where each row represents a user interaction with a recipe.

| Column | Description |
|--------|-------------|
| `user_id` | User ID |
| `recipe_id` | Recipe ID |
| `date` | Date of the interaction |
| `rating` | Rating given by the user |
| `review` | Text review written by the user |

To answer our research question, we focus on the following relevant columns:

| Column | Description |
|--------|-------------|
| `minutes` | Total time required to prepare the recipe |
| `nutrition` | Nutrition information in the format [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)] |
| `n_steps` | Number of preparation steps required |

Given the datasets, we investigate whether recipes that align more closely with fitness-oriented nutrition differ in preparation time compared to less nutritious recipes. To facilitate this analysis, we first separated the values stored in the `nutrition` column into their corresponding columns, including `calories (#)`, `total fat (PDV)`, `sugar (PDV)`, `protein (PDV)`, and others. PDV, or percent daily value, represents how much a nutrient in a serving of food contributes to the recommended daily intake.

Using these nutritional variables, we constructed a measure of recipe “fitness” based on the difference between protein and fat daily value percentages (`protein_pdv - total_fat_pdv`). Recipes with relatively higher protein content and lower fat content were categorized as **High Fitness**, while the remaining recipes were categorized as **Low Fitness**. This grouping allows us to compare preparation times between recipes that are more aligned with fitness-oriented nutrition and those that are not.

The most relevant columns for answering our question include `minutes`, which records the total preparation time of each recipe, `protein_pdv` and `fat_pdv`, which represent the nutritional composition of the recipe, and `n_steps` and `n_ingredients`, which capture the structural complexity of the recipe.


## Data Cleaning

To prepare the dataset for analysis, we performed several data cleaning and transformation steps. First, we merged the `recipes` and `interactions` datasets so that each recipe contains both its nutritional information and user interaction data such as ratings. The merge was performed using the recipe ID as the key.

Next, we replaced ratings of `0` with missing values (`NaN`). In this dataset, a rating of zero does not represent an actual rating but instead indicates that a user did not leave a rating. Treating these values as missing prevents them from artificially lowering average rating calculations.

Because each recipe can receive multiple ratings, we then calculated the **average rating for each recipe** by grouping by recipe ID and computing the mean rating across users. This value was stored in a new column called `avg_rating`.

The original dataset stores nutrition information as a list inside the `nutrition` column. To make these values usable for analysis, we separated this column into individual numeric columns representing each nutrient:

- calories  
- total fat (PDV)  
- sugar (PDV)  
- sodium (PDV)  
- protein (PDV)  
- saturated fat (PDV)  
- carbohydrates (PDV)  

PDV (Percent Daily Value) represents how much a nutrient contributes to the recommended daily intake.

To reduce the influence of extreme outliers, we removed recipes with calorie values greater than **2000 calories**, as these likely represent bulk mixtures or ingredient blends rather than individual recipes. We also removed recipes with preparation times greater than **600 minutes** or less than or equal to zero.

Finally, we constructed a **fitness score** to measure how aligned a recipe is with fitness-oriented nutrition: recipes with relatively higher protein and lower fat content receive higher scores.

To simplify comparisons, recipes were divided into two groups based on the **median fitness score**:

- **High Fitness** – recipes with scores above the median  
- **Low Fitness** – recipes with scores below the median  

---

## Distribution of Preparation Time

<iframe src="images/univariate1.html" width="900" height="500"></iframe>

The distribution of preparation time is strongly **right-skewed**, meaning most recipes require relatively short preparation times while a smaller number take much longer. The majority of recipes fall below approximately 100 minutes, though a long tail extends toward more complex recipes with longer preparation times. This pattern supports our decision to remove extreme outliers above 600 minutes during the cleaning process.

---

## Distribution of Fitness Score

<iframe src="images/univariate2.html" width="900" height="500"></iframe>

The distribution of the fitness score is centered around zero with both positive and negative values. Recipes with higher scores tend to have relatively higher protein content compared to fat, while negative scores indicate recipes with relatively higher fat content. This distribution allows us to divide recipes into **High Fitness** and **Low Fitness** groups for further analysis.

---

## Fitness Score vs Preparation Time

<iframe src="images/bivariate1.html" width="900" height="500"></iframe>

This scatter plot shows the relationship between recipe fitness score and preparation time. Although preparation time varies widely across recipes, there appears to be a slight tendency for recipes with higher fitness scores to have longer preparation times. However, the relationship is not strongly linear, suggesting that other factors such as recipe complexity may also influence preparation time.

---

## Average Preparation Time by Fitness Alignment

<iframe src="images/bivariate2.html" width="900" height="500"></iframe>

This bar chart compares the average preparation time between **High Fitness** and **Low Fitness** recipes. Recipes categorized as high fitness appear to take longer on average than low fitness recipes. This pattern suggests that healthier recipes may require more preparation effort, possibly due to additional ingredients or more involved cooking techniques.

---

## Interesting Aggregates

To further explore how recipe complexity influences preparation time, we created a pivot table that groups recipes by both **fitness alignment** and **number of preparation steps**. Recipes were divided into step-count categories ranging from very few steps to many steps. The table reports the **mean, median, and count** of preparation times within each group.

| Fitness Group | Step Group | Mean Minutes | Median Minutes | Count |
|---|---|---|---|---|
| High Fitness | Very Few | 67.39 | 15 | 10291 |
| High Fitness | Few | 67.91 | 30 | 21448 |
| High Fitness | Moderate | 66.20 | 40 | 45343 |
| High Fitness | Many | 77.11 | 50 | 28554 |
| Low Fitness | Very Few | 23.55 | 10 | 16737 |
| Low Fitness | Few | 36.88 | 20 | 28727 |
| Low Fitness | Moderate | 47.12 | 35 | 48648 |
| Low Fitness | Many | 66.48 | 50 | 26038 |

From this table we observe that recipes with **more preparation steps generally require longer preparation times**, which is expected since additional steps typically correspond to more complex cooking processes. However, even within the same step category, **High Fitness recipes consistently require more preparation time than Low Fitness recipes**. This suggests that healthier recipes may require additional preparation effort not fully captured by step count alone.


## Assessment of Missingness

Three columns — `date`, `rating`, and `review` — in the merged dataset contain a substantial amount of missing values. Because these variables capture user interaction with recipes, we assessed whether their missingness may be related to characteristics of the recipes themselves.

### MNAR Analysis

We believe that the missingness of the `review` column may be MNAR. Users are less likely to leave a review if they feel neutral or indifferent about a recipe, since writing a review requires time and effort. In contrast, users who feel strongly about a recipe — either positively or negatively — are more likely to share their experiences. This suggests that the likelihood of a review being missing depends on the user’s underlying opinion of the recipe, which is unobserved when no review is written.

### Missingness Dependency

We next examined the missingness of `rating` in the merged DataFrame by testing whether its missingness depends on other variables in the dataset. Specifically, we investigated whether the missingness of `rating` depends on `protein_pdv`, which represents the protein percentage daily value of the recipe, or on a randomly generated column `random_col`.

---

### Protein PDV and Rating

**Null Hypothesis:** The missingness of `rating` does not depend on the protein percentage daily value of the recipe.

**Alternative Hypothesis:** The missingness of `rating` does depend on the protein percentage daily value of the recipe.

**Test Statistic:** The absolute difference in mean `protein_pdv` between the distribution of recipes with missing ratings and the distribution of recipes without missing ratings.

**Significance Level:** 0.05

<iframe src="images/missingness_protein_kde.html" width="100%" height="550"></iframe>

We ran a permutation test by shuffling the missingness of `rating` 1000 times to generate simulated differences in the mean protein PDV between the two groups.

<iframe src="images/perm_missing_rating_protein.html" width="100%" height="550"></iframe>

The observed statistic is indicated by the red vertical line on the permutation distribution. Since the p-value we obtained (0.0) is less than the significance level of 0.05, we reject the null hypothesis. This suggests that the missingness of `rating` depends on `protein_pdv`.

---

### Random Column and Rating

**Null Hypothesis:** The missingness of `rating` does not depend on `random_col`.

**Alternative Hypothesis:** The missingness of `rating` does depend on `random_col`.

**Test Statistic:** The absolute difference in mean `random_col` between recipes with missing ratings and recipes without missing ratings.

**Significance Level:** 0.05

<iframe src="images/perm_missing_rating_random.html" width="100%" height="550" frameborder="0"></iframe>

We ran another permutation test by randomly shuffling the missingness of `rating` 1000 times to simulate the distribution of mean differences between the two groups.

The observed statistic is indicated by the red vertical line on the permutation distribution. Since the p-value we obtained (0.761) is greater than 0.05, we fail to reject the null hypothesis. This suggests that the missingness of `rating` does not depend on `random_col`, which is consistent with what we would expect from a randomly generated variable.

---

Overall, these results suggest that the missingness of ratings is not completely random. Instead, it appears to depend on certain recipe characteristics such as protein content, indicating that the data are unlikely to be Missing Completely At Random (MCAR).




## Hypothesis Testing
We aim to analyze the relationship between fitness-oriented recipes and their preparation time. Specifically, we investigate whether recipes with higher nutritional alignment tend to require more time to prepare.

To determine whether a recipe is fitness-oriented, we define a fitness score using **`protein_pdv`** and **`total_fat_pdv`** as:

`fitness_score = protein_pdv - total_fat_pdv`

Recipes with a fitness score above the mean are classified as high-fitness recipes, while those below the mean are classified as low-fitness recipes. We analyze preparation time using the **`minutes`** column.

We performed a permutation test to examine whether recipes that are more fitness-oriented tend to take longer to prepare. A permutation test is appropriate because it allows us to compare **`minutes`** between two groups without making assumptions about the underlying distribution.


### Hypotheses

- Null Hypothesis (H₀): There is no relationship between fitness-oriented recipes and **`minutes`**.
- Alternative Hypothesis (H₁): Fitness-oriented recipes require more **`minutes`** to prepare.

---

### Test Statistic

We use the **difference in mean `minutes`** between high-fitness and low-fitness recipes:

`mean(minutes | high fitness) − mean(minutes | low fitness)`

The mean is appropriate here because we are comparing the average preparation time between the two groups, which directly reflects our question of whether one group tends to take longer than the other.

---

### Methodology

1. Compute the observed difference in mean **`minutes`** between the two groups.
2. Randomly shuffle the **`fitness_binary`** labels across recipes.
3. Recompute the difference in means for each shuffle.
4. Repeat this process 10,000 times to generate a null distribution.
5. Calculate the p-value as the proportion of simulated statistics greater than or equal to the observed statistic.

---

### Results

- Observed statistic: 27.504
- p-value: approximately 0.0001
- Significance level: 0.05
<iframe
  src="images/fitness-permutation-plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

---

### Conclusion

Since the p-value is less than 0.05, we reject the null hypothesis. This suggests that the average preparation time (**`minutes`**) differs between high-fitness and low-fitness recipes.

Specifically, high-fitness recipes tend to take longer to prepare on average.

---

### Interpretation

One possible explanation is that recipes emphasizing higher **`protein_pdv`** and lower **`total_fat_pdv`** may require more ingredients or more complex preparation steps, leading to longer average cooking times.

In particular, recipes with higher fitness scores tend to require longer preparation times.



## Framing a Prediction Problem

We aim to predict the preparation time of a recipe, measured in minutes. This is a **regression problem** because the response variable, `minutes`, is continuous. Predicting preparation time is useful for understanding whether fitness-oriented recipes require more effort compared to less nutritious meals, which directly connects to our project’s central question.

We selected preparation time as our response variable because it reflects the effort required to cook a recipe and is a key factor influencing whether individuals choose to prepare healthier meals. By predicting preparation time from recipe characteristics, we can better understand how nutrition relates to convenience.

At the time of prediction, we assume access to features that are known before a recipe is prepared. These include nutritional attributes such as `calories`, `protein_pdv`, and `fat_pdv`, as well as structural features like the number of steps (`n_steps`). These variables describe the recipe itself and are available prior to any user interaction.

We do not use variables such as ratings or reviews, since these are only observed after users have cooked and evaluated the recipe. Including such variables would introduce information that is not available at prediction time and would lead to data leakage.

To evaluate our model, we use **Root Mean Squared Error (RMSE)**. RMSE measures the typical magnitude of prediction errors, with larger RMSE's indicating worse predictors, making it appropriate for assessing how accurately our model predicts preparation time.

## Baseline Model

For our baseline model, we used a **Linear Regression** model to predict the preparation time of a recipe, measured by `minutes`. Since `minutes` is a continuous variable, this is a **regression problem**.

The features used in our baseline model were `fitness_group`, `protein_pdv`, `total_fat_pdv`, and `n_steps`. The variable `fitness_group` is categorical, while the other features are quantitative. We one-hot encoded `fitness_group` and left the quantitative features unchanged.

We implemented the model using a **scikit-learn Pipeline**, which first preprocesses the data and then fits the Linear Regression model. We split the dataset into training and testing sets so that we could evaluate how well the model generalizes to unseen data.

To evaluate performance, we used **Root Mean Squared Error (RMSE)**. RMSE measures the typical size of prediction errors and penalizes larger errors more heavily, making it appropriate for a regression problem like ours.

Our baseline model achieved an RMSE of **76.63 minutes** on the test set. This means that, on average, the model’s predictions differ from the true preparation times by about 76 minutes.

While this baseline model provides a reasonable starting point, the relatively large RMSE suggests that it does not fully capture the factors that influence recipe preparation time. In the final model, we aim to improve performance by adding engineered features and introducing regularization.

## Final Model

To improve upon our baseline model, we engineered two additional features: `steps_per_ingredient` and `calories_per_ingredient`.

The feature `steps_per_ingredient` measures recipe complexity relative to the number of ingredients. We believed this would be useful because two recipes may have the same number of steps, but the one with fewer ingredients may require more effort per ingredient. The feature `calories_per_ingredient` captures caloric density at the ingredient level, which may help the model distinguish between simpler and more involved recipes.

Our final model used the features `fitness_group`, `protein_pdv`, `total_fat_pdv`, `n_steps`, `n_ingredients`, `steps_per_ingredient`, and `calories_per_ingredient`. As in the baseline model, `fitness_group` was one-hot encoded. All numerical features were standardized using **StandardScaler** so that they would be on comparable scales.

For the modeling algorithm, we used **Ridge Regression**. We chose Ridge because it adds regularization, which helps reduce overfitting and improve generalization when using multiple correlated numerical features. To tune the model, we performed **GridSearchCV** over the hyperparameter `alpha`, testing the values 0.01, 0.1, 1, 10, and 100 with 5-fold cross-validation.

The best value of `alpha` was selected based on cross-validated performance using RMSE.

<iframe src="./images/final_model_predictions.html" width="100%" height="600" style="border:none;"></iframe>

The plot above compares the model’s predicted preparation times to the actual preparation times. If the model were perfectly accurate, the points would lie on the red diagonal line. Instead, while the model captures general trends, many points fall away from that line, especially for recipes with very large preparation times. This suggests that the model performs better for more typical recipes than for extreme outliers.

Our final model achieved an RMSE of **76.36 minutes** on the test set. Compared to the baseline RMSE of **76.63 minutes**, this indicates that the final model improved predictive performance.

Overall, the final model performs better than the baseline because the engineered features better capture recipe complexity and the Ridge regularization helps the model generalize more effectively to unseen data.

## Fairness Analysis

To evaluate whether our model performs equally well across different types of recipes, we conducted a fairness analysis. Specifically, we examined whether the model predicts preparation time more accurately for one fitness group than the other.

We defined the two groups as:
- **Group X:** Recipes with High Fitness scores  
- **Group Y:** Recipes with Low Fitness scores  

Our evaluation metric was **Root Mean Squared Error (RMSE)**, since our model is a regression model predicting preparation time in minutes.

**Null Hypothesis:** The model is fair. The RMSE for high-fitness recipes and low-fitness recipes is approximately the same, and any observed difference is due to random chance.  

**Alternative Hypothesis:** The model is not fair. The RMSE differs between high-fitness recipes and low-fitness recipes.  

**Test Statistic:** The absolute difference in RMSE between the high-fitness and low-fitness recipe groups.  

**Significance Level:** 0.05  

<iframe src="./images/fairness_permutation.html" width="100%" height="550" style="border:none;"></iframe>

We performed a permutation test by randomly shuffling the fitness group labels 1000 times and computing the difference in RMSE between the two groups for each permutation. This created a null distribution representing the differences we would expect if the model performed equally well for both groups.

The observed RMSE difference was **37.71**, which is indicated by the red vertical line on the graph. Since the p-value (**0.0**) is less than the significance level of 0.05, we reject the null hypothesis.

This result suggests that the model’s prediction error differs significantly between high-fitness and low-fitness recipes. In other words, the model performs better for one fitness group than the other, indicating a potential fairness concern.

Our project aimed to understand differences in preparation time between recipes that align more closely with fitness-oriented nutrition and those that do not. While our model is able to estimate preparation time using nutritional and recipe structure features, the fairness analysis shows that it does not perform equally well across the two groups. This suggests that preparation time may depend on factors that differ between fitness groups in ways the model does not fully capture. As a result, predictions for one group may be less reliable than for the other, highlighting an important limitation of our model.
