Welcome to the second exciting part of the Language Development in ASD exercise
-------------------------------------------------------------------------------

In this exercise we will delve more in depth with different practices of
model comparison and model selection, by first evaluating your models
from last time against some new data. Does the model generalize well?
Then we will learn to do better by cross-validating models and
systematically compare them.

The questions to be answered (in a separate document) are: 1- Discuss
the differences in performance of your model in training and testing
data 2- Which individual differences should be included in a model that
maximizes your ability to explain/predict new data? 3- Predict a new
kid’s performance (Bernie) and discuss it against expected performance
of the two groups

Learning objectives
-------------------

-   Critically appraise the predictive framework (contrasted to the
    explanatory framework)
-   Learn the basics of machine learning workflows: training/testing,
    cross-validation, feature selections

Let’s go
--------

N.B. There are several datasets for this exercise, so pay attention to
which one you are using!

1.  The (training) dataset from last time (the awesome one you produced
    :-) ).
2.  The (test) datasets on which you can test the models from last time:

-   Demographic and clinical data:
    <a href="https://www.dropbox.com/s/ra99bdvm6fzay3g/demo_test.csv?dl=1" class="uri">https://www.dropbox.com/s/ra99bdvm6fzay3g/demo_test.csv?dl=1</a>
-   Utterance Length data:
    <a href="https://www.dropbox.com/s/uxtqqzl18nwxowq/LU_test.csv?dl=1" class="uri">https://www.dropbox.com/s/uxtqqzl18nwxowq/LU_test.csv?dl=1</a>
-   Word data:
    <a href="https://www.dropbox.com/s/1ces4hv8kh0stov/token_test.csv?dl=1" class="uri">https://www.dropbox.com/s/1ces4hv8kh0stov/token_test.csv?dl=1</a>

### Exercise 1) Testing model performance

How did your models from last time perform? In this exercise you have to
compare the results on the training data () and on the test data. Report
both of them. Compare them. Discuss why they are different.

-   recreate the models you chose last time (just write the model code
    again and apply it to your training data (from the first
    assignment))
-   calculate performance of the model on the training data: root mean
    square error is a good measure. (Tip: google the function rmse())
-   create the test dataset (apply the code from assignment 1 to clean
    up the 3 test datasets)
-   test the performance of the models on the test data (Tips: google
    the functions “predict()”)
-   optional: predictions are never certain, can you identify the
    uncertainty of the predictions? (e.g. google predictinterval())

\[HERE GOES YOUR ANSWER\]

### Exercise 2) Model Selection via Cross-validation (N.B: ChildMLU!)

One way to reduce bad surprises when testing a model on new data is to
train the model via cross-validation.

In this exercise you have to use cross-validation to calculate the
predictive error of your models and use this predictive error to select
the best possible model.

-   Use cross-validation to compare your model from last week with the
    basic model (Child MLU as a function of Time and Diagnosis, and
    don’t forget the random effects!)
-   (Tips): google the function “createFolds”; loop through each fold,
    train both models on the other folds and test them on the fold)

-   Now try to find the best possible predictive model of ChildMLU, that
    is, the one that produces the best cross-validated results.

-   Bonus Question 1: What is the effect of changing the number of
    folds? Can you plot RMSE as a function of number of folds?
-   Bonus Question 2: compare the cross-validated predictive error
    against the actual predictive error on the test data

``` r
#- Create the basic model of ChildMLU as a function of Time and Diagnosis (don't forget the random effects!).
basic <- lmer(CHI_MLU ~ Visit*Diagnosis + (1+Visit|Child.ID), data = df_train)
```

    ## Warning in checkConv(attr(opt, "derivs"), opt$par, ctrl =
    ## control$checkConv, : Model failed to converge with max|grad| = 0.00252572
    ## (tol = 0.002, component 1)

``` r
#- Make a cross-validated version of the model. (Tips: google the function "createFolds";  loop through each fold, train a model on the other folds and test it on the fold)

df_train <- subset(df_train, (!is.na(df_train$CHI_MLU))) #Removing the NAs 

folds <- createFolds(unique(df_train$Child.ID), k = 5, list = FALSE) # Creating 5 folds divided upon each subject 
fold_k <- 1 # we create a variable called fold_index

#we loop over each id in the vector of unique ids to get a fold number in a column and make sure that each child get their fold number consistently
for (id in unique(df_train$Child.ID)){
  df_train$fold[df_train$Child.ID == id] <- folds[fold_k] # creating the column called fold, which contains the fold number associated with each child id
  fold_k <- fold_k + 1
}
  
performances <- c() # We create an empty vector for the performances
#### loop for running cross validation ####
for (k in 1:max(df_train$fold)){ 
  fold_k <- as.integer(as.character(k)) # Convert k to integer 
  
  # assign test and training set according to the current folds
  train_df <- as.data.frame(df_train[df_train$fold != fold_k, ]) #training set without the current foldindex
  test_df <- as.data.frame(df_train[df_train$fold == fold_k, ]) #test set with only the current foldindex
  
# We create our model
  model <- lmer(CHI_MLU ~ Visit*Diagnosis + (1+Visit|Child.ID), data = train_df, REML =F, lmerControl(optCtrl=list(xtol_abs=1e-8, ftol_abs=1e-8)))

# Run our predictions 
  prediction <- predict(model, test_df, allow.new.levels =T)
  
#Calculate the rmse for each fold 
  rmse <- rmse(test_df$CHI_MLU, prediction)
  message(paste("RMSE for fold", fold_k, rmse)) # print RMSE for each fold to console
  performances[k] <- rmse # register rmse value to 'performances'vector
}
```

    ## RMSE for fold 1 0.635556453765192

    ## RMSE for fold 2 0.70652529329183

    ## RMSE for fold 3 0.778874655912837

    ## RMSE for fold 4 0.714581952506953

    ## RMSE for fold 5 0.961419823056665

``` r
# calculate mean RMSE and print it
performances = mean(performances)
performances
```

    ## [1] 0.7593916

``` r
# Bonus Question 1: What is the effect of changing the number of folds? Can you plot RMSE as a function of number of folds?
# Bonus Question 2: compare the cross-validated predictive error against the actual predictive error on the test data
```

``` r
### Cross validation of another model with verbal IQ###

performances <- c() # We create an empty vector for the performances
#### loop for running cross validation ####
for (k in 1:max(df_train$fold)){ 
  fold_k <- as.integer(as.character(k)) # Convert k to integer 
  
  # assign test and training set according to the current folds
  train_df <- as.data.frame(df_train[df_train$fold != fold_k, ]) #training set without the current foldindex
  test_df <- as.data.frame(df_train[df_train$fold == fold_k, ]) #test set with only the current foldindex
  
# We create our model
  model <- lmer(CHI_MLU ~ Visit*Diagnosis*verbalIQ1 + (1+Visit|Child.ID), data = train_df, REML =F, lmerControl(optCtrl=list(xtol_abs=1e-8, ftol_abs=1e-8)))

# Run our predictions 
  prediction <- predict(model, test_df, allow.new.levels =T)
  
#Calculate the rmse for each fold 
  rmse <- rmse(test_df$CHI_MLU, prediction)
  message(paste("RMSE for fold", fold_k, rmse)) # print RMSE for each fold to console
  performances[k] <- rmse # register rmse value to 'performances'vector
}
```

    ## RMSE for fold 1 0.517896052902403

    ## RMSE for fold 2 0.49459844474655

    ## RMSE for fold 3 0.598921177691949

    ## RMSE for fold 4 0.538206990684724

    ## RMSE for fold 5 0.587564852782322

``` r
# calculate mean RMSE and print it
performances = mean(performances)
performances
```

    ## [1] 0.5474375

\[HERE GOES YOUR ANSWER\]

### Exercise 3) Assessing the single child

Let’s get to business. This new kiddo - Bernie - has entered your
clinic. This child has to be assessed according to his group’s average
and his expected development.

Bernie is one of the six kids in the test dataset, so make sure to
extract that child alone for the following analysis.

You want to evaluate:

-   how does the child fare in ChildMLU compared to the average TD child
    at each visit? Define the distance in terms of absolute difference
    between this Child and the average TD.

-   how does the child fare compared to the model predictions at Visit
    6? Is the child below or above expectations? (tip: use the predict()
    function on Bernie’s data only and compare the prediction with the
    actual performance of the child)

``` r
#We subset the data for Bernie - he has child.id 2
bernie <- subset(df_test, Child.ID==2)
td <- subset(df_train, Diagnosis == "TD")
asd <- subset(df_train, Diagnosis == "ASD")
asd_model <- lmer(CHI_MLU ~ Visit*verbalIQ1 + (1+Visit|Child.ID), data = asd, REML =F, lmerControl(optCtrl=list(xtol_abs=1e-8, ftol_abs=1e-8)))

td_model <- lmer(CHI_MLU ~ Visit*verbalIQ1 + (1+Visit|Child.ID), data = td, REML =F, lmerControl(optCtrl=list(xtol_abs=1e-8, ftol_abs=1e-8)))
```

    ## boundary (singular) fit: see ?isSingular

``` r
# The development over time
diff_1 <- abs(bernie$CHI_MLU[1]-mean(td$CHI_MLU[1]))
diff_2 <- abs(bernie$CHI_MLU[2]-mean(td$CHI_MLU[2]))
diff_3 <- abs(bernie$CHI_MLU[3]-mean(td$CHI_MLU[3]))
diff_4 <- abs(bernie$CHI_MLU[4]-mean(td$CHI_MLU[4]))
diff_5 <- abs(bernie$CHI_MLU[5]-mean(td$CHI_MLU[5]))
diff_6 <- abs(bernie$CHI_MLU[6]-mean(td$CHI_MLU[6]))
diff_1
```

    ## [1] 0.7322037

``` r
diff_2
```

    ## [1] 1.530839

``` r
diff_3
```

    ## [1] 1.796305

``` r
diff_4
```

    ## [1] 0.9315263

``` r
diff_5
```

    ## [1] 0.06484296

``` r
pred1 <- predict(td_model, bernie, allow.new.levels = TRUE)
pred2 <- predict(asd_model, bernie, allow.new.levels = TRUE)


bernie$CHI_MLU[6]-pred1[[6]] #This is Bernies MLU at visit 6
```

    ## [1] 0.1691664

``` r
#Bernie is below expectations of td 

bernie$CHI_MLU[6]-pred2[[6]]
```

    ## [1] -0.4689433

``` r
#He is slightly above expectations of an ASD child
```

\[HERE GOES YOUR ANSWER\]

### OPTIONAL: Exercise 4) Model Selection via Information Criteria

Another way to reduce the bad surprises when testing a model on new data
is to pay close attention to the relative information criteria between
the models you are comparing. Let’s learn how to do that!

Re-create a selection of possible models explaining ChildMLU (the ones
you tested for exercise 2, but now trained on the full dataset and not
cross-validated).

Then try to find the best possible predictive model of ChildMLU, that
is, the one that produces the lowest information criterion.

-   Bonus question for the optional exercise: are information criteria
    correlated with cross-validated RMSE? That is, if you take AIC for
    Model 1, Model 2 and Model 3, do they co-vary with their
    cross-validated RMSE?

### OPTIONAL: Exercise 5): Using Lasso for model selection

Welcome to the last secret exercise. If you have already solved the
previous exercises, and still there’s not enough for you, you can expand
your expertise by learning about penalizations. Check out this tutorial:
<a href="http://machinelearningmastery.com/penalized-regression-in-r/" class="uri">http://machinelearningmastery.com/penalized-regression-in-r/</a>
and make sure to google what penalization is, with a focus on L1 and
L2-norms. Then try them on your data!
