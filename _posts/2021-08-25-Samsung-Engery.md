---
layout: post
title:  "SmartThings plugs in Home Assistant energy"
excerpt: "Enabling SmartThings plugs on the new Home Assistant energy dashboard"
date:   2021-08-25 22:00:00
---

I wanted to get some smartplugs with built in energy monitoring, specifically Zigbee models.
There are not many but the Samsung SmartThings models are suitable and [supported](https://www.zigbee2mqtt.io/devices/GP-WOU019BBDWG.html) by Zigbee2MQTT.

They connect immediately to my Sonoff CC2531 [dongle](https://itead.cc/product/cc2531-usb-dongle/) and appeared in Home Assistant.

As it happened they arrived just after Home Assistant had released their [Energy Managment](https://www.home-assistant.io/blog/2021/08/04/home-energy-management/) system. When I tried to add them to the energy dashboard nothing was listed apart from a vague error "`No matching statistics found`" and a link to [documentation](https://developers.home-assistant.io/docs/core/entity/sensor/#long-term-statistics). The documentation was also not so clear to me. Eventually I saw some forum threads and was able to piece together what was needed.

I edited `configuration.yaml` with the following entries:

```yaml
homeassistant:
  customize:
    sensor.samsung1_energy:
      last_reset: "2021-07-30T00:00:00+00:00"
      state_class: measurement
      device_class: energy
    sensor.samsung2_energy:
      last_reset: "2021-07-30T00:00:00+00:00"
      state_class: measurement
      device_class: energy
```

Using the Visual Studio Code [add-on](https://github.com/hassio-addons/addon-vscode) is the easiest way to make these edits.

After the restart the dashboard was live:

![Energy dashboard]({{ site.url }}/assets/images/energy.JPG)

It's still basic but already has been interesting to see that `Samsung1` which has my server plugged in consistently uses 90W of energy. I can even see a small increase nightly after 3am when [Snapraid](https://www.snapraid.it/) runs a sync job which spins up all the hard drives.
