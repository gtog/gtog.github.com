---
layout: post
title: "Measuring the Intensity of Historical Crises with VaR (Part 2)"
description: "Part 2 of a multi-part look at tail risk."
category: finance
tags: [finance, VaR, R, PerformanceAnalytics, quantmod, knitr, tail risk]
---
{% include JB/setup %}

Adam Duncan, December 2012

If you missed the first part of this analysis, be sure to check it out on gtog.github.com.  
In this part of the analyis, I'm going to look at the actual 1 day negative returns / VaR estimates  
("VaR breaks") across a numnber of different asset classes. The hope is to arrive at some general  
conclusions about how big historical events have been as measured by a simple VaR metric. Armed with this  
information, we can start to think about how to craft a reasonable tail risk hedging strategy. Crafting  
a good tail risk hedging strategy involves a lot more than what we're going to examine here, but this is  
where the foundation for a good program lies. You have to know the potential magintudes of various events  
before you can correctly size and design a good hedging program.   

I also promised to clean up the summary function in a faster format. So, here goes.  

As usual, import some libraries:

### Setting up the data

Let's go out and get the data we want to examine. This time we'll put each of the data elements in a list.  
We can pass this list of variables to our new summary functions.  

{% highlight r %}
tickers.equity = c("SP500", "DJIA")  # SP500 and DJIA
tickers.fx = c("DEXUSEU", "DEXMXUS", "DEXUSAL", "DEXJPUS", "DEXKOUS")  # EURUSD, USDMXN, AUDUSD, USDJPY, and USDKRW
tickers.commod = c("DCOILWTICO", "GOLDAMGBD228NLBM", "DCOILBRENTEU")  # WTI and Gold
tickers.rates = c("DSWP2", "DSWP10", "DSWP30")  # 2,10, and 30 yr US swap rates.
{% endhighlight %}


We'll also need a getData() function to help with getting all the data into our workspace:  

{% highlight r %}
getData <- function(tickers) {
    for (i in 1:length(tickers)) {
        suppressWarnings(getSymbols(tickers[i], src = "FRED", auto.assign = getOption("getSymbols.auto.assign", 
            TRUE), env = parent.frame()))
    }
}
{% endhighlight %}

We can now call this function with our ticker variables as arguments:   

{% highlight r %}
getData(tickers.equity)
getData(tickers.fx)
getData(tickers.commod)
getData(tickers.rates)
{% endhighlight %}


Now that all the variables are in our workspace, we can make some lists...  

{% highlight r %}
data.equity <- list(SP500, DJIA)
data.fx <- list(DEXUSEU, 1/DEXMXUS, DEXUSAL, DEXJPUS, 1/DEXKOUS)
data.commod <- list(DCOILWTICO, GOLDAMGBD228NLBM, DCOILBRENTEU)
data.rates <- list(DSWP2, DSWP10, DSWP30)
{% endhighlight %}


### Redacting our summary function...
As promised in [Part 1](http://gtog.github.com/finance/2013/03/27/tail-risk-part-1/), let's re-write the    
summary function to be a little more flexible and a little more efficient. First we need to make sure  
we have access to the varFun function from Part 1...  


Now we can re-write the summary function, vsum()...   

{% highlight r %}
vsum <- function(datalist, p, window, method, inv.var, draw) {
    # Now, tickers is a LIST. We'll cycle through the list and generate our
    # summary output.  We can dispense with having to get the data on the fly.
    # 'draw' is a boolean that tells the function whether to plot the ratio
    # series.
    
    n <- length(datalist)
    out <- data.frame()
    
    for (i in 1:n) {
        ratios <- varFun(datalist[[i]], p, window, method, inv.var, "ratios")
        ratios <- subset(ratios, subset = (ratios < 0))
        out[i, 1] <- names(datalist[[i]])
        out[i, 2] <- min(ratios)
        out[i, 3] <- mean(ratios)
        out[i, 4] <- median(ratios)
        if (draw == TRUE) {
            plot.xts(ratios, las = 1, main = names(datalist[[i]]), ylab = "act.ret/VaR est.")
            abline(h = -1, col = "red")  # a line to demark the ratios where actual return exactly equaled the VaR estimate.
        }
    }
    names(out) <- c("name", "max.ratio", "mean.ratio", "median.ratio")
    return(out)
}
{% endhighlight %}



### Plotting historical VaR Breaks  

Let's make use of our new summary function and look at just the FX data:  

{% highlight r %}
par(mfrow = (c(1, 2)))
vb.summary <- vsum(data.fx, 0.95, 1260, method = "historical", inv.var = FALSE, 
    draw = TRUE)
{% endhighlight %}

![center](https://github.com/gtog/gtog.github.com/figs/var_part2_markdown.RMD/plotFXData1.png) ![center](https://github.com/gtog/gtog.github.com/tree/master/figs/var_part2_markdown.RMD/plotFXData2.png) 

{% highlight r %}
vb.summary
{% endhighlight %}



{% highlight text %}
##      name max.ratio mean.ratio median.ratio
## 1 DEXUSEU    -2.959    -0.4664      -0.3529
## 2 DEXMXUS   -11.083    -0.4850      -0.3397
## 3 DEXUSAL   -80.792    -0.5614      -0.3661
## 4 DEXJPUS    -9.259    -0.4852      -0.3408
## 5 DEXKOUS   -32.176    -0.6584      -0.3133
{% endhighlight %}

![center](https://github.com/gtog/gtog.github.com/tree/master/figs/var_part2_markdown.RMD/plotFXData3.png) 


The AUDUSD data looks a little suspicious with a -80x VaR realization. That probably merits some investigation.  
Our new summary function gives us the ability to produce a table of VaR breaks, some summary statistics about  
the breaks, and plot the breaks if we want. Let's run the whole lot so we can see the table of the breaks and  
the associated plots.  
Warning: Running them all will take a little while. Actually, let's go ahead and do that and time the code:  

{% highlight r %}
ptm <- proc.time()
par(mfrow = (c(2, 2)))  # We'll do 4 plots per page.
data.master <- list(SP500, DJIA, DEXUSEU, 1/DEXMXUS, DEXUSAL, DEXJPUS, 1/DEXKOUS, 
    DCOILWTICO, GOLDAMGBD228NLBM, DCOILBRENTEU, DSWP2, DSWP10, DSWP30)
vb.summary <- vsum(data.master, 0.95, 1260, method = "historical", inv.var = FALSE, 
    draw = TRUE)
{% endhighlight %}


![center](https://github.com/gtog/gtog.github.com/tree/master/figs/var_part2_markdown.RMD/plotAllData1.png) 

![center](https://github.com/gtog/gtog.github.com/tree/master/figs/var_part2_markdown.RMD/plotAllData2.png) 

![center](https://github.com/gtog/gtog.github.com/tree/master/figs/var_part2_markdown.RMD/plotAllData3.png) 

{% highlight r %}
proc.time() - ptm
{% endhighlight %}

{% highlight text %}
##    user  system elapsed 
##  433.61    0.03  436.29
{% endhighlight %}

{% highlight r %}
par(mfrow = c(1, 1))
{% endhighlight %}

![center](https://github.com/gtog/gtog.github.com/tree/master/figs/var_part2_markdown.RMD/plotAllData4.png) 

### Summing up...  
So what do the different crises look like on average from a 95% historical VaR perspective? Meaning,  
on average, how bad would we have done if we were using a simple VaR framework to estimate our 1-day tail risk?  

{% highlight r %}
vb.summary
{% endhighlight %}



{% highlight text %}
##                name max.ratio mean.ratio median.ratio
## 1             SP500   -16.574    -0.4979      -0.3508
## 2              DJIA   -23.045    -0.4810      -0.3312
## 3           DEXUSEU    -2.959    -0.4664      -0.3529
## 4           DEXMXUS   -11.083    -0.4850      -0.3397
## 5           DEXUSAL   -80.792    -0.5614      -0.3661
## 6           DEXJPUS    -9.259    -0.4852      -0.3408
## 7           DEXKOUS   -32.176    -0.6584      -0.3133
## 8        DCOILWTICO    -9.326    -0.4761      -0.3401
## 9  GOLDAMGBD228NLBM   -12.074    -0.4904      -0.3239
## 10     DCOILBRENTEU    -5.043    -0.4817      -0.3583
## 11            DSWP2    -6.183    -0.5875      -0.4046
## 12           DSWP10    -9.708    -0.5826      -0.4122
## 13           DSWP30    -8.602    -0.6145      -0.4517
{% endhighlight %}



{% highlight r %}
mean(vb.summary$max.ratio, trim = 0.1)
{% endhighlight %}



{% highlight text %}
## [1] -13.01
{% endhighlight %}

So, throwing out the 10% best and worst cases for the VaR breaks, we get a trimmed mean of -12.5x VaR.  
From a visual inspection of the data, it would appear that for non-equity asset classes, somewhere between  
5-9x daily VaR seems to capture the worst case scenario for 1 day negative excessions. Equities and high beta  
FX pairs would fall somewhere in the 12-18x VaR range. Well, that's all for now. In the next part, we'll examine  
the duration aspect of crises. How long does it take to get to the bottom of an average crisis?  
How large are the cumulative drawdowns on average and how long does it take to get back to flat on average?  

Thanks for reading!

