---
title: "Practical Machine Learning Project"
author: "Preetam Debasish Saha Roy"
date: "Sunday, January 25, 2015"
output: html_document
---

In this study we use data about personal activity using devices such as Jawbone Up, Nike FuelBand, and Fitbit. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, our goal is to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways.

**Data**

For this problem we are provided with two sets of data-training data and test data. We will use training dataset to build our prediction algorithm and then predict "classe" variable in the test dataset for 20 cases.

**Cleaning the Data**

Let's dive into the problem. First explore the dataset. A quick look into the dataset shows us there are lot of missing values, NA valus and divison by zero values. In order to build our prediction algorithm first we need to take care of this NA values by cleaning up the data. Before we start doing anything we should set the seed first, so that our results are reproducible.

```{r warning=FALSE}
set.seed(12)
traindata<-read.csv("pml-training.csv",header=T,na.string=c("","#DIV/0!","NA"))
dm1<-dim(traindata)
nacname<-NULL
for(i in 1:dim(traindata)[2]){
  if(class(traindata[,i])=="logical"){
    traindata[,i]<-as.character(traindata[,i])
  }
}
for(i in 1:dim(traindata)[2]){
  if(is.na(unique(traindata[,i]))){
    nacname<-c(nacname,i)    
  }      
}
traindata<-traindata[,-nacname]
dm2<-dim(traindata)
```

Here, while reading the training dataset we use na.string to declare blank values, "NA " and "DIV/0!" as NAs. Next we change some of the logical variables to chararcter variables for preprocessing purpose. Finally we remove the columns from the training dataset which have only NA values. This data cleaning reduces number of columns in our dataset from **`r dm1[2]`** to **`r dm2[2]`**. 

**Data Partition**

Now, we will divide our dataset in to two groups, one for training the model and the other to cross-validate our model.For this purpose we will be using "Caret" package in R. We divide our dataset in 50% training and 50% for testing (cross-validation).

```{r warning=FALSE,message=FALSE}
library(caret)
intrain<-createDataPartition(traindata$classe,p=0.5,list = F)
training<-traindata[intrain,]
testing<-traindata[-intrain,]
```

**Model Building**

For this study we will be random forest prediction algorithm on our training dataset. Before using random forest algorithm on our training dataset we exclude first 7 columns from model building as they are mostly counter type  variables. For example, the first column denotes the number of sample in the dataset. We build our model on this dataset and if we did not achieve satisfactory accuracy, then we can look to include this variables  to improve out of sample error of our prediction algorithm.

```{r warning=FALSE,message=FALSE}
library(randomForest)
dss<-training[,8:60]
mf<-randomForest(classe~.,data=dss)
```

**Cross-validation**

To test the accuracy of our prediction algorithm, we need to cross-validate our predictions on the test dataset (created from the training dataset) before we apply our prediction on the actual test dataset (consisting of 20 samples.) 

```{r}
confusionMatrix(testing$classe,predict(mf,testing))
```

So, we have achieved **more than 99%** overall accuaracy (So, out of sample error is pretty low). Now, we are pretty confident to predict with this data on the original test cases.

**Predicting the Test cases**

```{r}
testdata<-read.csv("pml-testing.csv",header=T)
predict(mf,testdata)
```

So, this are our predictions for the 20 sample cases.

