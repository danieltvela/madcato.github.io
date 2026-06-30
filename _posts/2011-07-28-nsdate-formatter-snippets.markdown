---
layout:     post
title:      "NSDate formatter snippets"
date:       2011-07-28 12:19:00
author:     "Daniel Vela"
og_image: "/img/post-bg-01.jpg"
locale:       en
lang-ref:   nsdate-formatter-snippets
---

## Get a string from date object

{% highlight objc %}
NSDateFormatter* fmt = [[NSDateFormatter alloc] init];
[fmt setTimeStyle:NSDateFormatterNoStyle];
[fmt setDateStyle:NSDateFormatterLongStyle];

NSLog(@“Thanksgiving is: %@”, [fmt stringFromDate:thanksgiving]);
{% endhighlight %}

## Get a date object from a string

{% highlight objc %}
NSDateFormatter* fmt = [[NSDateFormatter alloc]init];
[fmt setDateFormat:@“dd/MM/yyyy HH:mm”];
[fmt setTimeZone: [NSTimeZone timeZoneWithName:@“Europe/Madrid”]]; // Use [NSTimeZone knownTimeZoneNames]; for other timezones [fmt setCalendar:cal];

NSDate* date = [fmt dateFromString:@“10/06/2010 9:30”];
{% endhighlight %}

## For localization use dateFormatFromTemplate

{% highlight objc %}
NSLocale *locale = [NSLocale currentLocale];

NSString *dateFormat;
NSString *dateComponents = @“yMMMMd”;

dateFormat = [NSDateFormatter dateFormatFromTemplate:dateComponents options:0 locale:locale];
NSLog(@“Date format for %@: %@”,
[locale displayNameForKey:NSLocaleIdentifier value:[locale localeIdentifier]], dateFormat);
{% endhighlight %}

![apple watches]({{ site.url }}/assets/tumblr_inline_mjxj1zDBhW1qz4rgp.png)

20110404.20

