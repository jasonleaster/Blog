---
layout: post
title: "Int Object in Python"
date: 2016-02-03 00:56:31 +0800
comments: true
tags: [C, Python]
categories: [Python]
---


### Preparation

Here, we are goint to analysis how Python deal with `Int Object` in itself.

First of all, we should have a environment which help us to explore the detail of the implementation in `Python 2.7`. At the same time, our operation shouldn't influence the origianl Python in your workstation, if you have install Python before.

You can get the source code from offical website of Python. And you will find that there have a file `configure` in the directory of the source.

*In the file configure, you can replace all string 'Python' with 'pyLab'. And then move all file prefix with `Python` into `pyLab`(mainly in directory Misc/ and Modules)*

Don't forget to run the configuration file -- configure with the option `--prefix=Location where you want the Python which is for lab to install `

Finally, just run `make & make install`

Repeat myself again, the reason that I move all 'Python' into 'pyLab' is all for making a difference with the original instance which is official and it shouldn't be influence by any operation from us.


### Rules of Object in Python.

* Objects are structures allocated on the heap.  
* Objects are never allocated statically or on the stack; they must be accessed through special macros and functions only.  

<!-- more -->

* An object has a 'reference count' that is increased or decreased when a pointer to the object is copied or deleted; when the reference count reaches zero there are no references to the object left and it can be removed from the heap.

* An object has a 'type' that determines what it represents and what kind of data it contains.  An object's type is fixed when it is created. Types themselves are represented as objects; an object contains a pointer to the corresponding type object.  The type itself has a type pointer pointing to the object representing the type 'type', which contains a pointer to itself!).

* Objects do not float around in memory; once allocated an object keeps the same size and address.  Objects that must hold variable-size data can contain pointers to variable-size parts of the object.  Not all objects of the same type have the same size; but the size cannot change after allocation.  

* Objects are always accessed through pointers of the type 'PyObject \*'. 


### Firt glance at the implementation.

``` C++

/*
   There are two basic different type of Object in Python.
   1. PyObject.    (Immutable Object eg: int, string object which's 
                                size is known when it is creating.)
   2. PyVarObject. (Mutable Object eg: list object which's size is 
                    unknown when it is creating and support the 
                    operation of deleting and inserting)
                        -- notes by Jason Leaster
 */
typedef struct _object {
    PyObject_HEAD
} PyObject;

typedef struct {
    PyObject_VAR_HEAD
} PyVarObject;
```

Actually, PyObject are created by Macro `PyObject_HEAD`

``` C++

[Include/object.h]

...

#define _PyObject_HEAD_EXTRA            \
    struct _object *_ob_next;           \
    struct _object *_ob_prev;

#define _PyObject_EXTRA_INIT 0, 0,

#define PyObject_HEAD                   \
    _PyObject_HEAD_EXTRA                \
    Py_ssize_t ob_refcnt;               \
    struct _typeobject *ob_type;

/* 
   There also have some Macro to get the data member of PyObject.
                    -- notes by Jason Leaster
 */
#define Py_REFCNT(ob)           (((PyObject*)(ob))->ob_refcnt)
#define Py_TYPE(ob)             (((PyObject*)(ob))->ob_type)
#define Py_SIZE(ob)             (((PyVarObject*)(ob))->ob_size)

...

```

Typically, the Int Object in Python `PyIntObject` is a `immutable object`. Once we created the object, we can't change the value in that object.

Back to our topic, let's look at the definition of `PyIntObject`.

``` C++

//[Include/intobject.h]

typedef struct{
    PyObject_HEAD
    long ob_ival;
} PyIntObject;

```

Aha, the `PyIntObject` is just a simple capsulation of `long` in C.
This object maintain a long data member.

``` C++

static int
int_compare(PyIntObject *v, PyIntObject *w)
{
    /*
    You will sigh that "the author of Python is a master in 
    programming". For efficient, they use `register` keyword and 
    try to apply for using register to store the local variable.
                            -- notes by Jason Leaster.
    */
    register long i = v->ob_ival;
    register long j = w->ob_ival;
    return (i < j) ? -1 : (i > j) ? 1 : 0;
}

```

How about the detail of printing a integer in Python ?

``` C++

/* ARGSUSED */
static int
int_print(PyIntObject *v, FILE *fp, int flags)
    /* flags -- not used but required by interface */
{
    long int_val = v->ob_ival;
    Py_BEGIN_ALLOW_THREADS
    fprintf(fp, "%ld", int_val);
    Py_END_ALLOW_THREADS
    return 0;
}

```

There have many others method which are defined in `PyNumberMethods`. Python programmer can get the information about the object with a data member `__doc__`, like this:

![images](/images/img_for_2016_02_03/doc.png)

The implementation of that is just a plain text string in `Object/intobject.c`

``` C++

PyDoc_STRVAR(int_doc,
"int(x=0) -> int or long\n\
int(x, base=10) -> int or long\n\
\n\
Convert a number or string to an integer, or return 0 if no arguments\n\
are given.  If x is floating point, the conversion truncates towards zero.\n\
If x is outside the integer range, the function returns a long instead.\n\
\n\
If x is not a number or if base is given, then x must be a string or\n\
Unicode object representing an integer literal in the given base.  The\n\
literal can be preceded by '+' or '-' and be surrounded by whitespace.\n\
The base defaults to 10.  Valid bases are 0 and 2-36.  Base 0 means to\n\
interpret the base from the string as an integer literal.\n\
>>> int('0b100', base=0)\n\
4");

```


### Initialization of PyIntObject

Python support many different API to construct a `PyIntObject`.

``` C++

[Include/intobject.h]
...
PyAPI_FUNC(PyObject *) PyInt_FromString(char*, char**, int);
#ifdef Py_USING_UNICODE
PyAPI_FUNC(PyObject *) PyInt_FromUnicode(Py_UNICODE*, Py_ssize_t, int);
#endif
PyAPI_FUNC(PyObject *) PyInt_FromLong(long);
PyAPI_FUNC(PyObject *) PyInt_FromSize_t(size_t);
PyAPI_FUNC(PyObject *) PyInt_FromSsize_t(Py_ssize_t);
...

```

Before we dig into the implementation of the initialization process.
Let's do a lab.

![images](/images/img_for_2016_02_03/different_address.png)

`id(object)` will returen the address of object in CPython.(If you are interesting in the implementation 
of built-in method id(object), you can jump to the end of this article and I presented how CPython implement this function.)

Try to answer the following question.

Why int object `m` and `n` of value `1000` have different address?

Why int object `x` and `y` of value `1`    have the same  address?

The answer relate to the mechanism with numbers in Python.

*In daily programming stuff, small number are used frequently. But large number may used frequent more than small numbers*

If there isn't a special mechanism, Python will allocate memory(call malloc() in C) again and again. This stratege isn't efficient at run time.
So, Python support a mechanism which will create small number only once but not bigger number.

There also have the other question. What means big? What means small?
It's not clearly in theory. But in the implementation, there have a trade off. People can't cache all integer number for the limitation of memory(RAM).

``` C++

[Object/intobject]
...
/*
NSMALLPOSINTS:    Numbers of small positive integers number
NSMALLNEGINTS:    Numbers of small negative integers number
        --notes by Jason Leaster
*/
#ifndef NSMALLPOSINTS  
#define NSMALLPOSINTS           257
#endif
#ifndef NSMALLNEGINTS  
#define NSMALLNEGINTS           5
#endif
#if NSMALLNEGINTS + NSMALLPOSINTS > 0
/*  References to small integers are saved in this array so that they
    can be shared.
    The integers that are saved are those in the range
    -NSMALLNEGINTS (inclusive) to NSMALLPOSINTS (not inclusive).
*/
static PyIntObject *small_ints[NSMALLNEGINTS + NSMALLPOSINTS];
#endif
...

```

According to the implementation, the small number is [-5, 257). You can also modify the source code and recomplie it to change the range of small number, if you would like to.

>   Integers are quite normal objects, to make object handling uniform.
   (Using odd pointers to represent integers would save much space
   but require extra checks for this special case throughout the code.)
   Since a typical Python program spends much of its time allocating
   and deallocating integers, these operations should be very fast.
   Therefore we use a dedicated allocation scheme with a much lower
   overhead (in space and time) than straight malloc(): a simple
   dedicated free list, filled when necessary with memory from malloc().
   block\_list is a singly-linked list of all PyIntBlocks ever allocated,  linked via their next members.  PyIntBlocks are never returned to the
   system before shutdown (PyInt\_Fini).
   free\_list is a singly-linked list of available PyIntObjects, linked
   via abuse of their ob\_type members.

``` C++

#define BLOCK_SIZE      1000    /* 1K less typical malloc overhead */
#define BHEAD_SIZE      8       /* Enough for a 64-bit pointer */
#define N_INTOBJECTS    ((BLOCK_SIZE - BHEAD_SIZE) / sizeof(PyIntObject))

struct _intblock {
    struct _intblock *next;
    PyIntObject objects[N_INTOBJECTS];
};

typedef struct _intblock PyIntBlock;

static PyIntBlock *block_list = NULL;
static PyIntObject *free_list = NULL; // Initliazed by fill_free_list()

```

### Insert and Delete

A initialization routine from `long type` integer number.

``` C++

PyObject *
PyInt_FromLong(long ival)
{
    register PyIntObject *v;
    // try to use small number pool
#if NSMALLNEGINTS + NSMALLPOSINTS > 0
    if (-NSMALLNEGINTS <= ival && ival < NSMALLPOSINTS) {
        v = small_ints[ival + NSMALLNEGINTS];
        Py_INCREF(v);
#ifdef COUNT_ALLOCS
        if (ival >= 0)
            quick_int_allocs++;
        else
            quick_neg_int_allocs++;
#endif
        return (PyObject *) v;
    }
#endif

   // if ival is not a small number, create a new instance object.
   if (free_list == NULL) {
       if ((free_list = fill_free_list()) == NULL)
           /*
              fill_free_list() will create a general number pool 
              for PyIntObjects.
              But if there is no memory, return NULL
            */
           return NULL;
   }
   /* Inline PyObject_New */
   v = free_list;
   free_list = (PyIntObject *)Py_TYPE(v); 
   // Py_TYPE(ob)  (((PyObject*)(ob))->ob_type) --notes by Jason Leaster
   PyObject_INIT(v, &PyInt_Type);
   v->ob_ival = ival;
   return (PyObject *) v;
}

```

### Time for Hacking
Modify the `int_print` function in `Object/intobject.c`.
Here is my modification:

``` C++

/*
 Programmer : Jason Leaster
 Date       : 2016.02.03

 Modify from Robert Chen's code.
 */

tatic int values[10];
static int refcounts[10];

static int
int_print(PyIntObject *v, FILE *fp, int flags)
         /* flags -- not used but required by interface */
{
        // <<< modified by Jason Leaster
        PyIntObject* intObjectPtr = NULL;
        PyIntBlock* p    = block_list;
        PyIntBlock* last = NULL;
        
        int count = 0;
        int i = 0;
        while(p != NULL)
        {
            count++;
            last = p;
            p = p->next;
        }
        
        intObjectPtr = last->objects;
        intObjectPtr+= N_INTOBJECTS - 1;
            
        printf(" Address @%p\n", v);
        
        for(i = 0; i < 10; i++, --intObjectPtr)
        {
            values[i] = intObjectPtr->ob_ival;
            refcounts[i] = intObjectPtr->ob_refcnt;
        }
        
        printf(" Values: ");
        for(i = 0; i < 10; i++)
        {
            printf("%d\t", values[i]);
        }
        printf("\n");

        printf(" refcnt: ");
        for(i = 0; i < 10; i++)
        {
            printf("%d\t", refcounts[i]);
        }
        printf("\n");

        printf(" block_list count: %d\n", count);
        printf(" free_list : %p\n", free_list);

        // >>> Origianl
        long int_val = v->ob_ival;
        Py_BEGIN_ALLOW_THREADS
        fprintf(fp, "%ld", int_val);
        Py_END_ALLOW_THREADS
        return 0;
}


```

Tips for debugging:
`getrefcount` in module `sys` is helpful to get the reference number of object.


![images](/images/img_for_2016_02_03/printout.png)



### Extention reading

There are lots of built-in methods in Python. id(Object) is one of them.

According to the build-in doc for function id(Object). 

> In [1]: print id.\_\_doc\_\_  
id(object) -> integer  
Return the identity of an object.  This is guaranteed to be unique among
simultaneously existing objects.  (Hint: it's the object's memory address.)

There is a hint and say the return value of built-in function id(object) is the object's memory address.

All the function do is just 5-line codes in the implementation of CPython.

``` Python

[Python/bltinmodule.c]
static PyObject *
builtin_id(PyObject *self, PyObject *v)
{
        return PyLong_FromVoidPtr(v);
}

...

static PyMethodDef builtin_methods[] = {
    ...
    {"id",              builtin_id,         METH_O, id_doc}, // id_doc is just the value of id.__doc__
    {"isinstance",  builtin_isinstance, METH_VARARGS, isinstance_doc},
    {"len",             builtin_len,        METH_O, len_doc},
    ...
}

```

That's all about the built-in method id(Object)

--------

Photo by Jason Leaster, in ChangeDe Hunan
![images](/images/img_for_2016_02_03/street.jpg)

