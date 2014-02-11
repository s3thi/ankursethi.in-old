---
author: ankur
comments: true
date: 2012-11-29 09:43:40+00:00
layout: post
slug: mobile-tweaks-and-chrome-extension
title: Mobile Tweaks and Chrome Extension
wordpress_id: 14
categories:
- Programming
---



I just made some changes to my website's CSS to make it more readable on small screens. I'm not completely happy with how it looks - the header looks jarringly out of place and code samples are all messed up - but at least now you don't have to play with your browser's zoom settings to be able to read the text properly. Baby steps.

I also wrote a little Chrome extension for adding bookmarks to my [homegrown bookmarks manager](http://ankursethi.in/bookmarks). I was pleasantly surprised at how easy it is to extend Chrome. The APIs are well-designed and well-documented, and a number of [examples](http://developer.chrome.com/extensions/samples.html) are available from [Google's developer website](https://developer.chrome.com). I can see myself writing more extensions for Chrome in the future.

My bookmarks extension is nothing to write home about. It consists of the mandatory `manifest.json` file, four or five lines of JavaScript that open a new tab when a toolbar button is clicked, and a terrible icon I quickly designed in Photoshop. The [code is on GitHub](https://github.com/GeneralMaximus/can_o_bookmarks) so that my friends and family can laugh at my terrible Photoshop skills.


