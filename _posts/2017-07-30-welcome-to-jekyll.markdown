---
layout: post
title:  "Building a queue in Swift"
date:   2017-07-30 19:06:12 -0400
categories: swift
---
### Before We start
#### Another header

> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.

*list
*list
*list

*************

This is [an example](http://example.com/ "Title") inline link.

[This link](http://example.net/) has no title attribute.

You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight swift linenos %}
import Foundation

struct User {
    var shownSurvey: Bool
    var promotion: Bool
    var onboarding: Bool
}

let u = User(shownSurvey: false, promotion: false, onboarding: false)

enum EventType {
    case onboarding
    case survey
    case promotion

    var eventName: String {
        switch self {
        case .onboarding: return "onboarding"
        case .promotion: return  "promotion"
        case .survey: return "survey"
        }
    }
}



class QueueManager {

    var user: User

    init(user: User) {
        self.user = user
        setUpAlerts()
        curveBall()
    }

    func curveBall() {
        sleep(10)
        queue.append(.onboarding)
    }

    private var queue:[EventType] = [] {
        didSet {

            guard !queue.isEmpty else { return }

            handleQueue()
        }
    }

    private func setUpAlerts() {
        var localQueue:[EventType] = []
        print("setting up")
        if !user.onboarding{
            localQueue.append(.onboarding)
        }

        if !user.promotion {
            localQueue.append(.promotion)
        }

        if !user.shownSurvey {
            localQueue.append(.survey)
        }

        guard !localQueue.isEmpty else { return }
        queue.append(contentsOf: localQueue)
    }

    private func handleQueue() {
        print("handling queue \(queue)")
        if let firstEvent = queue.first {
            performAction(with: firstEvent.eventName, completion: {
                self.handleCompletedEvent(event: firstEvent)
                self.queue.remove(at: 0)
            })
        }
    }

    func handleCompletedEvent(event type: EventType) {
        print("handling completed")
        print("_________________________________")
        switch type {
        case .onboarding: return user.onboarding = true
        case .promotion: return user.promotion = true
        case .survey: return user.shownSurvey = true
        }
    }


    private func performAction(with name: String, completion: @escaping (()->())) {
        sleep(4)
        print("action \(name)")
        completion()
    }
}

let q = QueueManager(user: u)
//print_hi('Tom')
//=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
