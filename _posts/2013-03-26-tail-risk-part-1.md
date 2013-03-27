---
layout: post
title: "Tail Risk Part 1"
description: "Looking at historical events through a VaR lense."
category: Finance
tags: [VaR, tail risk, R]
---
{% include JB/setup %}

<!DOCTYPE html>
<!-- saved from url=(0014)about:internet -->
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>

<title>Analyzing Crisis Events Using Value-at-Risk (part 1)</title>

<style type="text/css">
body, td {
   font-family: sans-serif;
   background-color: white;
   font-size: 12px;
   margin: 8px;
}

tt, code, pre {
   font-family: 'DejaVu Sans Mono', 'Droid Sans Mono', 'Lucida Console', Consolas, Monaco, monospace;
}

h1 { 
   font-size:2.2em; 
}

h2 { 
   font-size:1.8em; 
}

h3 { 
   font-size:1.4em; 
}

h4 { 
   font-size:1.0em; 
}

h5 { 
   font-size:0.9em; 
}

h6 { 
   font-size:0.8em; 
}

a:visited {
   color: rgb(50%, 0%, 50%);
}

pre {	
   margin-top: 0;
   max-width: 95%;
   border: 1px solid #ccc;
   white-space: pre-wrap;
}

pre code {
   display: block; padding: 0.5em;
}

code.r, code.cpp {
   background-color: #F8F8F8;
}

table, td, th {
  border: none;
}

blockquote {
   color:#666666;
   margin:0;
   padding-left: 1em;
   border-left: 0.5em #EEE solid;
}

hr {
   height: 0px;
   border-bottom: none;
   border-top-width: thin;
   border-top-style: dotted;
   border-top-color: #999999;
}

@media print {
   * { 
      background: transparent !important; 
      color: black !important; 
      filter:none !important; 
      -ms-filter: none !important; 
   }

   body { 
      font-size:12pt; 
      max-width:100%; 
   }
       
   a, a:visited { 
      text-decoration: underline; 
   }

   hr { 
      visibility: hidden;
      page-break-before: always;
   }

   pre, blockquote { 
      padding-right: 1em; 
      page-break-inside: avoid; 
   }

   tr, img { 
      page-break-inside: avoid; 
   }

   img { 
      max-width: 100% !important; 
   }

   @page :left { 
      margin: 15mm 20mm 15mm 10mm; 
   }
     
   @page :right { 
      margin: 15mm 10mm 15mm 20mm; 
   }

   p, h2, h3 { 
      orphans: 3; widows: 3; 
   }

   h2, h3 { 
      page-break-after: avoid; 
   }
}

</style>

<!-- Styles for R syntax highlighter -->
<style type="text/css">
   pre .operator,
   pre .paren {
     color: rgb(104, 118, 135)
   }

   pre .literal {
     color: rgb(88, 72, 246)
   }

   pre .number {
     color: rgb(0, 0, 205);
   }

   pre .comment {
     color: rgb(76, 136, 107);
   }

   pre .keyword {
     color: rgb(0, 0, 255);
   }

   pre .identifier {
     color: rgb(0, 0, 0);
   }

   pre .string {
     color: rgb(3, 106, 7);
   }
</style>

<!-- R syntax highlighter -->
<script type="text/javascript">
var hljs=new function(){function m(p){return p.replace(/&/gm,"&amp;").replace(/</gm,"&lt;")}function f(r,q,p){return RegExp(q,"m"+(r.cI?"i":"")+(p?"g":""))}function b(r){for(var p=0;p<r.childNodes.length;p++){var q=r.childNodes[p];if(q.nodeName=="CODE"){return q}if(!(q.nodeType==3&&q.nodeValue.match(/\s+/))){break}}}function h(t,s){var p="";for(var r=0;r<t.childNodes.length;r++){if(t.childNodes[r].nodeType==3){var q=t.childNodes[r].nodeValue;if(s){q=q.replace(/\n/g,"")}p+=q}else{if(t.childNodes[r].nodeName=="BR"){p+="\n"}else{p+=h(t.childNodes[r])}}}if(/MSIE [678]/.test(navigator.userAgent)){p=p.replace(/\r/g,"\n")}return p}function a(s){var r=s.className.split(/\s+/);r=r.concat(s.parentNode.className.split(/\s+/));for(var q=0;q<r.length;q++){var p=r[q].replace(/^language-/,"");if(e[p]){return p}}}function c(q){var p=[];(function(s,t){for(var r=0;r<s.childNodes.length;r++){if(s.childNodes[r].nodeType==3){t+=s.childNodes[r].nodeValue.length}else{if(s.childNodes[r].nodeName=="BR"){t+=1}else{if(s.childNodes[r].nodeType==1){p.push({event:"start",offset:t,node:s.childNodes[r]});t=arguments.callee(s.childNodes[r],t);p.push({event:"stop",offset:t,node:s.childNodes[r]})}}}}return t})(q,0);return p}function k(y,w,x){var q=0;var z="";var s=[];function u(){if(y.length&&w.length){if(y[0].offset!=w[0].offset){return(y[0].offset<w[0].offset)?y:w}else{return w[0].event=="start"?y:w}}else{return y.length?y:w}}function t(D){var A="<"+D.nodeName.toLowerCase();for(var B=0;B<D.attributes.length;B++){var C=D.attributes[B];A+=" "+C.nodeName.toLowerCase();if(C.value!==undefined&&C.value!==false&&C.value!==null){A+='="'+m(C.value)+'"'}}return A+">"}while(y.length||w.length){var v=u().splice(0,1)[0];z+=m(x.substr(q,v.offset-q));q=v.offset;if(v.event=="start"){z+=t(v.node);s.push(v.node)}else{if(v.event=="stop"){var p,r=s.length;do{r--;p=s[r];z+=("</"+p.nodeName.toLowerCase()+">")}while(p!=v.node);s.splice(r,1);while(r<s.length){z+=t(s[r]);r++}}}}return z+m(x.substr(q))}function j(){function q(x,y,v){if(x.compiled){return}var u;var s=[];if(x.k){x.lR=f(y,x.l||hljs.IR,true);for(var w in x.k){if(!x.k.hasOwnProperty(w)){continue}if(x.k[w] instanceof Object){u=x.k[w]}else{u=x.k;w="keyword"}for(var r in u){if(!u.hasOwnProperty(r)){continue}x.k[r]=[w,u[r]];s.push(r)}}}if(!v){if(x.bWK){x.b="\\b("+s.join("|")+")\\s"}x.bR=f(y,x.b?x.b:"\\B|\\b");if(!x.e&&!x.eW){x.e="\\B|\\b"}if(x.e){x.eR=f(y,x.e)}}if(x.i){x.iR=f(y,x.i)}if(x.r===undefined){x.r=1}if(!x.c){x.c=[]}x.compiled=true;for(var t=0;t<x.c.length;t++){if(x.c[t]=="self"){x.c[t]=x}q(x.c[t],y,false)}if(x.starts){q(x.starts,y,false)}}for(var p in e){if(!e.hasOwnProperty(p)){continue}q(e[p].dM,e[p],true)}}function d(B,C){if(!j.called){j();j.called=true}function q(r,M){for(var L=0;L<M.c.length;L++){if((M.c[L].bR.exec(r)||[null])[0]==r){return M.c[L]}}}function v(L,r){if(D[L].e&&D[L].eR.test(r)){return 1}if(D[L].eW){var M=v(L-1,r);return M?M+1:0}return 0}function w(r,L){return L.i&&L.iR.test(r)}function K(N,O){var M=[];for(var L=0;L<N.c.length;L++){M.push(N.c[L].b)}var r=D.length-1;do{if(D[r].e){M.push(D[r].e)}r--}while(D[r+1].eW);if(N.i){M.push(N.i)}return f(O,M.join("|"),true)}function p(M,L){var N=D[D.length-1];if(!N.t){N.t=K(N,E)}N.t.lastIndex=L;var r=N.t.exec(M);return r?[M.substr(L,r.index-L),r[0],false]:[M.substr(L),"",true]}function z(N,r){var L=E.cI?r[0].toLowerCase():r[0];var M=N.k[L];if(M&&M instanceof Array){return M}return false}function F(L,P){L=m(L);if(!P.k){return L}var r="";var O=0;P.lR.lastIndex=0;var M=P.lR.exec(L);while(M){r+=L.substr(O,M.index-O);var N=z(P,M);if(N){x+=N[1];r+='<span class="'+N[0]+'">'+M[0]+"</span>"}else{r+=M[0]}O=P.lR.lastIndex;M=P.lR.exec(L)}return r+L.substr(O,L.length-O)}function J(L,M){if(M.sL&&e[M.sL]){var r=d(M.sL,L);x+=r.keyword_count;return r.value}else{return F(L,M)}}function I(M,r){var L=M.cN?'<span class="'+M.cN+'">':"";if(M.rB){y+=L;M.buffer=""}else{if(M.eB){y+=m(r)+L;M.buffer=""}else{y+=L;M.buffer=r}}D.push(M);A+=M.r}function G(N,M,Q){var R=D[D.length-1];if(Q){y+=J(R.buffer+N,R);return false}var P=q(M,R);if(P){y+=J(R.buffer+N,R);I(P,M);return P.rB}var L=v(D.length-1,M);if(L){var O=R.cN?"</span>":"";if(R.rE){y+=J(R.buffer+N,R)+O}else{if(R.eE){y+=J(R.buffer+N,R)+O+m(M)}else{y+=J(R.buffer+N+M,R)+O}}while(L>1){O=D[D.length-2].cN?"</span>":"";y+=O;L--;D.length--}var r=D[D.length-1];D.length--;D[D.length-1].buffer="";if(r.starts){I(r.starts,"")}return R.rE}if(w(M,R)){throw"Illegal"}}var E=e[B];var D=[E.dM];var A=0;var x=0;var y="";try{var s,u=0;E.dM.buffer="";do{s=p(C,u);var t=G(s[0],s[1],s[2]);u+=s[0].length;if(!t){u+=s[1].length}}while(!s[2]);if(D.length>1){throw"Illegal"}return{r:A,keyword_count:x,value:y}}catch(H){if(H=="Illegal"){return{r:0,keyword_count:0,value:m(C)}}else{throw H}}}function g(t){var p={keyword_count:0,r:0,value:m(t)};var r=p;for(var q in e){if(!e.hasOwnProperty(q)){continue}var s=d(q,t);s.language=q;if(s.keyword_count+s.r>r.keyword_count+r.r){r=s}if(s.keyword_count+s.r>p.keyword_count+p.r){r=p;p=s}}if(r.language){p.second_best=r}return p}function i(r,q,p){if(q){r=r.replace(/^((<[^>]+>|\t)+)/gm,function(t,w,v,u){return w.replace(/\t/g,q)})}if(p){r=r.replace(/\n/g,"<br>")}return r}function n(t,w,r){var x=h(t,r);var v=a(t);var y,s;if(v){y=d(v,x)}else{return}var q=c(t);if(q.length){s=document.createElement("pre");s.innerHTML=y.value;y.value=k(q,c(s),x)}y.value=i(y.value,w,r);var u=t.className;if(!u.match("(\\s|^)(language-)?"+v+"(\\s|$)")){u=u?(u+" "+v):v}if(/MSIE [678]/.test(navigator.userAgent)&&t.tagName=="CODE"&&t.parentNode.tagName=="PRE"){s=t.parentNode;var p=document.createElement("div");p.innerHTML="<pre><code>"+y.value+"</code></pre>";t=p.firstChild.firstChild;p.firstChild.cN=s.cN;s.parentNode.replaceChild(p.firstChild,s)}else{t.innerHTML=y.value}t.className=u;t.result={language:v,kw:y.keyword_count,re:y.r};if(y.second_best){t.second_best={language:y.second_best.language,kw:y.second_best.keyword_count,re:y.second_best.r}}}function o(){if(o.called){return}o.called=true;var r=document.getElementsByTagName("pre");for(var p=0;p<r.length;p++){var q=b(r[p]);if(q){n(q,hljs.tabReplace)}}}function l(){if(window.addEventListener){window.addEventListener("DOMContentLoaded",o,false);window.addEventListener("load",o,false)}else{if(window.attachEvent){window.attachEvent("onload",o)}else{window.onload=o}}}var e={};this.LANGUAGES=e;this.highlight=d;this.highlightAuto=g;this.fixMarkup=i;this.highlightBlock=n;this.initHighlighting=o;this.initHighlightingOnLoad=l;this.IR="[a-zA-Z][a-zA-Z0-9_]*";this.UIR="[a-zA-Z_][a-zA-Z0-9_]*";this.NR="\\b\\d+(\\.\\d+)?";this.CNR="\\b(0[xX][a-fA-F0-9]+|(\\d+(\\.\\d*)?|\\.\\d+)([eE][-+]?\\d+)?)";this.BNR="\\b(0b[01]+)";this.RSR="!|!=|!==|%|%=|&|&&|&=|\\*|\\*=|\\+|\\+=|,|\\.|-|-=|/|/=|:|;|<|<<|<<=|<=|=|==|===|>|>=|>>|>>=|>>>|>>>=|\\?|\\[|\\{|\\(|\\^|\\^=|\\||\\|=|\\|\\||~";this.ER="(?![\\s\\S])";this.BE={b:"\\\\.",r:0};this.ASM={cN:"string",b:"'",e:"'",i:"\\n",c:[this.BE],r:0};this.QSM={cN:"string",b:'"',e:'"',i:"\\n",c:[this.BE],r:0};this.CLCM={cN:"comment",b:"//",e:"$"};this.CBLCLM={cN:"comment",b:"/\\*",e:"\\*/"};this.HCM={cN:"comment",b:"#",e:"$"};this.NM={cN:"number",b:this.NR,r:0};this.CNM={cN:"number",b:this.CNR,r:0};this.BNM={cN:"number",b:this.BNR,r:0};this.inherit=function(r,s){var p={};for(var q in r){p[q]=r[q]}if(s){for(var q in s){p[q]=s[q]}}return p}}();hljs.LANGUAGES.cpp=function(){var a={keyword:{"false":1,"int":1,"float":1,"while":1,"private":1,"char":1,"catch":1,"export":1,virtual:1,operator:2,sizeof:2,dynamic_cast:2,typedef:2,const_cast:2,"const":1,struct:1,"for":1,static_cast:2,union:1,namespace:1,unsigned:1,"long":1,"throw":1,"volatile":2,"static":1,"protected":1,bool:1,template:1,mutable:1,"if":1,"public":1,friend:2,"do":1,"return":1,"goto":1,auto:1,"void":2,"enum":1,"else":1,"break":1,"new":1,extern:1,using:1,"true":1,"class":1,asm:1,"case":1,typeid:1,"short":1,reinterpret_cast:2,"default":1,"double":1,register:1,explicit:1,signed:1,typename:1,"try":1,"this":1,"switch":1,"continue":1,wchar_t:1,inline:1,"delete":1,alignof:1,char16_t:1,char32_t:1,constexpr:1,decltype:1,noexcept:1,nullptr:1,static_assert:1,thread_local:1,restrict:1,_Bool:1,complex:1},built_in:{std:1,string:1,cin:1,cout:1,cerr:1,clog:1,stringstream:1,istringstream:1,ostringstream:1,auto_ptr:1,deque:1,list:1,queue:1,stack:1,vector:1,map:1,set:1,bitset:1,multiset:1,multimap:1,unordered_set:1,unordered_map:1,unordered_multiset:1,unordered_multimap:1,array:1,shared_ptr:1}};return{dM:{k:a,i:"</",c:[hljs.CLCM,hljs.CBLCLM,hljs.QSM,{cN:"string",b:"'\\\\?.",e:"'",i:"."},{cN:"number",b:"\\b(\\d+(\\.\\d*)?|\\.\\d+)(u|U|l|L|ul|UL|f|F)"},hljs.CNM,{cN:"preprocessor",b:"#",e:"$"},{cN:"stl_container",b:"\\b(deque|list|queue|stack|vector|map|set|bitset|multiset|multimap|unordered_map|unordered_set|unordered_multiset|unordered_multimap|array)\\s*<",e:">",k:a,r:10,c:["self"]}]}}}();hljs.LANGUAGES.r={dM:{c:[hljs.HCM,{cN:"number",b:"\\b0[xX][0-9a-fA-F]+[Li]?\\b",e:hljs.IMMEDIATE_RE,r:0},{cN:"number",b:"\\b\\d+(?:[eE][+\\-]?\\d*)?L\\b",e:hljs.IMMEDIATE_RE,r:0},{cN:"number",b:"\\b\\d+\\.(?!\\d)(?:i\\b)?",e:hljs.IMMEDIATE_RE,r:1},{cN:"number",b:"\\b\\d+(?:\\.\\d*)?(?:[eE][+\\-]?\\d*)?i?\\b",e:hljs.IMMEDIATE_RE,r:0},{cN:"number",b:"\\.\\d+(?:[eE][+\\-]?\\d*)?i?\\b",e:hljs.IMMEDIATE_RE,r:1},{cN:"keyword",b:"(?:tryCatch|library|setGeneric|setGroupGeneric)\\b",e:hljs.IMMEDIATE_RE,r:10},{cN:"keyword",b:"\\.\\.\\.",e:hljs.IMMEDIATE_RE,r:10},{cN:"keyword",b:"\\.\\.\\d+(?![\\w.])",e:hljs.IMMEDIATE_RE,r:10},{cN:"keyword",b:"\\b(?:function)",e:hljs.IMMEDIATE_RE,r:2},{cN:"keyword",b:"(?:if|in|break|next|repeat|else|for|return|switch|while|try|stop|warning|require|attach|detach|source|setMethod|setClass)\\b",e:hljs.IMMEDIATE_RE,r:1},{cN:"literal",b:"(?:NA|NA_integer_|NA_real_|NA_character_|NA_complex_)\\b",e:hljs.IMMEDIATE_RE,r:10},{cN:"literal",b:"(?:NULL|TRUE|FALSE|T|F|Inf|NaN)\\b",e:hljs.IMMEDIATE_RE,r:1},{cN:"identifier",b:"[a-zA-Z.][a-zA-Z0-9._]*\\b",e:hljs.IMMEDIATE_RE,r:0},{cN:"operator",b:"<\\-(?!\\s*\\d)",e:hljs.IMMEDIATE_RE,r:2},{cN:"operator",b:"\\->|<\\-",e:hljs.IMMEDIATE_RE,r:1},{cN:"operator",b:"%%|~",e:hljs.IMMEDIATE_RE},{cN:"operator",b:">=|<=|==|!=|\\|\\||&&|=|\\+|\\-|\\*|/|\\^|>|<|!|&|\\||\\$|:",e:hljs.IMMEDIATE_RE,r:0},{cN:"operator",b:"%",e:"%",i:"\\n",r:1},{cN:"identifier",b:"`",e:"`",r:0},{cN:"string",b:'"',e:'"',c:[hljs.BE],r:0},{cN:"string",b:"'",e:"'",c:[hljs.BE],r:0},{cN:"paren",b:"[[({\\])}]",e:hljs.IMMEDIATE_RE,r:0}]}};
hljs.initHighlightingOnLoad();
</script>




</head>

<body>
<!-- Automatically generated by RStudio [12861c30b10411e1afa60800200c9a66] -->

<h3>Analyzing Crisis Events Using Value-at-Risk (part 1)</h3>

<p>Adam Duncan &mdash; <em>Dec 14, 2012, 3:46 PM</em></p>

<pre><code class="r"># Adam Duncan, December 2012


# This is the first part of a tail risk hedging piece I am working on. The idea
# of this first part is to import a bunch of different financial time
# series and quantify what &quot;tail risk&quot; actually is. This term gets used too frequently,
# yet few people really understand what the difference is between regular &quot;risk&quot; and 
# &quot;tail risk.&quot; Everyone is interested in tail risk hedging, but when you query people
# about different events and their relative magnitudes, people give wildly different
# answers. So, I am going to use the VaR framework as a yard stick to measure the 
# degree to which certain historical events exceed our expectations. 

# This is no way is meant to lead into or be about the pros and cons of VaR. It&#39;s
# just a convenient measuring tool that everyone in finance, whether you believe
# in it&#39;s usefulness or not, can relate to. Like it or not, it&#39;s part of the daily
# business of modern finance.

# So, let&#39;s get started. First, we are going to use the VaR routines from 
# Performance Analytics. The documentation is copious and I&#39;ll try to make some
# sensible assumptions about its use as we go, but the actual VaR methodology 
# is not the focus. Rather, we are interested to know the extent to which realized
# returns exceed this measure as we move through time. Specifically, what does
# the distribution of these &quot;VaR Breaks&quot; look like?

# Here is how you call VaR fromt the Performance Analytics package...
# VaR(R=ret,
    #p=.95,
    #method=&quot;gaussian&quot;,
    #clean=&quot;none&quot;,
    #portfolio_method=&quot;single&quot;,
    #weights=NULL,
    #mu=ret.mu,
    #sigma=ret.sig,
    #m3=NULL,#skewness of series
    #m4=NULL,#kurtosis of series
    #invert=TRUE)

# Libararies
library(quantmod,warn.conflicts=FALSE)
</code></pre>

<pre><code>Loading required package: Defaults
</code></pre>

<pre><code>Loading required package: xts
</code></pre>

<pre><code>Loading required package: zoo
</code></pre>

<pre><code>Attaching package: &#39;zoo&#39;
</code></pre>

<pre><code>The following object(s) are masked from &#39;package:base&#39;:

as.Date, as.Date.numeric
</code></pre>

<pre><code>Loading required package: TTR
</code></pre>

<pre><code class="r">library(PerformanceAnalytics,warn.conflicts=FALSE)
library(xts,warn.conflicts=FALSE)
library(knitr,warn.conflicts=FALSE)

# We need to get some data from FRED database and set some initial parameters...
tickers=c(&quot;SP500&quot;,&quot;DJIA&quot;,&quot;DEXUSEU&quot;,&quot;DSWP5&quot;,&quot;DCOILWTICO&quot;,&quot;DEXMXUS&quot;)
for (i in 1:length(tickers)){
  getSymbols(tickers[i],src=&quot;FRED&quot;,auto.assign=TRUE)
}

# Let&#39;s just focus on one data set for the moment...
data&lt;-DEXMXUS # Be careful when selecting data. For example, USDMXN goes UP in a crisis...
data&lt;-1/data # Since we&#39;re using USDMXN as an example will flip it to MXNUSD so it goes down in a crisis. 

# Set some intitial parameters...
var.yrs&lt;-5
biz.days&lt;-252
hist.len&lt;-var.yrs*biz.days
p=.95

# Calculate a return series for the data object...
ret&lt;-periodReturn(data,period=&quot;daily&quot;,subset=NULL,type=&quot;log&quot;)

# Calculate a rolling mean,std. dev., skewness and kurtosis for our series, 
# given our VaR lookback length choice: var.yrs...
ret.mu&lt;-rollapply(ret,width=hist.len,FUN=mean)
ret.sig&lt;-rollapply(ret,width=hist.len,FUN=function(ret) apply(X=ret,MARGIN=2, FUN=sd))
ret.skew&lt;-rollapply(ret,width=hist.len,FUN=function(ret) skewness(ret,method=&quot;moment&quot;))
ret.kurt&lt;-rollapply(ret,width=hist.len,FUN=function(ret) kurtosis(ret,method=&quot;moment&quot;))

# We can now use rollapply to calculate the daily VaR we would have seen each day
# as we went through history. I choose a historical VaR method to keep things simple and a single underlying.
# You can easily choose &quot;gaussian&quot; and pass skewness and kurtosis, if you want to make it more consistent
# with how practitioners use parametric VaR estimates. Since we are using VaR as a measuring stick to 
# measure the intesnity of crises, parametric isn&#39;t completely necessary. Having said that, it will affect
# the ratio of actual realized returns to a priori VaR estimates. With adjustements for skewness and kurtosis,
# ratios will be smaller, all else equal. That is, crises events will seem less severe when the distribution
# is adjusted for skewness and excess kurtosis.

# Let&#39;s examine the results of all that to make sure we&#39;re on track...
par(mfrow=c(2,2))
plot.xts(ret.mu,las=1,main=&quot;Rolling Mean Returns&quot;,cex.main=.9)
plot.xts(ret.sig*sqrt(biz.days)*100,las=1,main=&quot;Rolling Vol&quot;,cex.main=.9)
plot.xts(ret.skew,las=1,main=&quot;Rolling Skewness&quot;,cex.main=.9)
plot.xts(ret.kurt,las=1,main=&quot;Rolling Kurtosis&quot;,cex.main=.9)
</code></pre>

<p><img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAfgAAAH4CAMAAACR9g9NAAAAw1BMVEX9/v0AAAAAADkAAGUAOTkAOWUAOY8AZmUAZrU5AAA5ADk5AGU5OQA5OTk5OWU5OY85Zo85ZrU5j7U5j9plAABlADllAGVlOQBlOTllZgBlZjllZmVlZo9lj9pltdpltf2POQCPOTmPOWWPZjmPZmWPj7WPtY+P29qP2/21ZgC1Zjm1ZmW1jzm1j2W1tWW1/rW1/tq1/v27u7u+vr7T09PajznatWXa24/a2/3a/rXa/tra/v39tWX924/9/rX9/tr9/v3DjPvLAAAAQXRSTlP/////////////////////////////////////////////////////////////////////////////////////ADBV7McAAAAJcEhZcwAACxIAAAsSAdLdfvwAABySSURBVHic7Z2Pg5y4dcefvFdnbN/VdXHca9x29q69dCdJk9tbc0njmV3+/78qSOKHAGmQGMEXxPucl5uRQLzHF54EgyQqmF1CaAMYDIQ2gMFAaAMYDIQ2gMFAaAMYDIQ2gMFAaAMYDIQ2gMFAaAMYDIQ2gMFAaAMYDIQ2gMFAaAMYDIQ2gMFAaAMYDIQ2gMFAaAMYDIQ2gMFAaAMYDIQ2gMFAaAMYDDRTuZc3QojXX6svbx/0v95KucjU38jmf3ycyciF8TkmL/fl4cirlYZHLB40U7mXd4/F88dj9cWmekn+zYfiD99ahTc2P9+lIrzXMXn9VYlfrTObMTRTubWTJyEO7dl9fvWtkI6Jb95I//N/+o+//8+/ZeX311/Ly+HuUee3m8ucv9+Luz/fPZbfLm+/vfuzWuP5o9jg2eB1TMqVpN71OrMZQzOVq8JaJq/W54+Z4eRD6XmZeNHCv/7v3//LfZYfilMmo9xR57ebq5xy9bMWvtyoLuHnv81k+Ix4HRN5FF5/bdaZzRiaqdzyxM3FUSpXnA6tk0rAMlHLm7/+0+d/vc/Ks1scylNeHHV+u7nKaYUvrxj9qbzixXEmy+fD65gUeXbK2nVmM4ZmKrd08uW+UizrOWlc8b+K/1JXfOlmJs91Q3i5ucqRwe+d3KIVXq0/k+Xz4XVMiss/f/9QbPqKL0/hrF+fKSfLi/s31RX/17f/e5+V1++rh7N49W1mCi83VznPH+9+VjVg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" alt="plot of chunk unnamed-chunk-1"/> </p>

<pre><code class="r">
# Great. All seems to be in order. One more to go...the rolling VaR estimates.

ret.var&lt;-rollapply(ret,width=hist.len,
                   FUN=function(ret) VaR(R=ret,
                                         p=p,
                                         method=&quot;historical&quot;,
                                         clean=&quot;none&quot;,
                                         portfolio_method=&quot;single&quot;,
                                         weights=NULL,
                                         mu=ret.mu, 
                                         sigma=ret.sig,
                                         m3=ret.skew,#skewness of series. not required for &quot;historical&quot; method
                                         m4=ret.kurt,#kurtosis of series. not required for &quot;historical&quot; method
                                         invert=FALSE),
                   align=&quot;right&quot;
                   )

#let&#39;s have a look at that handiwork...
par(mfrow=c(1,1))
plot.xts(ret.var,las=1,main=&quot;Rolling VaR Estimate&quot;,cex.main=.9)
</code></pre>

<p><img src="data:image/png;base64,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" alt="plot of chunk unnamed-chunk-1"/> </p>

<pre><code class="r">summary(ret.var)
</code></pre>

<pre><code>     Index            daily.returns    
 Min.   :1998-11-18   Min.   :0.00724  
 1st Qu.:2002-05-23   1st Qu.:0.00848  
 Median :2005-11-29   Median :0.00891  
 Mean   :2005-11-27   Mean   :0.00978  
 3rd Qu.:2009-06-02   3rd Qu.:0.01111  
 Max.   :2012-12-07   Max.   :0.01436  
</code></pre>

<pre><code class="r">
# Now that we have the rolling VaR series, we can calculate the ratios of actual returns to VaR
# estimates. This will give us the magnitude of outcomes in multiples of daily value at risk.
# We need to trim off the first part of the series that was used to estimate the mean/sd etc.
# this makes the return subset the same length as ret.var.

first.date&lt;-index(first(ret.var))
ret.sub&lt;-window(ret,from=first.date)

# Now, divide the ACTUAL returns by the prior day&#39;s VaR etimate. This ratio
# represents the multiple that the actual negative return was of the VaR estimate.
# A ratio of -4x means that the realized negative return was 4x larger in magnitude than than the VaR
# estimate.

act.var.ratio&lt;-ret.sub/lag(ret.var,k=1)
var.breaks&lt;-subset(act.var.ratio,subset=(act.var.ratio$daily.returns&lt;0))
summary(var.breaks)
</code></pre>

<pre><code>     Index            daily.returns    
 Min.   :1998-11-24   Min.   :-11.094  
 1st Qu.:2002-05-17   1st Qu.: -0.631  
 Median :2005-10-09   Median : -0.341  
 Mean   :2005-11-11   Mean   : -0.492  
 3rd Qu.:2009-04-26   3rd Qu.: -0.158  
 Max.   :2012-12-04   Max.   : -0.001  
</code></pre>

<pre><code class="r">plot(var.breaks,las=1,main=&quot;Actual Negative Return / Daily VaR Estimate&quot;,cex.main=.9) # line plot of the breaks.
</code></pre>

<p><img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAfgAAAH4CAMAAACR9g9NAAAAqFBMVEX9/v0AAAAAADkAAGUAOTkAOWUAOY8AZmUAZo8AZrU5AAA5ADk5AGU5OQA5OTk5OWU5OY85ZmU5ZrU5j7U5j9plAABlADllAGVlOQBlOTllOY9lZmVlj49ltdpltf2POQCPOTmPOWWPZgCPj2WPtY+P29qP2/21ZgC1Zjm1tWW124+1/v27u7u+vr7T09Pajzna24/a/tra/v39tWX924/9/rX9/tr9/v1I9PyxAAAAOHRSTlP/////////////////////////////////////////////////////////////////////////ADtcEcoAAAAJcEhZcwAACxIAAAsSAdLdfvwAABjkSURBVHic7Z0PY+S2ccWDu1qNdG5ycSTbTVrp4jqntApvY+1K8/2/WQni3wAEuSCXXHB33vNJKxDgYDA/8pEryeLvCBKp39VOAKojgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBcqgBeqrYI/3N28hsYPaedj9/mW9h++5nf/wQ5K9lNK3Q6GZQM/v9rRLkYbjQd8/vhC9PZwm8bdK7bPQBZjE59PWwW/+7dPHun7023c6cB/fBkA39sj7Lf3XAYGae1v7egdi88ZdlF8qBA3ymdogpGJz6eNgn97+PPTffu6a8+k9yelbv7V1lafZs/t168O/B8edaH35iRrh/61fdUDfjN7/HdbYF3kvTsL9X6Hu3uzgYd9e/jT3cf/u/vTnfWD3b0d/fZw76c8tAFvXs1prrfH+ei4HvyOT3C4+2v7xS9K6V1cdq97bg0VtFHwbQF3bVnbl7agGt7BENq3FdPbDPj/+sE22pd9B/3RDLB77DXSRzuA/JlpN7Cwbw/thUVfOswZ/v7FjW7H+Cnbfzs3Nz3fvHbHlM/HnvEtzo8v+iCwA7r92rgt9Z1L3008dJ06izYK/tmcRhq+sUZLqLucPjrwj//4n/YAUd11tTtO1KMZYPdoqbfs7QCy1+J7shsi8LdkvKBj8fbTC3nwfkoTcGfuPTRufTGynV1ccme8Ptcffd7usNAHlc8uJFVJ2wT/9qC626Ue+F13anvw+z+04K27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" alt="plot of chunk unnamed-chunk-1"/> </p>

<pre><code class="r">
# For USDMXN, we can see that 2008 was 11x daily value at risk. Other, smaller crises 
# episodes occur at around 4-5x daily VaR. Because we are using 5 years of history for the 
# VaR calculation, we miss the de-pegging of USDMXN that happened in 1994. That was on the order of 25x VaR. 

hist(var.breaks,main=&quot;Histogram of VaR Breaks (Multiples of Daily VaR&quot;,cex.main=.9) # histogram of the VaR breaks.
</code></pre>

<p><img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAfgAAAH4CAMAAACR9g9NAAAAnFBMVEX9/v0AAAAAADkAAGUAOWUAOY8AZmUAZo8AZrU5AAA5ADk5AGU5OQA5OTk5OWU5OY85ZmU5ZrU5j485j7U5j9plAABlADllAGVlOQBlOTllOY9lZmVltdpltf2POQCPOTmPOWWPZgCPjzmPtY+P29qP2/21ZgC1Zjm1jzm1/v3ajzna24/a/rXa/tra/v39tWX924/9/rX9/tr9/v21NieCAAAANHRSTlP///////////////////////////////////////////////////////////////////8AS1ciZQAAAAlwSFlzAAALEgAACxIB0t1+/AAADJVJREFUeJztnQ9f47YZgGuubKHXlrUjd7feEtr1ko2sI3/8/b/bLNkO4UigsWwl8DzPj14gsSyhJ6/0KljuN6Ug+ebUDZDToHgoioeieCiKh6J4KIqHongoioeieCiKh6J4KIqHongoioeieCiKh6J4KIqHongoioeieCiKh6J4KIqHongo5yJ+dTUplxez8NA+89Ox51gUF7OynL+7K8v1eNSetyiK0fa7y/unxdbjm6+f+TBbj0OpRRFOtm1g1aSHBu409UBLYo07h++WONTKbJyZ+IcnNtNjO6IpsQx9vWw7PJ43/vDw3Vc8FT+/vF+Pf7i830x/2BX/uEnPiG8ODEcsdn6n3RKHWpmNMxMf+yrEyWYawnNZh0z1zOdish7/7erd3Tw8v7r6XD38VhS1suVDicZjCKjmyCpQr24eaqjP0pw4HhIKBD+LNj7DE+vxzx/v1h//9e4ulJuHh3+GCv4bAvdzqLiW1bavsfaoJeGIcK62IeEUobpRebiV2Tgf8VWXRfGhA+Yh2kaxL6pACLEwD+Kr3lxW74Zw2Kjq7cpXDMjmsDYgm8LtkQ8RH4fTeJbmTVAfUjkIY01Tb9kK+/vtbHX9nx3xk7pJk1B5dcpm7I51PJTcbUl4ufp225Dqa7GdzQ60MhvnI76N+BAvk3q4DF4rLeFhGcSPytrfpO2oeiBtDmvFh9N8N9seWTTjQh1Vo3iWEKKhlnhIGANuQoXbsK3OWp1vPlmOFnvFx0AOGuvTPJR83JJG/LbJ1dd6PFnUecaBVmbj/MSHYbDKqfaJr7qmUr0s2lDbLz6UqDp3e2QzhjYa2rOEp+pD1uOL7+PPsd5yK345mk9eEN9O4G3Jp+JDYx4aUp3pr9Ob51qZjfMTH4fLi9l2qI89Uw/19bS4+Fr8TonI/NtPj45sj4rzaDxLKFFJqg+JpsI8e9Okl63ZH/8xi+Krlx+LD8lHK6s+YltytyVNK7cNqcep9s2yv5XZOD/xcRwOXfI0uYvKir/EaNsR3+RYW/HLdjHVHPkwRzSpXCwRp/VwSHgmKmyH2zq5q56vpMT06+L7JvD3JHehqm3Jxy1plnPbhtRtaVeU+1uZjXMR/xLLnPGw2Lfc3/LcAv5Fnq4dT8RrEJ899Vl/eO5dliJ+5xOhE/MaxMsAKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHsrL4uPFzcXZXBYs/fCi+M202Un47CYDeW28KH798e7Ro7wNjHgoL8/x67Fz/BvErB6K4qG4nINicgfF5RwUIx6KyzkoZvVQuogvWnpvjTxD8RJHne1PJXdhtN8zxSs+Ky91d//iY0K/5y7iis9KfvGr6/u9yznFZyW3+PHFl19DxF8/GesVn5XM4uNtnUflcs9yTvFZyS6+p5okEcVDUTwUxUPJntU3nws9ze4Un5XcEb+ZHrpjuOKzkn2oP3j3dsVnxTkeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHkpu8aurIvDuLrEmSSSz+M10Eh+Xl/dpNUkimcWvP949euxckyRixEPJPcevx87xZ4FZPRTFQ3E5B8XkDorLOShGPBSXc1DM6qGcXnzR0qGsdMblHBSTOygu56AY8VBczkE5fVbfrSZJRPFQFA9F8VByL+fGzcd0T7M7xWcld8Rvpje91CSJZB/q1x9mfdQkiTjHQ1E8FMVDUTwUxUNRPBTFQ1E8FMVDUTwUxUNRPBTFQ1E8FMVDUTyUIcSvx6MBWiK9MkzEL4vi4sCFNj3VJIkMNtRvpkUx6bMl0ivDiF9dhYjfszOyt5okkWHm+KdbItNbIr1iVg9lEPHLanZfHJvdKT4rgwz18dL51ftjZnjFZ2YI8fXdD/bc+6DHmiSRQYb6uENuz70PeqxJEjG5g6J4KANl9Qfuc9NfTZLIMB/gHPVZbaeaJJFBxB/1UW23miSRQYb6+aGbH/RXkyQyzFDvHH/2mNVDUTyUQcRvpsXlH4fuddNPTZLIMJ/V36yu7/2s/qwZaDlXiT92Uaf4rAwX8Qsj/pwZao4vimMvv1J8VszqoSgeip/cQRku4hdHfmCv+KwMJ97l3FkznPilQ/05M+Acf+TVGIrPilk9FMVDGXCoP3JBp/isDBLxi1H7T48tkV4Z7mJLl3NnzUB/nSuN+DNnuL/OHXsDJMVnxaweiuKheLElFC+2hOLFllC82BKKF1tCMauH4v54KIPM8bfH3rn6+JokEa+yheIcD0XxUPoX3y21U3xmhhG/uj7+hvWKz4rioSgeygDiO11jq/jMmNVDUTwUxUNRPBTFQ1E8FMVDUTwUxUNRPBTFQ1E8FMVDUTwUxUNRPBTFQ1E8FMVDUTwUxUPJLX51degKXMVnJbP4zbS+i/2eOyMpPiuZxbcb6/ZssFN8Vox4KLnn+MM3TVB8VszqoSgeiss5KCZ3UE6/nCtajqpJEjHiobicg2JWD0XxUE6Q3IXRfs89zRWflfziY0K/+imxJkkkv/h4KzT/OndqcosfX3z5NUT80xvhKT4r2ZO7zbQYlUuXc6fGrB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHorioSgeiuKhKB6K4qEoHori3yjFS7xU/qjaFH82pIpV/CtF8VAUD0XxUBQPRfFQFA9F8VAUD0XxUBQPRfFQFA9F8VAUD0XxUBQPRfFQFA9F8VAUD0XxUBQPRfFQFA9F8VAUD0XxUBQPRfFQFP9GGXr/e9/iV1exVe/uEmvCM7TYnsVvppP4uLy8T6vp1fNixJ44onsWv/549+ix3OmBryqW09Kv+GciXl4xL8/x63F8O+2Z4+UVk5LVyytG8VAUD0XxUBQPRfFQFA9F8VAUD0XxUBQPpUfxJ/7blJxMvOVfUXnFQ8srHlpe8dDyioeWVzy0vOKh5f0AB4rioSgeiuKhKB6K4qEoHorioSgeiuKh9CZ+9f6uvnvGJKH8elx03o1d1Z20o3czLS5mCeUfdpR3I6Hvyg4915f4Zej19YdZufquU+/F8qHnFqNuDQh1L1L28M8nqfcAWCSIS+m7suzQcz2Jn1/8XkXsMlQ97/Lb1+XDXTdi5HdgdX2/e9eOo0kp27Tgx08J4rv3XeD4nut3qC/rd27X8lFex/KpEb+6/i1tqN/cfkka6svufVd26Lm+xW+mN93Lh6G28y+fkh+E+q8msfs6s7hJm+MT+q7s0HPp4udFEUapWvx6fHTbd8p3jfhwijA/Ljtmd7F8wmjTlO8svu6CDn235dQRH8ImoXzKHJ80WlSsf0krv4iXtndXl9J3J5/jU9oeyoexrmtWnxLxkXniUJ+4nEvy3qHnehVfv+u7/QLJ6/hlkZacVXUn3tkrSXxK35UnXMfLK0PxUBQPRfFQFA9F8VAUD0XxUBQPRfFQFA9F8VAUD0XxUBQPRfFQFA9F8VAUD0XxUBR/aNvS5jZtC+WZo3jFv33CdfOVz3pD8urHT/Fy6vnn7U/NFdb169WBm+nl/fKN/v+UUeJDEK+u/wgbkt/ftTsY5pf37U/zuCthXb9eHVz9HLaodN3jcdagxJeLm/BVxi1H7Y6jMNTPJ3EHV2W82S1dPWxuv79J2cB65rDEr67/F2bueRi+W/H/nm3Fj4t6N058fTP9+Zf7OO4n3ijjPGGJ39z+fn2/Hk/iUP8Q8ZvppN2zWYbdSJNmqK9Hh9QbZZwnLPHloript3d+N9uKH23fBtWcvowzfng9JHe3syBd8a+feJeZRVF8+ynGePivyuovZts9mxfb12Mm+P5ublYvbwnFQ1E8FMVDUTwUxUNRPBTFQ1E8FMVDUTwUxUNRPBTFQ1E8FMVDUTwUxUP5P9isFXAlXnwfAAAAAElFTkSuQmCC" alt="plot of chunk unnamed-chunk-1"/> </p>

<pre><code class="r">

# Now we can just loop through each of the data series we imported and run
# the same analysis for each one. Then we&#39;ll examine the individual results as well as
# aggregate them so we can draw some conclusions about how big, in VaR terms, past crises have been.

# First, here is the workhorse function for this type of analysis. This is the function I use most when
# I want to examine VaR breaks for a given time series.
# This function takes raw time series data in xts format (x), and returns one of a number
# of items depending on the value of &#39;rtype&#39; passed to the function.
# &#39;rtype&#39; can take on the following valueS: &quot;vars&quot;,&quot;returns&quot;,&quot;ratios&quot;,&quot;vol&quot;,&quot;skew&quot;,&quot;kurt&quot;,&quot;mean&quot;
# &#39;window&#39; is the length of the lookback period in days.
# &#39;p&#39; is the quantile desired (e.g. .95, .99, etc.)
# &#39;inv.var&#39; sets whether you want positive or negative VaRs returned from VaR. See Performance Analytics::VaR

# Based on user input for rtype, we&#39;ll take the appropriate action...
# We&#39;re only going to support the &quot;historical&quot; VaR method for now.
varFun&lt;-function(x,p,window,method,inv.var,rtype){


  ret&lt;-periodReturn(x,period=&quot;daily&quot;,subset=NULL,type=&quot;log&quot;)

  if (rtype==&quot;vars&quot;){
    ret.var&lt;-rollapply(ret,width=window,
                       FUN=function(ret) VaR(R=ret,
                                             p=p,
                                             method=&quot;historical&quot;,
                                             clean=&quot;none&quot;,
                                             portfolio_method=&quot;single&quot;,
                                             weights=NULL,
                                             #mu=ret.mu,
                                             #sigma=ret.sig,
                                             #m3=ret.skew,#skewness of series
                                             #m4=ret.kurt,#kurtosis of series
                                             invert=inv.var),
                       align=&quot;right&quot;)
    ret.var&lt;-as.xts(ret.var)
    names(ret.var)&lt;-c(&quot;VaR&quot;)
    return(ret.var)
  } else {
    if (rtype==&quot;returns&quot;){
      ret&lt;-as.xts(ret)
      names(ret)&lt;-c(&quot;returns&quot;)
      return(ret)
    } else {
      if (rtype==&quot;ratios&quot;){
        ret.var&lt;-rollapply(ret,width=window,
                           FUN=function(ret) VaR(R=ret,
                                                 p=p,
                                                 method=&quot;historical&quot;,
                                                 clean=&quot;none&quot;,
                                                 portfolio_method=&quot;single&quot;,
                                                 weights=NULL,
                                                 #mu=ret.mu,
                                                 #sigma=ret.sig,
                                                 #m3=ret.skew,#skewness of series
                                                 #m4=ret.kurt,#kurtosis of series
                                                 invert=inv.var),
                           align=&quot;right&quot;)
        first.date&lt;-index(first(ret.var))
        ret.sub&lt;-window(ret,from=first.date)
        act.var.ratio&lt;-as.xts(ret.sub/ret.var)
        names(act.var.ratio)&lt;-c(&quot;ratios&quot;)
        return(act.var.ratio)
      } else {
        if (rtype==&quot;vol&quot;){
          ret.sig&lt;-rollapply(ret,width=window,FUN=function(ret) apply(X=ret,MARGIN=2, FUN=sd))
          names(ret.sig)&lt;-c(&quot;vols&quot;)
          return(ret.sig)
        } else {
          if (rtype==&quot;skew&quot;){
            ret.skew&lt;-as.xts(rollapply(ret,width=window,FUN=function(ret) skewness(ret,method=&quot;moment&quot;)))
            names(ret.skew)&lt;-c(&quot;skew&quot;)
            return(ret.skew)
          } else {
            if (rtype==&quot;kurt&quot;){
              ret.kurt&lt;-as.xts(rollapply(ret,width=window,FUN=function(ret) kurtosis(ret,method=&quot;moment&quot;)))
              names(ret.kurt)&lt;-c(&quot;kurtosis&quot;)
              return(ret.kurt)
            } else {
              if (rtype==&quot;mean&quot;){
                ret.mu&lt;-as.xts(rollapply(ret,width=window,FUN=mean))
                names(ret.mu)&lt;-c(&quot;means&quot;)
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

v&lt;-varFun(data,.95,1260,&quot;historical&quot;,FALSE,&quot;ratios&quot;)
plot.xts(subset(v,subset=(v&lt;0)),las=1)
</code></pre>

<p><img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAfgAAAH4CAMAAACR9g9NAAAAhFBMVEX9/v0AAAAAADkAAGUAOWUAOY8AZrU5AAA5ADk5AGU5OWU5OY85j7U5j9plAABlADllAGVlOQBlOY9ltbVltf2POQCPOTmPOWWPZgCPj2WPtY+P29qP2/21ZgC1tWW1/v27u7u+vr7T09Pajzna/rXa/tra/v39tWX924/9/rX9/tr9/v2a0VFFAAAALHRSTlP/////////////////////////////////////////////////////////AMfWCYwAAAAJcEhZcwAACxIAAAsSAdLdfvwAABf4SURBVHic7Z0LY+O2sYUL7/U66dZO0mvntum6DletJXv+//+7fOAxAIcUSJECqTmnsSUQ4GAwH3lIeV3zLwSp1F9KJwCVEcArFcArFcArFcArFcArFcArFcArFcArFcArFcArFcArFcArFcArFcArFcArFcArFcArFcArFcArFcArFcArFcArFcArFcAr1e7Bf758eRvqe32uvx3uvi8QJdVo1M8XY74SnR4e7bcN6pbBv5oa2cfT1wWipBqIao+RV2Na8q9N0NfhQ6qkdgf+9GBMXcqW1MfT/Xv95t/G3L/7nvpsrMv+3J52dfUP9dvXhoL9Pi+KbbebukY/6seT7T09fHn7eKqDHE19trfftqe9gW+q2xBiyEy3xfV055t57JB9vtSefGw2fzwFd54apdmHgxeiNgeMfXtsz/a60UTuvm1PewN/eujKyJE9tuV3Pe1r292YdFv2wGlmlFT9qAcT/KQ9x19bu3Cusj3tDXx7RiYm/daW2fUcuzOzhtLUvgN5qFFwgFOjNOJnfD/qwYTjioGv92+/bU97A99aas2jj8z1SODrgf954ifexCiN+uCjqM0Rk1o9+7Y57Q58o5pF7c3PNQtm0q7HmTUx8HXx/9m7x5oQJZUYtbkdiG/ucMYvp/aENO2l192f2Te9nubNV3trVXc25f+wJ+jUKGkWcdSg5OMcrvELyt09N9b6v70PYi2HV+M33L9bo+0+dTnwk6OkWURRe7I/wMFdfUkdI6c+fVsGwlH6qY4wCp/jiyn6GdvHLwtdbvN+Hoif3JVU1s/qV4mKn9VD2xPAKxXAKxXAKxXAKxXAKxXAKxXAKxXAKxXAKxXAKxXAKxXAKxXA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" alt="plot of chunk unnamed-chunk-1"/> </p>

<pre><code class="r">
# If you just have a bunch of tickers and want to see a summary of the VaR breaks, this will do it.
# This function loops through each of the data elements in &#39;tickers&#39; and calculates some
# summary information. It returns a data frame of the results.
# Note: This is the slow way to do this. It&#39;s better to import the data first, and then past a list
# of raw data variables to this function. Getting the data on the fly is slow and then you 
# don&#39;t have use of the variables on exit. I&#39;ll update this summary function in the next post.

vsum&lt;-function(tickers,p,window,method,inv.var){
  n&lt;-length(tickers)
  out&lt;-data.frame()

  for (i in 1:n){
    data&lt;-getSymbols(tickers[i],src=&quot;FRED&quot;,auto.assign=FALSE)
    ret&lt;-periodReturn(data,period=&quot;daily&quot;,subset=NULL,type=&quot;log&quot;)
    ret.mu&lt;-rollapply(ret,width=hist.len,FUN=mean)
    ret.sig&lt;-rollapply(ret,width=hist.len,FUN=function(ret) apply(X=ret,MARGIN=2, FUN=sd))
    ret.skew&lt;-rollapply(ret,width=hist.len,FUN=function(ret) skewness(ret,method=&quot;moment&quot;))
    ret.kurt&lt;-rollapply(ret,width=hist.len,FUN=function(ret) kurtosis(ret,method=&quot;moment&quot;))

    ret.var&lt;-rollapply(ret,width=window,
                       FUN=function(ret) VaR(R=ret,
                                             p=p,
                                             method=method,
                                             clean=&quot;none&quot;,
                                             portfolio_method=&quot;single&quot;,
                                             weights=NULL,
                                             mu=ret.mu,
                                             sigma=ret.sig,
                                             m3=ret.skew,#skewness of series
                                             m4=ret.kurt,#kurtosis of series
                                             invert=inv.var),
                       align=&quot;right&quot;
    )
    first.date&lt;-index(first(ret.var))
    ret.sub&lt;-window(ret,from=first.date)
    act.var.ratio&lt;-ret.sub/ret.var

    out[i,1]&lt;-tickers[i]
    out[i,2]&lt;-max(act.var.ratio)
    out[i,3]&lt;-mean(act.var.ratio)
    out[i,4]&lt;-quantile(act.var.ratio,p,na.rm=TRUE)
  }
  names(out)&lt;-c(&quot;name&quot;,&quot;max.ratio&quot;,&quot;mean.ratio&quot;,&quot;Q.p&quot;)
  return(out)
}
</code></pre>

</body>

</html>
