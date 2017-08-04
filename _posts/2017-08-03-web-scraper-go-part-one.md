---
layout: post
title:  "Go lang scraper: part one"
date:   2017-08-02 19:06:12 -0400
categories: Go
---
### Building a Fantasy Football API

The past moth i've been playing around with [Go](https://golang.org/).  According to their homepage:


> Go is an open source programming language that makes it easy to build simple, reliable, and efficient software.


And from my little experience with it I would have to agree with that.  I will be writing a series about building a webscraper api using go while utilizing some of it's powerful features.

With Football starting right around the corner we will be building a api that will return us formatted fantasy football data in json.  Users will be able to make a network request with a position and the api will return data for all players in the 2016 season.


In this post we are going to start simple by talking about how to make network request in go.

### Starting Off
This series won't include an onboarding guide for Go but there is plenty of resources to get you started:
* [Tour of Go](https://tour.golang.org/welcome/1)
* [Go Docs](https://golang.org/doc/code.html)
* [Effective Go](https://golang.org/doc/effective_go.html)
* [Go by example](https://gobyexample.com/)

### Network request
We will be using http://fftoday.com as our source for our fantasy football data.  From the homepage you will see different positions you can look for.  Click quarterbacks and take a look at the url.

{% highlight markdown %}
http://fftoday.com/stats/playerstats.php?Season=2016&GameWeek=&PosID=10&LeagueID=
{% endhighlight %}

You can see that there are params for season, game week, position and league id.  I'm going to give a value of 26955 to the league id for ESPN (Thats the league I personally play in).  You can change the league by clicking on the drop down on top to the right of the table. Later we will be changing the posId to get info for different postions.

Next we will make the request to get this pages html.  Lets see how it looks:

{% highlight go %}
package main

import (
	"net/http"
	"io/ioutil"
)

func main() {
	makeRequest()
}

func makeRequest() {
	response, error := http.Get("http://fftoday.com/stats/playerstats.php?Season=2016&GameWeek=&PosID=10&LeagueID=26955")

	if error != nil {
		println("error!")
	}

	defer response.Body.Close()

	body, err := ioutil.ReadAll(response.Body)

	if error != nil {
		println("error!")
	}

	println(string(body))
}
{% endhighlight %}

After running: go run main.go in your projects root directory you should see the pages html printed out in your console.

Im not gonna go into packages and imports since it's out of scope, theres awesome documentation on [go's website](https://golang.org/doc/code.html#PackageNames) about it that will go into much greater detail than I can provide.  

If you use an IDE it will automatically pull in the proper imports.  Im currently using [gogland by jetbrains](https://www.jetbrains.com/go/) for free.  It's a beta but its been working pretty well thus far.

Otherwise let's break down this code!

Main is the first function run in the program, it is simply calling our makeRequest function.

The first line in this func is making our request to fftoday.

{% highlight go %}
response, error := http.Get("http://fftoday.com/stats/playerstats.php?Season=2016&GameWeek=&PosID=10&LeagueID=26955")
{% endhighlight %}

You can see that we have two properties being assigned to this call,  response and error.  This is because the request will come back with either a successful response or error.  Below that line we are checking to make sure there was no error before proceeding.

{% highlight go %}
if error != nil {
	println("error!")
}
{% endhighlight %}

After the error check we hit this line of code.

{% highlight go %}
defer resp.Body.Close()
{% endhighlight %}

A defer statement defers the execution of a function until the surrounding function returns. [Source](https://tour.golang.org/flowcontrol/12).

This will make sure that we will always exit from reading the request response before we leave the function.

On the next line we read the contents of our request and then print it out.

{% highlight go %}
body, err := ioutil.ReadAll(response.Body)

if error != nil {
	println("error!")
}

println(string(body))
{% endhighlight %}

Again we assign two properties to this call and check for errors.  Pretty simple but this is an important piece of our project.

In the next post we will talk about parsing our returned html!
