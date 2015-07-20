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
