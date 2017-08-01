---
layout: post
title:  "UIColor extension"
date:   2017-07-31 19:06:12 -0400
categories: swift
---
### How I extend UIColor

Theres many ways to go about getting your brand colors in your project but I love the simplicity of this solution.  It keeps the consistency of apples UIColor convention by extending UIColor.
Start off by implementing this method in a UIColor extension.

{% highlight swift %}
extension UIColor {
    fileprivate class func color(withRed red:CGFloat, green:CGFloat, blue:CGFloat, alpha:CGFloat = 1) -> UIColor {
          return UIColor(red:red/255.0, green:green/255.0, blue: blue/255.0, alpha:alpha)
    }
}
{% endhighlight %}

Next create your own class variables for all your colors.

{% highlight swift %}
extension UIColor {
    public class var lightGrayBlue: UIColor { return UIColor.color(withRed: 152, green: 178, blue: 188) }
    public class var burntOrnage: UIColor { return UIColor.color(withRed: 255, green: 96, blue: 66) }
    public class var lightBurntOrange: UIColor { return UIColor.color(withRed: 255, green: 154, blue: 134) }
}
{% endhighlight %}

Now your brand colors will be accessible like any other UIColor

{% highlight swift %}
let bgColor: UIColor = .lightGrayBlue
{% endhighlight %}
