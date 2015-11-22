# Coursera - Machine Learning - Course Project
Amine BENHAMZA  
22 November 2015  

## Executive Summary
This report explains the design and outcomes of the predictive study of Human Activity Recognition, particularly Weight Lifting Exercises. The goal is to predict the manner in which people did their exercise. Tolerated `Out of Sample Error: 5%` 

Find [here] more details. Data are taken from the same source.

The model fitted is a Random Forest with accuracy of 0.8%

## Study Main Blocks

. Load Raw data and perform cleansing

. Partition Data

. Feature Selection

. Fit the model

. Calculate Out Of Sample Error

### Load Raw data and perform cleansing
Load the data from the csv file provided by the mooc


```r
pmlRaw = read.csv("pml-training.csv");
dim(pmlRaw)
```

```
## [1] 19622   160
```

Then, remove all the predictors which records exceed 25% of rows' total

```r
pmlPre <- pmlRaw[, colSums(is.na(pmlRaw)) < nrow(pmlRaw)/4]
dim(pmlPre)
```

```
## [1] 19622    93
```

Remove the dataset `pmlRaw` dataset for memory optimization

```r
rm(pmlRaw)
```

### Parition Data
We'll apply the ratio 70/30 to partition the data


```r
suppressMessages(suppressWarnings(library(caret)))
inTrain <- createDataPartition(y = pmlPre$classe, p = 0.7, list = FALSE)
pmlTrain <- pmlPre[inTrain, ]
dim(pmlTrain)
```

```
## [1] 13737    93
```

### Feature Selection

**Step 1:** remove useless predictors

```r
removeIndex <- grep("timestamp|X|user_name|window", names(pmlTrain))
pmlTrain <- pmlTrain[, -removeIndex]
dim(pmlTrain)
```

```
## [1] 13737    86
```

**Step 2:** find the "Near Zero Variance" predictors and exclude them

```r
nzvTrain <- nzv(pmlTrain); length(nzvTrain)
```

```
## [1] 33
```

```r
pmlTrain <- pmlTrain[,-nzvTrain]
dim(pmlTrain)
```

```
## [1] 13737    53
```

**Step 3:** find attributes that are highly corrected and exclude them

```r
corrMatrixTrain <- cor(pmlTrain[,-dim(pmlTrain)])
highlyCorrTrain <- findCorrelation(corrMatrixTrain, cutoff=0.75)
pmlTrain <- pmlTrain[, -highlyCorrTrain]
dim(pmlTrain)
```

```
## [1] 13737    32
```


### Fit the model
The **random forest** method will be applied to fit the model, **3-folds cross validation** will be applied too.


```r
control = trainControl(method = "cv", number = 3,allowParallel=T)
model <- train(classe ~ ., data = pmlTrain, method="rf",trControl = control)
```

```
## Loading required package: randomForest
```

```
## Warning: package 'randomForest' was built under R version 3.1.3
```

```
## randomForest 4.6-12
## Type rfNews() to see new features/changes/bug fixes.
```

```r
model$finalModel
```

```
## 
## Call:
##  randomForest(x = x, y = y, mtry = param$mtry) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 2
## 
##         OOB estimate of  error rate: 0.84%
## Confusion matrix:
##      A    B    C    D    E class.error
## A 3900    6    0    0    0 0.001536098
## B   20 2630    8    0    0 0.010534236
## C    0   21 2369    6    0 0.011268781
## D    1    0   45 2204    2 0.021314387
## E    0    0    1    5 2519 0.002376238
```

### Test the model and Measure the Out Of Sample Error

Prepare the testing data

```r
pmlTest <- pmlPre[-inTrain, ]
dim(pmlTest)
```

```
## [1] 5885   93
```

Remove useless predictors : X, user_name....

```r
pmlTest <- pmlTest[, -removeIndex]
dim(pmlTest)
```

```
## [1] 5885   86
```

Remove near zeo Variance on Test Data

```r
pmlTest <- pmlTest[,-nzvTrain]
dim(pmlTest)
```

```
## [1] 5885   53
```

Remove highly correlated data

```r
pmlTest <- pmlTest[, -highlyCorrTrain]
dim(pmlTest)
```

```
## [1] 5885   32
```

Measure the OOSE

```r
predictions <- predict(model, pmlTest)

accuracy <- sum(predictions == pmlTest$classe)/length(predictions)

1-accuracy
```

```
## [1] 0.007646559
```
