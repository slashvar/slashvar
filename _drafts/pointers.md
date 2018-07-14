---
Title: Pointers
---

Let's talk about pointers and memory. It's one of my favorite subject and in my
opinion one of the most important notion in programming.

Of course, talking about pointers implies talking about C or C++, they are the
two languages where you need them most and you can avoid them, but pointers are
everywhere in programming, that just that sometimes, they're hidden by the
language.

This post requires at least some basic knowledge of C, mostly its syntax and the
most obvious part of its semantics. If you're not a C programmer (but you're
    familiar with languages that use C syntax like Java) don't be afraid you'll
probably be able to understand most the text and, I hope, you'll learn something
useful.

## A bit of theory ##

For the sake of clarity, the rest of the text will assume a simple memory model
which more or less maps to what you could find on modern PC running Linux, that
is a model where memory addresses are just unsigned integers indexing bytes.

The semantics of the C programming language, in order to support (almost) any
kind of architectures is more complex. This model explains some particular
undefined behavior.

In short, what is defined is that you got addresses and you don't know they're
nature, the language only guaranteed the expected behavior of arrays and of
pointers used as array. The whole definition of pointers is a bit obscure, it
defined mostly when pointers are supposed to be equivalent and defined.

That is to say, you can live with it as long as you don't try to rely to much on
the numerical nature of pointers outside of arrays. But keep in mind, that
anything that is not explicitely defined (or defined as implementation
    dependant) may be compiled differently than what you expect.

So far for the theory.

## There's no arrow ##

The most common illustration of pointers involves boxes and arrows. After
several years of teaching C, it appears to me that this is the worst way of
presenting pointers.

A pointer is a value (often numerical) like any others. In order to manipulate
it, you need to store it (in a variable, a field of a structure, an array cell
    ... ) and you can copy it in another storage, it can be valid or
uninitialised, it can become invalid ...

```c
int x = 42;	// we an address to init our pointer
int *p;		// p is a variable that can contain a pointer to an int
p = NULL;	// p contains the null address, an existing but unusable pointer
p = &x;		// p now contains the address of x
int *q = p;	// q is a variable containing the same address as p
int y = 0;	// another variable in order to have an address
p = &y;		// p contains the address of y, q still contains the one of x
printf("%d %d\n", *p, *q);
```

When declaring a pointer as in `int *p`, you define a storage (a variable) where
you can put an address, but until you initialize it with a proper address, it is
unusable, just like any other variable.

Just like an address in real life, if you write it in you contact book, it
doesn't mean that it will be updated when the people living here moves or if the
house is destroyed, and if you have a line for the address of a contact but don't
put anything inside (or put something that is not an address) you won't be able to
send any letter to that address, it is not valid !

Once you got this in mind, you'll be able to understand most errors related to
pointers. It doesn't mean that you won't make errors or that you'll find them
easily, but at least you what you're playing with.

Let's now do the basic.

## Basic usage of pointers ##

The following piece of code illustrate classic usage of pointers, read it and
read the comments !

```c
#include <stdio.h>
#include <stdlib.h>

/*
 * reset(a) puts 0 at the address a
 */
void reset(int *a)
{
  *a = 0;
}

/*
 * swap(a, b) swap the integers stored at address a and b
 */
void swap(int *a, int *b)
{
  int c = *a; // copy in C the value stored at address a
  *a = *b;    // copy at the address in a the value at b
  *b = c;     // copy the value from c at the address in b
}

/*
 * divide(a, b, m) returns a/b (as int) an stores at m the rest
 * if b is null, returns 0 and don't touch what m points to
 */
int divide(int a, int b, int *m)
{
  if (b == 0) {
    return 0;
  }
  *m = a % b;
  return a / b;
}

int main()
{
  int x = 27;
  printf("x = %d\n", x);
  reset(&x); // send to reset the address of x
  printf("x = %d\n", x);

  int a = 2, b = 42;
  printf("a = %d\nb = %d\n", a, b);
  swap(&a, &b);
  printf("a = %d\nb = %d\n", a, b);

  int m = -1;
  printf("a/b = %d\n", divide(a, b, &m));
  printf("m = %d\n", m);

  return 0;
}
```

You can compile that code and run it:

```
> make basic.c
cc basic -o basic.c
> ./basic
x = 27
x = 0
a = 2
b = 42
a = 42
b = 2
a/b = 21
m = 0
```

Just to resume what we've seen: `reset(a)` is a function that use a pointer to
store a value outside of its scope; `swap(a, b)` exchange the content of 2
addresses and `divide(a, b, m)` computes integer division and store the rest at
an address provided. This three functions illustrate common (basic) use case of
pointers: be able to modify variables (or any storage) outside of the current
function.

Even if it's not the official vocabulary, you can see as a reference call
semantics, you pass a reference to a local variable in order to modify it.

The traditional call semantics of almost every language is a call by value:
function's parameters values are copied into the parameter storage. If you want
to modify variables local to the caller, you must pass the address of those
variables. The callee always assumes that you pass valid addresses (it can't
    verify them anyway ... )

## Arrays and pointers arithmetic ##

In most programming languages, an array is a contiguous area containing a
collection of object of the same type. They are of a fixed size (at some point
    at least, but you may be able to update their size).

In C, the implementation of arrays are related to pointers, but to the contrary
of what a lot of people think, arrays and pointers are not exactly the same, but
I'll show you that later.

If my address are numerical, accessing an element of array is as simple as
taking the address of the beginning and add the amount of bytes corresponding to
the number of elements to skip. For example, if I have 4 bytes integers and I
want to access the third element, I will add shift the address by 16 bytes (skip
    the first two). Knowing the size of the elements is annoying, the compiler
know their type and thus their size. This is where pointer arithmetic shows up !

Adding an integer `n` to a pointer, moves the address by `n` objects, provided
that this pointer points to an array of at least `n` objects (the language
    guaranteeds the existing of the address of the last object plus one).