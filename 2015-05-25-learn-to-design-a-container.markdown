---
layout: post
title: "Learn to Design a Container"
date: 2015-05-25 19:14:38 +0800
comments: true
categories: Cplusplus
---

Container is a collection which help us to store data of different types of data structure.

There are only two types of container in C++/C :

* array
* structure

C++ could provide more container but it didn’t.

**Give a man a fish and you feed him for a day; teach a man to fish and you feed him for a lifetime; knowledge is the best charity;**

:)

<!-- more -->
Here we are gona to design a container which is like array but not the same.

![images](/images/img_for_2015_05_24/arch.png)

You could find the implementation of this container on my github.

[Our Container Implementation](https://github.com/jasonleaster/Rumination_On_C_plus_plus/blob/master/chapter_13/con_array.h)

You could test out container by this program.

``` C++
#include "con_array.h"

int main()
{
    Array<int> *ap = new Array<int> (10);
    Pointer<int> p(*ap, 5);
    //delete ap;

    for (int i = 0; i < 10; i++)
    {
        (*ap)[i] = i;
    }

    *p = 42;

    cout << "The size of Array " << ap->size() <<endl;
    cout << (*ap) << endl;

    ap->resize(20);
    cout << "After resize(), the size of Array " << ap->size() <<endl;
    cout << (*ap) << endl;

    return 0;
}
```

You could also see the output beblow there:

![images](/images/img_for_2015_05_24/output1.png)

---------
Photo by Jason Leaster in ChangDe, China

![images](/images/img_for_2015_05_24/cherry_blossom.png)

