---
author: ankur
comments: true
date: 2013-07-17 15:56:14+00:00
layout: post
slug: simulating-a-slow-internet-connection
title: Simulating a Slow Internet Connection
wordpress_id: 245
categories:
- AngularJS
- Django
- Programming
---

I am currently working on a single page web application written with AngularJS that communicates with a REST API written with Django and Tastypie. Since I run both the client and the server locally on my machine, every HTTP request that my AngularJS application makes receives a response from the REST API in tens of milliseconds. This is not ideal.

In the real world, Internet connections have latencies that range anywhere from a few hundred milliseconds to tens of seconds. To give my user a smooth experience even on a slow internet connection, I need to ensure that she receives appropriate feedback whenever she performs an action that requires a round-trip to the server. For example, when she navigates to a view that requires a large amount of data to be fetched from the server, my application needs to display a loading spinner of some sort on the screen to indicate progress. I cannot have the UI be completely blank for the time it takes my API to respond to the HTTP request.

Unfortunately, if I run the application locally, it becomes impossible for me to test my progress indicators. The request-response cycle completes so quickly that they are replaced by the actual content within a split second.

After searching the Internet in vain for a solution that would let me simulate a "real" Internet connection from within my browser, I wrote a Django middleware that uses time.sleep() to delay each HTTP response that my application returns by 0 to 4 seconds.

[code language="python"]
import random
import time

class SlowPony(object):
    def process_response(self, request, response):
        time.sleep(random.randint(0, 4))
        return response
[/code]

Then I added this middleware to my MIDDLEWARE_CLASSES:

[code language="python"]
MIDDLEWARE_CLASSES = (
    # ...
    'my_application.middleware.SlowPony',
)
[/code]

I don't like this solution. For one, this does not cause any of the requests to time out, which happens frequently on mobile Internet connections. It's better than nothing, though.

I find it surprising and disappointing that neither Firefox nor Chrome let me simulate slow Internet connections via their developer tools. Fast, reliable, low-latency Internet connections are a rarity, especially since a large and growing number of people browse the web using mobile Internet. This situation is unlikely to change for several years in the future, and tools to test our web applications in such scenarios are either incomplete or non-existent.
