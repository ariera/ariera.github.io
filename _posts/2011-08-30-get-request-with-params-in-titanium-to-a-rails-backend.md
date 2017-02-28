---
layout: post
title: GET request with params in Titanium to a Rails backend
comments: true
---
I'm currently developing an iPhone app with [Appcelerator's Titanium](http://www.appcelerator.com/) that communicates with a Rails backend. Doing so I found a few problems and here is how I solved them. This is the first time I work with Titanium, so don't expect something fancy ; )

## Ajax GET request with params:
We'll use `Titanium.Network.HTTPClient`, if you have never used it I suggest that you take a look a this [awesome example in the Titanium wiki](http://wiki.appcelerator.org/display/guides/Handling+Remote+Data+with+HTTPClient+and+JSON) that perfectly covers the basics.

Say our rails app is a blog at the domain: _http://www.railsblog.com_, and we want to get the all posts that are written in _Spanish_ by the author _ariera_. My first attempt to do so looked like this:


{% highlight javascript %}
    var postsReq = Titanium.Network.createHTTPClient();
    postsReq.open("GET", "http://www.railsblog.com/posts");
    var params = {
    	language: "spanish",
    	author: "ariera"
    };
    postsReq.send(params);
{% endhighlight %}

It took me some time to understand why, but **that code doesn't work as you would expect** it to work. Although we clearly specified that we wanted to do a `GET` request it will do it as a `POST`. The reason why this happens is that **when you pass any parameter to the _send_ method it automatically converts the request into `POST`**.

My solution was to manually construct the url with parameters using `encondeURI`:

{% highlight javascript %}
    var url = "http://www.railsblog.com/posts";
    var params = "?language=spanish&author=ariera";
    var encodedURI = encodeURI(url + params);
    
    var postsReq = Titanium.Network.createHTTPClient();
    postsReq.open("GET", encodedURI);
    postsReq.send();
{% endhighlight %}


And that finally worked. But was that all? Of course not ;-P

## Titanium enable your Rails app:
When testing the iPhone app with my local copy of this railsblog I got the most _weird succinct not-descriptive_ error Rails ever gave me:

{% highlight bash %}
Started GET "/posts?language=spanish&author=ariera" for 127.0.0.1
  Processing by PostController#index as 
Completed 500 Internal Server Error in 1ms

NoMethodError (undefined method `ref' for nil:NilClass):
  

Rendered .../gems/actionpack-3.1.0.rc4/lib/action_dispatch/middleware/templates/
    rescues/_trace.erb (1.1ms)
Rendered .../gems/actionpack-3.1.0.rc4/lib/action_dispatch/middleware/templates/
    rescues/_request_and_response.erb (0.9ms)
Rendered .../gems/actionpack-3.1.0.rc4/lib/action_dispatch/middleware/templates/
    rescues/diagnostics.erb within rescues/layout (4.8ms)
{% endhighlight %}

that `NoMethodError (undefined method 'ref' for nil:NilClass)` error drove me crazy for a while. Apparently this is due to a mix of user agent and Mime Types handled incorrectly by Rails. Following [this stackoverflow answer](http://stackoverflow.com/questions/5126085/ruby-on-rails-mobile-application/5130756#5130756) I did this:

{% highlight ruby %}
    # config/initializers/mime_types.rb
    Mime::Type.register_alias "text/html", :titanium
{% endhighlight %}

{% highlight ruby %}
    # app/controllers/application_controller.rb
    before_filter :handle_titanium
    
    def titanium_user_agent?
      @mobile_user_agent ||= ( request.env["HTTP_USER_AGENT"] && 
        request.env["HTTP_USER_AGENT"][/Titanium/] )
    end
    
    def handle_titanium
      request.format = :titanium if titanium_user_agent?
    end
{% endhighlight %}