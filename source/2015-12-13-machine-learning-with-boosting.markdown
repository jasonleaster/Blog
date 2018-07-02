---
layout: post
title: "Machine Learning with Boosting"
date: 2015-12-13 17:00:47 +0800
comments: true
categories: MachineLearning
---

This blog will talk about the theory and implementation about famouse
concept in machine learning -- `Boosting`.

All algorithms are implemented in Python.

There are two main tasks which people want to finish with Machine Learning.

* Classification
* Regression

There are a lot of other ways to do it but now we focus on `boosting` algorithm. You know that it's a fantastic way to make our work done.


### Adaboost for classification

If you never hear about adaboost, I recommend you to finish the 7-th lab in MIT 6.034. It will help you a lot to understand what I'm taking about. But this lab didn't build adaboost completely. So, I implement it individually.


Give the input training samples which have tag with it.

<div style = "text-align:center"><img src = "/images/img_for_2015_12_13/equation.png" align = "middle"> </div>

where x[i] is the feature of the i-th sample point and y[i] is the `label` (soemtimes we call it as `tag`) with the sample point.


In this algorithm, there are only two different label of samples {-1, +1}.


Some classifier like decision tree also can work correctly about classification. But it's also easy to overfitting. So, we can't use it in some special situation. Instread of using decision tree, we use `decision stump` which is a special type of decision tree which's depth is only one. So we call it as `decision stump`.

<div style = "text-align:center"><img src = "/images/img_for_2015_12_13/stump.png" align = "middle"> </div>


<div style = "text-align:center"><img src = "/images/img_for_2015_12_13/weakClassifier.png" align = "middle"> </div>

`Yoav Freund` and `Robert Schapire` create this algorithm __AdaBoost__ which means adaptive boosting.


<div style = "text-align:center"><img src = "/images/img_for_2015_12_13/AdaBoost.png" align = "middle"> </div>

Test case:

There are training points with two different label. What if we input a point which's type is unkown, what the result will be?

![images](/images/img_for_2015_12_13/samples.png)

The test result is below there:

![images](/images/img_for_2015_12_13/result.png)

Just create a object of class `Adaboost` with your training samples with label. like this:

``` python

import adaboost
a = AdaBoost(Original_Data, Tag)
# The up bound of training time to avoid the algorithm won't stop for not meeting the training accuracy.
times = 5 

a.train(times)

a.prediction(UnkownPoints)

```

API `prediction()` of class AdaBoost will return the result of prediction according to the model. All job done.

You could find other test case in my repository in github.

[Implementation of Adaboost in Python](https://github.com/jasonleaster/Machine_Learning/tree/master/Adaboost)

There is an [assignment](http://cs229.stanford.edu/extra-notes/boosting.pdf) about AdaBoost in Stanford CS 229, which will ask student to implement stump booster. But I don't really understand the skeleton of that source code. I think there must be something worng with that matlab script `stump_booster.m`. The week classifier can't lost the direction information.


``` matlab

%%% !!! Don't forget the important variable -- direction

API given by the course materials:
function [ind, thresh] = find_best_threshold(X, y, p_dist)
function [theta, feature_inds, thresholds] = stump_booster(X, y, T)

API of my solution:
function [ind, thresh, direction] = find_best_threshold(X, y, p_dist)
function [theta, feature_inds, thresholds, directions] = stump_booster(X, y, T)

```

Run `boost_example.m`, you will see the __classifier line__ with different iteration.

<div style = "text-align:center"><img src = "/images/img_for_2015_12_13/iter2.jpg" width = "400" height = "400" align = "middle"> </div>

<div style = "text-align:center"><img src = "/images/img_for_2015_12_13/iter5.jpg" width = "400" height = "400" align = "middle"> </div>

<div style = "text-align:center"><img src = "/images/img_for_2015_12_13/iter10.jpg" width = "400" height = "400" align = "middle"> </div>


### Boosting Tree

We have knew to use `AdaBoost` to do classification. `Boosting Tree` will help us to do regression.

We also use decision stump as the weak classifier. But implementation of decision stump in this algorithm is not the same as that in AdaBoost.

There are ten samples in my test module:

``` python

Original_Data = numpy.array([
        [2],
        [3],
        [4],
        [5],
        [6],
        [7],
        [8],
        [9],
        [10],
        [1]
        ]).transpose()

ExpVal = numpy.array([
        [5.70],
        [5.91],
        [6.40],
        [6.80],
        [7.05],
        [8.90],
        [8.70],
        [9.00],
        [9.05],
        [5.56]
        ]).transpose()

```

The expected value of Original_Data[i] is ExpVal[i]. The input is from 1 to 10. How about to predict the output when the input is 1 or 11?

Let's test it. Here is the result:
![images](/images/img_for_2015_12_13/output_of_boosting_tree.png)

Just used 11 weak classifier to construct a stronger classifier to do the regressio. The output is reasonable.

Here is my implementation of `Boosting Tree`
[Implementation of Boosting Tree in Python](https://github.com/jasonleaster/Machine_Learning/tree/master/Boosting_Tree)

Reference:

1. MIT-6.034, Artificial Intelligence. Lab-7
2. << The statistic methods >> by HangLi.
3. [Wikipedia](https://en.wikipedia.org/wiki/AdaBoost)

------

Photo by Jason Leaster

![images](/images/img_for_2015_12_13/street.png)
