---
layout: post
title: "Brief introduction to opcodes in Python"
date: 2016-02-06 18:39:50 +0800
comments: true
categories: python
---

Python program is compiled down to bytecode which is sort of like assembly for the python virtual machine. The interpreter executes each of these bytecodes one by one.

Let's look the following code. It's easy to understand what it is doing without my explanation.

``` python

[demo.py]

x = 1
y = 2
z = x + y
print "x, y, z", x, y, z

```
<!-- more -->

Programmer can use python module `dis` to transform that code into the assemble program of Python.

``` python

python -m dis "./demo.py"
#line number 
#in the       #assemble code           # object name in the source code
#source code
1           0 LOAD_CONST               0 (1)
            3 STORE_NAME               0 (x)

2           6 LOAD_CONST               1 (2)
            9 STORE_NAME               1 (y)

3          12 LOAD_NAME                0 (x)
           15 LOAD_NAME                1 (y)
           18 BINARY_ADD
           19 STORE_NAME               2 (z)

4          22 LOAD_NAME                0 (x)
           25 PRINT_ITEM
           26 LOAD_NAME                1 (y)
           29 PRINT_ITEM
           30 LOAD_NAME                2 (z)
           33 PRINT_ITEM
           34 PRINT_NEWLINE
           35 LOAD_CONST               2 (None)
           38 RETURN_VALUE

########################################################

"""
In the interpreter of Python, programmer also can call
the built-in function @compile and import module @dis
to do the same thing.
"""

In [5]: source = open("./hello.py").read()

In [6]: code = compile(source, "demo", "exec")

In [7]: import dis

In [8]: dis.dis(code)

```

It's not hard to understand the `opcode`. In implementation of CPython, all opcode defined as Macro in the header file `Include/opcode.h`. 

``` C

[Include/opcode.h]
...
#define LOAD_CONST  100 /* Index in const list */
#define LOAD_NAME   101 /* Index in name list */
...

```

The evaluation machine of Python is a stack-based machine.
If you find the detail of `18 BINARY_ADD`, you will find that this opcode 
will pop the two object on the stack and then add them together.

``` C

[Python/ceval.c]
...

/* Stack manipulation macros */

/* The stack can grow at most MAXINT deep, as co_nlocals and
      co_stacksize are ints. */

#define STACK_LEVEL()     ((int)(stack_pointer - f->f_valuestack))
#define EMPTY()           (STACK_LEVEL() == 0)
#define TOP()             (stack_pointer[-1])
#define SECOND()          (stack_pointer[-2])
#define THIRD()           (stack_pointer[-3])
#define FOURTH()          (stack_pointer[-4])
#define PEEK(n)           (stack_pointer[-(n)])
#define SET_TOP(v)        (stack_pointer[-1] = (v))
#define SET_SECOND(v)     (stack_pointer[-2] = (v))
#define SET_THIRD(v)      (stack_pointer[-3] = (v))
#define SET_FOURTH(v)     (stack_pointer[-4] = (v))
#define SET_VALUE(n, v)   (stack_pointer[-(n)] = (v))
#define BASIC_STACKADJ(n) (stack_pointer += n)
#define BASIC_PUSH(v)     (*stack_pointer++ = (v))
#define BASIC_POP()       (*--stack_pointer)
...

```




