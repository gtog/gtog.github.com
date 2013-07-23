---
layout: post
title: "Monitoring an ETF Portfolio in R - Part 1"
description: "An implementation of the 'endowment portfolio' using ETFS"
category: finance
tags: [Portfolio Construction, Finance, Investing, Performance Analytics]
---
{% include JB/setup %}


Adam Duncan
Also avilable on [R-bloggers.com](http://www.r-bloggers.com "R-bloggers.com")  


Some time ago, I read an interesting article about an interview with David Swensen, the renownd
money manager from Yale University's investment office. He's quite famous and is considered by many
to be the architect of the modern "endowment portfolio."  The point of the article was to suggest
a way for ordinary investors to replicate the "endowment portfolio" using widely available, low cost ETFs. 
This is not the original piece I read, but this [article](http://www.investmentu.com/2013/March/david-swensen-and-the-yale-model.html)conveys the essentials of the original post. This idea appealed to me and I wrote down the tickers and weights in my notebook 
for some later date when I could take action on it.

From the article I linked to above...

"Here’s a look at Swensen’s asset allocation recommendations:"

Asset Class | %
----------- | --
Domestic Stocks	| 30%
Foreign Developed Stocks | 15%
Emerging Market Stocks | 5%
Real Estate and Natural Resources | 20%
U.S. Treasury Bonds | 15%
U.S. Inflation-Protected Securities (TIPS) | 15%

All of the ETFs suggested in the original article were Vanguard ETFs and that seemed fine with me. I
should also note that this simple ETF portfolio differs from a typical endowment portfolio in many 
non-trivial ways. But, my goals were pretty modest at the start and the allocations seemed pretty
sensible.

So, on October 17th, 2012 I deployed an initial investment of 250,000 into the following portfolio
of ETFs. This post is going to show you how to monitor such a portfolio in R. For people familiar with quantmod and Performance Analytics, the discussion will be pretty rudimentary. In the next post I will try to raise some questions about VaR excessions, drawdowns, and de-risking. So, let's get started.

First we need to specify a data source and the tickers for our portfolio. I'll also fetch data for the DOW and the SP500, as well as the 2yr US swap rate. All for benchmarking purposes.


{% highlight r %}
library(quantmod, warn.conflicts = FALSE, quietly = TRUE)
library(PerformanceAnalytics, warn.conflicts = FALSE, quietly = TRUE)

# Set up the data source and the tickers for all the ETFs...
data.source = c("yahoo")
tickers.etf = c("VGSIX", "VUSTX", "VGTSX", "VFISX", "VTSMX", "VFITX", "VEIEX", 
    "VIPSX")
tickers.human = c("Real Est", "USTSY Long", "For Eq", "USTSY Short", "All Eq", 
    "USTSY Int.", "EM Eq", "Infl Sec.")
tickers.bench = c("DJIA", "^GSPC")
tsy.2y <- c("DGS2")

# Get the data from yahoo and FRED...
suppressWarnings(getData(tickers.etf, data.source))
{% endhighlight %}



{% highlight text %}
## VGSIX 1
{% endhighlight %}



{% highlight text %}
## As of 0.4-0, 'getSymbols' uses env=parent.frame() and auto.assign=TRUE by
## default.
## 
## This behavior will be phased out in 0.5-0 when the call will default to
## use auto.assign=FALSE. getOption("getSymbols.env") and
## getOptions("getSymbols.auto.assign") are now checked for alternate
## defaults
## 
## This message is shown once per session and may be disabled by setting
## options("getSymbols.warning4.0"=FALSE). See ?getSymbol for more details
{% endhighlight %}



{% highlight text %}
## VUSTX 2 
## VGTSX 3 
## VFISX 4 
## VTSMX 5 
## VFITX 6 
## VEIEX 7 
## VIPSX 8
{% endhighlight %}



{% highlight r %}
suppressWarnings(getData(tickers.bench, data.source))
{% endhighlight %}



{% highlight text %}
## DJIA 1 
## ^GSPC 2
{% endhighlight %}



{% highlight r %}
suppressWarnings(getData(tsy.2y, datasrc = "FRED"))
{% endhighlight %}



{% highlight text %}
## DGS2 1
{% endhighlight %}


I use of a couple of helper functions I wrote to make life a little easier:
getData(), makeIndex(), calcWeights(), and getOHLC(). The code for these functions can be found at the end of this post and on [github](https://github.com/gtog).

Put all the variables that were just loaded into the environment into a list and set the inception date
of the portfolio:


{% highlight r %}
etfs <- list(VGSIX, VUSTX, VGTSX, VFISX, VTSMX, VFITX, VEIEX, VIPSX)
first.date <- c("2012-10-17/")  # inception data for portfolio...

# Get all the closing prices into a vector starting from the inception
# date...
etfs_close <- getOHLC(etfs, "C")
etfs_close <- etfs_close[first.date]

# Specify the initial number of shares that were purchased and the initial
# investment...
shares <- c(2347.447, 977.96, 867.798, 1167.414, 2086.846, 1080.06, 1409.844, 
    2615.382)
init.invest <- 250000
{% endhighlight %}


Now that we have the relevant data and some information about the number of shares purchased and the original investment amount, we can set up a portfolio. First we need to sort out some basic mechanics. For example, we need to have the most recent closing price for each of the tickers we imported. We need this in order to mark-to-market the portfolio today:


{% highlight r %}
mtm_last <- sum(last(etfs_close) * shares)
mtm_last
{% endhighlight %}



{% highlight text %}
## [1] NA
{% endhighlight %}



{% highlight r %}
port_value <- as.xts(apply(etfs_close, 1, FUN = function(x) sum(x * shares)))
plot.xts(port_value, las = 1)  # A line chart of the portfolio value in dollars...
{% endhighlight %}

![center](http://gtog.github.io/figs/2013-07-22-implementing-swensen-in-R/mtmLast.png) 

What follows is a series of functions from the PerformanceAnalytics package that will help us look at
the performance of the portfolio. The benchmark portfolio will be a 60/40 stock and intermediate bond portfolio. I don't think using the SP500 is really a sensible benchmark because going all-in on stocks
seems, well non-sensical. As it turns out, the 60/40 (just like the 1/n) portfolio is actually tough to beat. 


{% highlight r %}
bench.60 <- periodReturn(GSPC[first.date][, 4], period = "daily", subset = NULL, 
    type = "log")
bench.40 <- periodReturn(VFITX[first.date][, 4], period = "daily", subset = NULL, 
    type = "log")

bench_ret <- 0.6 * bench.60 + 0.4 * bench.40  # Return stream for the 60/40 benchmark portfolio...
port_ret <- periodReturn(port_value, period = "daily", subset = NULL, type = "log")  # Ret. stream for our portfolio...
{% endhighlight %}



{% highlight text %}
## Warning: missing values removed from data
{% endhighlight %}



{% highlight r %}
port_index <- makeIndex(port_ret, inv = FALSE, ret = TRUE)  # Our portfolio indexed to 100 at inception...
bench_index <- makeIndex(bench_ret, inv = FALSE, ret = TRUE)  # Benchmark portfolio indexed to 100 at our inception date...

# Setting up some variables to mach the PerformanceAnalytics lexicon...
Ra <- port_ret
Rb <- bench_ret
dts <- index(Ra, 0)
Rb <- xts(Rb, dts)
Rf <- as.numeric(last(DGS2)/100/252)
{% endhighlight %}


The performance of our portfolio relative to the benchmark:

{% highlight r %}
chart.RelativePerformance(Ra, as.vector(Rb), main = "Relative Performace vs. Benchmark", 
    xaxis = TRUE)
{% endhighlight %}

![center](http://gtog.github.io/figs/2013-07-22-implementing-swensen-in-R/chartRelativePerfomance.png) 


How much our portfolio outperforms the benchmark on an anuualized basis:

{% highlight r %}
act.premium <- ActivePremium(Ra, Rb, scale = 252)
act.premium
{% endhighlight %}



{% highlight text %}
## [1] -0.04633
{% endhighlight %}


The weights of the portfolio at inception, now, and over time:

{% highlight r %}
# The initial weights of the portfolio:
weights_init <- (first(etfs_close) * shares)/init.invest
weights_init
{% endhighlight %}



{% highlight text %}
##            VGSIX.Close VUSTX.Close VGTSX.Close VFISX.Close VTSMX.Close
## 2012-10-17      0.2039     0.05152     0.05078     0.05034      0.3034
##            VFITX.Close VEIEX.Close VIPSX.Close
## 2012-10-17     0.05068      0.1515      0.1547
{% endhighlight %}



{% highlight r %}

# The weights of the portfolio now:
weights_last <- (last(etfs_close) * shares)/init.invest
weights_last
{% endhighlight %}



{% highlight text %}
##            VGSIX.Close VUSTX.Close VGTSX.Close VFISX.Close VTSMX.Close
## 2013-07-22      0.2264     0.04604     0.05387     0.04997      0.3563
##            VFITX.Close VEIEX.Close VIPSX.Close
## 2013-07-22          NA       0.142          NA
{% endhighlight %}



{% highlight r %}

# Change in weights since inception:
weights_chg <- as.vector(weights_last) - as.vector(weights_init)
weights_chg
{% endhighlight %}



{% highlight text %}
## [1]  0.0225355 -0.0054766  0.0030894 -0.0003736  0.0528389         NA
## [7] -0.0095305         NA
{% endhighlight %}



{% highlight r %}

# Display the weights of each asset over time:
port_weights <- calcWeights(etfs_close, shares, init.invest)
par(mfrow = c(2, 2))
for (i in 1:length(tickers.etf)) {
    plot.xts(port_weights[, i], las = 1, type = "l", main = paste(tickers.human[i]))
}
{% endhighlight %}

![center](http://gtog.github.io/figs/2013-07-22-implementing-swensen-in-R/initWeights1.png) ![center](http://gtog.github.io/figs/2013-07-22-implementing-swensen-in-R/initWeights2.png) 


We can also display the prices of each sector over time:

{% highlight r %}
par(mfrow = c(2, 2))
for (i in 1:length(tickers.etf)) {
    plot.xts(makeIndex(etfs_close[, i], inv = FALSE, ret = FALSE), las = 1, 
        type = "l", main = paste(tickers.human[i]))
}
{% endhighlight %}

![center](http://gtog.github.io/figs/2013-07-22-implementing-swensen-in-R/privesOverTime1.png) 

{% highlight text %}
## Warning: missing values removed from data
{% endhighlight %}



{% highlight text %}
## Warning: missing values removed from data
{% endhighlight %}

![center](http://gtog.github.io/figs/2013-07-22-implementing-swensen-in-R/privesOverTime2.png) 


Let's generate return streams for all of the ETFs in the portfolio and then perform some operations
on them:

{% highlight r %}
etfs.ret <- NULL  # A data frame that holds all of the etf return streams...
for (i in 1:length(tickers.etf)) {
    temp <- as.xts(periodReturn(etfs_close[, i], period = "daily", type = "log"))
    etfs.ret <- cbind(etfs.ret, temp)
}
{% endhighlight %}



{% highlight text %}
## Warning: missing values removed from data
{% endhighlight %}



{% highlight text %}
## Warning: missing values removed from data
{% endhighlight %}



{% highlight r %}
etfs.ret <- etfs.ret[, 2:9]
{% endhighlight %}



{% highlight text %}
## Error: subscript out of bounds
{% endhighlight %}



{% highlight r %}
names(etfs.ret) <- tickers.human
head(etfs.ret)
{% endhighlight %}



{% highlight text %}
##              Real Est USTSY Long    For Eq USTSY Short    All Eq
## 2012-10-17  0.0000000   0.000000  0.000000           0  0.000000
## 2012-10-18  0.0105385  -0.005329 -0.003423           0 -0.002203
## 2012-10-19 -0.0073193   0.011385 -0.011728           0 -0.016681
## 2012-10-22 -0.0059867  -0.005297  0.006226           0  0.000000
## 2012-10-23 -0.0092808   0.011317 -0.018796           0 -0.013264
## 2012-10-24 -0.0004663  -0.007530  0.001404           0 -0.002845
##            USTSY Int.     EM Eq  Infl Sec.
## 2012-10-17  0.0000000  0.000000  0.0000000
## 2012-10-18 -0.0008529 -0.001490  0.0006759
## 2012-10-19  0.0025565 -0.014265  0.0033727
## 2012-10-22 -0.0025565  0.010530 -0.0013477
## 2012-10-23  0.0025565 -0.018121  0.0026936
## 2012-10-24 -0.0008514  0.001523 -0.0033681
{% endhighlight %}


Here is a chart that shows the relative performance vs. the benchmark portfolio for each of the assets
in the portfolio. I don't love the color scheme, but we can fix that later.

{% highlight r %}
par(mfrow = c(1, 1))
chart.RelativePerformance(etfs.ret, as.vector(Rb), main = "Relative Performace vs. Benchmark", 
    colorset = c(1:8), xaxis = TRUE, ylog = FALSE, legend.loc = "bottomleft", 
    cex.legend = 0.8)
{% endhighlight %}



{% highlight text %}
## Error: improper length of one or more arguments to merge.xts
{% endhighlight %}


Here is a chart that shows the returns for each asset class in bar chart form with the 20 day Std Dev
and the most recent 1 day VaR (95%) observation indicated by a solid line:

{% highlight r %}
par(mfrow=c(1,1))
charts.BarVaR (etfs.ret[,1:8],
              width = 1, gap = 0,
              methods = "StdDev",  #c("none", "ModifiedVaR", "GaussianVaR", "HistoricalVaR", "StdDev", "ModifiedES", "p = 0.95"),
              p=.95,
              clean = "none",            
              all = TRUE,
              show.clean = FALSE,
              show.horizontal = TRUE,
              show.symmetric = FALSE,
              show.endvalue = TRUE,
              show.greenredbars = TRUE,
              #legend.loc = "bottomleft",
              ylim = NA,
              #lwd = 2,
              colorset = 1:12,
              lty = 1,
              ypad = 0,
              legend.cex = 0.8)
{% endhighlight %}

![center](http://gtog.github.io/figs/2013-07-22-implementing-swensen-in-R/returnAnalysis1.png) ![center](http://gtog.github.io/figs/2013-07-22-implementing-swensen-in-R/returnAnalysis2.png) 


Here is a nice table of CAPM parameters:

{% highlight r %}
table.CAPM(Ra, Rb, scale = 252, Rf = Rf, digits = 4)
{% endhighlight %}



{% highlight text %}
##                     daily.returns to daily.returns
## Alpha                                      -0.0002
## Beta                                        1.1288
## Beta+                                       1.0833
## Beta-                                       1.2131
## R-squared                                   0.8745
## Annualized Alpha                           -0.0529
## Correlation                                 0.9352
## Correlation p-value                         0.0000
## Tracking Error                              0.0000
## Active Premium                             -0.0463
## Information Ratio                             -Inf
## Treynor Ratio                               0.0475
{% endhighlight %}


And a look at some historical and parametric VaR estimates:

{% highlight r %}
chart.VaRSensitivity(Ra, methods = c("HistoricalVaR", "ModifiedVaR", "GaussianVaR"))
{% endhighlight %}

![center](http://gtog.github.io/figs/2013-07-22-implementing-swensen-in-R/varEstimates.png) 


In the next post, I'll continue the analysis with a look at the drawdowns, in particular, the most
recent drawdown which we still haven't fully recovered from. Here is a look at the largest drawdowns
since the inception date in October of 2012:


{% highlight r %}
table.Drawdowns(Ra, top = 5, digits = 4)
{% endhighlight %}



{% highlight text %}
##         From     Trough         To   Depth Length To Trough Recovery
## 1 2013-05-22 2013-06-24       <NA> -0.1007     42        23       NA
## 2 2012-10-19 2012-11-15 2012-12-10 -0.0364     34        18       16
## 3 2013-04-12 2013-04-18 2013-04-25 -0.0197     10         5        5
## 4 2012-12-19 2012-12-28 2013-01-02 -0.0171      9         7        2
## 5 2013-02-20 2013-02-25 2013-03-05 -0.0163     10         4        6
{% endhighlight %}


Keep in mind that with a 1% 1-day VaR (95%), the most recent drawdown was a 10x excession! That
ranks up there with some of the largest 1 day experiences from 2008-2009! (See my earlier pieces
on using VaR to examine historical crises) That's a bit alarming and something I'll discuss more next time.

Thanks for reading!


Disclaimer: This blog post is not intended as investment advice and was written for information
purposes only. Please take care when running your own portfolio. Nobody cares as much about 
your money as you do.
