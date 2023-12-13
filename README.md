# Predicting Ratings

This is a project for DSC80 at UCSD. It is an analyses of a two datasets of recipes and reviews that intends to build a model to predict the rating out of 5 stars given in each review.

By: Quinn Brandt

---

# Framing the Problem

The goal of this project is to predict the **response variable: average rating**, of a given recipe using the information from the merged datasets. That is information about the recipe itself, aswell as the individual review of the recipe. This is a problem of **regression** rather than classification. While the review column is ordinal and thus would require a classification problem to predict, the average review column is continuous and can be predicted using a regression model.

I will be measuring the accuracy of my model using **Root Mean Square Error**. I chose this over Mean Square error becasue it preserves the units of the problem and makes the result easier to interpret. As this is not a classification problem, metrics such as accuracy, precision, recall, and F-1 score would not be helpful.

Due to the fact that I am predicting average rating for the recipe rather than the singular rating, **all the columns** in this dataset would be available at the time of prediction, because I would have needed to collect all the reviews. Despite this, I still had to drop the rating column because it was used to create the average rating column.

---

# Baseline Model

For my first model I only used the **minutes** and **calories** columns. In this case minutes is a **discrete** variable while calories is **continuous**. For this first model, I only used quantitative features as they were more prevelent in my dataset. 

In the preprocessing for this model, I made 2 feature transformations. I noticed that both minutes and calories were right skewed with long tails in their distributions. To deal with this I first **logarithmically scaled** both the features. Due to the fact they they are in different units I then chose to apply **StandardScaler** to standardize both of these features. I also used a **PolynomialFeatures** transformer in my pipeline with degree 3. I chose 3 as a placeholder primarily as I planned to refine my choice during my hyperparameter search.

After splitting the data into training and testing data, I fit my first model and tested it on the train and test sets. This resulted in a **RMSE of 0.643** for the training set and **0.699** for the testing set. For a first model I think this a relatively good. Seeing as I am predicting the average rating for the recipe using only a single review I think it is not that bad because each individual review is subjective and not everyone will share the same opinion about a recipe.

---

# Final Model

For my final model I added 4 quantitative features and 1 qualitative features. The first three quantitative features are **review_len**, **protein (PDV)**, and **total fat (PDV)**. Each of these are **discrete** variables. At first I thought protein and fat would be continous but upon inspection of the dataset there are only integer values. All three of these were **logarithmically scaled** then **standardized**. The fourth quantitative feature was **n_steps** which again is **discrete**. For this I wanted to seperate simple and complex recipes so I used a **Binarizer** to do so. The qualitative feature I used comes from a column called 'tone'. I created this column by seeing if the words 'good' and 'bad' were used in the review. Originally I had a 3rd value 'neutral' which I used for reviews that contained both words or neither word. However I noticed that the 'neutral' and 'good' reviews had almost the same average of the average_rating column so I combined them. I used **OneHotEncoder** to handle the two values in the 'tone' column.

I added the review_len features because I noticed that longer reviews were correlated with higher average ratings. As for the protein and total fat features, I belived that these were important metrics in a recipe as fat has a very negative stigma while protein has a very positive one. N_steps was added becasue I thought people would be more likely to leave a negative review if the recipe was very complex. Finally when exploring the 'tone' data I noticed a major discrepency between reviews that contained the word 'bad' and those that did not.

I explored hyperparameters by using a GridSearchCV. I used this to determine the best value for the threshold of my Binarizer, the degree of my PolynomialFeatures, and the log function I used to scale my data. The values that I tested and the best out of each are as follows:

| hyperparameter   | values              | best      |
|:-----------------|--------------------:|----------:|
| log function     | loge, log2, log10   | log 10    |
| threshold        | 3, 5, 7, 10, 15, 20 | 15        |
| degree           | 1, 2, 3             | 1         |

After inputting my best hyperparameters to my function I fit it to the training data. After testing it using RMSE once again I got a training error of **0.642** and a testing error of **0.647**. This is better than my first model by about 0.05 meaning on average, each estimation is 0.05 closer to the real value. Considering the fact that each review is subjective I think this performance is good. There is a lot of discrepancy and a recipe could have a horrible review and still have a high average rating so for this reason I think it is challenging to build a model that is much better.

---

# Fairness Analysis

For my fairness analysis I chose to compare the **good/neutral** values and the **bad** values of the 'tone' column. The reason I chose to do this is because they had such a significant difference in their corresponding distributions of the 'avg_rating' column that I wanted to explore how this impacted our predictions. Me evaluation was the RMSE of each individual group. 

**Null Hypothesis:** Our model is fair. Its RMSE for reviews marked good/neutral and reviews marked bad are roughly the same. Any differences are due to random chance.

**Alternative Hypothesis:** Our model is unfair. The RMSE for reviews marked good/neutral is lower than for reviews marked bad.

The test statistic I chose was the **difference in RMSE** (RMSE bad - RMSE good/neutral). I went forward with a significance level of 0.05. After running my permuatation test 500 times, I was left with a p-value of 0.0. I reject the null hypothesis that our model works just as well for both groups. 

<iframe src="assets/perm_test.html" width=800 height=600 frameBorder=0></iframe>
    
