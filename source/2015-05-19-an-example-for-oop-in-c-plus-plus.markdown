---
layout: post
title: "An Example for OOP in C++"
date: 2015-05-19 23:13:08 +0800
comments: true
categories: Cplusplus
---

Generally, OOP( Object Oriented Progarmming) have three important elements:

* Abstraction of data
* Inheritance
* Dynamic Binding

Now, there is a demo for how to build an arithmetic expression-tree.

![images](/images/img_for_2015_05_19/exp_tree.png)

<!-- more -->

We want to build a node-tree like the tree in that picture to print a correct arithmetic expression.

Here you could see the relationship between the classes we used in our demo.

![images](/images/img_for_2015_05_19/class_relationship.png)

You know that the three classes `Int_node`, `Unary_node` and `Binary_node` are all inherit from the base class `Expr_node`.

Let’s look at the simple unary arithmetic expression `(-5)`. In this expression, there is only one operand `-` and one integer `5`. In expression `( 3 + 4)`, there are two integer and one operand.

We could find that there are only three different expression in representation of arithmetic expression. They are:

* Integer Expression
* Unary Expression
* Binary Expression

So, we find the common attributes on the three different types of expression and use a base-class `Expr_node` to represent this attribute.

If you have background in class handle, you may notice that `class Expr` is the handle for `Expr_node`.

We don’t need real objects of `class Expr_node`. What we need is the classes which inherit from that base-class. The meaning of the base-class is to provide the public interface.
	
``` C++
/*
    Programmer  :   EOF
    Date        :   2015.05.19
    File        :   exp_node.cpp
    E-mail      :   jasonleaster@gmail.com

 */
#include <iostream>
#include <string>

using namespace std;


/*
   This @Expr_node is the base-class.
 */
class Expr_node
{
    friend ostream& operator << (ostream&, const Expr_node&);
    friend class Expr;

    int use;// @use is a counter to avoid copying objects.

    //protected:
    public:
        Expr_node(): use(1) { }
        virtual void print(ostream&) const = 0;
        virtual ~Expr_node() { }
};


class Expr
{
    friend ostream& operator<<(ostream&, const Expr&);
    Expr_node* p;

    public:
        Expr():p(NULL){}
        Expr(int);
        Expr(const string&, Expr);
        Expr(const string&, Expr, Expr);
        Expr(const Expr& t) { p = t.p; ++p->use; };

        Expr& operator=(const Expr&);

        ~Expr() { if(--p->use == 0) delete p;}
};

ostream&
operator<<(ostream& o, const Expr_node& e)
{
    e.print(o);
    return o;
}

Expr&
Expr::operator=(const Expr& rhs)
{
    rhs.p->use++;
    if(--p->use == 0)
    {
        delete p;
    }

    p = rhs.p;
    return *this;
}

ostream&
operator<<(ostream& o, const Expr& t)
{
    t.p->print(o);
    return o;
}

class Int_node: public Expr_node
{
    friend class Expr;

    int n;

    Int_node(int k): n(k) { }
    void print(ostream& o) const { o << n;}
};

class Unary_node: public Expr_node
{
    friend class Expr;
    string op;
    Expr opnd;
    Unary_node(const string& a, Expr b):
        op(a), opnd(b) { }

    void print(ostream& o) const
    {
        o << "(" << op << opnd << ")";
    }
};

class Binary_node: public Expr_node
{
    friend class Expr;
    string op;

    Expr left;
    Expr right;

    Binary_node(const string& a, Expr b, Expr c):
        op(a), left(b), right(c) { }

    void print(ostream& o) const
    {
        o << "(" << left << op << right << ")";
    }
};

Expr::Expr(int n)
{
    p = new Int_node(n);
}

Expr::Expr(const string& op, Expr t)
{
    p = new Unary_node(op, t);
}

Expr::Expr(const string& op, Expr left, Expr right)
{
    p = new Binary_node(op, left, right);
}


int main()
{
    Expr t = Expr("*", Expr("-", 5), Expr("+", 3, 4));
    cout << t << endl;
    t = Expr("*", t, t);
    cout << t << endl;

    return 0;
}
```

You could run this program and will get the output like this one.

![images](/images/img_for_2015_05_19/output1.png)

### More Opeartion

We could evaluate the expression and add more types of node.

Here is the implementation. That’s cool!

``` C++
/*
    Programmer  :   EOF
    Date        :   2015.05.19
    File        :   8.5.cpp
    E-mail      :   jasonleaster@gmail.com

 */
#include <iostream>
#include <string>

using namespace std;


/*
   This @Expr_node is the base-class.
 */
class Expr_node
{
    friend ostream& operator << (ostream&, const Expr_node&);
    friend class Expr;

    int use;// @use is a counter to avoid copying objects.

    protected:
        Expr_node(): use(1) { }
        virtual void print(ostream&) const = 0;
        virtual ~Expr_node() { }
        virtual int eval() const = 0;
};


class Expr
{
    friend ostream& operator<<(ostream&, const Expr&);
    Expr_node* p;

    public:
        Expr():p(NULL){}
        Expr(int);
        Expr(const string&, Expr);
        Expr(const string&, Expr, Expr);
        Expr(const string&, Expr, Expr, Expr);
        Expr(const Expr& t) { p = t.p; ++p->use; };

        Expr& operator=(const Expr&);

        ~Expr() { if(--p->use == 0) delete p;}

        int eval() const {return p->eval();}
};

ostream&
operator<<(ostream& o, const Expr_node& e)
{
    e.print(o);
    return o;
}

Expr&
Expr::operator=(const Expr& rhs)
{
    rhs.p->use++;
    if(--p->use == 0)
    {
        delete p;
    }

    p = rhs.p;
    return *this;
}

ostream&
operator<<(ostream& o, const Expr& t)
{
    o << (*t.p);
    return o;
}

class Int_node: public Expr_node
{
    friend class Expr;

    int n;

    Int_node(int k): n(k) { }
    void print(ostream& o) const { o << n;}
    int eval() const { return n;}
};

class Unary_node: public Expr_node
{
    friend class Expr;
    string op;
    Expr opnd;
    Unary_node(const string& a, Expr b):
        op(a), opnd(b) { }

    void print(ostream& o) const
    {
        o << "(" << op << opnd << ")";
    }

    int eval() const
    {
        if(op == "-")
        {
            return -opnd.eval();
        }

        throw "error, bad op" + op + "int UnaryNode";
    }
};

class Binary_node: public Expr_node
{
    friend class Expr;
    string op;

    Expr left;
    Expr right;

    Binary_node(const string& a, Expr b, Expr c):
        op(a), left(b), right(c) { }

    void print(ostream& o) const
    {
        o << "(" << left << op << right << ")";
    }

    int eval() const
    {
        int op1 = left.eval();
        int op2 = right.eval();

        if(op == "-") return op1 - op2;
        if(op == "+") return op1 + op2;
        if(op == "*") return op1 * op2;
        if(op == "/") return op1 / op2;

        if(op == "/" && op2 != 0) return op1/ op2;

        throw "error, bad op" + op + "int BinaryNode";
    }
};

class Ternary_node:public Expr_node
{
    friend class Expr;

    string op;
    Expr left;
    Expr middle;
    Expr right;

    Ternary_node(const string& a, Expr b, Expr c, Expr d):
        op(a), left(b), middle(c), right(d) { }
    void print(ostream& o) const;
    int eval() const;
};

void Ternary_node::print(ostream& o) const
{
    o << "(" << left << " ? " << middle << ":" << right << ")";
}

int Ternary_node::eval() const
{
    if(left.eval())
    {
        return middle.eval();
    }
    else
    {
        return right.eval();
    }
}

Expr::Expr(int n)
{
    p = new Int_node(n);
}

Expr::Expr(const string& op, Expr t)
{
    p = new Unary_node(op, t);
}

Expr::Expr(const string& op, Expr left, Expr right)
{
    p = new Binary_node(op, left, right);
}

Expr::Expr(const string& op, Expr left, Expr middle, Expr right)
{
    p = new Ternary_node(op, left, middle, right);
}

int main()
{
    Expr t = Expr("*", Expr("-", 5), Expr("+", 3, 4));
    cout << t << " = " << t.eval() << endl;
    t = Expr("*", t, t);
    cout << t << " = " << t.eval() << endl;

    t = Expr("?",Expr(1),Expr(2),Expr(3));
    cout << t << " = " << t.eval() << endl;
    return 0;
}
```

![images](/images/img_for_2015_05_19/output2.png)



----------
Photo by Zhouyin in ShangHai, China

![images](/images/img_for_2015_05_19/me.png)
