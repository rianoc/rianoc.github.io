---
layout: post
title:  "Tides in Home Assistant"
excerpt: "Gathering Tide information to display in Home Assistant"
date:   2020-04-27 21:11:00
---

I wanted to have tide information displayed in Home Assistant.
Searching I found a [post](https://community.home-assistant.io/t/tide-times/9045) and related [repository](https://github.com/imikerussell/TideTimes). It scraped information from [Met Office](https://www.metoffice.gov.uk/weather/specialist-forecasts/coast-and-sea/beach-forecast-and-tide-times) but when I tested it I found it was no longer working with the current website.

So I then wrote my own scraper for [tidestimes.org.uk](https://www.tidetimes.org.uk/london-bridge-tower-pier-tide-times) which is now on [Github](https://github.com/rianoc/tides).

Currently it is scraping the full site but I have since seen that the same information can be found in an [rss feed](https://www.tidetimes.org.uk/london-bridge-tower-pier-tide-times.rss) which I may update the code to use instead.

My first version used [pandas](https://pandas.pydata.org/) but as the Hass.io VM I am using is based on [Alpine Linux](https://alpinelinux.org/) installing the package involved extra steps and a slow compilation phase. Removing pandas and instead using straight python was not difficult afterwards.

The result:

![Tides]({{ site.url }}/assets/images/tides.PNG)
