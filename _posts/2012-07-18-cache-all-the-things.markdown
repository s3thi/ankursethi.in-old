---
author: ankur
comments: true
date: 2012-07-18 19:45:28+00:00
layout: post
slug: cache-all-the-things
title: Cache all the Things!
wordpress_id: 21
categories:
- Programming
---

Yesterday night I installed memcached on the server that runs AnkurSethi.in and hooked it up to Django's caching framework. Since then, page load times are significantly lower and so is the server load. This little machine is now ready to handle whatever the Internet throws at it.




According to the Chrome developer tools, the slowest loading resource on AnkurSethi.in is now PT Sans, the font that I use for all the text on this website. Since I set up caching, resources from my own server take less than 700ms to reach me in New Delhi. On the other hand, PT Sans as served from the Google CDN takes up to 1 second to get here. This doesn't bother me much, though, since the browser caches the font after it is downloaded once, which means further page loads are blazing fast. Besides, web fonts are loaded asynchronously, so you can still read the page content while the font is loading in the background. Optimizing font loading time is simply not a productive endeavor.

Setting up caching in Django was easy enough, but I did run into one major issue: cache expiry. When I finish writing a journal entry, I often go back and edit it multiple times in order to fix lingering typos and grammatical errors. Sometimes I even do this after I've published it. I never want to serve up a journal entry with incorrect spelling or grammar. However, I had set a one hour expiry limit on the items in my cache. This meant that, even after I had made corrections to an existing journal entry, the server would continue to serve the incorrect entry from the cache for some time. In the worst case, "some time" could mean an hour, which was unacceptable.

The solution I used to fix this issue was simple, if inelegant. I hooked up the `post_save` signal that is defined on every Django model to a function that simply nukes the entire cache. It may sound like a terrible solution, but it works great in practice. Sure, rebuilding the cache is expensive, but not expensive enough to warrant a more complex solution. Having to rebuild the entire cache is a huge issue for large, distributed, high-traffic webapps where running one database query may take tens of seconds. It is, however, not an issue for my humble personal blog, which contains a few kilobytes of data, runs off a single machine and is lucky to see a few hundred visitors per day. On the whole, I'm very happy with my current setup.

One more item crossed off the TODO list for this website. Next item: support for syntax highlighting.


