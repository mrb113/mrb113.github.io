---
layout: post
title: When can lock optimization make performance worse?
---

# Struct protected by a lock
Imagine you have a struct which your program accesses frequently.


```
Typedef struct MyStruct
{
    Field Foo;
    Field Bar;
}
```

There are competing threads, so you access it like:

```
Lock(myStruct);
// Access Struct
Unlock (myStruct);
```

Your program accesses this struct all day long from multiple threads, so there's lots of contention on the struct. Your program's performance isn't looking too good. Let's try to fix it!

# Lock around the individual fields? 
To “fix” it, maybe you try to put locks around the fields themselves. For example,

```
Lock(myStruct->Foo);
// Access field in struct.
Unlock(myStruct->Foo);
```

Should be better, right??? Each thread who wants the object now only has to compete for the field it wants, rather than waiting for the whole thing.

*Perf on accessing the object got worse after the “fix”. Why?*

# Why didn't perf improve?
You may have reduced contention on the object itself, but every time the processors wanted to go get the field in memory, they had cache-line contention while waiting for competing processors to get the same object out of memory. i.e. Proc0 wants `myStruct->Foo`, but has to wait for Proc1 to get `myStruct->Bar` out if it’s in the same cache. If your fields aren't in the same cache, you're just punting the contention down the line after changing your lock mechanism.

# How do we fix it?
If you're encountering cache-line contention on struct fields, the fix is to *cache align* the struct to the size of your cache line. The [C++ align keyword](https://docs.microsoft.com/en-us/cpp/cpp/align-cpp?view=vs-2017) is one platform specific way to do that.

