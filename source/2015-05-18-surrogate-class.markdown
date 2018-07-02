---
layout: post
title: "Surrogate Class"
date: 2015-05-18 14:13:38 +0800
comments: true
categories: Cplusplus
---

Here is a question.

How could we design a type of class to enable it to include the classes which is different but have something in common ?

The solution is sorrogate

<!-- more -->

A demo about “Vehicle”:	

``` C++
/*
    Programmer  :  EOF
    Date        :  2015.05.18
    File        :  5.1.c
    E-mail      :  jasonleaster@gmail.com

 */

class Vehicle
{
    public:
        /*
           It will be OK, if you don't use virtual function.
        double weight() const;
        void start();
        */

        virtual double weight() const = 0;
        virtual void start() = 0;

        // ...
};

class RoadVehicle: public Vehicle {/* ... */};
class AutoVehicle: public RoadVehicle {/* ... */};
class Aircraft: public Vehicle {/* ... */};
class Helicopter: public Aircraft {/* ... */};

Vehicle parking_lot[1000];
```

If you gona to compile this source file, you will got errors and warnings like this:

images

The reason why we got error is that Vihecle is a base class, member function @weight and @start is pure virtual function. With the = 0 in the declaration, we declare that that pure virtual function could without definitaion. Only the class which is inherit from base class Vihecle.

So, the object Vehicle is not existed and we can’t define an array of that type.

If we change the virtual function into non-virtual function, we could pass the compile process.

But it’s very dangerous.

Eg, if we delete all pure virtual functions in Vehicle, what would happen?

``` C++
Vehicle parking_lot[1000];

Automobile x = /* ... */

Parking_lot[num_vihicles++] = x;
```

The answer is that there would be a cast on @x. @x is class Automobile which inhiret from base class Vehicle. @x will be cast into class Vehicle and then assign to object Parking_lot[num_vihicles]. In the process of casing, @x will lost anything which is not in Vehicle.

More explicit, the array Parking_lot is an array of class Vehicle but not Automobile.

It’s time release our scheme to solve that problem.

``` C++	

/**********************************************
Programmer  :   EOF
Date        :   2015.05.18
File        :   5.4.cpp
E-mail      :   jasonleaster@gmail.com

***********************************************/
#include <iostream>

class Vehicle
{
    public:
        /*
           It will be OK, if you don't use virtual function.
        double weight() const;
        void start();
        */

        virtual double weight() const = 0;
        virtual void start() = 0;
        virtual Vehicle* copy() const = 0;
        virtual ~Vehicle() { } ;
        // ...
};

class RoadVehicle: public Vehicle {/* ... */};
class AutoVehicle: public RoadVehicle {/* ... */};
class Aircraft: public Vehicle {/* ... */};
class Helicopter: public Aircraft {/* ... */};


class VehicleSurrogate
{
    public:
        /*
           Constructor
         */
        VehicleSurrogate();
        VehicleSurrogate(const Vehicle&);
        VehicleSurrogate(const VehicleSurrogate&);

        /*
           De-constructor
         */
        ~VehicleSurrogate();

        VehicleSurrogate& operator =(const VehicleSurrogate&);

        /*
           Operations from class Vehicle
         */
        double weight() const;
        void start();

        // ...

    private:
        Vehicle* vp;
};

VehicleSurrogate::VehicleSurrogate(): vp(0) { }
VehicleSurrogate::VehicleSurrogate(const Vehicle& v):
    vp(v.copy()) { }

VehicleSurrogate:: ~VehicleSurrogate()
{
    delete vp;
}

VehicleSurrogate::VehicleSurrogate (const VehicleSurrogate& v):
    vp(v.vp ? v.vp->copy() : 0) { }

VehicleSurrogate&
VehicleSurrogate::operator=(const VehicleSurrogate& v)
{
    if (this != &v)
    {
        delete vp;
        vp = (v.vp ? v.vp->copy() : 0);
    }

    return *this;
}

double VehicleSurrogate::weight() const
{
    if (vp == 0)
    {
        throw "empty VehicleSurrogate.weight()";
    }

    return vp->weight();
}

void VehicleSurrogate::start()
{
    if (vp == 0)
    {
        throw "empty VehicleSurrogate.start()";
    }

    vp->start();
}


class Automobile: public AutoVehicle
{
        double weight() const;
        void start();
        Vehicle* copy() const;
};

double
Automobile::weight() const
{
    return 0.0;
}

void
Automobile::start()
{
    std::cout << "Hello this is automobile" << std::endl;
}

Vehicle*
Automobile::copy() const
{
    return new Automobile(*this);
}



/*
   Here is our test unit.

   Attention! 
   Never use assignment between class objects out of function block.
   If your assignment operation is not in function block, you must got 
   an error from compiler.

   But it's ok, if you would like to define the varibles outside of blocks.
 */

VehicleSurrogate parking_lot[1000];

Automobile x;

int num_vehicles = 0;

int main()
{
    /*
       These two assignment statement are doing the same thing.
     */
    parking_lot[num_vehicles++] = x;

    //parking_lot[num_vehicles++] = VehicleSurrogate(x);

    return 0;
}
```

You may noticed that the key point of our solution is to use the class to represent a concept.

We defined a contructor without any parameters, so we define an array of this type of class.

If you have understand what we have done, you may feel excited. Yes, we create a container which could be used to represent any type of classes which are have relationships of inhiretance.

`parking_lot[num_vehicles++] = x;` is equal to `parking_lot[num_vehicles++] = VehicleSurrogate(x);`

This statement create a copy of object @x and attach @VehicleSurrogate onto object @x. If we destory elements in @parking_lot, the corresponding copy will be deleted.

_Finally, you have know that why this techonology named **Surrogate**._

The objects in this type of class could be used to represent others class objects which are have completly relationships of inhiretance.

-----------------
Margaret Heafield Hamilton who is a very famous computer scientist, systems engineer, and business owner. Picture blew there is that hamilton during the Apollo Program.

![images](/images/img_for_2015_05_18/margaret.png)

