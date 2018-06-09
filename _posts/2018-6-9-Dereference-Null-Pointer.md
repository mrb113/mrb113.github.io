---
layout: post
title: What do you get when you dereference a null pointer?
---

A while ago, one of my friends was testing out ideas for potential interview questions for candidates at his company. One of the options:

> What do you get when you dereference a null pointer?

I don't know if I think this is a good interview question, but it's fun to think about what a `NULL` really means.

# Code (C)

```
// Setting p to point to NULL.
// Usually this means we'll point it somewhere else later
char* p = NULL;

char mystery = *p;

// What will be in mystery???
```

# Undefined behavior
A `NULL` deref is undefined behavior. Your program will not let you do it - full stop. The program will raise an exception or crash when you try to deref. But, because this is supposed to be an interview question, let's keep going and think about it more.

# How is NULL defined in the code
In C (unlike other languages like C++), `NULL` is defined via a macro. This can vary per implementation, but in the simplest case:
```
#define NULL 0
```

Remember, [macros are just text replacement](http://mrbit.me/Macros/), so we can think of everywhere `NULL` is above as `0`.
```
char* p = 0;

char mystery = *p;

// What will be in mystery???
```

# Wait, so why don't I just get whatever's at address 0?
Based on the above, you're telling the compiler you want what's at address 0 to go in `mystery`. Why don't you actually get the contents of address 0 and instead you get an access violation?

# What is NULL, anyway?
The C language definition states that for each pointer type, there is a special value (the null pointer) which is distinguishable from all other pointer values and which is "guaranteed to compare unequal to a pointer to any object or function."

Surprise! 'NULL' isn't really zero. It's just a construct to denote somewhere that's not really in memory. However, it needs to be denoted in code somehow, so we just pick `0` to denote our "out of bounds" `NULL` value. If you want to build a compiler that interprets "michelle b is super cool" as `NULL` (guaranteed to compare unequal to any object or function), you could make it happen, though I might wonder if you're doing okay.

C++ makes this slightly less confusing by removing the macro business and introducing the `nullptr` keyword to indicate this concept.

# NULL pointer isn't a real "pointer"
In general, we think of pointers as things which "point" to addresses in memory. `NULL` doesn't actually point anywhere - it's just a way to describe stuff that doesn't point anywhere. 