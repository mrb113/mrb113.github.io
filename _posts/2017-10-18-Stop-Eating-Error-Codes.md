---
layout: post
title: Stop Eating Error Codes! Avoiding a Basic Software Anti-Pattern
---

# Anti-patterns
Loosely defined as "a commonly used, yet counterproductive solution to a problem".

If something gives you the "heebie jeebies", as I call it whenever I see weird-looking code, it's probably an anti-pattern. Recognizing anti-patterns in code is a valuable skill to have as both a programmer and a code reviewer.

# Anti-pattern: Eating error codes
I've seen this anti-pattern show up all the time (heck, I've probably done it before). Here's a quick demonstration using Windows/C programming constructs, but this is present in every scenario in any language where you can get a failure code back from a function.

```
//
// In Windows, HRESULT is a kind of error code.
// 
HRESULT WriteResultsToFile()
{
    ...

    // OpenResultsFile() returns S_OK on success, may return a variety of failure error codes.
    // For example, E_INVALID_PARAMETER if FilePath does not exist.
    HRESULT hr = OpenResultsFile(&FilePath);

    // Check to see if the function completed successfully
    if(FAILED(hr))
    {
        Trace("OpenResultsFile failed!");
        return E_FAIL;
    }

    // etc
    ...
}
```

Dude. `OpenResultsFile()` already gives you a specific failure case. Why did you overwrite it with the generic `E_FAIL`? You already know the error! Now we don't know anything about what went wrong because YOU ATE IT AND YOU SHOULD BE SAD.

## What you should do
Don't eat the error!

```
//
// In Windows, HRESULT is a kind of error code.
// 
HRESULT WriteResultsToFile()
{
    ...

    // OpenResultsFile() returns S_OK on success, may return a variety of failure error codes.
    // For example, E_INVALID_PARAMETER if FilePath does not exist.
    HRESULT hr = OpenResultsFile(&FilePath);

    // Check to see if the function completed successfully
    if(FAILED(hr))
    {
        Trace("OpenResultsFile failed! HRESULT: %!HRESULT!", hr);
        // Keep the existing value of hr
        goto Exit;
    }

    // etc
    ...
Exit:
    return hr;
}
```

This time, we keep the error we already got back from `OpenResultsFile()`.