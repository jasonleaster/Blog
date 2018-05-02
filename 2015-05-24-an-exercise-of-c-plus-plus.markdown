---
layout: post
title: "An Exercise of C++"
date: 2015-05-24 13:57:36 +0800
comments: true
categories: Cplusplus
---

In << The C++ Programming Language >> , there is a exercise for learner who is studying C++. It’s that student should write a program in C++ to manipulate “character picture”, something like this.

![images](/images/img_for_2015_05_24/output.png)

<!-- more -->

To abstract that picture, we use this `class Picture` in the header file.

	
``` C++
/*******************************************************
Programmer  :   EOF
Date        :   2015.05.22
File        :   picture.h
E-mail      :   jasonleaster@gmail.com

Description:
    Here is a demo in << Ruminations on C++ >>.

*******************************************************/

#include <iostream>
#include <string.h>

using namespace std;

class Picture
{
    /*
       All implementation of member function and friend functions
       could be found in "picture.cpp"
     */
    friend ostream& operator<<(ostream&, const Picture&);
    friend Picture frame(const Picture&);
    friend Picture operator & (const Picture&, const Picture&);
    friend Picture operator | (const Picture&, const Picture&);

    public:
        Picture(): height(0), width(0), data(0) { }
        Picture(const char* const*, int);
        Picture(const Picture&);
        ~Picture() { delete [] data; }

        Picture& operator = (const Picture&);

    private:
        int height;
        int width;

        char* data;
        void copyblock(int, int, const Picture&);

        /*
           This function is so cool! Don't you think so? :)
         */
        char& position(int row, int col)
        {
            return data[row * width + col];
        }

        char position(int row, int col) const
        {
            return data[row * width + col];
        }

        void clear(int, int, int, int);
        void init(int, int);
        static int max(int, int);
};
```

Here we should support three different operation to our user.

* Adding frame for picture.
* Connecting picture vertically
* Connecting picture horizontal

Aha~ We support that operation by provide there interfaces.

``` C++
Picture frame(const Picture& p);
Picture operator & (const Picture& p, const Picture& q);
Picture operator | (const Picture& p, const Picture& q);
```

We could test our implementation by this test program :)
	
``` C++
#include "picture.h"

using namespace std;

char* init[] = {"Paris", "in the", "Spring"};

int main()
{
    Picture p(init, 3);
    cout << p << endl;

    p = frame(p);
    cout << p << endl;

    p = p & p;
    cout << p << endl;


    p = p | p;
    cout << p << endl;

    Picture q(init,2);

    cout << q << endl;
    return 0;
}
```

Here is my implementation: file: picture.cpp
	
``` C++
/***************************************************
    Programmer  :   EOF
    File        :   picture.cpp
    Date        :   2015.05.24
    E-mail      :   jasonleaster@gmail.com

 ***************************************************/
#include "picture.h"

/*
   public functions of class @Picture
 */
Picture::Picture(const char* const* array, int n)
{
    int w = 0;
    int i = 0;

    for ( i = 0; i < n; i++)
    {
        w = Picture::max(w, strlen(array[i]));
    }

    init(n, w);

    for (i = 0; i < n; i++)
    {
        const char* src = array[i];
        int len = strlen(src);
        int j = 0;

        while(j < len)
        {
            position(i, j) = src[j];
            j++;
        }

        while(j < width)
        {
            position(i, j) = ' ';
            j++;
        }
    }
}

Picture::Picture(const Picture& p):
    height(p.height), width(p.width),
    data(new char[p.height * p.width])
{
    copyblock(0, 0, p);
}

Picture& Picture::operator=(const Picture& p)
{
    if(this != &p)
    {
        delete [] data;
        init(p.height, p.width);
        copyblock(0, 0, p);
    }

    return *this;
}

/*
    Private functions
 */
void Picture::copyblock(int row, int col, const Picture& p)
{
    for (int i = 0; i < p.height; i++)
    {
        for(int j = 0; j < p.width; j++)
        {
            position(i + row, j + col) = p.position(i, j);
        }
    }
}

void Picture::clear(int r1, int c1, int r2, int c2)
{
    for(int r = r1; r < r2; r++)
    {
        for(int c = c1; c < c2; c++)
        {
            position(r, c) = ' ';
        }
    }
}

void Picture::init(int h, int w)
{
    height = h;
    width  = w;
    data   = new char[height * width];
}

int Picture::max(int m, int n)
{
    return m > n ? m : n;
}
/*
   Friend functions.
 */
ostream& operator<<(ostream& o, const Picture& p)
{
    for (int i = 0; i < p.height; i++)
    {
        for (int j = 0; j < p.width; j++)
        {
            o << p.position(i, j);
        }

        o << endl;
    }

    return o;
}

/*
    This function @frame() will help us to add frame 
   into the old picture which was referenced by @p.
 */
Picture frame(const Picture& p)
{
    Picture r;

    r.init(p.height + 2, p.width + 2);

    for(int i = 1; i < r.height - 1; i++)
    {
        r.position(i, 0) = '|';
        r.position(i, r.width - 1) = '|';
    }

    for( int j = 1; j < r.width; j++)
    {
        r.position(0, j) = '-';
        r.position(r.height - 1, j) = '-';
    }

    r.position(0, 0) = '+';
    r.position(0, r.width - 1) = '+';
    r.position(r.height - 1, 0) = '+';
    r.position(r.height - 1, r.width - 1) = '+';

    r.copyblock(1, 1, p);
    return r;
}

/*
   We redefine the @& operator to connect two picture 
   @p and @q vertically.

   The height of the new picture will be @p.height + @q.height.
   The width of the new one will be max value between 
   @p.width and @q.width

   We also should clear the new empty region for alignment,
   if the size of the inputed two pictures are different.

 */
Picture operator & (const Picture& p, const Picture& q)
{
    Picture r;

    r.init(p.height + q.height, Picture::max(p.width, q.width));
    r.clear(0, p.width,  p.height, r.width);
    r.clear(p.height, q.width, r.height, r.width);

    r.copyblock(0, 0, p);
    r.copyblock(p.height, 0, q);

    return r;
}

/*
   Operator @| is like @& but different.
   This operation connect two pictures @p and @q horizonal.
 */
Picture operator | (const Picture& p, const Picture& q)
{
    Picture r;

    r.init(Picture::max(p.height, q.height),
            p.width + q.width);

    r.clear(p.height, 0, r.height, p.width);
    r.clear(q.height, p.width, r.height, r.width);

    r.copyblock(0, 0, p);
    r.copyblock(0, p.width, q);

    return r;
}
```

You know that that solution is not hard and is easy for us to understand. But there is a weakpoint that we lose the basic intern information of the old picture which we used to connect and create a new picture.

**we can’t seperate the real frame and pure picture after we finished our @frame operation.**

The same problem exist in others operation. We lose information which may be very useful for us in future.

So… Refactoring is needed.

### The second solution

The header file which contains the class that is used for abstrcation.

``` C++
/***********************************************
    Programmer  :   EOF
    Date        :   2015.05.22
    File        :   picture.h
    E-mail      :   jasonleaster@gmail.com

 ***********************************************/
#include <iostream>
#include <string>
#include <string.h>

using namespace std;

class Picture;

class P_Node;// base class
class String_Pic;
class Frame_Pic;
class VCat_Pic;
class HCat_Pic;


class Picture
{
    friend ostream& operator<<(ostream&, const Picture&);
    friend Picture frame(const Picture&);
    friend Picture operator& (const Picture&, const Picture&);
    friend Picture operator| (const Picture&, const Picture&);

    public:
        Picture():p(0) {} ;
        Picture(const char* const*, int);
        Picture(const Picture&);
        ~Picture();

        Picture& operator=(const Picture&);

        friend class String_Pic;
        friend class Frame_Pic;
        friend class HCat_Pic;
        friend class VCat_Pic;

    private:
        P_Node *p;

        Picture(P_Node* p_node) : p(p_node) { };

        int height() const;
        int width() const;
        void display(ostream&, int, int) const;
};

class P_Node
{
    friend class Picture;

    protected:
        P_Node(): use(1) { };

        virtual int height() const = 0;
        virtual int width() const = 0;
        virtual void display(ostream&, int, int) const = 0;
        virtual ~P_Node() { };

        int max(int x, int y) const;

    private:
        int use;
};

class String_Pic: public P_Node
{
    friend class Picture;

    public:
        String_Pic(const char* const*, int);
        ~String_Pic();

        int height() const;
        int width() const;
        void display(ostream&, int, int) const;

    private:
        char**data;
        int size;
};

class Frame_Pic: public P_Node
{
    friend Picture frame(const Picture&);
    public:
        Frame_Pic(const Picture&);

        int height() const;
        int width() const;
        void display(ostream&, int, int) const;

    private:
        Picture p;
};

class VCat_Pic: public P_Node
{
    friend Picture operator& (const Picture&, const Picture&);

    public:
        VCat_Pic(const Picture&, const Picture&);

        int height() const;
        int width() const;
        void display(ostream&, int, int) const;

    private:
        Picture top, bottom;
};

class HCat_Pic: public P_Node
{
    friend Picture operator | (const Picture&, const Picture&);
    public:
        HCat_Pic(const Picture&, const Picture&);

        int height() const;
        int width() const;
        void display(ostream&, int, int) const;

    private:
        Picture left, right;
};
```

Member functions are all implemented in this file “picture.cpp”.
	
``` C++
/********************************************************
    Programmer  :   EOF
    File        :   picture.cpp
    Date        :   2015.05.24
    E-mail      :   jasonleaster@gmail.com

 *******************************************************/

#include "picture.h"

/*
   Member function of @class-P_Node
 */
int P_Node::max(int x, int y) const
{
    return x > y ? x : y;
}

/*
   Member function of @class-Picture
 */
Picture::Picture(const char* const *str, int n):
        p(new String_Pic(str, n))
{
}

Picture::Picture(const Picture& orig):p(orig.p)
{
    orig.p->use++;
}

Picture::~Picture()
{
    if(--p->use == 0)
    {
        delete p;
    }
}

Picture&
Picture::operator=(const Picture& orig)
{
    orig.p->use++;
    if(--p->use == 0)
    {
        delete p;
    }

    p = orig.p;

    return *this;
}

int Picture::height() const
{
    return p->height();
}

int Picture::width() const
{
    return p->width();
}

void Picture::display(ostream& o, int x, int y) const
{
    p->display(o, x, y);
}

ostream&
operator << (ostream& os, const Picture& p)
{
    int ht = p.height();
    int wd = p.width();

    for(int i = 0; i < ht; i++)
    {
        p.display(os, i, wd);
        os << endl;
    }

    return os;
}


/*
   Member function of @class-String_Pic
 */
String_Pic::String_Pic(const char* const *p, int n):
    data(new char* [n]), size(n)
{
    for(int i = 0; i < n; i++)
    {
        data[i] = new char[strlen(p[i] + 1)];
        strcpy(data[i], p[i]);
    }
}

String_Pic::~String_Pic()
{
    for(int i = 0; i < size; i++)
    {
        delete [] data[i];
    }
    delete [] data;
}

int String_Pic::height() const
{
    return size;
}

int String_Pic::width() const
{
    int n = 0;

    for(int i = 0; i < size; i++)
    {
        n = max(n, strlen(data[i]));
    }
    return n;
}

static void pad(ostream& os, int x, int y)
{
    for (int i = x; i < y; i++)
    {
        os << " ";
    }
}

void String_Pic::display(ostream& os, int row, int width) const
{
    int start = 0;

    if(row >= 0 && row < height())
    {
        os << data[row];
        start = strlen(data[row]);
    }

    pad(os, start, width);
}


/*
   Member function of @class-Frame_Pic
 */
Frame_Pic::Frame_Pic(const Picture& pic):
    p(pic)
{ }

int Frame_Pic::height() const
{
    return p.height() + 2;
}

int Frame_Pic::width() const
{
    return p.width() + 2;
}

void Frame_Pic::display(ostream& os, int row, int wd) const
{
    if(row < 0 || row >= height())
    {
        //run-across ??
        pad(os, 0, wd);
    }
    else
    {
        if(row == 0 || row == height() -1)
        {
            os << "+";
            int i = p.width();
            while(--i >= 0)
            {
                os << "-";
            }

            os << "+";
        }
        else
        {
            os << "|";
            p.display(os, row - 1, p.width());
            os << "|";
        }
        pad(os, width(), wd);
    }
}

Picture frame(const Picture& pic)
{
    return new Frame_Pic(pic);
}

/*
   Member functions of @class-VCat_Pic
 */
VCat_Pic::VCat_Pic(const Picture& t, const Picture& b):
        top(t), bottom(b)
{ }

int VCat_Pic::height() const
{
    return top.height() + bottom.height();
}

int VCat_Pic::width() const
{
    return max(top.width(), bottom.width());
}

void VCat_Pic::display(ostream& os, int row, int wd) const
{
    if(row >= 0 && row < top.height())
    {
        top.display(os, row, wd);
    }
    else if(row < top.height() + bottom.height())
    {
        bottom.display(os, row-top.height(), wd);
    }
    else
    {
        pad(os, 0, wd);
    }
}

Picture operator & (const Picture& t, const Picture& b)
{
    return new VCat_Pic(t, b);
}

/*
   Member functions of @class-HCat_Pic
 */

HCat_Pic::HCat_Pic(const Picture& l, const Picture& r): left(l), right(r) {}

int HCat_Pic::height() const
{
    int n = max(left.height(), right.height());
    return n;
}

int HCat_Pic::width() const
{
    return left.width() + right.width();
}

void HCat_Pic::display(ostream& os, int row, int wd) const
{
    left.display(os, row, left.width());
    right.display(os, row, right.width());
    pad(os, width(), wd);
}

Picture operator | (const Picture& l, const Picture& r)
{
    return new HCat_Pic(l, r);
}
```

You could use this test program to test our solution :)	

``` C++
#include "picture.h"

using namespace std;

char* init[] = {"Paris", "in the", "Spring"};

int main()
{
    Picture p(init, 3);
    cout << p << endl;

    cout << frame(p) << endl;

    cout << (frame(p) | frame(p)) << endl;

    cout << (frame(p) & frame(p)) << endl;

    return 0;
}
```

--------
Photo by YanZixin in Changde, China 

![images](/images/img_for_2015_05_24/tree.png)
