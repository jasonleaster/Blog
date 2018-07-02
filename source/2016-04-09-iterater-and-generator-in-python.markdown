---
layout: post
title: "Iterater and generator in Python"
date: 2016-04-09 13:54:30 +0800
comments: true
categories: Python
---
Try to answer the following question?

1. How do generators save memory?

2. When is the best time to use a generator?

3. How can I use itertools to create complex generator workflows?

4. When is lazy evaluation beneficial, and when is it not?

<!-- more -->

Programmer who is familiar with another language start learning Python, they are taken aback by the difference in __for__ loop notation.

With the influence from others language, they may try to finish iteration job by this code:

``` Python

for i in range(N):
    do_work(i)

```

What the beginner don't know the implemenation of function range in Python(2.7). The first thing the **range()** function must precreate the list of all numbers within the range.

In Python2.7, range() produce a list but xrange() return a iterator.( In Python 3, the range() is replaced with the xrange() and there is no xrange() function anymore. From this modification, programmer can know that the develop team of Python aware that it's neccessary to force the user to use a generator when they want to iteration job.)

Here is a test for the differences between the range() and xrange() in memory allocation.

![images](/images/img_for_2016_04_09/range_xrange_time.png)
![images](/images/img_for_2016_04_09/range_xrange_mem.png)
