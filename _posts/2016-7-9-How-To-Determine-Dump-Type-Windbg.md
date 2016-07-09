---
layout: post
title: How to determine a crash dump type in WinDbg
---
Solving Windows crash dumps (particularly bugcheck 9F, for which I have a post on [debugging basics](http://mrbit.me/How-To-Solve-Bugcheck-9F-3/)) brings me a lot of satisfaction.

Windows has a few [different types of crash dumps](https://msdn.microsoft.com/en-us/library/windows/hardware/ff551880(v=vs.85).aspx) which have different levels of information available. Occasionally, I'd like to check which variety I'm looking at to see what information I can expect saved in the dump.

### How to view the crash dump type from WinDbg or KD

Use the `||` command.

```
kd> ||
.  0 64-bit Kernel bitmap dump: C:\Windows\MEMORY.DMP
```

You can see that this is from a [kernel dump](https://msdn.microsoft.com/en-us/library/windows/hardware/ff551867(v=vs.85).aspx).