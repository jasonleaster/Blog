---
layout: post
title: "Architecture of Python Virtual Machine"
date: 2016-02-21 11:42:50 +0800
comments: true
categories: python
---

Python virtual machine is the core part of this language. After compiling the original python code into `Opcode`(byte code), python VM will take the job left. Python will take every opcode from `PyCodeObject`.


<!-- more -->

### Executing environment in Python VM

Actually, all the things that VM do is simulating what the OS do to excute a program.

![images](/images/img_for_2016_02_21/stack.png)

The figure over there show the representation of the model of stack-based machine.

If you are fimilary with system programming in C, it's easy to understand the meaning of that figure.

But if you are an beginner with programming in C, you may try to finish the lab2( bomb ) in CSAPP. It will help beginner a lot to understand the mechanism of stack-based machine.

**We know that all the static information about the program store in `PyCodeObject`. But what about the dynamic information when the program is running in the Python VM?**

`PyCodeObject` can't include the dynamic information and the environment where program is running.

``` python
number = 2016

def f():
    number = 42
    print number # 42

```

You must know `number` in function `f()` and the variable which is not inside the block of function `f()` with the same name. That two variable are in different frame which means envrionment in running time.

In Python, there is a class to describe the envrionment at running time -- `PyFrameObject`. It's a simulation of stack frame in x86 platform.


You noticed that `PyFrameObject` is a size-variable Python Object class. Because this class maintain a `PyCodeObject` and stack in different block have different size. So `PyFrameObject` can't be a size fixed class.

``` C
[Inlcude/frameobject.h]

typedef struct _frame {
    PyObject_VAR_HEAD
    struct _frame *f_back;  /* previous frame, or NULL */

    PyCodeObject *f_code;   /* code segment */
    PyObject *f_builtins;   /* builtin symbol table (PyDictObject) */
    PyObject *f_globals;    /* global symbol table (PyDictObject) */
    PyObject *f_locals;     /* local symbol table (any mapping) */
    PyObject **f_valuestack;/* points after the last local */

    /* Next free slot in f_valuestack.  
    Frame creation sets to f_valuestack.
    Frame evaluation usually NULLs it, but a frame that yields sets it
    to the current stack top. */
    PyObject **f_stacktop;

    ...

} PyFrameObject;

```

This figure show the origanization of PyFrameObject like a single linked-list. `f_back` point to the previous frame. `f_valuestack` is something like `ebp` register in x86 and `f_stacktop` like the `esp` register.

![images](/images/img_for_2016_02_21/frame.png)



How to create a new PyFrameObject? The answer is function `PyFrame_New`.

``` C
[Object/frameobject.c]

PyFrameObject *
PyFrame_New(PyThreadState *tstate, PyCodeObject *code, PyObject *globals,
                    PyObject *locals)
{
    PyFrameObject *back = tstate->frame;
    PyFrameObject *f; /* New frame object */
    PyObject *builtins;
    Py_ssize_t i;

    ... 
    /* Big block of code to set value of @builtins */
    ...

    f->f_stacktop = f->f_valuestack;
    f->f_builtins = builtins;
    Py_XINCREF(back);
    f->f_back = back;
    Py_INCREF(code);
    Py_INCREF(globals);
    f->f_globals = globals;

    ...
    /* some details are ommited */
    ...

    f->f_locals = locals;
    f->f_tstate = tstate;
    f->f_lasti = -1;
    f->f_lineno = code->co_firstlineno;
    f->f_iblock = 0;

    _PyObject_GC_TRACK(f);
    return f;
}

```

Luckly, we could access `PyFrameObject` in Python at running time.

``` python
import sys

value = 3

def g():
    """
    _getframe([depth]) return a frame object from the call stack.
    If that is deeper than the call stack, ValueError is raised.  The default
    for depth is zero, returning the frame at the top of the call stack.
    """
    frame = sys._getframe()
    print "current function is : ", frame.f_code.co_name
    caller = frame.f_back
    print "caller function is : ", caller.f_code.co_name
    print "caller's local namespace : ", caller.f_locals
    print "caller's global namespace : ", caller.f_globals.keys()

def f():
    a = 1
    b = 2
    g()

def show():
    f()

show()

"""
the output of this program :
jasonleaster@ubuntu:~/Code_by_Jason/Python_language$ python caller.py 
current function is :  g
caller function is :  f
caller's local namespace :  {'a': 1, 'b': 2}
caller's global namespace :  ['g', 'f', '__builtins__', '__file__', 'show', 'value', '__package__', 'sys', '__name__', '__doc__']
"""

```


### Name, Scope and Namespace

We have knew the three different namespace -- locals, globals and builtins.
In Python, there is a important basic structure -- `module`.
More generally, every `.py` file is a module in Python. This concept help Python to divide namespace and reuse the program which have been writed.

Name is helpful to memory something which is not simple enough.

Assignment expression in Python have two things in common:

* Create a new object
* build a connection between that new object and a name

You may be familary with assignment like this `x = 1`. But expression like
`def function():` and `class A():`, all these expression are also assignment expressions. What assignment to do is that build a connection between the object and name. It's like a pair (name, object), (x, 1), (code, codeObject) and so on.

A namespace is corresponding with unique scope.

There is a comparasion between Python and C about namespace.

![images](/images/img_for_2016_02_21/compare.png)

Just guess what would happend if you run these two program ?

The right side C program will run correctly but the left side Python program will run into exception for "local variable 'a' referenced before assignment".

The reason is that no matter where the variable are decalred in the scope, all things in that scope can see that variable. But what is interesting is that the assignment expression are after the first print expression. When program run into the first print expression, the assginment are unfinished, 
**which means the local object haven't created yet.**

`global` keyword help programmer to declare a name which is in the global scope.

``` python
def f():
    global a
    print a
    a = 2016
    print a

```
But you should know that once you declare a variable with `global` keyword, all thing happen to the variable in the local scope will influence that object in the global scope and change it's value. After function `f()` finished, the value of a in global scope changed into 2016.

### Runtime Architecture of Python VM

This function is the core part of virtual machine in Python. Once we get the opcode from `PyFrameObject`, the function `PyEval_EvalFrameEx` will process that opcode and run it.

``` C
[Python/ceval.c]

...

/* Code access macros */

#define INSTR_OFFSET()  ((int)(next_instr - first_instr))
#define NEXTOP()        (*next_instr++)
#define NEXTARG()       (next_instr += 2, (next_instr[-1]<<8) + next_instr[-2])
#define PEEKARG()       ((next_instr[2]<<8) + next_instr[1])
#define JUMPTO(x)       (next_instr = first_instr + (x))
#define JUMPBY(x)       (next_instr += (x))

...

PyObject *
PyEval_EvalFrameEx(PyFrameObject *f, int throwflag)
{
    ...
    for(;;)
    {
            f->f_lasti = INSTR_OFFSET();// get the opcode
            ...
            opcode = NEXTOP();

            oparg = 0;   
            /* allows oparg to be stored in a register because
               it doesn't have to be remembered across a full loop */
            if (HAS_ARG(opcode))
                oparg = NEXTARG();
dispatch_opcode:
                                                                                    /* Main switch on opcode */
                                                                                        READ_TIMESTAMP(inst0);
            switch (opcode) {

                case NOP:
                    ...
                case LOAD_FAST:
                    ...
                case LOAD_CONST:
                    ...
                    ...
            }


    }

    ...
}

```

----

Photo by QianNan Qu. In XiangTan, HuNan, China.

![images](/images/img_for_2016_02_21/me.jpg)
