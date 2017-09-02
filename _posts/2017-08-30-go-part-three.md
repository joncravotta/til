---
layout: post
title:  "Go lang scraper: part three"
date:   2017-08-30 02:07:12 -0200
categories: Go
---

### Setting up routers

In our third installment of building a fantasy football api we will be setting up routers for our api to return data.

In our last piece we went over how to use goquery to parse html and moved that data into models.  We will now use those models to return json.

We will be using a library called [mux](https://github.com/gorilla/mux) to handle our routing.  It is a very popular project widely used in go.  Their github repo has a bunch of examples and resources on how to use it in otherways than what were doing in this tutorial.

To install run this in your terminal:

{% highlight markdown %}
go get -u github.com/gorilla/mux
{% endhighlight %}

Great, now you should be able to import this into your project:
{% highlight markdown %}
import (
"github.com/gorilla/mux"
...
)
{% endhighlight %}

Once that is set up your ready to start building routes.

We will build a set up routes function that will be responsible for creating the route.

{% highlight go %}
func setUpRoutes() {
	router := mux.NewRouter()
	router.HandleFunc("/stats", GetStat).Methods("GET")
	log.Fatal(http.ListenAndServe(":12345", router))
}
{% endhighlight %}

You can see that we start off by creating a new router and then we add a new route to the router by using the HandleFunc method.  We give it a path of "/stats" and tell the router that it is a get request.  We also pass something called GetStat, that is going to be the function responsible for getting and returning the data.  Next we setup the port it should be listening on which is going to be 12345 and then pass the router into it.

Now we just change our main func to call setUpRoutes:

{% highlight go %}
func main() {
	setUpRoutes()
}
{% endhighlight %}

Great now we need to build our GetStat function.
{% highlight go %}
func GetStat(w http.ResponseWriter, req*http.Request) {

	var players []Player

  	u := baseUrl + "?Season=" + season + "&GameWeek=" + "&PosID=qb"  + "&LeagueID=" + espnId

  	document, error := goquery.NewDocument(u)

  	if error != nil {
  		log.Fatal(error)
  	}

  	document.Find(".tablehdr").Parent().After(".tableclmhdr").Find("tr").Each(func(i int, s *goquery.Selection) {
  		 p := parseQB(s)
  		 players = append(players, p)
  	})

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(players)
}
{% endhighlight %}

We start off by creating an array of players, we will be adding players to this array as we parse them.  We then construct a url, ive set up some constants in my file to house some information about the url.  Next we make our go query request and than check for errors, and then parse our players.  Everything looks just like it did in our previous post.

The next two lines are important:

{% highlight go %}
w.Header().Set("Content-Type", "application/json")
{% endhighlight %}

This will set the response header type to return json, very important for the client to get the correct return type.

{% highlight go %}
json.NewEncoder(w).Encode(players)
{% endhighlight %}

This line will encode our array of players into json so it can be return to the client.  Hmmm, how does go know how to turn our models into json?  It doesn't, we need to add struct composition to our model, this will give it field name and allow us to add some other options.


{% highlight go %}
type GeneralStats struct {
	GamesPlayed string `json:"games_played",omitempty`
	Points string `json:"fantasy_points",omitempty`
	PointsPerGame string `json:"fantasy_points_per_game",omitempty`
}

type PassingStats struct {
	Completions string `json:"completions",omitempty`
	Attempts string `json:"attempts",omitempty`
	Yards string `json:"yards",omitempty`
	Tds string `json:"tds",omitempty`
	Interceptions string `json:"interceptions",omitempty`
}

type RushingStats struct {
	Attempts string `json:"attempts",omitempty`
	Yards string `json:"yards",omitempty`
	Tds string `json:"tds",omitempty`
}

type ReceivingStats struct {
	Targets string `json:"targets",omitempty`
	Receptions string `json:"receptions",omitempty`
	Yards string `json:"yards",omitempty`
	Tds string `json:"tds",omitempty`
}

type Player struct {
	Name string `json:"name",omitempty`
	Team string `json:"team",omitempty`
	General GeneralStats `json:"general",omitempty`
	Passing PassingStats `json:"passing",omitempty`
	Rushing RushingStats `json:"rushing",omitempty`
	Receiving ReceivingStats `json:"receiving",omitempty`
}
{% endhighlight %}


By doing this we gave the json clear + formatted identifiers and also added omitempty to all the fields.  If for some reason we could not parse or if there was no data on a player this would omit the field from the response.

Nice, so lets give this a try.  Run your program, head to a browser and go to:

{% highlight markdown %}
http://localhost:12345/stats
{% endhighlight %}

You should get a response that looks like this!
{% highlight json %}
[
    {
        "name": "Aaron Rodgers",
        "team": "GB",
        "general": {
            "games_played": "16",
            "fantasy_points": "376.0",
            "fantasy_points_per_game": "23.5"
        },
        "passing": {
            "completions": "401",
            "attempts": "610",
            "yards": "4,428",
            "tds": "40",
            "interceptions": "7"
        },
        "rushing": {
            "attempts": "67",
            "yards": "369",
            "tds": "4"
        },
        "receiving": {
            "targets": "",
            "receptions": "",
            "yards": "",
            "tds": ""
        }
    },
    {
        "name": "Matt Ryan",
        "team": "ATL",
        "general": {
            "games_played": "16",
            "fantasy_points": "345.5",
            "fantasy_points_per_game": "21.6"
        },
        "passing": {
            "completions": "373",
            "attempts": "534",
            "yards": "4,944",
            "tds": "38",
            "interceptions": "7"
        },
        "rushing": {
            "attempts": "35",
            "yards": "117",
            "tds": "0"
        },
        "receiving": {
            "targets": "",
            "receptions": "",
            "yards": "",
            "tds": ""
        }
    },
    {
        "name": "Drew Brees",
        "team": "NO",
        "general": {
            "games_played": "16",
            "fantasy_points": "332.3",
            "fantasy_points_per_game": "20.8"
        },
        "passing": {
            "completions": "471",
            "attempts": "673",
            "yards": "5,207",
            "tds": "37",
            "interceptions": "15"
        },
        "rushing": {
            "attempts": "23",
            "yards": "20",
            "tds": "2"
        },
        "receiving": {
            "targets": "",
            "receptions": "",
            "yards": "",
            "tds": ""
        }
    }
    .....
{% endhighlight %}

In my next post I will be going over how we can use request params to get different positions based on the clients need.
