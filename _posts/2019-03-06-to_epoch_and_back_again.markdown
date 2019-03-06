---
layout: post
title:      "To Epoch and Back Again"
date:       2019-03-06 06:27:26 +0000
permalink:  to_epoch_and_back_again
---


## Time is tricky.  
We take it for granted, and forget that it's weird.  Approaching 2:00am at the start of Daylight Savings Time, an hour disappears.  Some timezones make their hours disappear while others don't. While building [Luned](https://github.com/davisjustinw/luned), I referred to [this page](https://www.timeanddate.com/time/change/usa?year=2018) more than once.  

I'm a reformed emt, and emts, paramedics, nurses all have theories and superstitions.  The number one superstition... don't say the 'Q' word. 


`Say quiet in an ED == death stares and reprimands.`  


More theories:


```
full_moon == crazy day
change_in_weather == crazy day
change_in_weather == strokes
first_of_the_month == DUIs
```


And so with heaps of curiosity and a little hubris, I chose to build an app that combines 911 and weather data from Socrata and Dark Sky respectively.  While I thought coordinating data from two APIs  might prove challenging, I completly overlooked the ticking gorrilla... time.

## Ruby's many shades of time
During my journey I worked with four objects that represent time.  

* Date
* DateTime
* Time
* ActiveSupport::Timezone

I needed precision to the hour so I settled on DateTime.  But DateTime doesn't know timezones.  At first I didn't think I'd have to deal with timezones.  My 911 data was coming from Seattle so I thought it must be in Pacific time.  Calls are made to Dark Sky with latitude and longitude so I poorly assumed the response would come back in local time. I was wrong.

Socrata uses a sql like query language called SOQL for making GET requests with a basic structure like this.


```
https://dev.socrata.com/foundry/data.seattle.gov/grwu-wqtk?$select=<field1>,<field2>&$where=<conditional>
```


[There are several functions ](https://dev.socrata.com/docs/queries/)available for more complex queries but the basic structure is the same.  I followed the documentation's lead and tried using the date_trunc_ymd function in the conditional statement.  Truncate takes a date time string and zeros out the time allowing you to query for all rows from a given day.


```
date_trunc_ymd('2017-11-30T23:24:53'))
  => "2017-11-30T00:00:00"
```


Through testing I found the database was not in Pacific Time, so my data was off.  Because it was off I thought the data must be in UTC.  Because I thought it was in UTC I could no longer use the trunc_ymd because my Pacific days would no longer be in on the same days in UTC.  I refactored to use the [between](https://dev.socrata.com/docs/transforms/between.html) function.  Between allows you to pull rows between two dates.  To pull the correct dates I needed timzones. DateTime doesn't know time so I needed Time.

I refactored to use Time instead of DateTime.  I thought I could use the utc method to convert to utc to get the correct query. Still wrong.  So what ultimately worked?  I still struggle to picture this in my head...because time is weird.  And I think this database is weird.  My theory is the database was recorded in Pacific but made to think it was UTC. 

Yeah. I think.

## The Darkness
Let me pause.  Right about the time I made this discovery, I hit a low.  I love the challenge of code.  I love the small wins.  I love when red goes green.  But I was tired.  Work. Wife. Kid. Code. Sleep. Repeat.  The doubts creeped in.  The fatigue numbed my brain.  To me, what makes building software great is the shear number of opportunities you have to fail then get up and try again.  It's relentless.  How many times can your brain re-think the same problem, re-factor reality? That's what makes this fun, even when it's not.  

## The return
So what worked?  Besides not giving up? Or maybe making my app a little simpler?  I had to trick the the database.  Let's walk through an example.  I want a call that happened at 1600 on 3 March 2019.  WIth the Time class your machine will want to force to your local time. You can use UTC to force to UTC time.


```
TIme.new(2019, 3, 3, 16)
=> 2019-03-03 16:00:00 -0800

Time.utc(2019, 3, 3, 16)
=> 2019-03-03 16:00:00 UTC
```


So then I had to trick my time into "database" time.  


```
Time.zone = "Pacific Time (US & Canada)"

Time.utc(2019, 3, 3, 16).in_time_zone
=> Sun, 03 Mar 2019 08:00:00 PST -08:00

```


Yeah weird.  It's 8 hours the wrong way.  I actually believed the server was somewhere in the South Pacific, at one point.  Maybe because I wanted to go visit it at that point. And then here's the gross return trip.


```
time = Time.strptime(row["datetime"], "%Y-%m-%dT%H:%M:%S.%L").utc.to_s
time = Time.strptime(time, "%Y-%m-%d %H:%M:%S UTC").in_time_zone
```


The only way to get the response back to the correct time was to use strptime to force utc, convert to string, then trick it into Pacific.  Dark Sky was easy.  That was in straight up UTC.

Is it elegant?  Nah.  I'm pretty green so maybe there was a better way.  But after dozens of little failures.  This worked.  And I'm proud of it.  

Somedays it's just about not giving up.


