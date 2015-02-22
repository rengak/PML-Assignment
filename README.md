---
title: "PML_Assignment"
output: html_document
---

# Loading required packages

```{r}
set.seed(151987)
library(caret)
library(dplyr)
```

# Training data set

- Reading the CSV file with NA strings function and stripping white spaces

```{r}
training_raw <- read.csv("http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv",na.strings=c("NA",""), strip.white=T)
```

- # of rows and columns in the train set

```{r}
dim(training_raw)
```

- Cleaning up the training data set
  - Remove columns with NA values
  - Remove variables related to name, time which will not help in predicting the classes

```{r}
isNA <- apply(training_raw, 2, function(x) { sum(is.na(x)) })
training_raw <- select(training_raw,which(isNA == 0))
training_raw <- select(training_raw,-c(X, user_name, new_window, num_window, raw_timestamp_part_1,
                   raw_timestamp_part_2, cvtd_timestamp))
```

# Training and Test Data sets

- Training and Test data sets are created in 60/40 ratio 

```{r}
inTrain <- createDataPartition(training_raw$classe, p=0.6, list=F)
training <- training_raw[inTrain,]
testing <- training_raw[-inTrain,]
```

# Model Building

- Random forests are used to create the model
- 4 fold cross validation is used

```{r}
ctrl <- trainControl(allowParallel=T, method="cv", number=4)
model <- train(classe ~ ., data=training, model="rf", trControl=ctrl)
```

# Cross Validating with Test Data set

- Out of sample error is <1%

```{r}
pred <- predict(model, newdata=testing)  
sum(pred == testing$classe) / length(pred)
```

- Confusion Matrix and Statistics

```{r}
confusionMatrix(testing$classe, pred)
```

