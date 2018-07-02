---
layout: post
title: "Memory Model of Objects in C++"
date: 2015-06-13 19:48:42 +0800
comments: true
categories: Cplusplus
---

In this blog, I assume that you have a basic background about C++ and know that there is a “monster” we call it as `virtual table` :)

If there is virtual member function in your class’s definition, there will be a virtual table.

Compiler will generate a pointer which’s name is `_vprt` and use this pointer to find the virtual table.

<!-- more -->

The function below there is very important for us to understand the C++ object model.
	
``` C++
template<class T>
unsigned long* get_element(T &obj, int offset)
{
    return ((unsigned long*)*((unsigned long*)(&obj))) + offset ;
}
```

Function `get_element` take the reference of object which’s type is T(I used template technology) and `offset`.

We get the address of this object by `&obj` and then we cast it into pointer which point to unsigned long and then we dereference that pointer. Acctually, the pointer which we dereferenced in function `get_element` is the `_vprt`.

After we get `_vprt`, we cast it into `unsigned long*` again and add `offset` to get others pointers in virtual table which contains all pointers that point to class virtual member functions.

If we dereference the return value of this function, we will get the pointer which point to the begin of function. Fantastic :)

### Single Object

First of all, let’s take a glance at this demo:
	
``` C++
/*******************************************************************
Programmer  :   EOF
Date        :   2015.06.13
File        :   meomory_model_for_single_object.cpp
E-mail      :   jasonleaster@gmail.com

 ******************************************************************/

#include <iostream>

using namespace std;

typedef  void (*FUN) (void);

class Base
{
    public:

        int num;

        Base():prv_data(100), num(100){ }

        virtual void f(void){ cout << " Base::f()" << endl; }
        virtual void g(void){ cout << " Base::g()" << endl; }
        virtual void h(void){ cout << " Base::h()" << endl; }

        void show()
        {
            cout << "Base::num "        << num << endl;
            cout << "Base::prv_data "   << prv_data << endl;
            cout << "&Base::num "       << &num << endl;
            cout << "&Base::prv_data "  << &prv_data << endl;
        }

    private:
        /*
            Private Data
         */
        int prv_data;
};

template<class T>
unsigned long* get_element(T &obj, int offset)
{
    return ((unsigned long*)*((unsigned long*)(&obj))) + offset ;
}

int main()
{
    Base b;

    b.show();

    for(int i = 0; i < 3; i++)
    {
        /*
            The address which point virtual
            function's implementation in virtual table
        */
        cout << "Pointer in Virutal Table: " << (unsigned long*)*get_element(b, i) << endl;

        /* Calling function by pointer in virtual table */
        ((FUN)(*get_element(b, i))) ();
    }

    return 0;
}
```

There are data member `num` and `prv_data` with different access label in `class base`. There also have virtual functions and normal member functions in this class. So … what’s the memory model of this class look like?

![images](/images/img_for_2015_06_12/single_object_memory_model.png)

The output of that demo:

![images](/images/img_for_2015_06_12/output1.png)


### Single Inheritance

The inheritance relationship between the base class and derived class :

![images](/images/img_for_2015_06_12/single_inheritance_model.png)

It’s single inheritance that base class must be only single type.


``` C++
/*******************************************************************
Programmer  :   EOF
Date        :   2015.06.12
File        :   virtual_function_model.cpp
E-mail      :   jasonleaster@gmail.com

 ******************************************************************/

#include <iostream>

using namespace std;

typedef  void (*FUN) (void);

class Base
{
    public:

        int num;

        Base():prv_data(100), num(100) { }

        virtual void f(void){ cout << " Base::f()" << endl; }
        virtual void g(void){ cout << " Base::g()" << endl; }
        virtual void h(void){ cout << " Base::h()" << endl; }

        void show()
        {
            cout << "Base::num "        << num << endl;
            cout << "Base::prv_data "   << prv_data << endl;
            cout << "&Base::num "       << &num << endl;
            cout << "&Base::prv_data "  << &prv_data << endl;
        }

    private:
        /*
           Private  Data
         */
        int prv_data;

};

class Derived : public Base
{
    public:

        int num;

        Derived():num(200), prv_data(200) { }

        virtual void x(void){ cout << " Derived::x()" << endl; }
        virtual void y(void){ cout << " Derived::y()" << endl; }
        virtual void f(void){ cout << " Derived::f()" << endl; }
        virtual void g(void){ cout << " Derived::g()" << endl; }

        void show()
        {
            cout << "Derived::num "        << num << endl;
            cout << "Derived::prv_data "   << prv_data << endl;
            cout << "&Derived::num "       << &num << endl;
            cout << "&Derived::prv_data "  << &prv_data << endl;

            Base::show();
        }

    private:
        int prv_data;
};

template<class T>
unsigned long* get_element(T &obj, int offset)
{
    return ((unsigned long*)*((unsigned long*)(&obj))) + offset ;
}

int main()
{
    Base b;
    Derived d;

    b.show();

    cout << endl;

    d.show();

    cout << endl;

    for(int i = 0; i < 3; i++)
    {
        /*
            The address which point virtual
            function's implementation in virtual table
        */
        cout << "Pointer in Virutal Table: " << (unsigned long*)*get_element(b, i) << endl;

        /* Calling function by pointer in virtual table */
        ((FUN)(*get_element(b, i))) ();
    }

    cout << endl;

    for(int i = 0; i < 5; i++)
    {
        cout << "Pointer in Virutal Table: " << (unsigned long*)*get_element(d, i) << endl;
        ((FUN)(*get_element(d, i))) ();
    }

    return 0;
}
```

Memory Model:

![images](/images/img_for_2015_06_12/single_inheritance_model.png)

Output:

![images](/images/img_for_2015_06_12/output2.png)

We find that:

* The virtual table pointer _vprt is at the beginning of the object.

* Data member store into object and have nothing with the access label but according to the sequence of declaration.

* In single inheritance model, the virtual function wichi is re-implement will be update in all virtual table.


### Multiply Inheritance

Now, you find that there is only one virtual table in our object under single inheritance situation. But … How about things going on with multiply inheritance(MI) ?

![images](/images/img_for_2015_06_12/multiple_inheritance_model.png)

Let’s go and test it.

``` C++
/*******************************************************************
Programmer  :   EOF
Date        :   2015.06.13
File        :   virtual_function_for_multiple_inheritance.cpp
E-mail      :   jasonleaster@gmail.com

 ******************************************************************/

#include <iostream>

using namespace std;

typedef  void (*FUN) (void);

class Base_1
{
    public:

        int num;

        Base_1():prv_data(100), num(100) { }

        virtual void f(void){ cout << " Base_1::f() \t"; }
        virtual void g(void){ cout << " Base_1::g() \t"; }
        virtual void h(void){ cout << " Base_1::h() \t"; }

    private:
        /*
           Private  Data
         */
        int prv_data;

};

class Base_2
{
    public:

        int num;

        Base_2():prv_data(200), num(200) { }

        virtual void f(void){ cout << " Base_2::f() \t"; }
        virtual void x(void){ cout << " Base_2::x() \t"; }

    private:
        /*
           Private  Data
         */
        int prv_data;

};

class Base_3
{
    public:

        int num;

        Base_3():prv_data(300), num(300) { }

        virtual void g(void){ cout << " Base_3::g() \t"; }

    private:
        /*
           Private  Data
         */
        int prv_data;

};

class Derived : public Base_3, public Base_2, public Base_1
{
    public:

        int num;

        Derived():num(42), prv_data(42) { }

        virtual void y(void){ cout << " Derived::y() \t" ; }

        /*
           Here we re-implement the virtual function @f().
           Compiler will rewrite the virtual table
         */
        virtual void f(void){ cout << " Derived::f() \t"; }


    private:
        int prv_data;
};

template<class T>
unsigned long* get_element(T &obj, int offset = 0, int vprt_offset = 0)
{
    return ((unsigned long*)*((unsigned long*)(&obj) + vprt_offset)) + offset ;
}

template<class T>
void call_vir_func(T &obj, int offset = 0, int vprt_offset = 0)
{
    ((FUN)(*get_element(obj, offset, vprt_offset))) ();
    cout << "Address of function: " ;
    cout << (int*)*get_element(obj, offset, vprt_offset) << endl;
}

int main()
{
    Derived d;

    cout << "Virtual Table of @Base_3 and @Derived:" << endl;
    call_vir_func(d, 0, 0);
    call_vir_func(d, 1, 0);
    call_vir_func(d, 2, 0);

    cout << "Virtual Table of @Base_2:" << endl;

    call_vir_func(d, 0, sizeof(Base_3)/8);
    call_vir_func(d, 1, sizeof(Base_3)/8);

    cout << "Virtual Table of @Base_1:" << endl;
    call_vir_func(d, 0, sizeof(Base_3)/8 + sizeof(Base_2)/8);
    call_vir_func(d, 1, sizeof(Base_3)/8 + sizeof(Base_2)/8);
    call_vir_func(d, 2, sizeof(Base_3)/8 + sizeof(Base_2)/8);

    return 0;
}
```

Memory Model:

![images](/images/img_for_2015_06_12/multiple_inheritance.png)

Output:

![images](/images/img_for_2015_06_12/output3.png)

We can declare some conclusions that:

* Each base class have their own’s virtual table. Assume that The number of base class is N in multiple inheritance. The number of the virtual table is N-1.The virtual table of derived function will be combine into the first base class in the declaration queue of base class.

* The base virtual table will be update if the derived class rewrite implementation of virtual function in base class.

### Repeat Inheritance

![images](/images/img_for_2015_06_12/repeat_inheritance_model.png)

``` C++
/*******************************************************************
Programmer  :   EOF
Date        :   2015.06.14
File        :   virtual_function_for_repreat_inheritance.cpp
E-mail      :   jasonleaster@gmail.com

 ******************************************************************/

#include <iostream>

using namespace std;

typedef  void (*FUN) (void);

class Base
{
    public:

        int num;

        Base():prv_data(100), num(100) { }

        virtual void f(void){ cout << " Base::f()" << endl; }
        virtual void g(void){ cout << " Base::g()" << endl; }
        virtual void h(void){ cout << " Base::h()" << endl; }

        void show()
        {
            cout << "Base::num "        << num << endl;
            cout << "Base::prv_data "   << prv_data << endl;
            cout << "&Base::num "       << &num << endl;
            cout << "&Base::prv_data "  << &prv_data << endl;
        }


    private:
        /*
           Private  Data
         */
        int prv_data;

};

class Derived_1: public Base
{
    public:

        int num;

        Derived_1():prv_data(200), num(200) { }

        virtual void f(void){ cout << " Derived_1::f()" << endl; }
        virtual void x(void){ cout << " Derived_1::x()" << endl; }

        void show()
        {
            cout << "Derived_1::num "        << num << endl;
            cout << "Derived_1::prv_data "   << prv_data << endl;
            cout << "&Derived_1::num "       << &num << endl;
            cout << "&Derived_1::prv_data "  << &prv_data << endl;

            Base::show();
        }

    private:
        /*
           Private  Data
         */
        int prv_data;

};

class Derived_2: public Base
{
    public:

        int num;

        Derived_2():prv_data(300), num(300) { }

        virtual void g(void){ cout << " Derived_2::g()" << endl; }

        void show()
        {
            cout << "Derived_2::num "        << num << endl;
            cout << "Derived_2::prv_data "   << prv_data << endl;
            cout << "&Derived_2::num "       << &num << endl;
            cout << "&Derived_2::prv_data "  << &prv_data << endl;

            Base::show();
        }

    private:
        /*
           Private  Data
         */
        int prv_data;

};

class Derived : public Derived_2, public Derived_1
{
    public:

        int num;

        Derived():num(42), prv_data(42) { }

        virtual void y(void){ cout << " Derived::y()" << endl; }

        /*
           Here we re-implement the virtual function @f().
           Compiler will rewrite the virtual table
         */
        virtual void f(void){ cout << " Derived::f()" << endl; }

        void show()
        {
            cout << "Derived::num "        << num << endl;
            cout << "Derived::prv_data "   << prv_data << endl;
            cout << "&Derived::num "       << &num << endl;
            cout << "&Derived::prv_data "  << &prv_data << endl;

            Derived_1::show();
            Derived_2::show();
        }


    private:
        int prv_data;
};

template<class T>
unsigned long* get_element(T &obj, int offset = 0, int vprt_offset = 0)
{
    return ((unsigned long*)*((unsigned long*)(&obj) + vprt_offset)) + offset ;
}

template<class T>
void call_vir_func(T &obj, int offset = 0, int vprt_offset = 0)
{
    ((FUN)(*get_element(obj, offset, vprt_offset))) ();
    cout << "\t Address of function: " ;
    cout << (int*)*get_element(obj, offset, vprt_offset) << endl;
}

int main()
{
    Derived d;

    cout << "Virtual Table of @Derived, @Derived_2 and @Base:" << endl;
    call_vir_func(d, 0, 0);
    call_vir_func(d, 1, 0);
    call_vir_func(d, 2, 0);
    call_vir_func(d, 3, 0);


    cout << "Virtual Table of @Derived_1 and @Base:" << endl;
    call_vir_func(d, 0, sizeof(Derived_2)/8);
    call_vir_func(d, 1, sizeof(Derived_2)/8);
    call_vir_func(d, 2, sizeof(Derived_2)/8);
    call_vir_func(d, 3, sizeof(Derived_2)/8);

    d.show();
    return 0;
}
```

Memory Model:

![images](/images/img_for_2015_06_12/repeat_inheritance.png)

You find that The base object was inherited again and again Output:

![images](/images/img_for_2015_06_12/output4.png)

It’s may not what we want that the base class is inherited repeatedly. So C++ use the `virtual base` technology to solve this problem :)


### Diamond Inheritance

![images](/images/img_for_2015_06_12/diamond_inheritance_model.png)

``` C++
/*******************************************************************
Programmer  :   EOF
Date        :   2015.06.14
File        :   virtual_function_for_diamond_inheritance.cpp
E-mail      :   jasonleaster@gmail.com

 ******************************************************************/

#include <iostream>

using namespace std;

typedef  void (*FUN) (void);

class Base
{
    public:

        int num;

        Base():prv_data(100), num(100) { }

        virtual void f(void){ cout << " Base::f()" << endl; }
        virtual void g(void){ cout << " Base::g()" << endl; }
        virtual void h(void){ cout << " Base::h()" << endl; }

        void show()
        {
            cout << "Base::num "        << num << endl;
            cout << "Base::prv_data "   << prv_data << endl;
            cout << "&Base::num "       << &num << endl;
            cout << "&Base::prv_data "  << &prv_data << endl;
        }


    private:
        /*
           Private  Data
         */
        int prv_data;

};

class Derived_1: virtual public Base
{
    public:

        int num;

        Derived_1():prv_data(200), num(200) { }

        virtual void f(void){ cout << " Derived_1::f()" << endl; }
        virtual void x(void){ cout << " Derived_1::x()" << endl; }

        void show()
        {
            cout << "Derived_1::num "        << num << endl;
            cout << "Derived_1::prv_data "   << prv_data << endl;
            cout << "&Derived_1::num "       << &num << endl;
            cout << "&Derived_1::prv_data "  << &prv_data << endl;

            Base::show();
        }

    private:
        /*
           Private  Data
         */
        int prv_data;

};

class Derived_2: virtual public Base
{
    public:

        int num;

        Derived_2():prv_data(300), num(300) { }

        virtual void g(void){ cout << " Derived_2::g()" << endl; }

        void show()
        {
            cout << "Derived_2::num "        << num << endl;
            cout << "Derived_2::prv_data "   << prv_data << endl;
            cout << "&Derived_2::num "       << &num << endl;
            cout << "&Derived_2::prv_data "  << &prv_data << endl;

            Base::show();
        }

    private:
        /*
           Private  Data
         */
        int prv_data;

};

class Derived : public Derived_2, public Derived_1
{
    public:

        int num;

        Derived():num(42), prv_data(42) { }

        virtual void y(void){ cout << " Derived::y()" << endl; }

        /*
           Here we re-implement the virtual function @f().
           Compiler will rewrite the virtual table
         */
        virtual void f(void){ cout << " Derived::f()" << endl; }

        void show()
        {
            cout << "Derived::num "        << num << endl;
            cout << "Derived::prv_data "   << prv_data << endl;
            cout << "&Derived::num "       << &num << endl;
            cout << "&Derived::prv_data "  << &prv_data << endl;

            Derived_1::show();
            Derived_2::show();
        }


    private:
        int prv_data;
};

template<class T>
unsigned long* get_element(T &obj, int offset = 0, int vprt_offset = 0)
{
    return ((unsigned long*)*((unsigned long*)(&obj) + vprt_offset)) + offset ;
}

template<class T>
void call_vir_func(T &obj, int offset = 0, int vprt_offset = 0)
{
    ((FUN)(*get_element(obj, offset, vprt_offset))) ();
    cout << "\t Address of function: " ;
    cout << (int*)*get_element(obj, offset, vprt_offset) << endl;
}

int main()
{
    Derived d;
//    d.show();

    cout << "Virtual Table of @Derived::Derived_2" << endl;
    call_vir_func(d, 0, 0);
    call_vir_func(d, 1, 0);
    call_vir_func(d, 2, 0);

    cout << "Virtual Table of @Derived::Derived_1" << endl;
    call_vir_func(d, 0, (sizeof(Derived_2) - sizeof(Base))/8);
    call_vir_func(d, 1, (sizeof(Derived_2) - sizeof(Base))/8);
/*
    cout << (int*)&d << endl;
    cout << (int*)&d.Derived_2::num << endl;
    cout << (int*)&d.Derived_1::num << endl;
    cout << (int*)&d.num << endl;
    cout << (int*)&d.Base::num << endl;
*/
    cout << "Virtual Table of @Derived::Derived_2::Base" << endl;

    call_vir_func(d, 0, (sizeof(Derived) - sizeof(Base))/8);
    call_vir_func(d, 1, (sizeof(Derived) - sizeof(Base))/8);
    call_vir_func(d, 2, (sizeof(Derived) - sizeof(Base))/8);

    return 0;
}
```

It’s a hard time to draw this picture. But thank god… I make it.

Memory Model:

![images](/images/img_for_2015_06_12/diamond_inheritance.png)

Output:

![images](/images/img_for_2015_06_12/output5.png)

ATTENTION: **Every class in C++ only have ONE virtual table, different objects of the same class will share the same virtual table of that class!**

----------

Photo by Jason Leaster images

![images](/images/img_for_2015_06_12/end.png)
