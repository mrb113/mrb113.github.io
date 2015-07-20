---
layout: post
title: testing in prod is bad but that is what is happening
---

When I first learned to program in high school and college, the emphasis was on correctness and not on performance. When I taught students programming in college as a teaching assistant, I would often point out a suboptimal code path to the student. I can't tell you how many times I heard the following phrase: "Well, processors are fast enough these days that it doesn't really matter. Even still, the compiler will optimize it away". 

*No! Stop that right now!* The tireless work of the industry to bring you faster processors and better compilers is not an excuse for the programmer to write subpar code! 
 
In fact, I probably said that one day, but then I grew up to be a systems programmer where saving a few extra clock cycles could be the difference between a laggy experience and a smooth one for a user. Optimization and performance is the name of the game. 

"But Michelle", you say, "My job isn't anything like that. It's not such a big deal if things aren't optimized like crazy". Fine, but what job are you in that you don't want to give your users the absolute smoothest experience in town? Even if you are only programming for practice and nobody will ever see your finished product, I urge you to think about performance even if it slows you down at first. 
Why? It becomes natural after a while to use performant patterns and you'll end up with faster code without even thinking about it. 

This article uses C for the examples since that's what's easiest for me to write, but I am avoiding using things specific to C so that any programmer can understand and apply them. 

---

*__Time for a game of "Can We Do Better"!__*

**Problem:** Write a simple algorithm that inserts a number into a table using a hash that takes a seed, such that inserting `x` into a table with seed 0 would look like: 
```
`
slot = Hash(x, 0);
table[slot] = x % tablesize - 1; 
``` 
######(We subtract 1 because the table is zero indexed. Naughty off-by-one errors are not welcome here).  

Got it? Simple hashing.

If `table[slot]` is already full, we try `Hash(x, 1)`, `Hash(x, 2)` until we have our slot. For the purposes of our problem, `Hash(x, seed)` is a good hash function that generates few collisions and will give a valid slot for some seed (that is, there won't be an infinite loop or stack overflow while we're looking for our hash slot. The function is good, people. Moving on). I'm also including int TimeConsumingOperation() which represents a time consuming operation that can fail - fairly standard thing that shows up often in real life. 

*p.s. This is a simplification of one of my favorite problems: Perfect Hashing. That's for another post, though. If you're interested in my performant perfect hashing program that this example is based on, but you can see it [here](https://github.com/mrb113/PerfectHash).* 

So, we have our unoptimized version that we banged together: (yes, I am hypothetically using stdbool.h. If you don't use C, you may not know that C does not come built in with boolean true/false values! So we have to add them in ourselves)

```
/* Inserts a value into the table array */ 
bool InsertIntoTableUnoptimized(int* table, int tablesize, int value) {
	int slot; 
	bool slotFound; 

	printf("Doing some time-consuming operations on the table"); 
	if (!TimeConsumingOperation()) {
		printf("Insertion into table failed!\n"); 
		return false; 
	}	 
	if(TableIsFull) {
		printf("Insertion into table failed!\n"); 
		return false; 
	}
	
	printf("Inserting %d into the table.\n", value); 
	// Insert into the table using the hash
	// Remember to declare any variables you need outside of the while loop!
	// "Nice loop you have there... shame if we had to alloacate an int or two
	// every time we went through it... "

	int seed = 0; 
	slotFound = false; 
	while (!slotFound) {
		slot = Hash(value, seed) % tablesize  - 1; 
		if (table[slot] == 0) {
			// Slot is occupied - try again.
			seed++; 
			break; 
		} else {
			// Slot is available - insert
			table[slot] = value; 
			slotFound = true; 
			return true; 
		} 
	}	
}
```
It answers our question and incorporates some good programming practices. You may have noticed some iffy things unfolding, but hey, it works!

**...but can we do better?**


###Removing duplicated code 
Duplicated code is one of those things that you hope your compiler will look kindly upon you for, but let's nuke it for readability and just in case we're using Joe Bob's Hillbilly Compiler. You see that each time we fail, we print an error message and return false. If you're not a C person or systems programmer, your first thought might be "make a method that you call on failure". Good idea, but I don't like it here. This may be good in certain languages/situations, but I'm gonna show you what the industry standard is for native code. If you don't already know what's coming, then chances are I might shock you a little due to some misinformation you may have heard. 

```
/* Inserts a value into the table array */
bool InsertIntoTableUnoptimized(int* table, int tablesize, int value) {
	int slot; 
	int seed; 
	bool slotFound; 

	printf("Doing some time-consuming operations on the table"); 

	if (!TimeConsumingOperation()) {
		goto Failure; 
	}
	 
	if(TableIsFull) {
		goto Failure; 
	}
	
	printf("Inserting %d into the table.\n", value); 
	// Insert into the table using the hash
	// Remember to declare any variables you need outside of the while loop!
	// "Nice loop you have there... shame if we had to alloacate an int or two
	// every time we went through it... "

	seed = 0; 
	slotFound = false; 
	while (!slotFound) {
		slot = Hash(value, seed) % tablesize - 1; 
		if (table[slot] == 0) {
			// Slot is occupied - try again.
			seed++; 
			break; 
		} else {
			// Slot is available - insert
			table[slot] = value; 
			slotFound = true; 
			return true; 
		} 
	}
Failure: 	
	printf("Insertion into table failed!\n"); 
	return false; 
}
```

Yes, that is a goto that you see! The goto has been unfairly prosecuted in programming classes across the globe and even by [Dijkstra himself](http://www.u.arizona.edu/~rubinson/copyright_violations/Go_To_Considered_Harmful.html). As a former teaching assistant for many programming classes, I am fully aware of the monstrosities that students are capable of, so I do understand banning it from programming classes. You guys are mature enough to wield the power of goto... right? I'm trusting you to not commit any sins with it now that you know that goto is actually an industry standard in failure and cleanup cases. 

**...but can we do better?**

## Fast Fail: 
You see how we have that big `TimeConsumingOperation()` happening right up front? Say that's O(N) or worse. We know that the check for `TableIsFull` is O(1)... and can fail. If that fails after we did the time consuming operation, we throw our hands up and say
 
>*"Why'd I even waste my time? Ya know, Marsha, I told myself we were staying together for the kids, but really that wasn't true and I just got hurt. I should have gotten outta there long ago".*

Whoa, okay, maybe not quite like that, but we really should do the short operation that can fail before we do anything else. This is called fast fail, and looks like this:

```
// Do the quick operation that can fail before we do anything that requires heavy lifting
if(TableIsFull) {
	goto Failure; 
}

printf("Doing some time-consuming operations on the table"); 
if (!TimeConsumingOperation()) {
	goto Failure; 
}	
``` 
This seems so easy (and it is!) but it tends to be a "gotcha" for even the most experienced programmers. What usually happens is that we'll go in and add something after the fact without thinking about how we should have really made it a fast fail. Someone catches us in code review and then we end up crying to Marsha about how we should have bailed... or something.

We've gone for the low hanging fruit already and our program looks pretty decent.

**...but can we do better?**

###Watch Your Loops
I'm not sure if this practice has a name, but we have to think really hard about whether our loops can get out of hand. In our case, we already have the guarantee that our Hash(value, seed) will eventually give us a value... but what if it takes 1000 iterations for a certain value to find a seed that works? What if that keeps happening for all zillion values that we're inserting? We can't afford that on Jimbo's Dinosaur Machine. Jimbo's gonna toss that machine right out the window and sue us for the cost of the new one... or something. 

What we really want is a version of fast fail for our loop.

```
// This number will really depend on your needs and the nature of your application
// Play around: Lower if you are extremely concerned about perf, higher if you just want to avoid a stack overflow
#define MAX_TRIES 20
...
while (!slotFound && seed < MAX_TRIES) {
	slot = Hash(value, seed) % tableSize - 1; 

	if (table[i] == -1) {
		// Slot is occupied - try again.
		seed++; 
		break; 
	} else {
		// Slot is available - insert
		table[slot] = value; 
		slotFound = true; 
		return true; 
	} 
}
// We tried the max number of times, but didn't find a slot
if (!slotFound) {
	goto Failure; 
}
```

Our code looks pretty good so far and strikes a pretty nice balance of optimization and readability. 

**...but can we do better?**

###Power of Two Tricks
We find a slot with the following calculation:
`slot = Hash(value, seed) % tableSize;`
Did anyone see that *mod* (%) in there?! It's looking so innocuous, but mod is division which is very expensive from a hardware perspective. If we care about minimizing the number of cycles, we want to banish it ASAP. We can do a dirty trick to stop doing that disgusting division, though. The space overhead could be higher, but if we're okay with that, let's go full speed ahead. 

Ready? 

Make the tableSize a power of two! Then we can take advantage of a binary operation instead of mod. This is much more efficient on a processor level because there are single processor operations for this as opposed to mod which requires special hardware or complicated algorithms, which makes things rough if we want to run this on Bertha's Cheapo FPGA. 
```
// Works if the tableSize is a power of 2. 
slot = Hash(value, seed) & tableSize - 1; 
```
Now, if you'll excuse me, I'll pull the "verification is left to the curious reader" trick now.

**...but can we do better?**

###Bonus Round: Inlining
Welcome to the bonus round! Here we'll talk about things that are a little more intense than your garden variety optimization that we did above and aren't necessary for most cases, but I encourage you to at least consider them to get into the spirit of optimization the next time you're coding. The point is that it could be worth it to experiment with your program and take advantage of facts that the compiler couldn't possibly know.

Remember that magic `Hash(x, seed)` function that we have? Well, let's say that we've determined that `Hash(x, 0)` works 65% of the time and that we don't have to try any more seeds after 0. If we are really really gung ho about minimizing cycles, we can take advantage of this fact and force the compiler to optimize for the path where `seed = 0`. 
```
Pseudocode: 
Hash(x, seed) {
	// Hash algorithm goes here
	// Seed gets added, subtracted, multiplied, shifted, whatever.
	// x + seed
	// etc
}
```

```
HashZeroInlined(x) {
	// Hash algorithm adds here. 
	// Replace seed with 0
	// x + 0, or just x
}
```
We'd of course use the `HashZeroInlined(x)` function first in our program. If it failed, then we'd go ahead with the loop that finds a seed that works using `Hash(x, seed). 
**...but can we do better?**

###Double Bonus Round: Compression
Oh my, now we're getting really fancy. What if our table has a large size and doesn't get insertions very often? 

You mean to tell me that we're keeping this behemoth table that's mostly empty in memory?! 

*Go wash that mouth out with soap, young lady!*

I'm not going to do any code for this portion, but think about it: If memory usage is important to us and we have a big data structure that's mostly empty and that we don't touch all that often, we can maybe think about using a compression algorithm on it. You're playing with fire a little bit here since you're adding extra overhead to uncompress it when you need it, but in some cases it's worth it. The point here is that you need to make considerations like this and do the necessary experimentation to determine whether or not it's worth it for your case. 

**...but can we do better?**

 You tell me! Drop me an email at michelle@mrbit.me. 