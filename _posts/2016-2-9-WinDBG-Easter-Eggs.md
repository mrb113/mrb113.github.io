---
layout: post
title: WinDBG Easter Eggs
---

Today I stumbled across (via the [OSR Twitter page](https://twitter.com/OSRDrivers/status/685119806723469314/photo/1) ) a funny Easter egg in the [!ndiskd](https://msdn.microsoft.com/en-us/library/windows/hardware/ff552270(v=vs.85).aspx) kernel debugger extension for WinDBG.

`!ndiskd.help` does as you’d expect and prints info about the commands within that extension:

![!ndiskd.help](../images/ndiskdhelp_regular.png)

If you ask for help with a sense of urgency:

`!ndiskd.help!`

![!ndiskd.help!](../images/ndiskdhelp_urgent.png)

It exclaims “Don’t Panic!”

---

I sent this to all 3 of my other friends who think WinDBG jokes are funny, including [Alex Ionescu](http://www.alex-ionescu.com/), who deserves credit for bringing my attention to another cutesy undocumented debugger extension. 

`!IToldYouSo`

![!IToldYouSo](../images/itoldyouso.png)

It is exactly the same thing as [!chksym](https://msdn.microsoft.com/en-us/library/windows/hardware/ff562234(v=vs.85).aspx), but with a more amusing name.

Let me know if you are aware of any other fun undocumented WinDBG Easter eggs like these - I'd love to hear.
