---
layout: post
title: "Volatility Regimes: Part 2"
description: "Examining the impact of volatility regimes on strategy selection."
category: finance
tags: [finance, Volatility, R, Regime Switching, strategy, clustering]
---
{% include JB/setup %}


Adam Duncan from January, 2013  
Also avilable on [R-bloggers.com](http://www.r-bloggers.com "R-bloggers.com")    

Strategy Implications  
---------------------------------

In this part of the volatility regimes analysis, we'll use the regime identification framework established in part 1 to draw conclusions about which strategies work best is each regime. That should prove useful to us and goes a long way to answering the question, "What strategies should I be pursuing right now?"  

This first thing we need to do is assemble some strategies. We'll use rules-based strategies as our potential candidates. There are many such strategies available. Almost every investment bank has a whole suite of these products available for users to access. The strategies run the gammut from carry to momentum to relative value. People who design hedge fund replication strategies have boiled down the money making process into basically 5 broad categories:  

1. Momentum  
2. Carry  
3. Value  
4. Volatility  
5. Multi-strat / Alpha  

Rules-based strategies have been designed to pursue each of these strategies, some more complicated than others, but all designed to operate within a pre-defined set of rules and procedures. This is good because it eliminates concerns about style drift and other considerations which might confound our conclusions about how various strategies perform in different states of the world. Additionally, the investible world is usually broken down into asset classes that look something like: 

1. Equities  
2. Fixed Income  
3. Commodities  
4. FX  
5. Emerging Markets  

Some strategies operate across all asset classes simultaneously. With 5 strategy types and 5 asset classes, we are well armed to put together a collection of strategies that we can test across the possible states of the world as defined by our volatility regime framework.  

I have familiarity with rules based strategies designed by JP Morgan and Credit Suisse. Other banks have extensive collections as well. I'll limit my choices to those I am familar with. All of this data is available via Bloomberg (tickers are listed). Here are the strategies (including the asset class) that we'll examine:  

* Momentum - Equity - AIJPMEUU (USD)  
* Momentum - FX - AIJPMF1U (USD)  
* Momentum - Commodity (Energy) - AIJPMCEU (USD)  
* Momentum - Fixed Income (short dated) - AIJPMMUU  
* Carry -  FX (G10)- AIJPCF1U  
* Carry - Fixed Income (2yr) - AIJPCB1U  
* Carry - Commodity - AIJPCC1U  
* Carry - All - GCSCS2UE  
* Volatility - Equity (Imp vs. Realized) - AIJPSV1U  
* Volatility - FX - CSVILEUS (long only)  
* Volatility - Equities (CS RVIX) - CSEARVIX  
* Value - Emerging Markets (bonds) - EMFXSEUS  
* Value - Equities (CS HOLT RAII) - RAIIHRVU  
* Value - Commodities (CS GAINS) - CSGADLSE  

That's a reasonably comprehensive basket of strategy types and asset classes that should allow us to draw some insight. You can always backtest your own favorite strategy and analyze it across the various regimes.But for now, these readily available strategies should suffice to give us direction. I've imported data starting in 1994. For almost all of the indices, this time period encapsulates all the available data. Note, however that our first volatility information from Part 1 doesn't start until 28-Dec-1998. So, the analysis will have an effective start date of 28-Dec-1998. 

Some of the time series are disappointingly short. We may have to disqualify them on the basis of not crossing enough regimes. But for now, we'll leave everything in and plow ahead. We can exclude the shorter series when we aggregate the results. Regime snippets shorter than 20 days are discarded in the analysis Anything in the tables that shows up as NA or NaN, is something that was just too short to provide meaningful insight. Returns and standard deviations are all annualized.  

The first thing we'll do is base all the indices to 100 so we can see the proper evolution of each index. Each index will grow from 100 at a rate driven by it's daily log returns. Then, we'll examine the returns of each index during each of the identified volatility regimes. From there, we can decide which strategies do best and when. Here are the unconditional returns for each strategy (decreasing order by Information Ratio):  
{% highlight text %}
         ann.return ann.stddev info.ratio          strat.type
CSEARVIX   0.110052    0.05735    1.91881          VIX Vol RV
AIJPCC1U   0.064988    0.03595    1.80772     Commodity Carry
CSGADLSE   0.162870    0.15399    1.05770   Commodities Value
RAIIHRVU   0.081325    0.07776    1.04586      Equities Value
AIJPMMUU   0.047916    0.06010    0.79732  Fixed Inc Momentum
AIJPCB1U   0.055611    0.07073    0.78628     Fixed Inc Carry
GCSCS2UE   0.068753    0.10001    0.68748           All Carry
AIJPSV1U   0.022259    0.06344    0.35088 Imp/Real Volatility
EMFXSEUS   0.018915    0.05748    0.32906      EM Bonds Value
AIJPCF1U   0.030464    0.11183    0.27241            FX Carry
AIJPMF1U   0.012083    0.10461    0.11550         FX Momentum
AIJPMEUU  -0.006767    0.16454   -0.04113     Equity Momentum
AIJPMCEU  -0.045902    0.28178   -0.16290  Commodity Momentum
CSVILEUS  -0.014686    0.06444   -0.22789    Long Only FX Vol
{% endhighlight %}

Ok. Let's get down to the hard work of calculating all the return snippets for each of our possible states. Recall that there are 9 possible states, though our sample only has 8 of the possible 9 present. Recall also that we have 14 different indices to run across all of the possible states. Great care has to be taken to calculate, track, and assemble the return episdoes by state and strategy. After some difficult calculation and assembly, we can have a look at the information ratios by strategy, by regime: (Recall that we had no observations for "FL" in our sample. The "FM" regime also did not have any episodes longer than our 20 day minimum constraint. Annualizing returns from very short episodes is misleading. So, those 2 entries in our table are blank.)  

![center](http://gtog.github.io/figs/volregimes_part2_knitr/showStrategyGrid.png) 

### Regime Perscriptions  

Let's focus in on the current regime, "SL", and see what the data tells us about the different strategies. First, we might want to ask some questions about the current regime, like:  

*When did it start?  

*How long do regimes like this usually last?  

*What does the model say about expectations for how long this regime will last?  

The current regime (“Steep/Low” or “S/L”) began on October 11, 2012. As of January 11th, the time the data set was last updated, the current regime was 67 days old. The plot here shows the number of “SL” occurances in the sample by length of each occurence. So, we have seen 1 day “SL” regimes 23 times (those are excluded in the broader analysis). Similarly, we have seen an “SL” regime of 81 days length exactly 1 time in the sample. The current regime is getting a bit long in the tooth with respect to most occurrences observed in the data. Indeed, we have only seen 1 longer “SL” regime (the 81 day episode just mentioned).  

![center](http://gtog.github.io/figs/volregimes_part2_knitr/regimeLength.png) 

Recall the Total Length of Stay estimates from section 2.3 in Part 1. The total number of days we expect to spend in “SL” over the next year was estimated at 43 days. We have already exceeded that estimate by 24 days as January 11th and, as of today have exceeded even the longest “SL” regime observed in the sample. The upper-limit of the 97.5% quantile for the SL total length of stay estimate is 66 days. 

Here are the information ratios for each of the strategies, conditional on being in an "SL" regime [full sample:1998-present]:  

{% highlight text %}
     ticker  metric       strategy.type
1  AIJPSV1U  1.0704 Imp/Real Volatility
2  AIJPCC1U  0.9854     Commodity Carry
3  RAIIHRVU  0.6190      Equities Value
4  GCSCS2UE  0.5380           All Carry
5  AIJPCF1U  0.5341            FX Carry
6  AIJPMMUU  0.0943  Fixed Inc Momentum
7  EMFXSEUS  0.0048      EM Bonds Value
8  AIJPMEUU -0.0548     Equity Momentum
9  AIJPMCEU -0.1307  Commodity Momentum
10 AIJPMF1U -0.3470         FX Momentum
11 AIJPCB1U -0.7766     Fixed Inc Carry
12 CSVILEUS      NA    Long Only FX Vol
13 CSEARVIX      NA          VIX Vol RV
14 CSGADLSE      NA   Commodities Value
{% endhighlight %}

We can highlight the performance of the best performing “SL” strategy, Equity Implied vs. Realized Vol, only during “SL” regime epsiodes, as shown here:  

![center](http://gtog.github.io/figs/volregimes_part2_knitr/plotSLBest.png)  

It's also clear that Implied vs. Realized Vol does well in other regimes besides “SL”. Instead of ranking strategies by regime, we might also want to rank regimes, by strategy. Here is a such a list. That is, for each strategy, how do the regimes stack up in order of Information Ratio. Now is also a good time to compare the unconditional performance of a given strategy to the array of conditional performances by regime. For example, Imp/Realized Vol has an “SL” conditional information ratio of 1.07, an “NL” IR of 1.12, and a “SH” IR of 2.13. Compare that to an unconditional (ie. “always on”) IR of .35. It's not a panacea, however. For example, Commodity Carry as a strategy is not improved by conditioning on volatility regime. Here is a view of the strategy performances shown 2 ways: by regime and by strategy:  

![center](http://gtog.github.io/figs/volregimes_part2_knitr/rankRegimesByStrategy1.png) ![center](http://gtog.github.io/figs/volregimes_part2_knitr/rankRegimesByStrategy2.png)  

Recall the 1-month transition probablility matrix (shown again here) we estimated in part 1. The highest 1 month transition probabilities are (in order): "NL", "SL", and "NM". Meaning, those are the 3 most likely transitions from the current state. We are most likely to either 1) Enter "NL", 2) Stay where we are, or 3)Enter "NM". If we transition to "NL", what will the top strategies be and how do they differ from the current best strategies?  

{% highlight text %}
      FH    FM    NH    NL    NM    SH    SL    SM
FH 0.165 0.002 0.368 0.016 0.082 0.210 0.016 0.141
FM 0.066 0.010 0.157 0.098 0.259 0.121 0.072 0.218
NH 0.055 0.003 0.306 0.035 0.137 0.228 0.031 0.205
NL 0.004 0.004 0.028 0.358 0.174 0.038 0.266 0.128
NM 0.015 0.008 0.094 0.138 0.288 0.106 0.103 0.247
SH 0.033 0.004 0.230 0.058 0.182 0.204 0.049 0.240
SL 0.003 0.004 0.028 0.361 0.161 0.038 0.279 0.126
SM 0.016 0.007 0.120 0.115 0.256 0.134 0.091 0.261
{% endhighlight %}
 
A transition to "NL" suggests the following strategies:  

{% highlight text %}
     ticker  metric       strategy.type
1  AIJPSV1U  1.1193 Imp/Real Volatility
2  AIJPMMUU  0.9568  Fixed Inc Momentum
3  AIJPCC1U  0.7648     Commodity Carry
4  RAIIHRVU  0.7219      Equities Value
5  AIJPMF1U  0.6084         FX Momentum
6  AIJPCF1U  0.4046            FX Carry
7  EMFXSEUS  0.3365      EM Bonds Value
8  AIJPMEUU  0.1674     Equity Momentum
9  AIJPMCEU  0.1511  Commodity Momentum
10 GCSCS2UE  0.0856           All Carry
11 AIJPCB1U -1.1578     Fixed Inc Carry
12 CSVILEUS      NA    Long Only FX Vol
13 CSEARVIX      NA          VIX Vol RV
14 CSGADLSE      NA   Commodities Value
{% endhighlight %}

Those results suggest that a our carry strategies will remain ok, but we'll want to add exposure to fixed income momentum, as well as equity value strategies and perhaps FX momentum.

If we transition from "SL" into "NM", then...  

{% highlight text %}
     ticker  metric       strategy.type
1  CSEARVIX  1.0724          VIX Vol RV
2  AIJPCC1U  0.7495     Commodity Carry
3  CSVILEUS  0.6205    Long Only FX Vol
4  AIJPCB1U  0.5590     Fixed Inc Carry
5  AIJPMF1U  0.4160         FX Momentum
6  RAIIHRVU  0.3255      Equities Value
7  GCSCS2UE  0.2927           All Carry
8  AIJPMMUU  0.2511  Fixed Inc Momentum
9  AIJPMCEU  0.2472  Commodity Momentum
10 AIJPCF1U  0.1448            FX Carry
11 AIJPSV1U  0.0534 Imp/Real Volatility
12 AIJPMEUU  0.0339     Equity Momentum
13 EMFXSEUS -0.5480      EM Bonds Value
14 CSGADLSE      NA   Commodities Value
{% endhighlight %}

...we'll favor VIX vol relative value, FX momentum, and perhaps long only vol strategies. Note that "NM" is sort of a midling state where many strategies do reasonably well, with a few standouts. Note the shift in some carry strategies on this transition. This is a situation when vols rise, led by the front end of the term structure. This pushes levels and term structure back into higher levels. Thus the "NM" designation.When this happens, carry strategies usually get hurt and the results seem to bear this out. Given the current "SL" prescriptions, one would probably want to hedge the possibility of entering the "NM" regime.  

### Conclusion
 
Whether or not you think volatility term structure and level are good conditioning information for strategy selection, the framework should nonetheless prove valuable. You could add aditional conditioning factors to the model and derive a different set of prescriptions. For some strategies, particularly vol based strategies, this model improves performance quite nicely. We might imrprove our stratgey selections by adding other asset class volatilities, or perhaps creating a composite volatility measure made up of may asset class vols and term structures.   

#### Next up: Part 3: Estimating a regime switching model.