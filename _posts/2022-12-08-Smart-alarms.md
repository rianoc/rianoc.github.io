---
layout: post
title:  "Smart Alarms"
excerpt: "Smart Alarms"
date:   2022-12-28 20:25:00
---

# Replacing legacy alarm with Yale Sync smart alarm

My house came a legacy hardwired alarm Aritech CS350 (discontinued 2006). I didn't trust it's reliability and wanted some remote monitoring functionality.

I wated something 100% stand alone so I went with a [Yale Sync](https://yalehome.co.uk/ia-310-yale-sync-smart-home-alarm-4-piece-kit/) smart home alarm.
Generally I have been happy with it apart from needing to move one of the Smoke/Heat/PIR sensors as the PIR sensor was getting falsely triggered by its proximity to a radiator.
I was easily able to bring this in to Home Assistant using the [Yale Smart Living](https://www.home-assistant.io/integrations/yale_smart_alarm/). It relies on cloud polling but that's fine for me. All it's sensors are battery powered and it's hub has a battery backup which means the unit functions in power outage.

# Using Konnected and Alarmo

I removed the legacy alarm panel but did not remove the sensors. I wanted to see if I could get some value from them to augment the Yale alarm. For this I came across [Konnected](https://konnected.io/). I got the most basic [6 Zone](https://konnected.io/collections/shop-now/products/6-zone-konnected-alarm-panel-siren-output) board. It was $99 in Aug 22 when I bought which did feel high, now in Dec 22 it is $59 which is much more reasonable.

I bought a 12V 2A adapter on Amazon and installed the Konnected board. My distrust of the old alarm was proved valid. One contact sensor zone on a door had a large delay to close and another on windows would trip in windy weather, I will adjust this sensors to fix the issue at a  later date. These issues would have had the legacy alarm constantly triggering when armed annoying my neighbours.

To bring in to Home Assistant was very easy by using the Konnected [integration](https://www.home-assistant.io/integrations/konnected/). This made the sensors available but to treat them as an alarm needs a separate integration. [Alarmo](https://github.com/nielsfaber/alarmo) is the integration which turns the sensors in to an alarm system. Using a mixture of Alarmo and standard automations I now have:

* If Yale is triggered then Alarmo triggers.
* If Alarmo is triggered it triggers the Yale panic button.
* If either alarm is set/unset the other will match it's state.

I have also integrated a Zigbee [leak detector](https://www.zigbee2mqtt.io/devices/LS21001.html) in to Alarmo by my washing machine in case of issues. Alarmo has functionality for sensors like this were you get alerted regardless of whether the alarm is set or not.

# Using Konnected to monitor hot water tank temperatures

To get most value out of the Konnected board I looked through the docs and saw you can attach temperature or humidity probes. to it. I thought this would be useful as by chance my alarm hub is situated beside my (very small 90L) water cylinder. Unfortunately the [docs](https://help.konnected.io/support/solutions/articles/32000026315-wiring-temp-humidity-or-temperature-probe-sensors) detailed this only using an add-on board which I did not want to pay for.

Since I saw the option in the software still I thought I would try anyway. The main issue was that the board did not expose a 5V line like the add-on boards did. Luckily I had a small Sparkfun [power supply](https://www.sparkfun.com/products/13032) lying around. I wired this up to to the 12V adapter to give me a 5V source. Then I wired in my 3 2 meter long DS18B20 temperature probes with the data line going to a free zone on the Konnected Board and their +/- going to the Sparkfun board. The sensors then showed up in Home Assistant without issue.

My existing home heating system by [EPH](https://www.home-assistant.io/integrations/ephember/) has 1 temperature probe which tells me the temperature at the top of the tank only.

![EPH Graph]({{ site.url }}/assets/images/EPH Graph.jpg)

Now with the Konnected board I have 3 additional sensors top/middle/bottom. This gives me much better insight into the water temperature in the tank.

![Triple graph]({{ site.url }}/assets/images/Triple graph.jpg)

I like to use the [Gauge Card](https://www.home-assistant.io/dashboards/gauge/) as well for the current status.

![Water-guages]({{ site.url }}/assets/images/Water-guages.jpg)
