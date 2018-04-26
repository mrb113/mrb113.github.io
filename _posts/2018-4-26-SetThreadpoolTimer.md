---
layout: post
title: SetThreadpoolTimer, or "Why is time so hard in Windows"
---

If you've ever written code to fire a timer in x seconds from now, it's generally pretty straightforward. Usually you just call some function and tell it "I want you to fire in 5 seconds" and "here's what method to call once the timer goes off!".

Recently, I noticed some odd behavior in the Win32 API `SetThreadpoolTimer`. 

Here's the MSDN doc page, which I'll be referencing a lot throughout this post.
https://msdn.microsoft.com/en-us/library/windows/desktop/ms686271(v=vs.85).aspx 

# SetThreadpoolTimer - brief overview

```
VOID WINAPI SetThreadpoolTimer(
  _Inout_  PTP_TIMER pti,
  _In_opt_ PFILETIME pftDueTime,
  _In_     DWORD     msPeriod,
  _In_opt_ DWORD     msWindowLength
);
```

Check out the info for the `pftDueTime`:
> A pointer to a `FILETIME` structure that specifies the absolute or relative time at which the timer should expire. If positive or zero, it indicates the absolute time since January 1, 1601 (UTC), measured in 100 nanosecond units. If negative, it indicates the amount of time to wait relative to the current time. For more information about time values, see File Times.

> If this parameter is `NULL`, the timer object will cease to queue new callbacks (but callbacks already queued will still occur). Note that if this parameter is zero, the timer will expire immediately.

There's a lot going on in this one - it can have 2 different meanings depending on what you pass.

1. Positive: Absolute time to expire (example: "thou shalt expire on Wednesday, April 26 2018 at 5:00pm")
2. Negative: Relative time to expire (example: "thou shalt expire in 5 seconds")

# Using SetThreadpoolTimer to fire a relative timer.

Let's to focus on case #2 from above - setting a relative time to expire (say 5 seconds).
Per the docs, we just need to create a `FILETIME` to represent 5 seconds, then pass it as a negative value to `SetThreadpoolTimer` to indicate a relative time.

Here's `FILETIME`

```
typedef struct _FILETIME {
DWORD dwLowDateTime;
DWORD dwHighDateTime;
} FILETIME, *PFILETIME;
```

Here's the `FILETIME` doc
https://msdn.microsoft.com/en-us/library/windows/desktop/ms724284(v=vs.85).aspx

## Converting to FILETIME

Of course, we can't just shove some number into FILETIME via casting and call it good - you see it has a high part and a low part. Your `FILETIME` will mean something different on a big endian vs. a little endian system if you cast to it.

Luckily, the docs have told us exactly what to do to get our `FILETIME`.
> you should copy the low- and high-order parts of the file time to a `ULARGE_INTEGER` structure... and copy the LowPart and HighPart members into the FILETIME structure.

So we want to put "-5 seconds" in the filetime. So we get out our `ULARGE_INTEGER` to put our number in... 

# Things start to go wrong

...wait. The `U` stands for something... *unsigned* large integer. How are we going to get a negative number into something unsigned??!?

"Never fear", you think. "I'll just use a `LARGE_INTEGER`. it's signed!"

So you get your high part and low part to put in the `FILETIME`.... but...

# And more wrong

The high part of the `LARGE_INTEGER` is a `LONG` (signed, lets us have our negative sign) and the high part of `FILETIME` is a `DWORD` which is... not signed. How are we ever going to get a negative number to give the function?

# All I wanted to do was set a timer :(

Doing this sign mumbo jumbo is technically incorrect, but it seems to still work. 
They must check for negatives somewhere along the line in order to make their function work as advertised. Casting to a signed number would make "negative" again because it was stored all along in its two's complement representation.

`SetThreadpoolTimerEx` appears to have the same issue. Anyone have a better idea?
