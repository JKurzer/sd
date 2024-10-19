# _SD
An outstanding unsolved problem in the work around games determinism is determinism in games. Jokes aside, one of the very hardest things for any determinism framework to do is fit other existing tools into the architecture, design, and mathematical systems it uses. This is one of the big drivers of massive rewrites and tech debt in the industry as a whole. This doesn't solve that.  

But it does solve it for us, for now, some of the time. As we've built out Artillery, we've stumbled on a really unpleasant problem. It's not enough to know broadly when something will run. You need to be able to make it run at that time on all machines that need a deterministic simulation. There's all sorts of approaches, and this is a variation and combination of a few of them. To steal a joke from Linus Torvalds, I've named it slow determinism, because I like naming things after myself and I'm quite slow and quite determined. 

Before we go any further: This work is proof of prior art by Oversized Sun and should be considered a formal academic publication. As such, citation is expected should you reuse this work. I recognize this is not the norm for the games industry, but it's about time that we started. Why? Because it's a nice way to prove that individual humans make games and individual humans matter. Please see the attached license. 

