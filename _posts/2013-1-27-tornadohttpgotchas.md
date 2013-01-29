---
layout: post
title: Tornado Async HTTP Gotchas
---

At [Venmo](https://venmo.com) we use [Tornado](tornadoweb.org) to power our API and async requests. Recently I found myself having to write a proxy using Tornado and ran into some frustrating issues involving small HTTP nuances that other libraries such as [Requests](http://docs.python-requests.org/) handle for you.

## Gotcha #1

Can you see what's wrong with this snippet?

{% highlight python %}
return http_client.fetch(url,
                         callback,
                         method="GET", # POST, PUT, DELETE, etc..
                         headers=headers,
                         body=None)
{% endhighlight %}

I looked at this for several hours. Turns out that `body` can not be `None` even if it's a `GET` request (requests with no body). Tornado expects an empty string even though the documentation lists `body` as a keyword argument that will default to `None`.

## Gotcha #2

Tornado does not encode arguments for you. I.e., you need to urlencode arguments and append them to either the query string or body depending on the type of request (`GET`, `POST`, `PUT`, `DELETE`, etc..)

Here's how I handled this situation:

{% highlight python %}
## Inside of my request handler.
import urllib

url = "http://www.someapi.com"
body = "" # Per the previous gotcha
encoded_params = urllib.urlencode(params)

if method == "GET":
    url += "?" + encoded_params
else:
    body = encoded_params
{% endhighlight %}

Annoying but not the end of the world.

## Gotcha #3
There's still one more issue regarding our previous gotcha. We still need to properly set our headers.

The above snippet now becomes:
{% highlight python %}
import urllib

url = "http://www.someapi.com"
body = "" # Per the previous gotcha
headers = ""
encoded_params = urllib.urlencode(params)

if method == "GET":
    url += "?" + params_str
else:
    body = params_str
    headers = { 'Content-Type' : 'application/x-www-form-urlencoded' }
{% endhighlight %}

And there you go, 3 silly caveats that I spent a fair amount of time debugging. Running into issues like this brings light to the dependence software engineers have on abstractions and how important it is to understand the layers beneath.

