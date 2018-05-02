---
layout: post
title: "Binary Search"
date: 2015-07-24 10:17:20 +0800
comments: true
categories: algorithm
---
This is a story why I write this algorithm again.

> In this days, I work as an intern for Alibaba. To be honest, it totally a bad time for the beginning days. More awful fact to me is that my mentor do his job mostly in Java and I have to pick it up as quickly as possible. It’s totally a stranger for me about Java. **What I have to say is that thanks god and my mentor.** He is so nice and never blame me for my problem. You know that I suffer pressure heavily for I learned nothing but time wasted in this two weeks. I asked for help from human resource manager and hope to find someone guide me to walk out this shadow. Bo who is my mentor. He understand me and know my feeling. Bo encourage me and try to let me believe things will be better. Today, I face to a problem which I think is very naive but I didn’t show the solution to a leader of a department very well. And I could feel that he doubt about my ability in programming. I feel shame about he ask me that “Do you have programming in C or C++ for ten thousand lines ?”.

<!-- more -->

Ok, the story is end and it don’t important to me anymore. What should I do is to pay attention to my weak ability and work hard to make me more stronger. Then I will prepare for the coming recruitment in autumn.

This is a classical question. How to find an element in sequenced data ? The solution is `Binary Search`. It’s the quickest way to find element in sorted data set.

The theory about `Binary Search` show in blew picture.

![images](/images/img_for_2015_07_24/binary_search.png)

Here you could see my code and show you how to implement a Binary Search in C++, Java and Python.	

``` C++
/**********************************************************
Programmer   :  EOF
E-mail       :  jasonleaster@163.com
File         :  binary_search.cpp
Date         :  2015.07.25

***********************************************************/

#include <iostream>

template<class T>
T BinarySearch(const T p_array[], int start, int end, T target)
{
    if (!p_array || start > end)
    {
        std::cout << "Empty Data set or bad scope "
                     "try to modify your parameter @start "
                     "or end. And make sure that @start smaller "
                     "than @end"
                   << std::endl;
        return -1;
    }

    int middle = -1;

    if(p_array[start] == target)
    {
        return start;
    }
    else if(p_array[end] == target)
    {
        return end;
    }

    for(;start < end;)
    {
        middle = (start + end)/2;

        if(p_array[middle] == target)
        {
            return middle;
        }
        else if (p_array[middle] < target)
        {
            start = middle + 1;
        }
        else
        {
            end = middle - 1;
        }
    }

    std::cout << "The element you find don't exist in the data set"
              << std::endl;

    return -1;
}

int main()
{
    int array[10] = {2, 4, 6, 8, 10, 12, 14, 16, 18, 20};

    int ret = -1;

    ret = BinarySearch(array, 0, sizeof(array)/sizeof(array[0]), 7);

    ret = BinarySearch(array, 0, sizeof(array)/sizeof(array[0]), 12);

    std::cout << ret << std::endl;
    return 0;
}
```

You could also find the other implementation follow this link:

[Binary Search](https://github.com/jasonleaster/Algorithm/tree/master/Binary_search)

----------
Photo by Jason Leaster, Hangzhou, China

![images](/images/img_for_2015_07_24/cute.png)


