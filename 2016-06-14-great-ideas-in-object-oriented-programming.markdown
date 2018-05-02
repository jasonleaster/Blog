---
layout: post
title: "Great ideas in Object Oriented Programming"
date: 2016-06-14 12:49:00 +0800
comments: true
categories: Java
---

> Object-oriented programming (OOP) is a programming paradigm based on the concept of "objects", which may contain data, in the form of fields, often known as attributes; and code, in the form of procedures, often known as methods. -- [Wikipedia](https://en.wikipedia.org/wiki/Object-oriented_programming)

OOP is just a programming paradigm and it helps programmer to build more robust and implement more abstract concept which are needed in their project.OOP isn't the treasure which only belongs to a special programming language. It offer a way to make the problem more easy to be understood and to be solved.

There are three gread ideas in OOP.

* Encapsulation 
* Composition, inheritance, and delegation
* Polymophsim 

<!-- more -->

### Encapsulation

The most basic idea in OOP is that each object encapsulates some data and code. The object takes requests from other client object. The object alone is responsible for its own state, exposing public message for clients, and declaring private the variables and the methods. The client depends on the simple public interface, and does not known about or depend on the details of the implementation.

**Encapsulation remind us to put details and complexity into a black box. The outside people(user) don't need to care about how this box was built but just use it with the instructions which the box suppled.**

Q: What if we want to implement a abstract concept in mathematic -- Point. 

Point, the most basic presentation of point is two coordinates in Descartes coordinate system(x, y). How to implement it with program ?

Consider the following implementation in java.

``` java

[MyPoint1.java]

public class MyPoint1 {
    public int x;   // x and y are public
    public int y;
                
    public MyPoint1(int x,int y) {
        this.x = x;
        this.y = y;
    }

    // Client Code
    public static void main(String[] args){

        MyPoint1 p = new MyPoint1(1, 2);

        /**
         * Bad style to try to use the detail information of a class.
         * Here, client programmer should not use data member of the class directly.
         * With the software iteration and release the library which implement the
         * class which have used in your project. You may have to rewrite all your
         * client program. Because the author of that library may change the 
         * implementation of that class and your code can't be used with the 
         * new library.
         */
        int z = p.x + p.y;// !! Bad style.

        System.out.print(z);
    }
}

```

Here all information is public, this is the most convenient for end users, but limits our ability to change things later in development. What would happend if you build a software and you find something should be changed with the implementation of `MyPoint1`. You may want to add more attributes into this class. You may also want to rewrite your code and change it into another coordinate system but not descartes coordinate system.

If the others module depends on your implementation of `MyPoints1` and use the data member of that class directly, you may have to rewrite your whole project and it will cost your a lot of time. It will be nightmare for programmer to be told "Hey, buddy. There is something changed with requirement, you have to rewrite your whole project. Just put the code into garbage collector. It can't be modified to satisfy the new requirement."

![images](/images/img_for_2016_06_14/wth.png)

Here is a bettern one solution. In `MyPoint2`, the detail of this class are setted as private which means that the client(user) can't use the information about the implementation directly. The only way that the user can use the information which they need is to use the interface that the class supply with. In this example, the interface which is public can access _getX()_  and _getY()_ .

``` java 

[MyPoint2.java]
public class MyPoint2 {

    private int x;  // x and y are private

    private int y;
                
    public MyPoint2(int x,int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() {
        return x;
    }

    public int getY() {
        return y;
    }

    // Client Code
    public static void main(String[] args){
        MyPoint2 p = new MyPoint2(1, 2);
        int z = p.getX() + p.getY(); // Good Style :)
        System.out.print(z);
    }

}

```

The advantage of accessing control and encapsulation is that we can replace the current implementation into another better one and make sure the client don't need to be modified for this changes in the original class(MyPoint2).

Here, we implement Point with polar coordinate system. All the interface that set as public in `MyPoint2` are unchanged. This make sure that the client which use `getX()` and `getY()` don't need to be modified for changes with the implementation of Point.

``` java

[MyPoint3.java]
public class MyPoint3 {
    private double r;
    private double theta;
                
    public MyPoint3(int x,int y) {
        this.r = Math.sqrt(x * x + y * y);
        theta = Math.atan( (double) y / x);
    }
                    
    public long getX() {
        return Math.round(r * Math.cos(theta));
    }
                        
    public long getY() {
        return Math.round(r * Math.sin(theta));
    }

    /**
     * Client Code don't need to be modified after the implemetation of Point changed
     */
    public static void main(String[] args){
        MyPoint2 p = new MyPoint2(1, 2);
        int z = p.getX() + p.getY(); // Good Style :)
        System.out.print(z);
    }
                            
}

```


### Composition, inheritance, and delegation

In object-oriented programming, inheritance is when an object or class is based on another object (prototypal inheritance) or class (class-based inheritance), using the same implementation (inheriting from an object or class) specifying implementation to maintain the same behavior (realizing an interface; inheriting behavior). In this ariticle, we only talk about the class-based inheritance.

What's the adavantages of inheritance ?

* Overriding
* Code Reuse

Many OOP programming language permit to replace the implementation of one or more method that it inherited from the base class. We call this feature as **Overriding**. There are two way to help us to reuse code. The one way is composition and the other way is inheritance.

You reuse code by creating new classes, but instead of creating them from scratch, you use existing classes that someone has already built and debugged.

In Java and Python (I don't know the implementation of C++), you always doing inheritance when you create an object.


> Class Object is the root of the class hierarchy. Every class has Object as a superclass. All objects, including arrays, implement the methods of this class.            Since: JDK1.0   -- [Java Doc notes](https://docs.oracle.com/javase/7/docs/api/java/lang/Object.html)


You could also read my article about [Moemory Model of Object in C++](http://jasonleaster.github.io/blog/2015/06/13/memory-model-of-objects-in-c-plus-plus/) and [In Object In Python](http://jasonleaster.github.io/blog/2016/02/03/memory-model-of-int-object-in-python/). If you are interesting in the implementation of inheritance, just read all about it.


#### Warning of using inheritance

In heritance is a clever and appealing techonology, it is best applied in somewhat rare circumstances where you have several deeply similar classes. It's a common error for beginning OOP programmers to try to use inheritance for everything. In constrat, application of modularity and encapsulation and API design may be less flashy, but they are incredibly common and powerful.



### Polymophsim

### OOP Design
