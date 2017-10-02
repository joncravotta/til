---
layout: post
title:  "Sinatra and Dalli"
date:   2017-10-01 02:06:12 -0400
categories: Ruby
---

## How to setup cacheing with Dalli and Sinatra
My go to tool for creating micro web api is [Sinatra](https://www.google.com).  It’s a great, easy to use little framework that helps me build apis quick.  I find myself using it as a middle layer between other apis and my apps, so a common thing I need to do is set up a cache.   My go to library for cacheing is [Dalli](https://github.com/petergoldstein/dalli).  According to its github:

> Dalli is a high performance pure Ruby client for accessing memcached servers. It works with memcached 1.4+ only as it uses the newer binary protocol. It should be considered a replacement for the memcache-client gem.  

So lets get started and see how it can I incorporate it into a sinatra project.

i.e. you have to [install and run ](https://gist.github.com/tomysmile/ba6c0ba4488ea51e6423d492985a7953)  memcache for Dalli to work.

I like creating an app controller that holds important functionality so other controllers can subclass.  I’ll start off by setting up the cache itself.

```ruby
class AppController < Sinatra::Base
	def initialize
  		@cache = setup_memcache
		super
  end

	def setup_memcache
    options = { :namespace => "app_v1", :compress => true }
    Dalli::Client.new('localhost:11211', options)
  end
end
```


I create an options hash with a namespace and the option of compressing.  By setting compress to true Dalli will gzip-compress values larger than 1K.  I create the client and pass in the options.

After the setup we will now implement methods to get a cache and set a cache:

```ruby
def cached?(key)
	get_cache(key)
end

def set_cache(key, value, time)
	@cache.set(key, value, time)
end

private

def get_cache(key)
	@cache.get(key)
end
```

`cached?` will retrieve the cache via a key.  `set_cache` will set a cache with a key, value and time which is set in seconds.

We can then create a new controller that subclasses `AppController`

```ruby
class TestController < AppController
  TEST_CACHE_KEY = "test_key_01"
  def initialize
    super
  end

  get "/set_cache" do
    set_cache(TEST_CACHE_KEY, "Hello, i'm cached", 60)
  end

  get "/get_cache" do
    cache = cached?(TEST_CACHE_KEY)
    cache ? cache : "CACHE FAILED"
  end
end
```

We then could test this out by calling `http://YOUR_PATH/set_cache` to set our cache.  After the cache is set we can call `http://YOUR_PATH/get_cache`, you should see `"Hello, i'm cached"` returned!

Now that we set this up locally I’m going to show you how we can get this set up on a live server using [heroku](https://www.heroku.com/)
