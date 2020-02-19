---
layout: post
title:  "Date and Time - using the user's current timezone for countdown to midnight"
date:   2020-02-19
---

**How can I automatically detect the current time zone of each user and store their midnight day cutoff correctly in the DB?**

I could make the user select thier timezone in a form dropdown, store that timezone in the `users` table and then 

But I shouldn't have to ask the user, there's gotta be an automatic way.

200wad, for example, knows my correct timezone and I never had to select it. How? Actually I did select it when I created the account and I can change it in my 'settings'. I just tried it out. Changing the timezone in the settings changes the 'countdown' that I see on the home page.  (so I've found a hack that would allow for cheating I suppose) 

Are there some ways to detect the user's time from their browser. hm. Yes, but doesn't seem robust, especially if there is not a consistent way across browsers.

**Rails Configurations**
`config.active_record.default_timezone = :local or :utc` - "determines whether to use Time.local (if set to :local) or Time.utc (if set to :utc) when pulling dates and times from the database. The default is :utc"

So what is Time.local? It's from the ruby class `Time` interprets the value in local timezone (hm where does it get the local timezone, system time?). Example:
`Time.local(2000,"jan",1,20,15,1)   #=> 2000-01-01 20:15:01 -0600`


`config.time_zone = 'Central Time (US & Canada)`


In the Rails Console:
`irb(main):016:0> Time.now
=> 2020-02-19 12:14:53 -0600
irb(main):017:0> Time.current
=> Wed, 19 Feb 2020 18:14:56 UTC +00:00`

In `irb`:
`irb(main):003:0> Time.now
=> 2020-02-19 12:23:11 -0600
irb(main):004:0> Time.current
Traceback (most recent call last):
        4: from /Users/bhumi/.rbenv/versions/2.6.3/bin/irb:23:in `<main>'
        3: from /Users/bhumi/.rbenv/versions/2.6.3/bin/irb:23:in `load'
        2: from /Users/bhumi/.rbenv/versions/2.6.3/lib/ruby/gems/2.6.0/gems/irb-1.0.0/exe/irb:11:in `<top (required)>'
        1: from (irb):4
NoMethodError (undefined method `current' for Time:Class)`


Listing timezone strings that rails knows about:
`rake time:zones:all`

When working with APIs that involve passing time as argument best to use ISO8601 string, as it is unambiguous, readable, and sortable, they look like this `2015-07-04T21:53:23Z`
`irb(main):005:0> Time.now.utc.iso8601
=> "2020-02-19T19:38:01Z"
irb(main):006:0> Time.now.iso8601    
=> "2020-02-19T13:38:09-06:00"`

**Takeaway Learnings**
Time.now = system time
Time.zone.now = current time in the time zone that is set (default in rails is UTC)
Time.current = Time.zone.now
Time.now.in_time_zone = Time.zone.now

Date.today = system date
Time.zone.today = application date for the time zone that is set (default is UTC)
Date.current = Time.zone.today

**Difference between UTC and GMT?**
GMT is an actual time zone, whereas UTC is a time standard that is used to keep time synchronized across the world. Since Coordinated Universal Time or UTC is a standard, there is no time zone, territory or country that uses it for a local time. UTC is not adjusted for daylight saving time. GMT was used for 100 years but now UTC is used around the world.

Atomic Clocks - 
Leap Seconds - 

Ruby:
[DateTime](https://ruby-doc.org/stdlib-2.6.1/libdoc/date/rdoc/DateTime.html) - does not track any leapseconds or daylight saving rules
[Time](https://ruby-doc.org/core-2.6.3/Time.html) - similar to above DateTime, takes in to account daylight saving time


Ruby Time and Rails ActiveSupport::TimeWithZone instances are interchangeable


References:
https://thoughtbot.com/blog/its-about-time-zones
https://api.rubyonrails.org/classes/ActiveSupport/TimeZone.html