---
layout: post
title: Code For Work - Windows USB Type-C Port Controller Driver (and explaining it under anesthesia)
---

My employer doesn't generally open-source code, so I am basking in the novelty of a driver sample I wrote being open sourced.

My main project at work since January (and probably for at least the next 6 months, if not longer) is the new USB Type-C stack in Windows, called UcmTcpciCx. It stands for the "USB Connector Manager Type-C Port Controller Interface Class Extension". I know that's a brilliant name - no autographs, please.

### How do you use UcmTcpciCx? Is it simple enough for a person on drugs to explain it?
I'm so glad you asked! I had hip surgery one week ago - as I was waking up from the anesthesia, my boyfriend [Cameron](https://github.com/cgutman) thought it'd be hilaaaarious to ask me to explain writing a client driver for UcmTcpciCx.
I can't promise this will be educational, but here goes:
(the following will open a link to the YouTube Video)

[![Youtube Anesthesia video - Type C](https://img.youtube.com/vi/jsfydau16fE/0.jpg)](https://www.youtube.com/watch?v=jsfydau16fE)

### Sample Client Driver
Anyway, a person writing a driver for their Type-C port controller on Windows would hook their driver into UcmTcpciCx. Here's the sample that I wrote:

[UcmTcpciCx Sample Client Driver on GitHub](https://github.com/Microsoft/Windows-driver-samples/tree/master/usb/UcmTcpciCxClientSample)