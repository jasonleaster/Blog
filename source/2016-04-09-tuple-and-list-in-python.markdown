---
layout: post
title: "Tuple and list in Python"
date: 2016-04-09 08:52:51 +0800
comments: true
tags: Python
categories: Python
---

Try to answer the following question.

What are lists and tuples good for?

What is the complexity of a lookup in a list/tuple?

How is that complexity achieved?

What are the differences between lists and tuples?

How does appending to a list work?

When should I use lists and tuples?

<!-- more -->

Lists and tuples are a class of data structure called arrays. An array is simply a flat list of data with some intrinsic ordering.
This demarcates another line between lists and tuples: **lists are dynamic arrays while tuples are static arrays.**

### Lists Versus Tuples

Differences between lists and tuple

1. lists are dynamic arrays; they are mutable and allow for resizing(changing the number of elements that are held).

2. Tuples are static arrays; they are immutable, and the data within them cannot be changed once they have been created.

3. Tuples are cached by the Python runtime, which means that we don't need to talk to the kernel to reserve memory every time we want to use one.


These differences outline the philosophical difference between the two: tuples are for describing multiple properties of one unchanging thing, and list can be used to store collections of data about completely disparate objects.

![images](/images/img_for_2016_04_09/tuple_list_construction_speed.png)
