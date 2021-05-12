---
date:
description: ""
featured_image: "/images/brand_pref_feature.webp"
title: "Brand Preference Analysis with R"
show_reading_time: true
---

This was my first project using R. So the start of the project involved installing and setting up R and R Studio, and familarizing myself with how to install/load packages, setup version control (using renv), and performing basic analysis and modeling tasks in the R ecosystem.

{{< figure src="/images/cust_brand_pref_rstudio_dataprep.png" title="Screenshot of R Studio Session" >}}


This is a 2 class classification problem.


```R
# LOAD PKGS & DATA
library(tidyverse)
library(here)
library(janitor)
library(skimr)
library(generics)
#ran in console--install.packages("caret", dependencies = c("Depends", "Suggests"))
library(caret)
#library(doSNOW) #enables doing training in parallel
```

```r
#read in dataset
surveydata <- read.csv(here("data", "CompleteResponses.csv"))
```




# Data Wrangling

```r
#rename vars
cln_surveydata <- surveydata %>%
  clean_names() %>%
  rename(region = zipcode) %>% # new name = old name
  rename(edu = elevel)

#verify var name changes
glimpse(cln_surveydata)
```

```r
#change predictor vars to factor dtype
cln_surveydata$edu <- as.factor(cln_surveydata$edu)
cln_surveydata$car <- as.factor(cln_surveydata$car)
cln_surveydata$region <- as.factor(cln_surveydata$region)

#change target var to factor--so treated as classification
cln_surveydata$brand <- as.factor(cln_surveydata$brand)

#verify changes
is.factor(cln_surveydata$brand)
```




## Variable distributions

Use ```hist()``` for continuous variables and ```barplot(table())``` for categorical variables.

```r
#plot distributions of continuous predictor vars
hist(cln_surveydata$salary)
hist(cln_surveydata$age)
hist(cln_surveydata$credit)
```

All variables are aprox. uniformly distributed.

{{< figure src="/images/brand_pref_age_salary_credit_hist.png" >}}


```r
#plot distributions of categorical predictor vars
barplot(table(cln_surveydata$edu),
        main = "Education Level",
        ylab = "Count")
barplot(table(cln_surveydata$car),
        main = "Car Type",
        ylab = "Count")
barplot(table(cln_surveydata$region),
        main = "Region",
        ylab = "Count")
```

{{< figure src="/images/brand_pref_edu_car_region_bar.png" >}}

```r
#plot distribution of obs in target var
barplot(table(cln_surveydata$brand))
```

{{< figure src="/images/brand_pref_brand_bar.png" >}}

```r
#get value counts for brand classes
cln_surveydata %>% count(brand)
# class 0 is 37.82582%
```

The classes are not balanced: brand 1 is preferred by more customers. As a result, during the modeling phase I will need to use stratified splitting/sampling for creating my training and testing datasets. The presence of a class imbalance will also be important when it comes to interpreting my model's performance measures  



# Modeling

```r
#set random number generator
set.seed(123)

#specify partition of 75/25 (p = .75), list = FALSE to output vector, not list
in_train <- createDataPartition(cln_surveydata$brand,
                                p = .75,
                                list = FALSE)

#use partition to create training dataset--[intraining,] tells R to use all columns when creating the training data
train_data <- cln_surveydata[in_train,]

#use partition to create testing dataset
test_data <- cln_surveydata[-in_train,]
```

*Coding Notes*
- ```set.seed()``` in R does the same job as it does in Python. It ensures that when the data is split, the same observations are selected to make up the training and testing sets every time the code is run, which is necessary for reproducibility



#### Parallel processing
When I began modeling this dataset, my training and testing steps were taking several minutes each to complete—that made the work frustrating, and limited how much experimentation I could do in a limited amount of time. So I researched how to implement parallel processing.

```r
#create cluster
pcluster <- makeCluster(4, type = "SOCK")

#register cluster
#registerDoSNOW(pcluster)
```




##  Random Forest Model


```r
# use 5-fold cross validation method
control <- trainControl(method = "repeatedcv",
                        number = 10,  
                        repeats = 1)

#create parameter tuning grid (FOR MANUAL TUNING ONLY)
#grid <- expand.grid(size = c(2,7,12,18,24,34), k = c(1,2,3,4,5))
```

```r
#train & auto-tune model
system.time(rf_fit1 <- train(brand~.,
                             data = train_data,
                             method = "rf",
                             trControl = control,
                             tunelength = 5))
```

*Coding Notes*
- Use the ```system.time()``` wrapper to get runtime for code execution.
- Runtime with cluster: 321.69, without cluster: 804.95


```r
#train & manually tune model
system.time(rf_fit2 <- train(brand~.,
                             data = train_data,
                             method = "rf",
                             trControl = control,
                             tunelength = grid))
```



#### Model Predictions & Evaluation

```r
#predict brand preferences for testing_data
(pred_rf_fit1 <- predict(rf_fit1, test_data))
```


```r
#view confusion matrix
(perform_rf_fit1 <- confusionMatrix(pred_rf_fit1,
                                    test_data$brand))
```
---

## Gradient Boosting Model (GBM)

```r
#train  GBM (gradient boosting) model on the training_data, 3-fold crossval, w/ system.time wrapper
system.time(gbm_fit1 <- train(brand~.,
                              data = train_data,
                              method = "gbm",
                              trControl = control,
                              train.fraction = 0.5))
```

#### Model Predictions & Evaluation

```r
#make brand preferences predictions for testing_data
preds_gbm_fit1 <- predict(gbm_fit1,
                          test_data)
```


```r
#view confusion matrix of gbm model performance on test data
(perform_gbm_fit1 <- confusionMatrix(preds_gbm_fit1,
                                     test_data$brand))
```


The RF and GBM models perform similarly in predicting (i.e. correctly classifying) customers' brand preferences in the testing data. However, the GBM was significantly *faster*.


#### Variable Importance

```r
#calculate importance of predictor vars in RF model
varImp(rf_fit1)
```

Outcome: top 3 scores — salary 100, age 65, credit 9. After top 3, scores for remaining variables are below 1.


```r
# need to load gbm pkg to get varImp for GBM model  
library(gbm)

#calculate the importance of each feature/predictor var
varImp(gbm_fit1)
```

Outcome: top 3 predictors — age, salary, credit. Interestingly, the most important variable in the gbm model is different from the rf model. Not sure what to make of that.


##### Feature Selection

Because three variables show significantly more importance than the other vars, I want to look at the relationship between those vars and brand preference more closely.

```r
#plot brand pref and salary
cln_surveydata %>%
  ggplot(aes(x = brand, y = salary)) +
  geom_violin()

#plot brand pref and age
cln_surveydata %>%
  ggplot(aes(x = brand, y = age)) +
  geom_violin()

#plot brand pref and credit
cln_surveydata %>%
  ggplot(aes(x = brand, y = credit)) +
  geom_violin()
```


I wonder if all or most of the predictive power of gbm_fit1 is coming from the top 3 vars. So I want to check that


```r
# age only model
system.time(gbm_agefit <- train(brand~. -salary -edu -car -region -credit,
                                   data = train_data,
                                   method = "gbm",
                                   trControl = control))

# predict brand pref for test_data
(pred_gbm_agefit <- predict(gbm_agefit, test_data))

#evaluate performance using confusion matrix
(perform_gbm_agefit <- confusionMatrix(pred_gbm_agefit,
                                          test_data$brand))
```

The age only model had *zero* predictive power.


```r
# credit only model
system.time(gbm_creditfit <- train(brand~. -salary -edu -car -region -age,
                                data = train_data,
                                method = "gbm",
                                trControl = control))

# predict brand pref for test_data
(pred_gbm_creditfit <- predict(gbm_creditfit, test_data))

#evaluate performance using confusion matrix
(perform_gbm_creditfit <- confusionMatrix(pred_gbm_creditfit,
                                       test_data$brand))
```

The credit only model had *zero* predictive power

```r
# salary only model
system.time(gbm_salaryfit <- train(brand~. -age -edu -car -region -credit,
                             data = train_data,
                             method = "gbm",
                             trControl = control))

# predict brand pref for the test_data
(pred_gbm_salaryfit <- predict(gbm_salaryfit, test_data))

#evaluate performance of rf_salaryfit using confusion matrix
(perform_gbm_salaryfit <- confusionMatrix(pred_gbm_salaryfit,
                                       test_data$brand))
```

The salary only model performed considerably worse than the original GB model (accuracy = 0.72)

Conclusion: because of the outsized variable importance score for the salary variable, I thought all the predictive power of the original GBM model might be coming just from the salary variable. However, the salary only model had zero predictive power. So even though the salary variable is significantly more important than any of the other variables, without the other variables it accurately predict customers' brand preferences.  


```r
# AGE & CREDIT ONLY
system.time(gbm_agecredfit <- train(brand~. -edu -car -region -salary,
                                   data = train_data,
                                   method = "gbm",
                                   trControl = control))

#use gbm_salaryfit to predict brand pref for the test_data
(pred_gbm_agecredfit <- predict(gbm_agecredfit, test_data))

#evaluate performance of rf_salaryfit using confusion matrix
(perform_gbm_agecredfit <- confusionMatrix(pred_gbm_agecredfit,
                                          test_data$brand))
```
The model trained using only the age and credit variables  did not perform any better than the model's zero information rate (a model's zero information rate is equal to fraction of observations in the largest class, e.g. if 60% of the observations are in class A, and 40% are in class B, then a model can achieve 60% accuracy without any information/training by classifying all cases as A).


```r
#AGE, SALARY, AND CREDIT ONLY
system.time(gbm_agecredsalfit <- train(brand~. -edu -car -region,
                                    data = train_data,
                                    method = "gbm",
                                    trControl = control))

#use gbm_salaryfit to predict brand pref for the test_data
(pred_gbm_agecredsalfit <- predict(gbm_agecredsalfit, test_data))

#evaluate performance of rf_salaryfit using confusion matrix
(perform_gbm_agecredsalfit <- confusionMatrix(pred_gbm_agecredsalfit,
                                           test_data$brand))
```
This model using only these three variables performed slightly better than the full model 0.9297 vs. 0.9232. So I will use the model with fewer variables moving forward.





### Predict Brand Preferences for the Incomplete Survey DATA
[INSERT EPLANATION HERE]

```r
#read in incomplete survey data
incompletesurveydata <- read.csv(here("data", "SurveyIncomplete.csv"))

glimpse(incompletesurveydata)
```


```r
#clean var names and rename vars
incompletesurveydata <- incompletesurveydata %>%
  clean_names() %>%
  rename(edu = elevel) %>%    # new name = old name
  rename(region = zipcode)

glimpse(incompletesurveydata)    #verify changes

#check for missing values
sum(is.na(incompletesurveydata))
```
```r
#change var dtypes to factor
incompletesurveydata$edu <- as.factor(incompletesurveydata$edu)
incompletesurveydata$car <- as.factor(incompletesurveydata$car)
incompletesurveydata$region <- as.factor(incompletesurveydata$region)
incompletesurveydata$brand <- as.factor(incompletesurveydata$brand)

glimpse(incompletesurveydata)   # verify changes
```

```r
#use final model (gbm_agecredsalfit) to predict brand pref for entirely new dataset (incompletesurveydata)
pred_incompletedata <- predict(gbm_agecredsalfit,
                               newdata = incompletesurveydata)

#evaluate performance
(perform_incompletedata <- confusionMatrix(pred_incompletedata, incompletesurveydata$brand))
```

```r
#stop parallel processing
stopCluster(pcluster)

#get model info
gbm_agecredsalfit$modelInfo
```





[Link to Project GitHub Repository](https://github.com/kpiatti/customer-brand-preferences)
