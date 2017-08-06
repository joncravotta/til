---
layout: post
title:  "Go lang scraper: part two"
date:   2017-08-05 02:06:12 -0400
categories: Go
---

### Parsing html

In our second installment of building a Fantasy Football api we will be learning how to parse the html of our request.

In our first post we learned how to make a network request using go.  We will continue off that.

You can use go's built in libraries to parse html but I found it easier to use [goquery](https://github.com/PuerkitoBio/goquery). It is very similar to nokogiri in ruby or beautiful soup in python.  It helps parsing html by utilizing css selectors, html tags etc. To install run this in your command line:

{% highlight markdown %}
go get github.com/PuerkitoBio/goquery
{% endhighlight %}

Then in your main.go file add it to your list of imports:
{% highlight markdown %}
import (
	"github.com/PuerkitoBio/goquery"
  ...
)
{% endhighlight %}

Instead of using the net/http library to make our request we will use goquery.
{% highlight go %}
func makeRequest() {
  document, error := goquery.NewDocument("http://fftoday.com/stats/playerstats.php?Season=2016&GameWeek=&PosID=10&LeagueID=26955")

  if error != nil {
    println("error!")
  }
}

{% endhighlight %}


This will return use a document object, with this we can begin to parse our html.

To parse the html we need to know the structure of it, we can check this out by using your browser inspector.  

![My helpful screenshot]({{ site.url }}/images/go_html_inspect.png)
