Predicting Outcomes for New Data
================================

Predicting the target values for new observations is
implemented the same way as most of the other predict methods in *R*.
In general, all you need to do is
call \man2[predict][predict.WrappedModel] on the object returned by [train](http://berndbischl.github.io/mlr/man/train.html)
and pass the data to be predicted.


Quick start
-----------

### Classification example

Let's train a Linear Discriminant Analysis on the ``iris`` data and make predictions 
for the same data set.


```splus
library("mlr")

task = makeClassifTask(data = iris, target = "Species")
lrn = makeLearner("classif.lda")
mod = train(lrn, task = task)
pred = predict(mod, newdata = iris)
pred
```

```
## Prediction:
## predict.type: response
## threshold: 
## time: 0.00
##    truth response
## 1 setosa   setosa
## 2 setosa   setosa
## 3 setosa   setosa
## 4 setosa   setosa
## 5 setosa   setosa
## 6 setosa   setosa
```



### Regression example

We fit a simple linear regression model to the ``BostonHousing`` data set and predict
on the training data.


```splus
library("mlr")
library("mlbench")
data(BostonHousing)

task = makeRegrTask(data = BostonHousing, target = "medv")
lrn = makeLearner("regr.lm")
mod = train(lrn, task)
predict(mod, newdata = BostonHousing)
```

```
## Prediction:
## predict.type: response
## threshold: 
## time: 0.00
##   truth response
## 1  24.0    30.00
## 2  21.6    25.03
## 3  34.7    30.57
## 4  33.4    28.61
## 5  36.2    27.94
## 6  28.7    25.26
```


### Clustering example

We cluster the ``iris`` data set without the target variable and predict using
the same data.


```splus
library("mlr")
task = makeClusterTask(data = iris[, -5])
lrn = makeLearner("cluster.XMeans")
mod = train(lrn, task)
predict(mod, newdata = iris[, -5])
```

```
## Prediction:
## predict.type: response
## threshold: 
## time: 0.04
##   response
## 1        2
## 2        2
## 3        2
## 4        2
## 5        2
## 6        2
```



Further information
-------------------

There are several possibilities to pass the observations for which to make
predictions.
The first possibility is via the ``newdata`` argument and was already shown in the 
examples above.
If the data for which predictions are required are already contained in 
the [Learner](http://berndbischl.github.io/mlr/man/makeLearner.html), it is also possible to pass the task and optionally specify 
the subset argument that contains the indices of the test observations.

Predictions are encapsulated in a special [Prediction](http://berndbischl.github.io/mlr/man/Prediction.html) object. The available
accessors are documented in the man page for [Prediction](http://berndbischl.github.io/mlr/man/Prediction.html).


### Classification example

In case of a classification task, the result of
\man2[predict][predict.WrappedModel] depends on the predict type, which was set
when generating the [Learner](http://berndbischl.github.io/mlr/man/makeLearner.html). The default is to predict class
labels.

We start again by loading **mlr** and creating a classification task for the 
``iris`` data set. We select two subsets of the data. We train a decision tree on the
first one and predict the class labels on the test set.


```splus
library("mlr")

# Define the classification task
task = makeClassifTask(data = iris, target = "Species")

# Define the learning algorithm
lrn = makeLearner("classif.rpart")

# Split the iris data into a training set for learning and a test set
training.set = seq(from = 1, to = nrow(iris), by = 2)
test.set = seq(from = 2, to = nrow(iris), by = 2)

# Now, we can train a decision tree using only the observations in train.set
mod = train(lrn, task, subset = training.set)

# Finally, predict the label on new values using the predict method
pred = predict(mod, newdata = iris, subset = test.set)
```


A `data.frame` that contains the true and predicted class labels can be accessed via


```splus
head(pred$data)
```

```
##     truth response
## 2  setosa   setosa
## 4  setosa   setosa
## 6  setosa   setosa
## 8  setosa   setosa
## 10 setosa   setosa
## 12 setosa   setosa
```


Alternatively, we can also predict directly from a `task`:


```splus
pred = predict(mod, task = task, subset = test.set)
head(as.data.frame(pred))
```

```
##    id  truth response
## 2   2 setosa   setosa
## 4   4 setosa   setosa
## 6   6 setosa   setosa
## 8   8 setosa   setosa
## 10 10 setosa   setosa
## 12 12 setosa   setosa
```


When predicting from a `task`, the resulting `data.frame` contains an additional column, 
called *ID*, which tells us which element in the original data set the prediction 
corresponds to. In the iris example the IDs and the rownames coincide.

In order to get predicted posterior probabilities, we have to change the ``predict.type``
of the learner.


```splus
lrn = makeLearner("classif.rpart", predict.type = "prob")
mod = train(lrn, task)
pred = predict(mod, newdata = iris, subset = test.set)
head(pred$data)
```

```
##     truth prob.setosa prob.versicolor prob.virginica response
## 2  setosa           1               0              0   setosa
## 4  setosa           1               0              0   setosa
## 6  setosa           1               0              0   setosa
## 8  setosa           1               0              0   setosa
## 10 setosa           1               0              0   setosa
## 12 setosa           1               0              0   setosa
```


As you can see, in addition to the predicted probabilities, a response
is produced by choosing the class with the maximum probability and
breaking ties at random.

The predicted posterior probabilities can be accessed via the [getProbabilities](http://berndbischl.github.io/mlr/man/getProbabilities.html) function.


```splus
head(getProbabilities(pred))
```

```
##    setosa versicolor virginica
## 2       1          0         0
## 4       1          0         0
## 6       1          0         0
## 8       1          0         0
## 10      1          0         0
## 12      1          0         0
```



### Binary classification

In case of binary classification, two things are worth mentioning. As you may recall, 
we can specify a positive class when generating the task. Moreover, we can set the
threshold value that is used to assign class labels based on the predicted 
posteriors.

To illustrate binary classification, we use the Sonar data set from the
[mlbench](http://cran.r-project.org/web/packages/mlbench/index.html) package. 
Again we create a classification task and a learner which predicts
probabilities, train the learner and then predict the class labels based on the
probabilities.


```splus
library("mlbench")
data(Sonar)

task = makeClassifTask(data = Sonar, target = "Class", positive = "M")
lrn = makeLearner("classif.rpart", predict.type = "prob")
mod = train(lrn, task = task)
pred = predict(mod, task = task)
head(pred$data)
```

```
##   id truth prob.M prob.R response
## 1  1     R 0.1061 0.8939        R
## 2  2     R 0.7333 0.2667        M
## 3  3     R 0.0000 1.0000        R
## 4  4     R 0.1061 0.8939        R
## 5  5     R 0.9250 0.0750        M
## 6  6     R 0.0000 1.0000        R
```


In a binary classification setting, we can adjust the threshold, used
to map probabilities to class labels using [setThreshold](http://berndbischl.github.io/mlr/man/setThreshold.html). Here, we set
the threshold for the *positive* class to 0.8:


```splus
pred = setThreshold(pred, 0.8)
head(pred$data)
```

```
##   id truth prob.M prob.R response
## 1  1     R 0.1061 0.8939        R
## 2  2     R 0.7333 0.2667        R
## 3  3     R 0.0000 1.0000        R
## 4  4     R 0.1061 0.8939        R
## 5  5     R 0.9250 0.0750        M
## 6  6     R 0.0000 1.0000        R
```

```splus
pred$threshold
```

```
##   M   R 
## 0.8 0.2
```



### Regression example

We again use the BostonHousing data set and learn a Gradient Boosting Machine
model. We take all observations with an odd index as training and all
observations with an even index as test. The procedure is the same as in the
classification case.


```splus
library(mlbench)
data(BostonHousing)

task = makeRegrTask(data = BostonHousing, target = "medv")

training.set = seq(from = 1, to = nrow(BostonHousing), by = 2)
test.set = seq(from = 2, to = nrow(BostonHousing), by = 2)

lrn = makeLearner("regr.gbm", n.trees = 100)
mod = train(lrn, task, subset = training.set)

pred = predict(mod, newdata = BostonHousing, subset = test.set)

head(pred$data)
```

```
##   truth response
## 1  21.6    22.23
## 2  33.4    23.34
## 3  28.7    22.26
## 4  27.1    22.16
## 5  18.9    22.16
## 6  18.9    22.16
```

