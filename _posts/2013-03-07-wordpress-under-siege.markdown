---
author: ankur
date: 2013-03-07 15:47:19+00:00
layout: post
slug: wordpress-under-siege
title: 'WordPress: Under Siege'
categories:
    - Programming
---

As I mentioned in [my last post](http://ankursethi.in/2013/03/okay-wordpress-you-win-this-round/), I recently switched my website from my homegrown Django blogging app to WordPress. Before installing WordPress on my VPS, I installed it on a VM so I could test the waters before jumping in. I created an Ubuntu 12.04 VM using VirtualBox and gave it a gigabyte of RAM to work with. After I had WordPress up and running, I created some test posts and played around with various plugins and themes that I could find on the [WordPress directory](http://wordpress.org/extend/). I was dismayed to discover that WordPress has terrible performance out of the box, even if you disable all installed plugins. The WordPress dashboard served by my Ubuntu VM would easily take 4-5 seconds to load, and individual posts would take at least 2-3 seconds to load. I found this unacceptable, so I started searching StackOverflow and the excellent [WordPress StackExchange](http://wordpress.stackexchange.com/) for answers.

The two most straightforward performance optimizations that I could find were:
  
  1. Install a PHP opcode cache.	
  2. Install a page caching plugin.

Installing an opcode cache on Ubuntu is easy:

{% highlight bash %}sudo apt-get install php-apc{% endhighlight %}

No extra configuration is required on Ubuntu. If you use a different distro, read the [php-apc documentation](http://php.net/manual/en/book.apc.php) on the PHP website.

Installing WP Super Cache is similarly easy and a number of excellent tutorials for setting it up are scattered around the Web. [Here](http://www.wpbeginner.com/beginners-guide/how-to-install-and-setup-wp-super-cache-for-beginners/) is a good one for Apache, and [here](http://rtcamp.com/tutorials/wordpress-nginx-wp-super-cache/) is one for nginx. I also recommend perusing [this GitHub repository](https://github.com/perusio/wordpress-nginx/) that contains a complete set of configuration files for serving WordPress through nginx.


### The Numbers

The numbers that follow are for a fresh install of WordPress 3.5.1 running on an Ubuntu 12.04 VM with 1GB of RAM, served by nginx 1.1.19, php-fpm 5.3.10, and backed by MySQL 5.5.29. The host OS is Mac OS X 10.8.2 running on a MacBook Pro. All testing done with [Siege](http://www.joedog.org/siege-home/) 2.74 hitting different pages of the WordPress website in a random order.

{% highlight bash %}siege -d5 -c100 -i -f url_list.txt -t5m{% endhighlight %}

Note that these numbers only reflect a general trend in WordPress performance under load. Real world page load performance depends on many factors, including network latency, page size, whether you're using a CDN or not, the number of separate JavaScript/CSS/image files per page, etc. The following numbers only indicate how quickly WordPress can push HTML to the client.

Despite the flawed testing methodology, these numbers are useful as rough indicators of the effectiveness of opcode and page caching.


#### Fresh Install Without Opcode Cache or Page Cache
    
    Transactions:		        2710 hits
    Availability:		      100.00 %
    Elapsed time:		      299.47 secs
    Data transferred:	        9.59 MB
    Response time:		        8.37 secs
    Transaction rate:	        9.05 trans/sec
    Throughput:		        0.03 MB/sec
    Concurrency:		       75.74
    Successful transactions:        2710
    Failed transactions:	           0
    Longest transaction:	        9.58
    Shortest transaction:	        0.20


#### Fresh Install With WP Super Cache and php-apc
    
    Transactions:		       11833 hits
    Availability:		      100.00 %
    Elapsed time:		      299.70 secs
    Data transferred:	       23.62 MB
    Response time:		        0.02 secs
    Transaction rate:	       39.48 trans/sec
    Throughput:		        0.08 MB/sec
    Concurrency:		        0.75
    Successful transactions:       11881
    Failed transactions:	           0
    Longest transaction:	        0.42
    Shortest transaction:	        0.00


#### Bonus: Numbers for ankursethi.in (this website)

This website uses the same software as my testing VM. The only difference is that it is hosted in somewhere in Germany on a Hetzner VQ12 VPS, and I'm hitting it with Siege from New Delhi in India.
    
    Transactions:		        8566 hits
    Availability:		       98.73 %
    Elapsed time:		      299.54 secs
    Data transferred:	       40.03 MB
    Response time:		        0.55 secs
    Transaction rate:	       28.60 trans/sec
    Throughput:		        0.13 MB/sec
    Concurrency:		       15.72
    Successful transactions:        8577
    Failed transactions:	         110
    Longest transaction:	        5.39
    Shortest transaction:	        0.38


### Closing Words

My test blog went from 39 transactions per second to 9 transactions per second with these two simple performance optimizations, and page load time went from ~8 seconds to 0.02 seconds. This page load time is for users who have not logged in or left a comment. I see a more modest 1.5-2 second load time for logged in users, which is still a 4x improvement. The concurrency number went from 75.74 to 0.75, [which is a good thing in this case](http://www.joedog.org/2012/02/concurrency-single-siege/).

These optimizations should be enough for a majority of low to medium traffic self-hosted WordPress blogs. For more advanced optimization techniques, I recommend reading [this excellent article](http://blog.newrelic.com/2013/02/07/web-performance-optimization-automation/) on the New Relic blog.