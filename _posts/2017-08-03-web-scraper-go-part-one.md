---
layout: post
title:  "Go web scraper part one"
date:   2017-08-02 19:06:12 -0400
categories: Go
---
### Go web scraper part one

{% highlight swift %}

func asyncHttpGets(urls []string) []*HttpResponse {
	ch := make(chan *HttpResponse)

	responses := []*HttpResponse{}
	client := http.Client{}

	for _, url := range urls {
		go func(url string) {
			concatUrl := baseUrl + url
			fmt.Printf("Fetching %s \n", concatUrl)
			resp, err := client.Get(concatUrl)
			ch <- &HttpResponse{concatUrl, resp, err}
			if err != nil && resp != nil && resp.StatusCode == http.StatusOK {
				resp.Body.Close()
			}
		}(url)
	}

	for {
		select {
		case r := <-ch:
			fmt.Printf("%s was fetched\n", r.url)
			if r.err != nil {
				fmt.Println("with an error", r.err)
			}
			responses = append(responses, r)
			if len(responses) == len(urls) {
				return responses
			}
		case <-time.After(50 * time.Millisecond):
			fmt.Printf(".")
		}
	}
	return responses
}

{% endhighlight %}
