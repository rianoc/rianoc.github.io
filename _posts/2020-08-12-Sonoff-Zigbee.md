---
layout: post
title:  "Sonoff Zigbee range"
excerpt: "Testing some of the Zigbee products from Sonoff"
date:   2020-08-12 13:09:00:00
---

I wanted to get some buttons to help with some common home automations. Sometimes asking Google Assistant or using an app is annoying or slow.

Initially I had looked into finding a way to use some very cheap Bluetooth tags which would not be range limited in my 
apartment. After looking more in to using them there didn't seem to be a tidy/reliable method to repurpose them. Instead when I saw some information on Zigbee buttons and sensors I bought a selection.

## Zigbee Button

The [SNZB-01](https://sonoff.tech/product/smart-home-security/snzb-01) button is compact and has single, double, and long press actions. I have placed them around rooms and assigned tasks using Home Assistant automations.

**Living room:**

* **Single press:** Toggle a Philips hue light + a Sonoff 26 plug with a string of LEDs plugged in to it.
* **Double press:** Toggle a Sonoff S26 plug with a pedestal fan plugged in to it.
* **Long press:** Stream RTÉ Radio on a Nest home mini. Background details can be found in a previous [post](https://rianoc.github.io/2020/08/08/Home-Assistant-Radio-Streaming/). The full automation resulted in:

```yaml
- id: '1597176597944'
  alias: RTÉ Living Room
  description: ''
  trigger:
  - device_id: 6e6ab8821a50495d89b21e31cc7add72
    discovery_id: 0x00124b001ca70e51 action_long
    domain: mqtt
    platform: device
    subtype: long
    type: action
  condition: []
  action:
  - data:
      option: Living Room
    entity_id: input_select.chromecast_radio
    service: input_select.select_option
  - data:
      option: RTE One
    entity_id: input_select.radio_station
    service: input_select.select_option
  - data:
      value: 0.6
    entity_id: input_number.volume_radio
    service: input_number.set_value
  - data: {}
    service: script.radio
  mode: single
  ```

I bought 3 so will be able to enable similar tasks in other rooms. One key one will be replacing my current "Goodnight" routine in Google Assistant. I bought a [LENOVO smart CLOCK](https://www.lenovo.com/gb/en/smart-clock/) but when I trigger the routine even from the touchscreen it responds with "Goodnight Rian", I want this to be silent. Generally I found the clock and Google Assistant lacking enough flexibility.

## Zigbee Temperature & Humidity Sensor

The [SNZB-02](https://sonoff.tech/product/smart-home-security/snzb-02) temperature & humidity sensor. It publishes values roughly every 8 minutes from what I've found. The one trick I feel they missed is that the external reset button could have also been enabled as a button as well.

## Zigbee USB Dongle

The [CC2531](https://www.itead.cc/cc2531-usb-dongle.html) USB Dongle was very quick to  integrate with Home Assistant using [Zigbeetomqtt](https://www.zigbee2mqtt.io/). The one thing I did find was my Raspberry Pi started dropping off the Wifi network after I installed it. This was due to interference between Zigbee and Wifi both on 2.4Ghz. To remedy I bought a [1M USB extension](https://www.startech.com/uk/Cables/USB-2.0/USB-2.0-Cables/3foot-Black-USB-20-Extension-Cable-A-to-A-Male-to-Female~USBEXTAA3BK) which solved the issue.

## Zigbee Bridge

The [ZBBridge](https://sonoff.tech/product/smart-home-security/zbbridge) has both Wifi and Zigbee radio which allows it to act as a stand alone bridge to bring Zigbee devices on to the network. I tested it briefly and it worked with the [eWeLink](https://sonoff.tech/ewelink) app. I am not currently using it as I do not want to actually use eWeLink. Instead I will eventually use [Tasmota](https://github.com/arendst/Tasmota) once it is no longer labelled [experimental](https://templates.blakadder.com/sonoff_ZBBridge.html). For now the USB dongle is enough.
