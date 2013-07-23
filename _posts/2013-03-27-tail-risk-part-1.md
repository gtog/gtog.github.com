---
layout: post
title: "Measuring the Intensity of Historical Crises with VaR"
description: "Part 1 of a multi-part look at tail risk."
category: finance
tags: [finance, VaR, R, PerformanceAnalytics, quantmod, knitr, tail risk]
---
{% include JB/setup %}


Adam Duncan, December 2012  
Also avilable on [R-bloggers.com](http://www.r-bloggers.com "R-bloggers.com")  

## Prelude
These posts are written with dual purpose: 1) Hopefully provide some insight or inspiration into a topical issue in finance from a practioners perspective, and 2) show how to use R to craft an analysis and produce nice output. The posts are written in a "walkthrough" style. All of the source files are available at my [github repository](https://github.com/gtog/VaR_analysis). Feel free to use the files for your own projects. I only ask that you credit appropriately.  

### Introduction: How big is a financial "crisis"?

This is the first part of a series of practical examinations of tail risk. The idea of this first part is to import a bunch of different financial time series and quantify what "tail risk" actually is. This term gets used too frequently, yet few people really understand what the difference is between regular "risk" and "tail risk." Everyone is interested in tail risk hedging, but when you query people about different events and their relative magnitudes, people give wildly different answers. So, I am going to use the VaR framework as a yard stick to measure the degree to which certain historical events exceed our expectations.  

This in no way is meant to lead into or be about the pros and cons of VaR. It's just a convenient measuring tool that everyone in finance, whether you believe in it's usefulness or not, can relate to. Like it or not, it's part of the daily business of modern finance.  

So, let's get started. First, we are going to use the VaR routines from the Performance Analytics package. The documentation is copious and I'll try to make some sensible assumptions about its use as we go, but the actual VaR methodology is not the focus. Rather, we are interested to know the extent to which realized returns exceed this measure as we move through time. Specifically, what does the distribution of these "VaR Breaks" look like?

Here is how you call VaR fromt the Performance Analytics package...  

VaR(R=ret,  
  p=.95,  
  method="gaussian",  
  clean="none",  
  portfolio_method="single",  
  weights=NULL,  
  mu=ret.mu,  
  sigma=ret.sig,  
  m3=NULL,#skewness of series  
  m4=NULL,#kurtosis of series  
  invert=TRUE)  

Libararies we'll need (and all dependencies, naturally)...  
*quantmod  
*PerformanceAnalytics  
*knitr  

We need to get some data from FRED database and set some initial parameters...  

{% highlight r %}
tickers = c("SP500", "DJIA", "DEXUSEU", "DSWP5", "DCOILWTICO", "DEXMXUS")
for (i in 1:length(tickers)) {
    suppressWarnings(getSymbols(tickers[i], src = "FRED", auto.assign = TRUE))
}
{% endhighlight %}

Let's just focus on one data set for the moment. For example, let's choose USDMXN. For this post, we'll just construct the analysis for USDMXN. In the next post, we'll run through many variables and try to draw some more conclusions about the magnitude of past crises.  

{% highlight r %}
data <- DEXMXUS  # Be careful when selecting data. For example, USDMXN goes UP in a crisis...
data <- 1/data  # Since we're using USDMXN as an example will flip it to MXNUSD so it goes down in a crisis.
{% endhighlight %}

Let's also set some intitial global parameters...  

{% highlight r %}
var.yrs <- 5
biz.days <- 252
hist.len <- var.yrs * biz.days
p = 0.95
{% endhighlight %}

Next we'll need to calculate a return series for the data object...  

{% highlight r %}
ret <- periodReturn(data, period = "daily", subset = NULL, type = "log")
{% endhighlight %}


Similarly, we'll need to calculate a rolling mean, std.dev., skewness and kurtosis for our series,   
given our VaR lookback length choice: var.yrs.:  

{% highlight r %}
ret.mu <- rollapply(ret, width = hist.len, FUN = mean)
ret.sig <- rollapply(ret, width = hist.len, FUN = function(ret) apply(X = ret, 
    MARGIN = 2, FUN = sd))
ret.skew <- rollapply(ret, width = hist.len, FUN = function(ret) skewness(ret, 
    method = "moment"))
ret.kurt <- rollapply(ret, width = hist.len, FUN = function(ret) kurtosis(ret, 
    method = "moment"))
{% endhighlight %}

We can now use rollapply to calculate the daily VaR we would have seen each day in the sample data. I choose a *historical* VaR method to keep things simple and a single underlying. You can easily choose "gaussian" and pass in skewness and kurtosis, if you want to make it more consistent with how practitioners make parametric VaR estimates. Since we are using VaR as a measuring stick to measure the intesnity of crises, parametric VaR isn't completely necessary. Having said that, it will affect the ratio of actual realized returns to *a priori* VaR estimates. With adjustments for skewness and kurtosis, ratios will be smaller, all else equal. That is, crises events will seem *less severe* when the distribution is adjusted for skewness and excess kurtosis.  

Ok, let's examine the results of all that to make sure things are looking like they should. I always like to pause to examine some graphics along the way to make sure things are behaving as expected. It's better to find our early that we have a problem. Some plots:  


{% highlight r %}
par(mfrow = c(2, 2))
plot.xts(ret.mu, las = 1, main = "Rolling Mean Returns", cex.main = 0.9)
plot.xts(ret.sig * sqrt(biz.days) * 100, las = 1, main = "Rolling Vol", cex.main = 0.9)
plot.xts(ret.skew, las = 1, main = "Rolling Skewness", cex.main = 0.9)
plot.xts(ret.kurt, las = 1, main = "Rolling Kurtosis", cex.main = 0.9)
{% endhighlight %}

![center](http://gtog.github.io/figs/var_part1_markdown.RMD/plotMoments.png) 


Great. All seems to be in order. One more to go...the rolling VaR estimates.  

{% highlight r %}
ret.var <- rollapply(ret, width = hist.len, FUN = function(ret) VaR(R = ret, 
    p = p, method = "historical", clean = "none", portfolio_method = "single", 
    weights = NULL, mu = ret.mu, sigma = ret.sig, m3 = ret.skew, m4 = ret.kurt, 
    invert = FALSE), align = "right")
{% endhighlight %}

We better have a look at that handiwork before we move on...  

{% highlight r %}
par(mfrow = c(1, 1))
plot.xts(ret.var, las = 1, main = "Rolling VaR Estimate", cex.main = 0.9)
{% endhighlight %}

![center](http://gtog.github.io/figs/var_part1_markdown.RMD/plotVaR.png) 

Now that we have the rolling VaR series, we can calculate the ratios of *actual returns* to VaR estimates. This will give us the magnitude of outcomes in multiples of daily value at risk. We need to trim off the first part of the series that was used to estimate the mean/sd etc. this makes the return subset the same length as ret.var.  

{% highlight r %}
first.date <- index(first(ret.var))
ret.sub <- window(ret, from = first.date)
{% endhighlight %}

Now, divide the actual returns by the prior day's VaR etimate. This ratio represents the multiple that the actual negative return was of the VaR estimate. A ratio of -4x means that the realized negative return was 4x larger in magnitude than than the VaR estimate.  

{% highlight r %}
act.var.ratio <- ret.sub/lag(ret.var, k = 1)
var.breaks <- subset(act.var.ratio, subset = (act.var.ratio$daily.returns < 
    0))
summary(var.breaks)
{% endhighlight %}

{% highlight text %}
      Index            daily.returns    
  Min.   :1998-11-24   Min.   :-11.094  
  1st Qu.:2002-06-14   1st Qu.: -0.625  
  Median :2005-12-23   Median : -0.339  
  Mean   :2006-01-16   Mean   : -0.486  
  3rd Qu.:2009-07-29   3rd Qu.: -0.156  
  Max.   :2013-05-01   Max.   : -0.001
{% endhighlight %}

{% highlight r %}
plot(var.breaks, las = 1, main = "Actual Negative Return(t) / Daily VaR Estimate(t-1)", 
    cex.main = 0.9)  # line plot of the breaks.
{% endhighlight %}

![center](http://gtog.github.io/figs/var_part1_markdown.RMD/varBreaks.png) 

For USDMXN, we can see that 2008 was 11x daily value at risk. Other, smaller crisis episodes occur at around 4-5x daily VaR. Because we are using 5 years of history for the VaR calculation, we miss the de-pegging of USDMXN that happened in 1994. That was on the order of 25x VaR.  

Here is a histogram of the VaR breaks for USDMXN:  

{% highlight r %}
hist(var.breaks, main = "Histogram of VaR Breaks (Multiples of Daily VaR", cex.main = 0.9)  # histogram of the VaR breaks.
{% endhighlight %}

![center](http://gtog.github.io/figs/var_part1_markdown.RMD/histogramUSDMXN.png) 


Now we can just loop through each of the data series we imported and run the same analysis for each one. Then we'll examine the individual results as well as aggregate them so we can draw some conclusions about how big, in VaR terms, past crises have been.  

### A Couple Workhorse Functions

First, here is the workhorse function for the analysis. This is the function I use most when I want to examine VaR breaks for a given time series. This function takes raw time series data in xts format (x), and returns one of a number of items depending on the value of 'rtype' passed to the function:  

*'rtype' can take on the following valueS: "vars","returns","ratios","vol","skew","kurt","mean"  
*'window' is the length of the lookback period in days.  
*'p' is the quantile desired (e.g. .95, .99, etc.)  
*'inv.var' sets whether you want positive or negative VaRs returned from VaR. See Performance Analytics::VaR  
Based on user input for rtype, the function takes the appropriate action. I'm only going to support the "historical" VaR method for now.  

{% highlight r %}
varFun<-function(x,p,window,method,inv.var,rtype){
  
  ret<-periodReturn(x,period="daily",subset=NULL,type="log")
  
  if (rtype=="vars"){
    ret.var<-rollapply(ret,width=window,
                       FUN=function(ret) VaR(R=ret,
                                             p=p,
                                             method="historical",
                                             clean="none",
                                             portfolio_method="single",
                                             weights=NULL,
                                             #mu=ret.mu,
                                             #sigma=ret.sig,
                                             #m3=ret.skew,#skewness of series
                                             #m4=ret.kurt,#kurtosis of series
                                             invert=inv.var),
                       align="right")
    ret.var<-as.xts(ret.var)
    names(ret.var)<-c("VaR")
    return(ret.var)
  } else {
    if (rtype=="returns"){
      ret<-as.xts(ret)
      names(ret)<-c("returns")
      return(ret)
    } else {
      if (rtype=="ratios"){
        ret.var<-rollapply(ret,width=window,
                           FUN=function(ret) VaR(R=ret,
                                                 p=p,
                                                 method="historical",
                                                 clean="none",
                                                 portfolio_method="single",
                                                 weights=NULL,
                                                 #mu=ret.mu,
                                                 #sigma=ret.sig,
                                                 #m3=ret.skew,#skewness of series
                                                 #m4=ret.kurt,#kurtosis of series
                                                 invert=inv.var),
                           align="right")
        first.date<-index(first(ret.var))
        ret.sub<-window(ret,from=first.date)
        act.var.ratio<-as.xts(ret.sub/ret.var)
        names(act.var.ratio)<-c("ratios")
        return(act.var.ratio)
      } else {
        if (rtype=="vol"){
          ret.sig<-rollapply(ret,width=window,FUN=function(ret) apply(X=ret,MARGIN=2, FUN=sd))
          names(ret.sig)<-c("vols")
          return(ret.sig)
        } else {
          if (rtype=="skew"){
            ret.skew<-as.xts(rollapply(ret,width=window,FUN=function(ret) skewness(ret,method="moment")))
            names(ret.skew)<-c("skew")
            return(ret.skew)
          } else {
            if (rtype=="kurt"){
              ret.kurt<-as.xts(rollapply(ret,width=window,FUN=function(ret) kurtosis(ret,method="moment")))
              names(ret.kurt)<-c("kurtosis")
              return(ret.kurt)
            } else {
              if (rtype=="mean"){
                ret.mu<-as.xts(rollapply(ret,width=window,FUN=mean))
                names(ret.mu)<-c("means")
                return(as.xts(ret.mu))
              } else {
                return(Null)
              }
            }
          }
        }
      }
    }
  }
}
{% endhighlight %}


We can now assign some variable, v, to hold the VaR breaks for the variable "data":  

{% highlight r %}
v <- varFun(data, 0.95, 1260, "historical", FALSE, "ratios")
{% endhighlight %}

And then plot them...  

{% highlight r %}
plot.xts(subset(v, subset = (v < 0)), las = 1, main = "Rolling Historical VaR Breaks", 
    ylab = "Raito of Actual (Neg) Returns to VaR")
{% endhighlight %}

![center](http://gtog.github.io/figs/var_part1_markdown.RMD/plotVarfun.png) 


If you just have a bunch of tickers and want to see a summary of the VaR breaks, this will do it. This function loops through each of the data elements in 'tickers' and calculates some summary information. It returns a data frame of the results.  

Note: This is the **VERY slow way** to do this. It's better to import the data first, and then past a list of raw data variables to this function. Getting the data on the fly is slow and then you don't have use of the variables on exit. I'll update this summary function in the next post.  

{% highlight r %}
vsum<-function(tickers,p,window,method,inv.var){
  n<-length(tickers)
  out<-data.frame()
  
  for (i in 1:n){
    data<-getSymbols(tickers[i],src="FRED",auto.assign=FALSE)
    ret<-periodReturn(data,period="daily",subset=NULL,type="log")
    ret.mu<-rollapply(ret,width=hist.len,FUN=mean)
    ret.sig<-rollapply(ret,width=hist.len,FUN=function(ret) apply(X=ret,MARGIN=2, FUN=sd))
    ret.skew<-rollapply(ret,width=hist.len,FUN=function(ret) skewness(ret,method="moment"))
    ret.kurt<-rollapply(ret,width=hist.len,FUN=function(ret) kurtosis(ret,method="moment"))
    
    ret.var<-rollapply(ret,width=window,
                       FUN=function(ret) VaR(R=ret,
                                             p=p,
                                             method=method,
                                             clean="none",
                                             portfolio_method="single",
                                             weights=NULL,
                                             mu=ret.mu,
                                             sigma=ret.sig,
                                             m3=ret.skew,#skewness of series
                                             m4=ret.kurt,#kurtosis of series
                                             invert=inv.var),
                       align="right"
    )
    first.date<-index(first(ret.var))
    ret.sub<-window(ret,from=first.date)
    act.var.ratio<-ret.sub/ret.var
    
    out[i,1]<-tickers[i]
    out[i,2]<-max(act.var.ratio)
    out[i,3]<-mean(act.var.ratio)
    out[i,4]<-quantile(act.var.ratio,p,na.rm=TRUE)
  }
  names(out)<-c("name","max.ratio","mean.ratio","Q.p")
  return(out)
}
{% endhighlight %}

### Next Up: Running this analysis across many assets  



