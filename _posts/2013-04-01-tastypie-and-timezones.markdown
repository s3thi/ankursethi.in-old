---
author: ankur
comments: true
date: 2013-04-01 07:12:16+00:00
layout: post
slug: tastypie-and-timezones
title: Tastypie and Timezones
wordpress_id: 187
categories:
- Django
- Programming
---

If you use Tastypie with Django's timezone support turned on (i.e, you have `USE_TZ = True` in your `settings.py`), you will notice that Tastypie helpfully converts all dates and times in your resources to the timezone specified in your `TIME_ZONE` setting before returning them. If you care about internationalization, this is not the behavior you want. Tastypie encodes dates and times in the  ISO8601 format by default. These dates and times have no timezone information attached to them, which means that the consumers of your API will have no idea how to correctly display them or convert them to other timezones.

This is what ISO8601 datetimes look like:

[code language="javascript"]{
    "end_time": "2013-04-01T06:32:06",
    "start_time": "2013-04-01T00:30:00"
}
[/code]

Both those datetimes are in the Asia/Kolkata (UTC+5:30) timezone. How can I tell? I can't. Unless I look in my `settings.py`. Not cool.

There are two solutions to this problem. First, you could add this line to your `settings.py`:

[code language="python"]
TASTYPIE_DATETIME_FORMATTING = 'rfc-2822'
[/code]

This will cause Tastypie to format dates and times using the format specified in [RFC2822](http://www.ietf.org/rfc/rfc2822.txt). Your dates and times will now include timezone information:

[code language="javascript"]
{
    "end_time": "Mon, 1 Apr 2013 12:02:06 +0530",
    "start_time": "Mon, 1 Apr 2013 06:00:00 +0530"
}
[/code]

The second solution, which is the solution I prefer, is simpler: use UTC everywhere on the server and let your API consumers deal with timezone conversions. Set your `TIME_ZONE` to `'UTC'` and sleep easy.

If your API consumer is a web application, I highly recommend using [Moment.js](http://momentjs.com) for all date and time operations.
