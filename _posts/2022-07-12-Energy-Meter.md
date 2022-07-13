---
layout: post
title:  "Whole House Energy Monitor"
excerpt: "Whole House Energy Monitor"
date:   2022-07-12 19:20:00
---

I previously tracked specific item electricity usage using smart plugs but wanted a whole home solution. I bought a device from [Frient](https://frient.com/products/electricity-meter-interface/) which would be supported over [Zigbee](https://www.zigbee2mqtt.io/devices/ZHEMI101.html).

Installing it was easy and so was adding to the Zigbee network. It appeared in HA (Home Assistant) instantly.

My meter is a [Kamstrup DK-8660](https://www.kamstrup.com/en-en/electricity-solutions/meters-devices/meters/omnipower-singlephase-meter) which has a diode which flashes 1,000 times per kWh which matches the default on the Frient reader so no configuration changes were needed.

![Electricity Meter]({{ site.url }}/assets/images/meter.jpg)

Unfortunately now in HA while I can see my home homes power usage it is harder to see the device by device breakdown. This is because I cannot easily tell HA that the device energy usage is a subset of the whole home energy usage.

![Electricity Graph]({{ site.url }}/assets/images/electricityGraph.jpg)

I was able to work around this somewhat for the current W usage but not for the kWh usage as it is an accumulating sensor the calculation will be more complicated.

My W method uses a template to subtract the individual devices from the whole home usage. I choose the max of 0 and the calulated value at the end to prevent negative readings when the smart plugs register usage before the Frient reads the usage.

{% raw %}
```yaml

template:
  - sensor:
      - name: "Other Power"
        unit_of_measurement: "W"
        state: >
          {% set dishwasher = states('sensor.dishwasher_power') | float(default=0) %}
          {% set dehumidifier = states('sensor.dehumidifier_power') | float(default=0) %}
          {% set server = states('sensor.server_power') | float(default=0) %}
          {% set washing_machine = states('sensor.washing_machine_power') | float(default=0) %}
          {% set meter = states('sensor.electricity_meter_power') | float(default=0) %}
          {{ max(0,(meter - (dishwasher + server + washing_machine + dehumidifier))) }}
```
{% endraw %}
