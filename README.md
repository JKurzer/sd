# **SD**
An outstanding unsolved problem in the work around games determinism is determinism in games. Jokes aside, one of the very hardest things for any determinism framework to do is fit other existing tools into the architecture, design, and mathematical systems it uses. This is one of the big drivers of massive rewrites and tech debt in the industry as a whole.   

Slow Determinism doesn't make this problem go away. But it does solve it for us, for now, enough of the time. As we've built out Artillery, we've stumbled on a really unpleasant problem. It's not enough to know broadly when something will run. You need to be able to make it run at that time on all machines that need a deterministic simulation. There's all sorts of approaches, and this is a variation and combination of a few of them. To steal a joke from Linus Torvalds, I've named it slow determinism, because I like naming things after myself and I'm quite slow and quite determined. 

Before we go any further: This work is proof of prior art by Oversized Sun and should be considered a formal academic publication. As such, citation is expected should you reuse this work. I recognize this is not the norm for the games industry, but it's about time that we started. Why? Because it's a nice way to prove that individual humans make games and individual humans matter. Please see the attached license. 

# Slow Determinism In Practice
Normally, I'd do a really in-depth write up that doesn't require a ton of knowledge. However, that's just not possible in a reasonable amount of time, so we're going to state our required knowledge upfront followed by our givens. We're assuming you're using the Artillery framework. You can do this in other frameworks, but it's easy to describe how it might work in artillery.  

**Prior Knowledge:**  
Artillery provides five concepts you will need to be familiar with to proceed. These are: Ticklites, Ticks, Shadow Now, Cadence, and Conserved Attributes. You should probably understand how we use calculate versus apply, how barrage+jolt use deltas, and the anatomy of the artillery busy worker thread.

**Axioms:**  
Given a system that does not execute on the cadence of artillery...  
Given that this system can be executed on command...  
Given that this system can be known to execute within some fixed number of frames...  
Given a minimum framerate that can reasonably be expected...  

## **Core Ideas:**  
### **Harvester Ticklite**  
A ticklite that runs for a specific number of ticks, and each calculate step, it accumulates the set of changes coming from a queue or other single specific source. These changes take the form of Ticklite Add Requests, Conserved Attribute Changes, FBPhysics Inputs, and Registration Requests. During each apply step, it adds the accumulated requests to a block of stored requests and then orders all requests in the block by a deterministic order. Finally, On Expire, it issues all requests in the block.  
   
### **Shadow Realm**  
A Barrage+Jolt World kept at a specific tick offset (K) from the current frame, effectively freezing it at the start of that tick. This is achieved by applying the K-th oldest delta from the main physics world each tick. This only requires that a chain of deltas be kept. Shadow Realms do not allow modification, only read operations like spherecasting, broadphase queries, narrowphase queries, and similar. Generally, only one Shadow Realm is required, but you COULD have more. Effectively, this creates a snapshot of the Jolt Physics World at a known Shadow Now, hence the name.  
  
### **Wide Cadence**   
We already use the concept of Cadence in a number of places. A Wide Cadence is a cadence where the tick interval is large enough that the total duration of that many ticks is longer than two frames of your expected minimum framerate. For this discussion, a Wide Cadence is 8 or more ticks at 120hz tickrate. That's 15hz, so if your game runs at 60fps with few dips, this is 4 frames. 

**Design:**  
Surprisingly, we have everything we need now. Take your vanilla UE system. Let's call it our tree, because honestly, it'll probably be a state tree or similar. Make a Shadow Realm at the start of your Wide Cadence. Trigger the tree from artillery at the start of your wide cadence. It runs whenever it runs. As long as it finishes in four frames, we're good. If it wants to query Game Sim or modify Game Sim, like always, it needs to use Artillery to do this. Querying is done with the Shadow Now associated with the start of the wide cadence. Physics queries are made against the Shadow Realm. Changes are pushed into a single unified queue. This queue has a Harvester Ticklite associated with it. The Harvester Ticklite expires at the end of your Wide Cadence. That's the entire top-level design, actually. Slow is smooth, smooth is fast. This means that you don't really need to rewrite most systems in UE, just expose stuff to them using blueprint specifiers in Artillery, which you were going to want to do anyway, and which is already done for most stuff.  

**Supporting Rollback**
Rollback can be supported one of two ways. The hard way, which I'm not going to talk about. You wanna do it that way, good luck. I'll tell you that it IS possible. Or you can start your Wide Cadence in the past using a prepped Shadow Realm and the existing Shadow Now support of Conserved Attributes. This is why we use 8 ticks for the example. We basically start it 4 in the past, and give it 4 ticks or two frames to finish. You might consider using 10 ticks for your wide cadence. 10 times a second is only a max delay of 100ms, but the 67ms of 15hz is vastly preferable. It's perceptible, but barely, and it's below the 100ms annoyance threshold.

**When Will Artillery Support This?**  
Public support for **SD** in Artillery currently has no timeline. Right now, our implementation is pretty customized to our use-case and it's wormed through our game-side systems for execution speed reasons. If someone wanted to implement it as a series of PRs, we would definitely accept them. We will likely eventually push support for SD into public Artillery, but the work required to do so is enormous and it probably won't be until after the release of our first game.
