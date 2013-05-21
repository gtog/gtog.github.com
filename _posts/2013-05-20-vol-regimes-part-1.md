---
layout: post
title: "Volatility Regimes: Part 1"
description: "Examining the impact of volatility regimes on strategy selection."
category: finance
tags: [finance, Volatility, R, Regime Switching, strategy, clustering]
---
{% include JB/setup %}


This is a 'do over' of a project I started while at my former employer in the fall of 2012. I presented part 1 of this  
framework at the FX Invest West Coast conference on September 11, 2012. I have made some changes and expanded the  
analysis since then. Part 2 is complete and will follow this post in the week or so.  


Adam Duncan September, 2012  
Also avilable on [R-bloggers.com](http://www.r-bloggers.com "R-bloggers.com")  

Part 1: Constructing a Regime Framework  
-----------------------------------
  
### Motivation  

People often aske me, "What trades should we be doing right now?" or "What trades do you like right now?"  
I have often answered these questions with a combination of personal preference and an informal sampling  
of trades other people have been talking about. Rather than continue offering such subjective responses,  
I decided to embark on a small quest to apply some discipline answering those questions. What if we had  
a roadmap that told us: "When the world looks like this, do that." What if we could map out all states  
of the world and test all our favorite strategies in each of those states? We could then assess where  
we are and apply the strategies that do the best. The following is the first in a series of pieces that  
will develop this framework. It will:  

* Give you a framework for assessing the current state of the world.  
* Give you a way to test and rank your favorite strategies in each state or regime.  
* Give you a way to estimate the conditional probability of migtrating to another state.  
* Give you a way to form expectations about how long each state might persist.  

In addition, we will try to estimate a more formal regime swithcing model to see if the transition probabilities   
can give us any foresight into regime transitions. You should come away from this reading with a useable tool for   
deciding which stategies you should be employing, given the state of the world as identified by the FX vol surface.  
In short, we want to know if conditioning our strategy selection based on the volatility regime leads to better  
performance. But, first things first. Let's map the world...  

### Mapping the World

Volatiltiy regimes are nothing more than states of the world defined by volatility characteristics.  
What sort of characteristics? Well, in this framework, we consider 2 factors: (1) level of volatility   
and (2) the term structure or slope. These 2 factors give us a lot of information about what state the world is  
currently in.  

In a simple 4 state world, we might let each of our 2 factors take on 2 levels each. For example,   
vol levels might be (1) High or (2) Low  and our term structure of vol might be (1) Steep or (2) Flat.   
One can imagine looking at the world in 2008 when vol levels were quite high and term structures were massively  
inverted (short dated vols traded above longer dated vols). We might call this state 1: HIGH and FLAT.  
As the policy makers introduced measures and time passed, short dated vols came down, but overall vol levels  
remained quite high. This might be called state 2: HIGH and STEEP. As more time passed and market participants  
became more comfortable with the fallout fromt he crisis, we experienced a parallel shift lower in all vols, but  
long date risk premiums remained. That is, long dated vols still traded expensive to short dated vols, albeit at  
much lower levels. We might call this state 3: LOW and STEEP. One can imagine coming full circle back to the risk  
seeking months and years before the crisis where long dated risk premiums collapse and the term structure  
flattens out. This is similar to the 2005-2006 experience. Vol levels settle in a low levels and the slope  
of the term structure is relatively flat. We can call this state 4: LOW and FLAT.  

#### The Data  

Source: Bloomberg  
Frequency: Daily  
Vol Tenors:  
* 1 month  
* 1 year  

Start: 30-Dec-1994  
End: 11-Jan-2013  
Currency Pair: EURUSD

Note that you could average together, say, all of the G-10 and EM vols together and use some composite vol  
index as your term structure and level variables. At least as far as G-10 vols go, the results do not differ  
significantly from just using EURUSD vol.  

#### A Visual Inspection  

Before I get too far in any analysis, I like to make a visual inspection of the data.  This is always useful  
and can reveal problems or new ideas before too much work has been done. Let's do that here:


{% highlight text %}
## Version 0.4-0 included new data defaults. See ?getSymbols.
{% endhighlight %}

![center](http://gtog.github.io/figs/volregimes_part1_knitr/visualInspection.png) 


The visual inspection of the data reveals that our simple 2 levels per factor assumption is probably not the right  
formulation. looking left to right along the x-axis it's pretty clear that there are 3 distint clusters in the  
vol level data. There also appears to be 3 reasonable levels across the term structure variable, with points  
above, below, and well below the horizontal zero-line.  

#### A Shallow Dive into the Realm of Cluster Analysis  
The first thing we'll want to do is to standardize our variables by transforming them into z-score measures.  
This is always a useful thing to do when dealing with variables of vastly different bases. It also makes us  
level agnostic, which is a good thing.  

Rather than going through each day in history and deciding whether that day was STEEP or FLAT and  
HIGH or LOW, we can employ a simple clustering algorithm to split up our standardized variables into  
an arbitrary number of groups. In this case we have 2 groupings per factor. So, we can use a k-means clustering  
algorithm with 2 means per factor to split each factor into groups.  

#### Is our 2 Levels per Factor (4-State World) Appropriate?  

One natural question is,"Have we selected the right number of levels for each factor?" What if some other  
number of groupings is more appropriate for each of our factors? There is a way to determine the appropriate  
number of groupings for a given factor. It's very similar to using a scree plot in PCA to determine the  
number of principal components you want to use. By iteratively running the k-means clustering algorithm  
with a variable number of means, we can examine the sum of squared distances between the elements of each  
group. If we plot the results, we'll get something like this:  

![center](http://gtog.github.io/figs/volregimes_part1_knitr/makeScreePlot.png) 


The scree plots suggest that 3 or 4 groups is defintely more appropriate. It might be more appropriate to  
allow each factor (term structure and level) to take on 3 levels (high, medium, and low) and (steep, normal, and   flat),  
for example. 4 clusters might also work, but we start to lose some intuition about the construction.  
For now, let's just stick with our 4 state world and see if we can map the historical observations.  

After running the k-means clustering algorithm with 2 means per factor, we should have each standardized factor  
assigned to one of 2 groups. We can inspect the results of this assignment:

![center](http://gtog.github.io/figs/volregimes_part1_knitr/plotGroupAssignment.png) 


Now that both factors have been assigned a grouping, we can plot the results. A trellis plot is a nice way  
to view the data by factor grouping:

![center](http://gtog.github.io/figs/volregimes_part1_knitr/plot4States.png) 


The plot is nice in some respects. We can clearly see the crisis periods in the upper left quadrant  
"High/Flat".  And, each of the other quadrants seems to have reasonable representation. Though, there  
is a tremendous amount of overplotting (dots on top of each other). This suggests that our 4 state grouping  
might not be sufficient. We knew this, from our scree plot. Let's re-run the analysis and allow the k-means  
algorithm to bucket each factor into 3 groupings.  



It's helpful to visualize how our term structure and level z-score variables were grouped by the clustering algorithm.  
The following plots show the time series of the z-scores colored by the group they were assigned to.   You can easily see  
the "Steep", "Normal" (for lack of a better word), and "Flat" groupings. Similarly for the   level z-score variable.  


![center](http://gtog.github.io/figs/volregimes_part1_knitr/plotZScoresByColor.png) 


Now we can plot our 9 states as we did for our 4 state mapping:  
![center](http://gtog.github.io/figs/volregimes_part1_knitr/plot9States.png) 


### The *State* of Play  

Now that we have some nice states of the world mapped out, we might like to know something about the frequency  
of occupying a particular state and other questions like:
* Conditional on being in state s, what is the probability of transitioning to state r?
* Over the next n periods, how long are we expected to be in any one of the states?

A state table is a very good way to represent multi-state data. What follows is adapted from excellent work  
done by Christopher Jackson at the Department of Epidemiology and Public Health, Imperial College, London. 
The interested reader is referred here: [Multi-state modelling with R: the msm package](http://rss.acs.unt.edu/Rdoc/library/msm/doc/msm-manual.pdf)  


The state table shows the number of times each state has followed another state in succession in the data. 
Here is the state table for our 9 state representation of the world:

{% highlight text %}
##     to
## from   FH   FM   NH   NL   NM   SH   SL   SM
##   FH   63    3    4    0    0    0    0    0
##   FM    4   25    0    0    5    0    0    0
##   NH    3    0  157    0    9   10    0    0
##   NL    0    1    0 1004   20    0   70    0
##   NM    0    5   10   21  521    0    1   37
##   SH    0    0    5    0    0   69    0   10
##   SL    0    0    0   69    3    0  717    4
##   SM    0    0    3    1   37    5    6  755
{% endhighlight %}


This table shows the number of times that one state has followed another state in successive observation times. So,  
conditional on being in the "SM" state (the bottom row of the table), we have migrated to "SL" on *n* occassions,  
to "SH" on *m* occassions, to "NM" on *p* occassions, and so on. This is informative. It is also worth noting the zero  
entries in the table. This means that from some states we have NEVER transitioned to some other states. For example, we    
have never transitioned from "SM" to "FH". Nor from "SM" to "FH". Note that the count of observed states is a function  
of the grouping algorithm. Each time we re-run the clustering, we'll get slightly different answers. This is expected  
and you should run the algorithm many times to establish confidence intervals about the estimates.  

While *counts* of historical occurrences is helpful, let's see if we can estimate actual transition probabilities.  
We'll use this state table to seed the estimation. First we need to estimate an initial transisiton intensity matrix.  
Once we have the initial intensity matrix constructed (easily done with the msm package), we can then estimate the  
transition probabilities.  Here are the results of that estimation:  

{% highlight text %}
##       FH    FM    NH    NL    NM    SH    SL    SM
## FH 0.907 0.036 0.051 0.000 0.004 0.001 0.000 0.000
## FM 0.098 0.770 0.004 0.002 0.121 0.000 0.000 0.004
## NH 0.015 0.000 0.887 0.001 0.045 0.048 0.000 0.004
## NL 0.000 0.001 0.000 0.923 0.017 0.000 0.059 0.001
## NM 0.001 0.007 0.015 0.032 0.886 0.001 0.003 0.057
## SH 0.000 0.000 0.051 0.000 0.004 0.838 0.000 0.106
## SL 0.000 0.000 0.000 0.080 0.004 0.000 0.911 0.005
## SM 0.000 0.000 0.004 0.002 0.042 0.006 0.007 0.939
{% endhighlight %}


Excellent. Now we can say something about the 1-day probability of transitioning from one state to another.  
You can see the persistence present in each state. That makes sense. It's sort of like asking, "What is the  
probability that firm XYZ will be downgraded *TOMORROW*?" Well, there is a very high liklihood that whatever  
rating firm XYZ has today, it will have it tomorrow, too. But, there is always some small chance of a downgrade event.   
Similarly here, there is a very high liklihood that we will maintain the current state tomorrow. It might  
be more useful for us to estimate, say, 1 month ahead probabilities. Here are the 1-month transition probabilites:  


{% highlight text %}
##       FH    FM    NH    NL    NM    SH    SL    SM
## FH 0.138 0.035 0.164 0.100 0.207 0.064 0.056 0.236
## FM 0.091 0.026 0.117 0.155 0.210 0.046 0.096 0.258
## NH 0.051 0.016 0.127 0.131 0.213 0.063 0.081 0.317
## NL 0.009 0.006 0.023 0.414 0.120 0.009 0.306 0.112
## NM 0.025 0.012 0.067 0.221 0.203 0.031 0.148 0.293
## SH 0.029 0.012 0.096 0.133 0.215 0.052 0.088 0.376
## SL 0.007 0.006 0.020 0.420 0.111 0.008 0.325 0.104
## SM 0.018 0.010 0.063 0.173 0.214 0.033 0.118 0.370
{% endhighlight %}


Now, instead of a 94% chance of being in, say, "SM" tomorrow, we can say that there is approximately 30% chance of  
still being in "SM" 1 month from now, having started in "SM" today. Similarly, starting in "SM", we can  
see that "SL", "NM", and "NL" are the 3 most likely transitions from "SM". Those are the states we should  
be focusing on as portfolio managers trying to anticipate the next most likely states of the world.  
Confidence intervals of the probability estimates are available, but not shown here.

The last thing we'll do before moving on is to estimate the "total length of stay" for each of our  
designated states of the world. That is, over the next, say, 1yr, how many days do we expect to spend  
in each state?  Here is a table that shows the expected length of stay in each state (in days):  


{% highlight text %}
## FH FM NH NL NM SH SL SM 
## 18  5 20 60 43  8 43 56
{% endhighlight %}


This concludes part 1 of Volatility Regimes: Identification and the Impact on Strategy Selection.  
In the next part of the analysis, we'll select a number of strategies and run them without selection  
based on regime and then with conditioning on Volatility Regime. We'll show how the conditioning  
information improves performance. We'll also draw some conclusions about which strategies do best in   
which regimes. Lastly, we'll estimate a regime switching model and see how well it does in picking  
up changes in regimes.  

