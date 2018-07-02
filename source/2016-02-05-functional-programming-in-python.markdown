---
layout: post
title: "Functional Programming in Python"
date: 2016-02-05 02:37:57 +0800
comments: true
categories: python
---

Functional programming is not only use functions.
Think of "y = f(x)" or "h = f * g"

For comparasion, there are different paradigms in programming.

* Structured/Procedural -- Functions, loops, conditions
* Object-Oriented Programming (OOP) -- Classes, objects, methods
* Functional Programming -- Decorators, comprehensions, and generators
* Logic Programming

Functions in Python are `first class value`. It can take functions as arguments and return functions like a value.

``` python

"""
function @calc take @f as an argument and use it as a function.
Something like function pointer in C.
"""
def calc(f, x, y):
    return f(x, y)

def add(x, y):
    return x + y

def sub(x, y):
    return x - y

calc(add, 10, 20)

```

<!-- more -->

In Python, there have anonymous functions which is called `Lambda Expression` which's body is limited to one expression.

``` python

def calc(f, x, y):
    return f(x, y)

calc(lambda x, y: x + y, 10, 20)

```

Here, lambda expression used as a argument which is passed into function `calc`. Function like `add` and `sub`, they are too short and don't need a special name to be identified. So, Python support a mechanism -- anonymous function which is called `Lambda Expression`.

There is a another demo. We will change the code from procedure oriented into functional style.

``` python

"""
In a procedure way.
"""
def incr(x):
    return x + 1

def increment_each(elements):
    results = []
    for elem in elements:
        results.append(incr(elem))
    return results

increment_each([1, 2, 3])
# -> [2, 3, 4]

"""
In a functional way.
"""
map(incr, [1, 2, 3])
# or
map(lambda x: x + 1, [1, 2, 3])

# if we want to get the length of each element in a iterable object
map(len, iterable_object)

```

`filter` is a related idea in functional tools like map.
`filter` return a new sequence where values are taken from the given sequence if they return True when passed to a given function.

``` python

filter(lambda x: x % 2 == 0, [1, 2, 3, 4])
# -> [2, 4]

```

`reduce` will accumulate and return a single result, given a sequence
and passing each value to a function along with the current result.

``` python

from functools import reduce
reduce(lambda accum, current: accum + current, [1, 2, 3], 0)
# -> 6

```

### Decorator

In Python, function is first-class value and can be used as returned value.
So, programmer can define nested functions in Python like this:

``` python

"""
Nested function
"""
def outer():
    def inner():
        print "I am the inner function"

    return inner

func = outer()

func()
# -> I am the inner function

"""
Closure
"""
def outer(var = 10):

    "Local scope of function @outer"

     def inner(number):
         print "I am the inner function!"
         """
         You can't change value of variable once the
         @outer function finished. You will get error
         if you modify variable which is not in scope
         of function @inner. You can't modify @var
         like this: var += 1 (You will get exception
         information like "UnboundLocalError")
         """
         return var + number

    return inner

func = outer(10)

func(90)
# -> 100

```

Look the demo below there.

``` python

def wrapper(func):
    def cheker(arguments_of_func):
        """
            @checker receive the same arguments as @func
            and do something that @func didn't do.
        """
        return new_retVal

    return checker

```

Actually, the reture value of `wrapper` is a function just little different from the original function passed into `wrapper` -- `func`. Function `wrapper` like a shell on the original function and return with a more powerful function. That's decorator. `wrapper` is a decorator.


__Decorator: A decorator is any callable Python object that is used to modify a function, method or class definition. A decorator is passed th original object being defined and returns a modified object, which is then bound to the name in the definition. Python decorators were inspired in part by Java annotations, and have a similar syntax; teh decorator syntax is pure syntactic sugar, using @ as the keyword               -- Wikipedia__

Here is a more generic decorators

``` python

def wrapper(func):
    def inner(*args, **kwargs)
        print "Show arguments: %s, %s" % (args, kwargs)
        return func(*args, **kwargs)
    return inner

@wrapper
def adder(x, y)
    return x + y

print adder(10, 90)

#-> Show arguments: (10, 90), {}
#   100

```

A better and more detailed explaination reader could read the blog
[Understanding Python Decoratos in 12 Easy Steps!](http://www.simeonfranklin.com/blog/2012/jul/1/python-decorators-in-12-steps/)
which is written by `simeon franklin`.


### Generator

A `generator` is a special type of iterator(not vice versa!). 
Generator is a factory that lazily produces values.

There are two types of generators in Python: generator functions and generator expression.

``` python

"""
Demo for generator
"""

def fib():
    prev, curr = 0, 1
    while True:
        """
            There is no @return keyword in generator,
        but there must be a @yield with a return object 
        if there is a  generator function.
        """
        yield curr 
        prev, curr = curr, prev + curr

f = fib()
f.next()

############################################

"""
Generator Expression
"""
numbers = [1, 2, 3, 4, 5, 6]

lazy_square = ( x * x for x in numbers)
"""
Type:        generator
String form: <generator object <genexpr> at 0x7fd3bc119fa0>
Docstring:   <no docstring>
"""

In [18]: lazy_square == list
Out[18]: False

In [19]: lazy_square
Out[19]: <generator object <genexpr> at 0x7fd3bc0da410>

In [20]: lazy_square.next()
Out[20]: 1

In [21]: lazy_square.next()
Out[21]: 4

In [22]: lazy_square.next()
Out[22]: 9

```


---------
Photo by Jason Leaster in ChangDe, HuNan, China.

What a big banana :)

![images](/images/img_for_2016_02_06/banana.png)
