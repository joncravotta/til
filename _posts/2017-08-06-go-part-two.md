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

To parse the html we need to know the structure of it, we can check this out by using your browsers inspector.  

![Inspector]({{ site.url }}/til/img/go.png)


Digging through we can see that there are numerous tables and we can find the one that includes the data we want.

Now we know what to look for when we parse our html, we can do something like this:


{% highlight go %}
document.Find(".tablehdr").Parent().After(".tableclmhdr").Find("tr").Each(func(i int, s *goquery.Selection) {
	//parse more later
})
{% endhighlight %}

The html structure was a little tricky since the rows we want to parse do not have a unique identifier, so we had to get the row with ".tablehdr" then get its parent.  After getting the parent (which would be the tbody) we say After ".tableclmhdr" since thats where our data starts.

We then use the Each method to loop through the rows.

Great, now we can start parsing each row. For this we will create a few struct to help model our data.

{% highlight go %}
type GeneralStats struct {
	GamesPlayed string
	Points string
	PointsPerGame string
}

type PassingStats struct {
	Completions string
	Attempts string
	Yards string
	Tds string
	Interceptions string
}

type RushingStats struct {
	Attempts string
	Yards string
	Tds string
}

type Player struct {
	Name string
	Team string
	General GeneralStats
	Passing PassingStats
	Rushing RushingStats
}
{% endhighlight %}


Looking at the inspector we can see that each row has 13 cells.  Since there is no unique id to these cells we will infer the property based on the count.

We will create a new func that will parse a qb for us.  Although not the prettiest solution, this will do the job and give us exactly what we need.

{% highlight go %}

func parseQB(s *goquery.Selection) Player {
	p := Player{}

	// Loop through each cell
	s.Find("td").Each(func(int int, s *goquery.Selection) {
		switch int {
		case 0: p.Name = s.Find("a").First().Text()
		case 1: p.Team = s.Text()
		case 2: p.General.GamesPlayed = s.Text()
		case 3:	p.Passing.Completions = s.Text()
		case 4: p.Passing.Attempts = s.Text()
		case 5: p.Passing.Yards = s.Text()
		case 6: p.Passing.Tds = s.Text()
		case 7: p.Passing.Interceptions = s.Text()
		case 8: p.Rushing.Attempts = s.Text()
		case 9: p.Rushing.Yards = s.Text()
		case 10: p.Rushing.Tds = s.Text()
		case 11: p.General.Points = s.Text()
		case 12: p.General.PointsPerGame = s.Text()
		}
	})

	return p
}

{% endhighlight %}

The func takes in the type of *goquery.Selection, this is what goquery returns as a value from the .Find search.

We also do an extra search for the 0 index, that first cell has the qbs name wrapped in a link tag so we just parse it out.

We can call this method from our first document loop:

{% highlight go %}
var players []Player

document.Find(".tablehdr").Parent().After(".tableclmhdr").Find("tr").Each(func(i int, s *goquery.Selection) {
	p := parseQB(s)
	players = append(players, p)
})

println(len(players))
println(players[0].name)
// prints 52
// prints Aaron Rodgers
{% endhighlight %}

We added a players slice and append each player as we parse them, we now have all the html data in clean model objects.

In our next post we will set up our server and return these players in a json format as a response to a network request.
