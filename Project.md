Machine Learning Project by Tim Sturges
============================================================
Human Activity Recognition - Predicting "classe" Of Exercise
============================================================

#### Summary
This project is a demonstration of machine learning skills as presented by Jeff 
Leek in the Machine Learning MOOC by Coursera. The task is to use the provided 
training data set to make predictions on the provided testing data set. 

The training data set contains a factor varaible **classe** with five levels: 
**A,B,C,D,E** which correspond to the manner in which a standardized exercise 
was performed. For the sake of brevity, it is both unnecessary and redundant to 
explain here the nature of the exercise and the **classe** variable. Instead, I 
suggest referring to the project from which the data sets originate, which has 
been graciously provided by the project's authors under the Creative Commons 
BY-SA license. See "References" section below.

#### Preprocessing The Data
In the following code chunk, the data is loaded into memory, then cleaned. Upon 
inspection of the training and testing data sets, it is apparent that there 
are 100 columns of entirely NA data in the testing set, and 67 columns mostly 
NA, NaN, blank, or empty values in the training set. Those 67 columns are all 
included in the list of 100 NA columns from the testing set. One reason is 
sufficient to eliminate these 100 columns from both data sets: even if we were 
able impute data for those 33 columns of training data, any model built 
including those features will be unusuable to predict on the testing set, since 
there is no data available in those features to use for that prediciton. 
Furthermore, even if this were not the case, those columns of missing data in 
the training set are up to 99% missing in most cases, therefore we would not 
expect K-Nearest Neighbor imputation, for example, to provide good results. 
Therefore, removal of these 100 columns from both sets is peferable to 
imputation of the missing data.

The first 7 columns are also removed on the basis that they are not expected to 
be good predictors of **classe**, since they contain an index, subject 
identifiers (names), and timeline data. Only numeric data from the sensors used 
to take measurements remains.

```r
trn <- read.csv("./data/pml-training.csv")
tst <- read.csv("./data/pml-testing.csv")
x <- sapply(tst,anyNA)
listNA <- which(x == 1)
trn_nona <- trn[,-listNA]
tst_nona <- tst[,-listNA]
trn_nona <- trn_nona[,-c(1:7)]
tst_nona <- tst_nona[,-c(1:7)]
```

#### Model Training
Here, the caret library is invoked, and the model is trained. The training data 
is split into its own training and testing subset, which is used to validate 
the model, and estimate expected out-of-sample error rate. The seed is set for 
reproducibility. Due to the nature of the measurements and the exercise being 
studied (a dumbbell curl), a linear model would not be appropriate. The first 
model attempted was a simple decision tree which yielded spectacularly poor 
results (48% accuracy), so the next attempt used a "treebag" model, with 10-fold 
cross-validation. Although computationally intensive, the results are 
outstanding: nearly 99% prediction accuracy! The out-of-sample error rate is 
about 1% This was only the second model attempted, and it is unlikely that other 
models will predict with better accuracy by a significant margin, so it is 
satisfactory for the project task. 


```r
library(caret)
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
set.seed(666)
inTrain <- createDataPartition(y=trn_nona$classe,p=0.75,list=F)
training <- trn_nona[inTrain,]
testing <- trn_nona[-inTrain,]
modTrCtrl <- trainControl(method="cv",number=10,p=0.75,allowParallel=T)
modFit <- train(classe ~.,method="treebag",data=training,trControl = modTrCtrl)
```

```
## Loading required package: ipred
## Loading required package: plyr
```

```r
confusionMatrix(testing$classe,predict(modFit,newdata=testing))
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1389    4    2    0    0
##          B   10  926   10    2    1
##          C    1    6  841    7    0
##          D    1    0    9  792    2
##          E    0    3    3    3  892
## 
## Overall Statistics
##                                        
##                Accuracy : 0.987        
##                  95% CI : (0.983, 0.99)
##     No Information Rate : 0.286        
##     P-Value [Acc > NIR] : <2e-16       
##                                        
##                   Kappa : 0.983        
##  Mcnemar's Test P-Value : NA           
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             0.991    0.986    0.972    0.985    0.997
## Specificity             0.998    0.994    0.997    0.997    0.998
## Pos Pred Value          0.996    0.976    0.984    0.985    0.990
## Neg Pred Value          0.997    0.997    0.994    0.997    0.999
## Prevalence              0.286    0.191    0.176    0.164    0.183
## Detection Rate          0.283    0.189    0.171    0.162    0.182
## Detection Prevalence    0.284    0.194    0.174    0.164    0.184
## Balanced Accuracy       0.995    0.990    0.984    0.991    0.997
```

#### Model Application
The model is ready to make predictions on the testing data set, so in the 
following code chunk the predict function is passed the fitted model, and the 
cleaned testing data. Then the predictions are submitted per the assignment 
parameters, using code compied with permission from the submission instructions. 

```r
modPredict <- predict(modFit,newdata=tst_nona)
pml_write_files <- function(x){
        n <- length(x)
        for(i in 1:n){
                filename <- paste0("problem_id_",i,".txt")
                write.table(x[i],file=filename,quote=F,row.names=F,col.names=F)
        }
}
setwd("./submission")
pml_write_files(modPredict)
```

#### References
This project uses, with permission, the "Weight Lifting Exercises Dataset", 
released under the Creative Commons BY-SA license. The data originates from the 
following source:

Velloso, E.; Bulling, A.; Gellersen, H.; Ugulino, W.; Fuks, H. Qualitative 
Activity Recognition of Weight Lifting Exercises. Proceedings of 4th 
International Conference in Cooperation with SIGCHI (Augmented Human '13) . 
Stuttgart, Germany: ACM SIGCHI, 2013.  
http://groupware.les.inf.puc-rio.br/public/papers/2013.Velloso.QAR-WLE.pdf  
http://groupware.les.inf.puc-rio.br/har  

This project also uses code copied with permission from the assignment's 
submission instructions, found here:  
https://class.coursera.org/predmachlearn-010/assignment/view?assignment_id=5
