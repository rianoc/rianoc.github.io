---
layout: post
title:  "Exposing Home Assistant to Google Assistant"
excerpt: "Exposing Home Assistant to Google Assistant"
date:   2023-01-03 22:00:00
---

I wanted to expose a device from my Home Assistant to Google Assistant. Longer term I may use the official [Nabu Casa](https://www.nabucasa.com/) @ €75 per annum but for now I wanted a lower cost option to test with.

Most guides use [DuckDNS](https://www.duckdns.org/) but I generally wanted to avoid the port forwarding it requires.
Instead the first thing I did was set up a [Cloudflare Tunnel](https://www.cloudflare.com/products/tunnel/) following this [video](https://www.youtube.com/watch?v=xXAwT9N-7Hw)/[blog](https://everythingsmarthome.co.uk/free-remote-access-for-home-assistant-with-cloudlfare/). The only change I made was to use a registrar I was already familiar with: [host.ie](https://www.host.ie/). My chosen domain came at a cost of €10 for a year.

To tie this exposed URL in to Google Assistant I followed this [video](https://www.youtube.com/watch?v=a423xLw0M3w) along with the official [documentation](https://www.home-assistant.io/integrations/google_assistant/).

This all worked correctly and made 156 devices available in Google Assistant. In reality the number of devices is much lower but to map items in Home Assistant splits many devices out in to their constituent entities. For example a lot of the devices are parts of my TRVs which in Google is mapped to Thermostats but do not actually offer me any control.

Humidity sensors values come across correctly but temperature ones do not. I'm going to look in look into if these issues and others can be worked around to get better value out of this set up going forward.

Bulbs and switches do work. I won't use these much currently as I most use [SonoffLAN](https://github.com/AlexxIT/SonoffLAN) and [Hue](https://www.home-assistant.io/integrations/hue/). However, the door is now opened for me to flash my [S26](https://tasmota.github.io/docs/devices/Sonoff-S26-Smart-Socket/) smart plugs with Tasmota so I wouldn't have to rely on the Sonoff cloud any more which would be an improvement. Moving my bulbs to the same Zigbee mesh as all my remotes would also have benefits but I am more hesitant to do this as some Hue specific features would be lost.
