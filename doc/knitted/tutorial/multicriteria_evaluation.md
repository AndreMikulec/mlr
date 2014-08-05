Multi-criteria Evaluation
========================

In this example, we have to deal with some of the implementation details of
**mlr** and the characteristics of different measures.
In a lot of cases you get more than just one measure value but are interested in
just one aggregated value.
For example, a 10-fold cross validation computes 10 values for the chosen
performance measure.
The aggregated value is the mean of these 10 numbers.
**mlr** knows how to handle it because each `measure` knows how it is aggregated:


```splus
library(mlr)
mmce$aggr  #Mean misclassification error
```

```
## Aggregation function: test.mean
```

```splus
rmse$aggr  #Root mean square error
```

```
## Aggregation function: test.sqrt.of.mean
```


You can also create a new `measure` by using a different aggregation (see [aggregations](http://berndbischl.github.io/mlr/man/aggregations.html)).


```splus
mmceAggrBySd = setAggregation(mmce, test.sd)

task = makeClassifTask(data = iris, target = "Species")
lrn = makeLearner("classif.rpart")
rdesc = makeResampleDesc("CV", iters = 5)
r = resample(lrn, task, rdesc, measures = list(mmce, mmceAggrBySd))
```

```
## [Resample] cross-validation iter: 1
## [Resample] cross-validation iter: 2
## [Resample] cross-validation iter: 3
## [Resample] cross-validation iter: 4
## [Resample] cross-validation iter: 5
## [Resample] Result: mmce.test.mean=0.0733,mmce.test.sd=0.0279
```


It is even possible to create your own aggregation function by calling the
internal **mlr** function [makeAggregation](http://berndbischl.github.io/mlr/man/makeAggregation.html).
You can use internal (not exported) functions of R packages by prefixing the
name of the function with the name of the package, i.e. `packagename:::function()`.
In this case:

```splus
mlr:::makeAggregation(id = "some.name", fun = function(task, perf.test, perf.train, 
    measure, group, pred) {
    # stuff you want to do with perf.test or perf.train
})
```

```
## Aggregation function: some.name
```

Remember: it is important that the head of the function looks exactly as above!
`perf.test` and `perf.train` are both numerical vectors containing the measure values.
In the usual case (e.g. cross validation), the `perf.train` vector is empty.

Practical Example: Evaluate the Range of Measures
-------------------------------------------------

Let's say you are interested in the range of the obtained measures:

```splus
my.range.aggr = mlr:::makeAggregation(id = "test.range", fun = function(task, 
    perf.test, perf.train, measure, group, pred) diff(range(perf.test)))
```


Now we can run a feature selection based on the first measure in the provided
list and see how the other measures turn out.

```splus
library(ggplot2)
ms1 = mmce
ms2 = setAggregation(ms1, my.range.aggr)
ms1min = setAggregation(ms1, test.min)
ms1max = setAggregation(ms1, test.max)
(res = selectFeatures(lrn, task, rdesc, measures = list(ms1, ms2, ms1min, ms1max), 
    control = makeFeatSelControlExhaustive(), show.info = FALSE))
```

```
## FeatSel result:
## Features (2): Sepal.Width, Petal.Width
## mmce.test.mean=0.0467,mmce.test.range=0.0333,mmce.test.min=0.0333,mmce.test.max=0.0667
```

```splus
perf.data = as.data.frame(res$opt.path)
p = ggplot(aes(x = mmce.test.mean, y = mmce.test.range, xmax = mmce.test.max, 
    xmin = mmce.test.min, color = as.factor(Sepal.Width), pch = as.factor(Petal.Width)), 
    data = perf.data) + geom_point(size = 4) + geom_errorbarh(height = 0)
print(p)
```

![plot of chunk MulticriteriaEvaluation](figs/multicriteria_evaluation/MulticriteriaEvaluation.png) 

