---
layout: post
title: "Why should we use C++"
date: 2015-05-15 22:56:11 +0800
comments: true
categories: Cplusplus
---

After reading the chapter 0, 1 and 2 of `<< Reminations on C++ >>`, I try to make myself more sober and do some summaries for this question.

**Why should we use C++ ?**

What I want to do is to give the answer in advance and then give the reason.

Here we go.

<!-- more -->

In OOP (Object Oriented Programming), what we should do is focus on data but not the process. The key concept in C++ is class which is use for abstraction. Wait… what we use it for ? what’s that we do abstract operation on? The answer is the data which we used to abstract objects or concepts.

Abstraction is a very powerful operation to help us focus on our question and solve our problems.

What means abstraction, never afraid it, it just a operation that we focus on things that we are interesting and ignore something not important but do existed.

It isn’t totally new things, C++ will form us to treat thing more pragmatism. In daily life, custom’s requirements mostly change when time flows. So we have to solve this big problem in a gentle way. You do not need to rewrite all your code which you have writed but what you should do is to design your code in pragmatism.

Here is a demonstration for how to design your code.

This code was writed in C but not C++. And the purpose of this code is explicit. It use two function @trace_on and @trace_off to control the function @trace will output the string which pointer @s point to onto the standary output stream.

``` C++
#include <stdio.h>

static int noisy = 1;

void trace(char *s)
{
    if (noisy)
    {
        printf("%s\n", s);
    }
}

void trace_on()
{
    noisy = 1;
}

void trace_off()
{
    noisy = 0;
}

int main()
{
    trace("begin main()\n");

    // the body of main function

    trace("end main()\n");
}
```

Weakness:

You may have noticed that this code can only output the string into the standary output stream. If you want to modify this code and enable it have the ability to output into others stream, you may do a lot of work to meet your requirement.

Everyone could call @trace_on and @trace_off, it may not what we want if we want do a good job on encapsulation. The reason we want our code more encapsulated is that we want our code more robust and modularity. This great mind helps us to keep complex things in a simple way. We solve a big problem in some little sub-problem. Our contribution is how to design our code and enble it more encapsulation.

C++ do a good job on local functions which C don’t support. Here is a demo. The function @print in class Trace can only be called by class Trace object like this t.print(), you can’t call it individually. If there is another different one implementation of function @print in others class, they won’t interfere each other. This is the encapsulation what we want.

This is what we promoted in C++.

If you want to output into different stream, just use different construct function is OK. If you want to close the output, it’s convinent to call the member function @off.

All work we do is for only two things

* int noisy
* FILE *f

Yes, the data.
	
``` C++
#include <stdio.h>

class Trace()
{
    public:
        Trace()
        {
            noisy = 0;
            f = stdout;
        }

        Trace(FILE * ff)
        {
            noisy = 0;
            f = ff;
        }

        void print(char *s)
        {
            if (noisy)
            {
                fprintf(f, "%s", s);
            }
        }

        void on()
        {
            noisy = 1;
        }

        void off()
        {
            noisy = 0;
        }

    private:
        int noisy;
        FILE *f;
};

int main()
{
    Trace t(stderr);
    t.print("begin main()\n");

    // The body of main function

    t.print("end main()\n");
}
```

----------------------
Photo by Jason Leaster in Changde, China.

![images](/images/img_for_2015_05_16/bridge.png)
