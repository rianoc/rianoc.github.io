---
layout: post
title:  "Smart controlling a heat recovery unit"
excerpt: "Smart controlling a heat recovery unit using an IR blaster"
date:   2023-06-10 12:00:00
---

The kitchen in our house gets very cold. One reason was a poorly installed vent. It had a slidable open close mechanism but with a cheap warped plastic cover it made no difference.
Looking at the vent in cold weather with a thermal camera I could see that it was very cold surrounding it.
Opening it up further I saw that it's 100mm pipe was not connected end to end and instead laying in a 160mm x 130mm rectangular hole.

![ZEPHYR]({{ site.url }}/assets/images/ZEPHYR_cold.jpg)

Rather than replace it with a similar but higher quality vent I looked further into what else might be available. I then came across the [ZEPHYR Decentralized Heat Recovery Device](https://www.bskhvac.com.tr/en/product-detail/heat-recovery-units/zephyr-decentrelized-heat-recovery-device).

![ZEPHYR]({{ site.url }}/assets/images/ZEPHYR.png)

Rather than describe it myself I'll use their own description:

_"BSK Zephyr can recover heat by using a 2-way fan  heating and cooling of the ceramic heat exchanger inside. The device operates on a cycle that extracts and supplies air in periods of 70 seconds. While the air is extracted out from the room, the ceramic heat exchanger will be heated, when the air flow direction is reversed and fresh air from outside is supplied to the room, thanks to the heat stored in the ceramic core, it will be blown inside in a conditioned manner. BSK Zephyr, whose power consumption does not exceed a few watts and can reach 90% thermal efficiency, also allows you to save energy while meeting your ventilation needs. You can connect multiple devices to each other via wireless communication, and you can control your devices with the included IR remote."_

I had an electrician install it as it needs power and the hole needed to be increased to fit a 160mm pipe. When not turned on a servo retracts allowing it to seal with a magnet. Along with some foam lining it insulates well and the thermal camera shows much less cold air entering.

The device has WiFi but unfortunately it is only for creating a device to device mess. This leaves the IR remote as the only way to control the device. I bought a
[Moes IR Blaster](https://www.zigbee2mqtt.io/devices/UFO-R11.html#moes-ufo-r11) so I could control it from Home Assistant.
I used the Zigbee2MQTT UI to learn the IR codes and then created Home Assistant Scripts for each one. One thing I found strange was that the same button learned twice would give very different codes but thankfully the ZEPHYR responded correctly in all cases.

The code for the scripts:

```yaml
zephyr_power:
  alias: Zephyr power
  sequence:
  - service: mqtt.publish
    data:
      qos: 0
      retain: false
      topic: zigbee2mqtt/0xe0798dfffe05dfff/set
      payload: '{"ir_code_to_send": "CIUjkBFYAgICWGABAgICWCABAgICWCABAQIC4AEDAW0G4BUDAQICQAMFoQICAlgCwCsDAgJYAkALAwICWAJAB8ADAQICgAMBbQbgAS8Jo5uFI6oIWAL//+ACBwIIWAI="}'
  mode: single
  icon: mdi:power
zephyr_night_mode:
  alias: Zephyr night mode
  sequence:
  - service: mqtt.publish
    data:
      qos: 0
      retain: false
      topic: zigbee2mqtt/0xe0798dfffe05dfff/set
      payload: '{"ir_code_to_send": "BaQjixE5AuAXAQGJBuAVA8ABwCfAAUAPQAFAB0ADwAFAC0ADQAFABw+Mm6Qjswg5Av//pCOzCDkC"}'
  mode: single
  icon: mdi:weather-night
zephyr_intake_only:
  alias: Zephyr intake
  sequence:
  - service: mqtt.publish
    data:
      qos: 0
      retain: false
      topic: zigbee2mqtt/0xe0798dfffe05dfff/set
      payload: '{"ir_code_to_send": "C3wBfAH1IYIRaAIdAkADQAFAB8ADwAEDaAJcBuAVA8AjgAcAHWAHAR0C4AEDA1wGaAJAD+ADB0ALgAMJHQKGm/UhfwhoAg=="}'
  mode: single
  icon: mdi:bank-transfer-in
zephyr_exchange:
  alias: Zephyr exchange
  sequence:
  - service: mqtt.publish
    data:
      qos: 0
      retain: false
      topic: zigbee2mqtt/0xe0798dfffe05dfff/set
      payload: '{"ir_code_to_send": "BagjoRE+AuAXAQF9BuAZA+ADAUAv4AcBQBPAA0ABwAtABwmmm6gjugg+Av//4BoHAgg+Ag=="}'
  mode: single
  icon: mdi:bank-transfer
zephyr_exhaust:
  alias: Zephyr fan medium
  sequence:
  - service: mqtt.publish
    data:
      qos: 0
      retain: false
      topic: zigbee2mqtt/0xe0798dfffe05dfff/set
      payload: '{"ir_code_to_send": "BY4jnhE8AuAXAQGABuAVA0AB4Acj4AMBQBvgBwFAE8ADD7abjiO/CDwC//+OI78IPAI="}'
  mode: single
  icon: mdi:fan-speed-2
zephyr_fan_minimum:
  alias: Zephyr fan minimum
  sequence:
  - service: mqtt.publish
    data:
      qos: 0
      retain: false
      topic: zigbee2mqtt/0xe0798dfffe05dfff/set
      payload: '{"ir_code_to_send": "BW8jhhFKAkABARsCgAMBSgJACQFKAoAFAkoCGyABAoIGSiADwAdAC+AJAwMbAkoCQBdAB0ADQAFAB0ADA0oCGwLAG0AHgAOAEw2CBkoC5ZtvI6sISgL//+ACBwIISgI="}'
  mode: single
  icon: mdi:fan-speed-1
zephyr_fan_maximum:
  alias: Zephyr fan maximum
  sequence:
  - service: mqtt.publish
    data:
      qos: 0
      retain: false
      topic: zigbee2mqtt/0xe0798dfffe05dfff/set
      payload: '{"ir_code_to_send": "BYAjjhFDAuAXAQFuBuAdA0ABwCvgCwFAG8ABQAvAAw+fm4AjnAhDAv//gCOcCEMC"}'
  mode: single
  icon: mdi:fan-speed-3
zephyr_humidity_minimum:
  alias: Zephyr humidity minimum
  sequence:
  - service: mqtt.publish
    data:
      qos: 0
      retain: false
      topic: zigbee2mqtt/0xe0798dfffe05dfff/set
      payload: '{"ir_code_to_send": "BaAjcxFDAuAXAQFoBuAhA0ABQC/gDwHAGwJoBnsgA0AHEUMCoJugI5gIQwL//6AjmAhDAg=="}'
  mode: single
  icon: mdi:home-floor-1
zephyr_humidity_medium:
  alias: Zephyr humidity medium
  sequence:
  - service: mqtt.publish
    data:
      qos: 0
      retain: false
      topic: zigbee2mqtt/0xe0798dfffe05dfff/set
      payload: '{"ir_code_to_send": "BtYjWxFxAj3gFgEBdQbgAwMAceAMD+AFAQJxAj3gBgEBdQbgCwMAcWAXD7Sb1iOsCD0C///WI6wIPQI="}'
  mode: single
  icon: mdi:home-floor-2
zephyr_humidity_maximum:
  alias: Zephyr humidity maximum
  sequence:
  - service: mqtt.publish
    data:
      qos: 0
      retain: false
      topic: zigbee2mqtt/0xe0798dfffe05dfff/set
      payload: '{"ir_code_to_send": "BacjcRFGAuAXAQFnBuAZA0ABwCfgCwFAG8ABAmcGgCADQAeAAw+Om6cjmwhGAv//pyObCEYC"}'
  mode: single
  icon: mdi:home-floor-3
zephyr_exhaust_2:
  alias: Zephyr exhaust
  sequence:
  - service: mqtt.publish
    data:
      qos: 0
      retain: false
      topic: zigbee2mqtt/0xe0798dfffe05dfff/set
      payload: '{"ir_code_to_send": "Bn4jURF1AkDgFgECaAZ1IANAB+ADA+AFE8ABAWgGQAMCdQJA4AYBAWgGgAPAAcAPQAcPppt+I5kIQAL//34jmQhAAg=="}'
  mode: single
  icon: mdi:bank-transfer
```

On the Home Assistant dashboard I was able to make an almost exact replica of the remote. This gives the convenience of being able to quickly control the device from my phone rather than searching for the remote.

![ZEPHYR]({{ site.url }}/assets/images/ZEPHYR_remote.jpg)

![ZEPHYR]({{ site.url }}/assets/images/ZEPHYR_UI.jpg)

The code for the dashboard grid:

```yaml
square: false
columns: 3
type: grid
cards:
  - show_name: false
    show_icon: true
    type: button
    tap_action:
      action: toggle
    entity: sensor.kitchen_temperature_humidity
    show_state: true
  - show_name: true
    show_icon: true
    type: button
    tap_action:
      action: toggle
    entity: script.zephyr_power
    name: Power
  - show_name: true
    show_icon: true
    type: button
    tap_action:
      action: toggle
    entity: script.zephyr_night_mode
    name: Fan 25%
  - show_name: true
    show_icon: true
    type: button
    tap_action:
      action: toggle
    entity: script.zephyr_intake_only
    name: Intake mode
  - show_name: true
    show_icon: true
    type: button
    tap_action:
      action: toggle
    entity: script.zephyr_exchange
    name: Exchange mode
    icon: mdi:bank-transfer
  - show_name: true
    show_icon: true
    type: button
    tap_action:
      action: toggle
    entity: script.zephyr_exhaust_2
    name: Exhaust mode
    icon: mdi:bank-transfer-out
  - show_name: true
    show_icon: true
    type: button
    tap_action:
      action: toggle
    entity: script.zephyr_fan_minimum
    name: Fan 50%
  - show_name: true
    show_icon: true
    type: button
    tap_action:
      action: toggle
    entity: script.zephyr_exhaust
    name: Fan 75%
  - show_name: true
    show_icon: true
    type: button
    tap_action:
      action: toggle
    entity: script.zephyr_fan_maximum
    name: Fan 100%
  - show_name: true
    show_icon: true
    type: button
    tap_action:
      action: toggle
    entity: script.zephyr_humidity_minimum
    name: ' Humidity 40%'
  - show_name: true
    show_icon: true
    type: button
    tap_action:
      action: toggle
    entity: script.zephyr_humidity_medium
    name: Humidity 60%
  - show_name: true
    show_icon: true
    type: button
    tap_action:
      action: toggle
    entity: script.zephyr_humidity_maximum
    name: Humidity 80%
```

Overall the device has worked out very well. The main drawback being IR control which gives Home Assistant no feedback on state making automations difficult. Internally the ZEPHYR is powered by an ESP microcontroller so possibly a custom firmware could be written to allow WiFi control. Unfortunately I don't see myself doing that and as a niche product it's unlikely anyone else will either.

![ZEPHYR]({{ site.url }}/assets/images/ZEPHYR_internals.jpg)

![ZEPHYR]({{ site.url }}/assets/images/ZEPHYR_ESP.jpg)
