---
author: ankur
comments: true
date: 2012-07-16 15:27:39+00:00
layout: post
slug: a-whole-new-can-of-beans
title: A Whole New Can of Beans
wordpress_id: 12
categories:
- Programming
---



I'm envious of people who have consistently posted to their blogs for years. I, too, have blogged semi-regularly over the years, but I've never managed to stick to a single blog for long.

All I remember about my first blog is that it was mostly a link dump, but I do remember my second one. Titled _Infuriated_, it was a rant blog where the 15-year-old me wrote angry, vitriol-laden posts about things that annoyed me. A few months later, just when I was running out of swear words, someone introduced me to the world of Free Software. I scrapped _Infuriated_ and started a short-lived blog called _The Free Speech Blog_, where I wrote exactly one holier-than-thou post about something disgusting that Sony had done. Then I proceeded to neglect all my blogs for a couple of months until one day I got angry at something that had happened at school and created yet another rant blog on WordPress.com (you see a pattern here?) This one was called _Badger Alert!_, and ended up being very different from _Infuriated_. While _Infuriated_ was just pure vitriolic fun, _Badger Alert!_ had content that was more spontaneous and sincere than anything I had written before or have written since. You see, by this time I was well into my rebellious teenage years, and the smallest things would tick me off. _Badger Alert!_ was my "personal space" on the internet, somewhere I could talk about my feelings and ocassionally vent my anger. It got very pretentious sometimes, but on the whole it was the most fun I ever had writing.

The last blog I wrote before I dropped off the grid for close to two years was titled _A Series of Uncool Events_, hosted at Uncool.in (that URL redirects to AnkurSethi.in now). I didn't have much fun with it, and I wasn't particularly proud of what I posted there, yet it ended up being my most popular blog. The most popular post on Uncool.in, titled _Computer Science FAIL: Higher Education in India_, was a list of dangerously incorrect information that a professor at my college had fed my class during our first Introduction to Computer Science lecture. The post stayed on the Hacker News front page for a little time and sparked a handful of comments. It was more popular on reddit, where it spawned a long comment thread complete with flamewars and bad puns. It was was blogged and re-blogged across the Internet, leading to over a hundred thousand page views in the space of a week or two. At one point, it even showed up on the Cincom Smalltalk community blog. The resulting increase in PageRank ensured at least a few hundred hits every day. For the first time ever, my blog showed up as the number one result when you ran a search for my name on Google. Eventually I got rid of Uncool.in too, because I have a medical condition commonly known as being a complete idiot.

After Uncool.in, I became apathetic about blogging. I didn't feel I had enough to say, or that what I had to say mattered, or that blogging was worth it at all. I'd purchased a nice beefy VPS for hosting my personal website and blog, but I didn't set it up with a blogging tool. For many months both AnkurSethi.in and Uncool.in pointed at a plain HTML page with my email address and twitter handle on it.

If you're part of the tech community, a blog or a personal webpage is a wonderful thing to have. I wouldn't say it's _essential_, because some of the best engineers out there don't care for or need a web presence, but it's definitely a huge plus. Blogging leads to good things. For me, it was one of the ways I found people who shared my interests. Every time one of my posts got more than a few hundred hits, I'd get emails from people who wanted to talk or just say hi. Blogging was also a way to get involved in discussions about topics I _really_ cared about. Instead of merely participating in discussions that were trending on Hacker News or reddit, I could start discussions that I wanted to have. Now, it wasn't as if I wrote insightful posts that sparked deeply nested comment threads across the Internet. No. The best discussions I had via my blog were simply the ones I had with people I already knew, and most of these took place over IRC, IM or beer. Still, there was the ocassional post that would get attention, and that was nice.

As soon as I stopped blogging, I started feeling the negative effects. I noticed a drop in communication sent my way, and participated in fewer online discussions on the whole (IRC flamewars are not discussions). As my long-form writing output decreased, my thinking started to get muddled. Plus, there were so many things I was mulling over but was unable to discuss with anyone. I wanted to talk about how great [Racket](http://racket-lang.org) was, about the state of programming documentation, about things I was doing at college, about my little side projects and hacks, about Linux and free software, about privacy, about the books I was reading and the music I was listening to. I tried writing in real, physical, paper-and-pencil journal for a while, but it didn't feel the same as writing publicly on a blog. Twitter and Facebook were useless for the kind of conversations I wanted to have, and reddit and Hacker News had their own issues. Eventually, I installed WordPress on my new VPS so I could start blogging again.

Despite having spent several hours setting up the new WordPress blog and getting it to behave just right, I never really posted to it because my frustrations with WordPress had grown to a point where I couldn't even stand to look at the interface (a blog post for some other day). The blog just sat there, attracting only bots and spammers.

But then something wonderful happened. Earlier this year, I started getting into web development and, as a way of getting to know how to build, deploy and maintain a Django app, I decided to replace WordPress with a custom blogging tool of my own creation. Can 'O Beans was born.

If you go look at [the Can 'O Beans code on GitHub](http://github.com/GeneralMaximus/can_o_beans), you'll notice two things. One, some of the code — especially the template code — is atrocious. Once you get over that, the second thing you'll notice is that Can 'O Beans is as simple as a blogging tool can possibly be. It consist of exactly one database model, plus some templates and CSS. I didn't even build a publishing interface, choosing instead to use the Django admin for writing new posts.

Can 'O Beans as I'm using it now was "finished" a few weeks after I started building it, but I didn't deploy it for _months_ because I couldn't bring myself to dive into the big bad world of WSGI servers and reverse proxies and databases and what have you. Then last month, I found that I needed to deploy several Django apps for a project I'm working on. This forced me to sit down for three days and absorb everything I could about using Django in production. I wanted to experiment with what I had learned on my own server before I went live with it on the server my project is hosted on, so Can 'O Beans finally got a chance to shine.

And kids, that is why AnkurSethi.in once again has the pleasure of hosting real content instead of an unfriendly white HTML page.


