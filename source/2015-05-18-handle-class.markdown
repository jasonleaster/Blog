---
layout: post
title: "Handle Class"
date: 2015-05-18 19:13:34 +0800
comments: true
categories: Cplusplus
---

For some types of class, if we could avoid the copy operation, it will be great advantage.

Sometimes, copy cost a lot.

It’s not a good way to use pointer to avoid copy operation. It’s unsafe.

What if we want the attribute of polymiorphism and reduce the cost of copy ?

The answer is Handle which is a special class.

<!-- more -->

We use count to avoid copying. The count can not be a part of handle. If we do that, each handle have to know others handles’ location(it’s easy for us to get address of members who are in the same class). Only we do that, we could update the count correctly. We also could not let the count be a part of objects. Becasue we have to rewrite the type of class which have already existed.

So, we define a new type of class to store the count and object.

![images](/images/img_for_2015_05_19/arch.png)
	

``` C++
/**********************************************
    Programmer  :   EOF
    Date        :   2015.05.19
    File        :   6.6.cpp
    E-mail      :   jasonleaster@gmail.com

**********************************************/
#include <iostream>

using namespace std;

class Point
{
    public:
        Point(): xval(0), yval(0) { }
        Point(int x, int y): xval(x), yval(y) { }
        int x() const { return xval; }
        int y() const { return yval; }
        Point& x(int xv) { xval = xv; return *this; }
        Point& y(int yv) { yval = yv; return *this; }

    private:
        int xval, yval;
};

/*
   This class is used for store the @Point class and count @u
 */
class UPoint
{
    friend class Handle;
    Point p;
    int counter;

    UPoint(): counter(1) { }
    UPoint(int x, int y): p(x, y), counter(1)  { }
    UPoint(const Point& p0): p(p0), counter(1) { }
};

class Handle
{
    public:
        Handle();
        Handle(int, int);
        Handle(const Point&);
        Handle(const Handle&);

        Handle& operator=(const Handle&);

        ~Handle();

        int x() const;
        Handle& x(int);

        int y() const;
        Handle& y(int);

    private:
        /*
            All we have done in @Handle is to maintain this data member :)
         */
        UPoint *up;
};

Handle::~Handle()
{
    if (--up->counter == 0)
    {
        delete up;
    }
}

/*
   Constructor function of @Handle
 */
Handle::Handle(): up(new UPoint) { }
Handle::Handle(int x, int y): up(new UPoint(x, y)) { }
Handle::Handle(const Point& p): up(new UPoint(p)) { }
Handle::Handle(const Handle& h): up(h.up) { ++up->counter; }

Handle&
Handle::operator = (const Handle& h)
{
    ++(h.up->counter);
    if(--(up->counter) == 0)
    {
        delete up;
    }

    up = h.up;
    return *this;
}

int Handle::x() const { return up->p.x(); }
int Handle::y() const { return up->p.y(); }

Handle& Handle::x(int x0)
{
    up->p.x(x0);
    return *this;
}

Handle& Handle::y(int y0)
{
    up->p.y(y0);
    return *this;
}

int main()
{
    Handle h(3, 4);
    Handle h2 = h;

    h2.x(5);

    int n = h.x();

    cout << "n = " << n << endl;
    return 0;
}
```


In this scheme, we meet a weak point of the mechanisim that we have to attach the handler onto the class (in our demo, it’s @UPoint).

It’s not convenient for us to use that scheme for different types of class which are inhiret from base class.

We sperate data and reference count by changing our handle into this:

``` C++
class Handle
{
    public:
        // member unchanged

    private:
        Point* p_Point;
        UseCount u;
};
```

Here is the other and better implementation for handle.

In this implementation, we use a very cool teachnology – Copy On Write(COW). You may have see this teachnology in Operating System which use it to implement a very important function – `fork` .

``` C++
/**********************************************
    Programmer  :   EOF
    Date        :   2015.05.19
    File        :   7.3.cpp
    E-mail      :   jasonleaster@gmail.com

**********************************************/
#include <iostream>

#define COW

using namespace std;

class Point
{
    public:
        Point(): xval(0), yval(0) { }
        Point(int x, int y): xval(x), yval(y) { }
        int x() const { return xval; }
        int y() const { return yval; }
        Point& x(int xv) { xval = xv; return *this; }
        Point& y(int yv) { yval = yv; return *this; }

    private:
        int xval, yval;
};



class UseCount
{
    public:
        UseCount();
        UseCount(const UseCount&);
        ~UseCount();
        //another things

        bool only();
        bool reattach(const UseCount& );
        bool makeonly();

    private:
        int* p_counter;
};

UseCount::UseCount(): p_counter(new int(1)) { }

UseCount::UseCount(const UseCount& u): p_counter(u.p_counter)
{
    ++(*p_counter);
}

UseCount::~UseCount()
{
    if (--(*p_counter) == 0)
    {
        delete p_counter;
    }
}

bool UseCount::only() { return *p_counter == 1; }

bool UseCount::reattach(const UseCount& u)
{
    ++(*u.p_counter);
    if(--*p_counter == 0)
    {
        delete p_counter;
        p_counter = u.p_counter;
        return true;
    }

    p_counter = u.p_counter;
    return false;
}

bool UseCount::makeonly()
{
    if(*p_counter == 1)
    {
        return false;
    }

    --(*p_counter);
    p_counter = new int(1);

    return true;
}



class Handle
{
    public:
        Handle();
        Handle(int, int);
        Handle(const Point&);
        Handle(const Handle&);

        Handle& operator=(const Handle&);

        ~Handle();

        int x() const;
        Handle& x(int);

        int y() const;
        Handle& y(int);

        void show_pointer(void)
        {
            cout << p_Point << endl;
        }

    private:
        // what we added
        Point *p_Point;
        UseCount u;
};

Handle::~Handle()
{
    if (u.only())
    {
        delete p_Point;
    }
}

/*
   Constructor function of @Handle
 */
Handle::Handle(): p_Point(new Point) { }
Handle::Handle(int x, int y): p_Point(new Point(x, y)) { }
Handle::Handle(const Point& p0): p_Point(new Point(p0)) { }
Handle::Handle(const Handle& h): u(h.u), p_Point(h.p_Point) { }

Handle&
Handle::operator = (const Handle& h)
{
    if(u.reattach(h.u))
    {
        delete p_Point;
    }

    p_Point = h.p_Point;
    return *this;
}

#ifndef COW

int Handle::x() const { return p_Point->x(); }
int Handle::y() const { return p_Point->y(); }

Handle& Handle::x(int x0)
{
    p_Point->x(x0);
    return *this;
}

Handle& Handle::y(int y0)
{
    p_Point->y(y0);
    return *this;
}

#else
/*
   "Copy On Write"
 */

int Handle::x() const
{
    return p_Point->x();
}

Handle& Handle::x(int x0)
{
    if(u.makeonly())
    {
        p_Point = new Point(*p_Point);
    }

    p_Point->x(x0);
    return *this;
}

#endif

int main()
{
    Handle h(3, 4);
    Handle h2 = h;

    cout << "before rewriting" << endl;
    h.show_pointer();
    h2.show_pointer();

    h2.x(5);

    cout << "after rewriting" << endl;
    h.show_pointer();
    h2.show_pointer();

    int n = h.x();

    cout << "n = " << n << endl;
    return 0;
}
```

You will get this output :)

![images](/images/img_for_2015_05_19/output.png)

-----------
Photo by Jason Leaster in XiangTan University

![images](/images/img_for_2015_05_19/scenery.png)

