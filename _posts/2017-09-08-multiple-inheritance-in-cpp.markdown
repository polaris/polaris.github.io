---
layout: post
title:  "Multiple Inheritance in C++"
date:   2017-09-08 07:00:00 +0200
tags: cpp
---
There is the *general* recomendation that multiple inheritance is to be avoided. However, I think it is worthwhile to understand multiple inheritance and its consequences. Especially when using abstract base classes as interfaces, inheriting (implementing) multiple interfaces can make a lot of sense. If multiple inheritance is really needed be aware of some problems that might come up.

#### Bad design

Sometimes multiple inheritance is used where the object-oriented concept *composition* should be used. This mistake is ususally a misunderstanding of the concept of inheritance. Inheritance creates a *is-a* relationship between to classes. The derived class is a specialization of the class it inherits from. For example, a car is a vehicle. However, sometimes inheritance is misused to create a *has-a* relationship. In such cases composition is the more accurate relationship. For example, a car *has* an engine; the class car is not a specialization of the class engine, or the other way around, the class engine is not an abstraction of the class car.

#### Ambiguities

Consider the following example program. Here a class `C` inherits from classes `A` and `B`. Both, `A` and `B` have exactly the same method `foo()`. Note that the two methods have different definitions.

{% highlight cpp %}
class A {
public:
    int foo() { }
};

class B {
public:
    void foo(int bar) { }
};

class C : public A, public B {
};

int main() {
    C c;
    c.foo();
}
{% endhighlight %}

This program will not compile because the compiler does not know which of the two `foo()` methods to use.

{% highlight text %}
error: member 'foo' found in multiple base classes of different types
{% endhighlight %}

With a little modification we can help the compiler to choose a method:

{% highlight cpp %}
int main() {
    C c;
    c.A::foo();
}
{% endhighlight %}

#### The diamond problem

The diamond problem occurs when two superclasses `B` and `C` of a class `D` derive from the same base class `A`. In this case `D` gets two copies of all attributes in `A`. This structure causes ambiguity and very likely unintended behaviour too. Namely, the constructor and destructor of class `A` will be called twice.

Here is the code for the depicted scenario:

{% highlight cpp %}
#include <iostream>

using namespace std;

class A {
public:
    A() { cout << "A::A()\n"; }
    virtual ~A() { cout << "A::~A()\n"; }
};

class B : public A {
public:
    B() { cout << "B::B()\n"; }
    virtual ~B() { cout << "B::~B()\n"; }
};

class C : public A {
public:
    C() { cout << "C::C()\n"; }
    virtual ~C() { cout << "C::~C()\n"; }
};

class D : public B, public C {
public:
    D() { cout << "D::D()\n"; }
    virtual ~D() { cout << "D::~D()\n"; }
};

int main() {
    D d;
}
{% endhighlight %}

The output of this program is:

{% highlight text %}
A::A()
B::B()
A::A()
C::C()
D::D()
D::~D()
C::~C()
A::~A()
B::~B()
A::~A()
{% endhighlight %}

#### Virtual inheritance for the rescue

Depending on the actual class this is just a waste of resources or a more severe problem. The duplicate call to the constructor of class `A` might acquire limited resources or it might open a network connection. Furthermore, the destructor of class A might free something twice which could be problematic.

There is an easy way to solve the problem of duplication: virtual inheritance. By using the keyword `virtual` for the inheritance of classes `B` and `C`.

{% highlight cpp %}
class B : public virtual A {
public:
    B() { cout << "B::B()\n"; }
    virtual ~B() { cout << "B::~B()\n"; }
};

class C : public virtual A {
public:
    C() { cout << "C::C()\n"; }
    virtual ~C() { cout << "C::~C()\n"; }
};
{% endhighlight %}

With this modification the output changes to:

{% highlight text %}
A::A()
B::B()
C::C()
D::D()
D::~D()
C::~C()
B::~B()
A::~A()
{% endhighlight %}
