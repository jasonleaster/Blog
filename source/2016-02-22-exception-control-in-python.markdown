---
layout: post
title: "Exception Control in Python"
date: 2016-02-22 11:31:34 +0800
comments: true
categories: python
---

Pythoner may be familiar with exception control like the demo beblow there:

``` python
try:
    i = 0
    while True:
        i += 1

except KeyboardInterrupt:
    print "after abort:", i

print "get here"

```

<!-- more -->

If you press down `ctrl + c`, there will trigger a interruption by keyboard. The inner infinite loop will stop and jump into the exception handler. There are another examples of exception and interruption in the computer. Programmer may also ask what's the benefite to handle the exception ...

What if there is something unpredictable and it will stop the program to run correctly?

If you do 1 divide 0 in your program, the CPU don't know how to compute that expression. In the level of operating system, OS will handle it as an exception and tell programmer that it doesn't work. The program must be stopped or killed.

``` python
try:
    1/0
except:
    print "Welcome Chapter 42 -- Guideline to galaxy"

```

Although there trig a exception for diveding zero, but we catch it and handle it correctly. So, the program end correctly.

The mechanism of exception handling come from operating system. 


Here is the definition of exception object in Python. (Don't forget that everything in Python is object).

``` C
[Include/pyerrors.h]

...

typedef struct {
    PyObject_HEAD
    PyObject *dict;
    PyObject *args;
    PyObject *message;
} PyBaseExceptionObject;

...

/* Predefined exceptions */

PyAPI_DATA(PyObject *) PyExc_BaseException;
PyAPI_DATA(PyObject *) PyExc_Exception;
PyAPI_DATA(PyObject *) PyExc_StopIteration;
...
PyAPI_DATA(PyObject *) PyExc_ZeroDivisionError;
PyAPI_DATA(PyObject *) PyExc_EOFError;

...

```

Here is a example about how python deal with the exception.

What would happen if there is a expression `1/0`.
I use IPython interpreter do this demo.

``` python
In [18]: code = compile("1/0", "testscript", mode= "exec")

In [19]: dis.disassemble(code)
      1           0 LOAD_CONST               0 (1)
                  3 LOAD_CONST               1 (0)
                  6 BINARY_DIVIDE       
                  7 POP_TOP             
                  8 LOAD_CONST               2 (None)
                 11 RETURN_VALUE 

```
The assemble code (opcode) of expression `1/0` in Python is compiled into that opcode code. It isn't difficult to understand what the `LOAD_CONST` do.


Let's dig into the detail of `BINARY_DIVIDE` in Python/ceval.c

``` C
[Python/ceval.c]
    for(;;)// big for loop
        ...
        TARGET_NOARG(BINARY_DIVIDE)
            {
                if (!_Py_QnewFlag) {
                    w = POP();
                    v = TOP();
                    x = PyNumber_Divide(v, w); //
                    Py_DECREF(v);
                    Py_DECREF(w);
                    SET_TOP(x);
                    if (x != NULL) DISPATCH();
                    break;
                }
            }
        ...

        /* 
           set up the basic information for the reason 
           why exception happened  -- notes by Jason Leaster
         */
        /* Quickly continue if no error occurred */

        if (why == WHY_NOT) {
            if (err == 0 && x != NULL) {
                READ_TIMESTAMP(loop1);
                continue; /* Normal, fast path */
            }
            why = WHY_EXCEPTION;
            x = Py_None;
            err = 0;
        }

        ...

        /* Log traceback info if this is a real exception */
        if (why == WHY_EXCEPTION) {
            PyTraceBack_Here(f);
            ...
        }
        ...

[Objects/abstract.c]
BINARY_FUNC(PyNumber_Subtract, nb_subtract, "-")

/* PyNumber_Divide is just the other name of function nb_divide 
   and then you will find that the nb_divide is just a function pointer
   in struct PyNumberMethods.

   The REAL implementation is function @int_classic_div in 
   Objects/intobject.c. The @int_classic_div will call function
   @i_divmod in Objects/intobject.c

                                    -- notes by Jason Leaster
 */
BINARY_FUNC(PyNumber_Divide, nb_divide, "/") 

BINARY_FUNC(PyNumber_Divmod, nb_divmod, "divmod()")

[Objects/intobject.c]
i_divmod(register long x, register long y,
           long *p_xdivy, long *p_xmody)
{
    long xdivy, xmody;

    if (y == 0) {
        /*
         Keypoint!
         Here is where Python setup a new exception for "divide zero"
         problem.
                                            -- notes by JasonLeaster
         */
        PyErr_SetString(PyExc_ZeroDivisionError,
                "integer division or modulo by zero");
        return DIVMOD_ERROR;
    }

    ... ...
}

```

It's the last mile to get the target place.
`PyErr_SetString` finish the job about the initialization of `divide zero` exception.

``` C
[Python/errors.c]

void
PyErr_Restore(PyObject *type, PyObject *value, PyObject *traceback)
{
    PyThreadState *tstate = PyThreadState_GET();
    PyObject *oldtype, *oldvalue, *oldtraceback;

    ... 

    /* Save these in locals to safeguard against recursive
       invocation through Py_XDECREF */
    oldtype = tstate->curexc_type;
    oldvalue = tstate->curexc_value;
    oldtraceback = tstate->curexc_traceback;

    tstate->curexc_type = type;
    tstate->curexc_value = value;
    tstate->curexc_traceback = traceback;
    ...
}

void
PyErr_SetObject(PyObject *exception, PyObject *value)
{
    ...

    /* log this exception message into thread state */
    PyErr_Restore(exception, value, (PyObject *)NULL);
}

void
PyErr_SetString(PyObject *exception, const char *string)
{
    PyObject *value = PyString_FromString(string);
    PyErr_SetObject(exception, value);
    Py_XDECREF(value);
}

```

After the VM (ceval.c) know why there is a exception, the VM will take step
into handling this exception. We have knew that the big `for switch loop` execute the opcode step by step. 
Once there is something wrong with the opcode at runtime, the VM will setup
 exception object by `PyErr_SetObject`. And then VM also will setup the traceback object of type `PyTracebackObject`.

``` C
/* Traceback interface */

typedef struct _traceback {
    PyObject_HEAD
    struct _traceback *tb_next;
    struct _frame *tb_frame;
    int tb_lasti;
    int tb_lineno;
} PyTracebackObject;

```

You may notice that the `PyTracebackObject` is a single direction linked-list. 

``` C
static PyTracebackObject *
newtracebackobject(PyTracebackObject *next, PyFrameObject *frame)
{
    PyTracebackObject *tb;
    if ((next != NULL && !PyTraceBack_Check(next)) ||
            frame == NULL || !PyFrame_Check(frame)) {
        PyErr_BadInternalCall();
        return NULL;
    }
    tb = PyObject_GC_New(PyTracebackObject, &PyTraceBack_Type);
    if (tb != NULL) {
        Py_XINCREF(next);
        tb->tb_next = next;
        Py_XINCREF(frame);
        tb->tb_frame = frame;
        tb->tb_lasti = frame->f_lasti;
        tb->tb_lineno = PyFrame_GetLineNumber(frame);
        PyObject_GC_Track(tb);
    }
    return tb;
}

int
PyTraceBack_Here(PyFrameObject *frame)
{
    PyThreadState *tstate = PyThreadState_GET();
    PyTracebackObject *oldtb = (PyTracebackObject *) tstate->curexc_traceback;
    PyTracebackObject *tb = newtracebackobject(oldtb, frame);
    if (tb == NULL)
        return -1;
    tstate->curexc_traceback = (PyObject *)tb;
    Py_XDECREF(oldtb);
    return 0;
}

```

``` python
import sys

def h():
    print "in frame :", sys._getframe()
    print "in function :", sys._getframe().f_code.co_name
    1/0

def g():
    print "in frame :", sys._getframe()
    print "in function :", sys._getframe().f_code.co_name
    h()

def f():
    print "in frame :", sys._getframe()
    print "in function :", sys._getframe().f_code.co_name
    g()

f()

```

This program will trig the exception at function h().
Here is the output of that program.

![images](/images/img_for_2016_02_22/traceback.png)

![images](/images/img_for_2016_02_22/frameobjectlist.png)

### Hacker Time

![images](/images/img_for_2016_02_22/hacktime0.png)

-----
Photo by Jason Leaster, ChangeDe, HuNan, China
![images](/images/img_for_2016_02_22/bridge.jpg)


