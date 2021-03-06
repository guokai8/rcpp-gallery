---
title: Getting attributes to use xts objects
author: Dirk Eddelbuettel
license: GPL (>= 2)
tags: basics xts
summary: We show how to access attributes and illustrate the use with an xts object.
layout: post
src: 2013-01-12-getting-attributes-for-xts-example.cpp
---
[An earlier post](../setting-object-attributes) illustrated that R object attributes
can be set at the C++ level.  Naturally, we can also read them from an object. This proves 
particularly useful for [xts](http://cran.r-project.org/package=xts) objects which are, in 
essence, numerical matrices with added attributed that are used by a rich set of R operators 
and functions. 

Here, we show how to access these attributes.




{% highlight cpp %}
#include <Rcpp.h>

using namespace Rcpp;
 
// [[Rcpp::export]]
std::vector<std::string> xtsAttributes(NumericMatrix X) {
    std::vector<std::string> nm = X.attributeNames();
    return nm;
}
{% endhighlight %}


A first example simply creates a random `xts` object of twenty observations. 
We then examine the set of attributes and return it in a first program.

{% highlight r %}
    suppressMessages(library(xts))
    set.seed(42)
    n <- 20
    Z <- xts(100+cumsum(rnorm(n)), order.by=ISOdatetime(2013,1,12,20,21,22) + 60*(1:n))
    xtsAttributes(Z)
{% endhighlight %}



<pre class="output">
[1] &quot;dim&quot;         &quot;index&quot;       &quot;class&quot;       &quot;.indexCLASS&quot; &quot;tclass&quot;     
[6] &quot;.indexTZ&quot;    &quot;tzone&quot;      
</pre>


The same result is seen directly in R:

{% highlight r %}
    names(attributes(Z))
{% endhighlight %}



<pre class="output">
[1] &quot;dim&quot;         &quot;index&quot;       &quot;class&quot;       &quot;.indexCLASS&quot; &quot;tclass&quot;     
[6] &quot;.indexTZ&quot;    &quot;tzone&quot;      
</pre>



{% highlight r %}
    all.equal(xtsAttributes(Z), names(attributes(Z)))
{% endhighlight %}



<pre class="output">
[1] TRUE
</pre>


Now, given the attributes we can of course access some of these.
The `index()` function `xts` objects returns the index. Here, we know
we have a `Datetime` object so we can instantiate it at the C++ level.
(Real production code would test types etc).

{% highlight cpp %}
// [[Rcpp::export]]
DatetimeVector xtsIndex(NumericMatrix X) {
    DatetimeVector v(NumericVector(X.attr("index")));
    return v;
}
{% endhighlight %}


{% highlight r %}
    xtsIndex(Z)
{% endhighlight %}



<pre class="output">
 [1] &quot;2013-01-12 20:22:22 CST&quot; &quot;2013-01-12 20:23:22 CST&quot;
 [3] &quot;2013-01-12 20:24:22 CST&quot; &quot;2013-01-12 20:25:22 CST&quot;
 [5] &quot;2013-01-12 20:26:22 CST&quot; &quot;2013-01-12 20:27:22 CST&quot;
 [7] &quot;2013-01-12 20:28:22 CST&quot; &quot;2013-01-12 20:29:22 CST&quot;
 [9] &quot;2013-01-12 20:30:22 CST&quot; &quot;2013-01-12 20:31:22 CST&quot;
[11] &quot;2013-01-12 20:32:22 CST&quot; &quot;2013-01-12 20:33:22 CST&quot;
[13] &quot;2013-01-12 20:34:22 CST&quot; &quot;2013-01-12 20:35:22 CST&quot;
[15] &quot;2013-01-12 20:36:22 CST&quot; &quot;2013-01-12 20:37:22 CST&quot;
[17] &quot;2013-01-12 20:38:22 CST&quot; &quot;2013-01-12 20:39:22 CST&quot;
[19] &quot;2013-01-12 20:40:22 CST&quot; &quot;2013-01-12 20:41:22 CST&quot;
</pre>


Further operations such as subsetting based on the datetime vector
or adjustments to time zones are left as an exercise.

Edited on 2014-03-28 to reflect updated / simpliefied attributes functions.

