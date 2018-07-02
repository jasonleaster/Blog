---
layout: post
title: "Machine Learning With K-Means"
date: 2015-12-30 16:03:13 +0800
comments: true
categories: MachineLearning
---

K-Means is a classical unsupervised clustering Learning Algorithm. The detail of the theory about K-Means that you can find it in Wikipedia. Now I introduce to implement this algorithm by myself.

If you are interesting in the implementation and change it into a better version, you could find it in my github repository and give me some advices. I will be appreciated.

-------

So consider about if I want to classify the data into three different cluster. How could I make it?

![images](/images/img_for_2015_12_30/original.png)

Here is the result:

![images](/images/img_for_2015_12_30/result.png)

With the mean values:

``` python

Means =
[[3.5       1       6]
 [1.66666   6       6]]

```

<!-- more -->

In the implementation, I just choose the euclidean distance equation as my sensor to calculate the distance between samples. You could assign the `self.distance` with your function which is in your application.

Here, I show you how to classify the sample point in `K-Means`.

``` python

def classify(self):
    for i in range(self.SampleNum):
        minDis = +numpy.inf
        label  = None
        for k in range(self.classNum):
            d = self.distance(self._Mat[:, i].tolist(), self.meanVal[:, k].tolist())
            if d < minDis:
                minDis = d
                label  = k

        self.classification[i][0] = label
        self.classification[i][1] = minDis

```
And, here you will glance at the main procesure of this algorithm.

``` python

    """
    After you initialized this class, just call this
    function and K Means Model will be built
    """
    def train(self):
        while True:

            if self.stopOrNot():
                return

            self.classify()

            for k in range(self.classNum):
                mean    = None
                counter = 0
                for i in range(self.SampleNum):
                    if self.classification[i][0] == k:
                        if mean == None:
                            mean =  numpy.array(self._Mat[:, i])*1.
                        else:
                            mean += self._Mat[:, i]

                        counter += 1.

                mean /= counter
                self.meanVal[:, k] = mean

```

Hope my work will help you in some day. Thank you. 

Yous, EOF

------------
Photo by Annabella

Aha! Look! What a big shark. I'm fighting ...

![images](/images/img_for_2015_12_30/bigshark.png)
