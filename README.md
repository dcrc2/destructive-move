# destructive-move

The paper `destructivemove.html`
([view here](http://htmlpreview.github.io/?https://github.com/dcrc2cpp/destructive-move/blob/master/destructivemove.html))
is an attempt to show how 'destructive move' could be added to C++.

This paper aims to achieve full support for destructive
move, including:

  - It is possible to destructively move from objects
    which have automatic storage duration.
  - It is possible to write user-defined destructive
    move operations for a type. Both trivial and
    non-trivial destructive move operations are supported.
  - "Perfect forwarding" of prvalues (to the extent
    which is possible without breaking ABI for existing
    classes).

## Summary of this paper

This version of destructive move is expressed in terms of
three language features:

  1. *Moveable values.* A moveable value is a variable whose
     declaration indicates that it can be destructively moved from.
     In most respects a moveable value is similar to an
     ordinary non-reference variable, but there are some
     differences (e.g. moveable value parameters affect
     function overloading).

  2. A *forwarding operator,* like the one proposed in P0644.
     When applied to an lvalue reference or an rvalue reference,
     the operator is essentially just a shorthand for
     `std::forward`; but when it is applied to a moveable value,
     it performs a destructive move.

  3. *Destructuring operations.* These might look similar to
     structured bindings, which we already have in C++. But what we
     want for destructive move is a *true* destructuring operation:
     this is an operation which ends lifetime of the complete object and
     provides access to its subobjects in the form of moveable values.

As an example, these three features combine to provide a way to
write custom destructive move operations for a type:

```cpp
class C
{
    T m_t;
    U m_u;
    
public:
    C(C ~ other.*) : m_t(>>other.m_t), m_u(>>other.m_u) { }
};
``` 

Here the constructor parameter is a moveable value;
the declaration `other.*` indicates that
destructuring takes place; and the initialization of members
uses the forwarding operator `>>`.


## Pass-by-value vs pass-by-owning-reference

In the current version of the paper, a moveable value is
formally a kind of reference. This is not the only way to do it:
the alternative is to allow functions to move from parameters
which are passed by value.

The problem is that it's not clear how to use pass-by-value
while maintaining backwards-compatibility. In particular,
in the Itanium ABI, arguments passed by value are destroyed
by the caller after the function returns. This is not compatible
with allowing the function to move from its parameter.

One possibility would be to specify explicitly in the function
declaration that a different calling convention should be used.
Something like:

```cpp
void f(T t) moveable;
```

which indicates that all by-value parameters can be moved from.
(It would also affect other things: as in the paper, functions
which have moveable value parameters can be overloaded with
functions which have reference parameters.)

I would be interested to hear views on whether this would be
a better approach than the one in the paper, or whether there
is perhaps a better way to work around the ABI issue.

