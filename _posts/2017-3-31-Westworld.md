---
layout: post
title: Observations on the future of computing (if the "Westworld" TV series is to believed)
---

I've had a lot of free time the past week and a half because I had hip surgery last Wednesday. During my time on the couch, I watched the entirety of HBO's highly-addictive sci-fi/Western series, [Westworld](http://www.hbo.com/westworld). Here are some silly observations about the future of tech if the future is the one imagined in the show.

The show takes place sometime in the future - the premise as described by Wikipedia: "The story takes place in the fictional Westworld, a technologically advanced Wild West–themed amusement park populated by android hosts. Westworld caters to high-paying guests, who may indulge in whatever they wish within the park, without fear of retaliation from the hosts."

### Computing power

These guys are two of the androids in the park. They look and act real, which obviously requires a lot of computing power!
![Androids](../images/westworld.jpg)

For perspective, [Google's DeepMind AI used 1202 CPUs and 176 GPUs to beat the world champion of the Chinese board game Go](http://www.businessinsider.com/heres-how-much-computing-power-google-deepmind-needed-to-beat-lee-sedol-2016-3). 

Presumably, the future of Westworld is having luck with Moore's Law in order to have hundreds of realistic androids packed with that much computing power. For some reason, they have an AI and hardware in each individual host rather than powering the AI in "the cloud" - that sounds super expensive! If they bring a host offline, they can't use those computing resources for anything! But hey, the major plot points of the show would be totally ruined if they could do it in the cloud.

### Processors and OS versions

HBO set up a [website for Delos Incorporated](http://delosincorporated.com/), which is the corporate owner of the Westworld theme park in the show. It's more of a fun little Easter egg for viewers than anything else. They'd update it with relevant information to match the plot as more episodes came out. 

*-very mild spoilers, but nothing more than you can gain by just visiting delosincorporated.com on your own-*

Delos Incorporated is currently experiencing some technical malfunctions as of the end of Season 1. Their website reflects this (screenshot taken 3/31/2017 - I assume it will change when Season 2 comes out).

![Delos website](../images/delos.PNG)

Wow, there are a lot of fun things we can learn about Delos Incorporated and the "future" here from the stack trace that's printed on the screen!

- Delos Incorporated is using Red Hat.
- They're using an old kernel, especially because this is the future! 2.6.32 reached End of Life in February 2016. Hopefully Red Hat is keeping that up to date for them.
- They're using 64 bit Intel CPUs! awesome.
- Less awesome: They seriously need either a BIOS update or to replace CPU 8. "hard LOCKUP" means that the CPU hasn't responded to an interrupt within the alloted time. Come on, guys.

What do you think?
