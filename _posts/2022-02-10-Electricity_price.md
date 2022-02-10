---
layout: post
title:  "Automate Electricity prices in Home Assistant"
excerpt: "Automate Electricity prices in Home Assistant"
date:   2022-02-10 22:15:00
---

I've been making some use of [Energy Management](https://www.home-assistant.io/blog/2021/08/04/home-energy-management/) in Home Assistant. I have several Samsung smart plugs [reporting](https://rianoc.github.io/2021/08/25/Samsung-Engery/) energy information.

My electricity tariff breaks down to four time windows:

```txt
Day rate   08:00 - 17:00 : 33c/KWh
Peak rate  17:00 – 19:00 : 35c/KWh
Day rate   19:00 - 23:00 : 33c/KWh
Night rate 23:00 - 08:00 : 25c/KWh
```

Tracking price using an entity is supported. I chose to write an automation which makes use of [MQTT Discovery](https://www.home-assistant.io/docs/mqtt/discovery/) to present the price. It updates hourly:

```yaml
alias: Electricity Price
description: ''
trigger:
  - platform: time_pattern
    minutes: '1'
condition: []
action:
  - service: mqtt.publish
    data:
      topic: homeassistant/sensor/electricityPrice/config
      payload: >-
        { "name":"electricityPrice",  "unique_id":"electricityPrice", 
        "state_topic":"custom/electricityPrice", 
        "unit_of_measurement":"EUR/kWh",  "value_template": "{{"{{
        value_json.rate}}" }}"}
      retain: true
  - service: mqtt.publish
    data:
      topic: homeassistant/sensor/electricityTariff/config
      payload: >-
        { "name":"electricityTariff",  "unique_id":"electricityTariff", 
        "state_topic":"custom/electricityPrice",  "value_template": "{{"{{
        value_json.tariff}}" }}"}
      retain: true
  - choose:
      - conditions:
          - condition: time
            after: '08:00:00'
            before: '17:00:00'
        sequence:
          - service: mqtt.publish
            data:
              topic: custom/electricityPrice
              payload: '{"tariff":"day","rate":0.33}'
      - conditions:
          - condition: time
            after: '17:00:00'
            before: '19:00:00'
        sequence:
          - service: mqtt.publish
            data:
              topic: custom/electricityPrice
              payload: '{"tariff":"peak","rate":0.35}'
      - conditions:
          - condition: time
            after: '19:00:00'
            before: '23:00:00'
        sequence:
          - service: mqtt.publish
            data:
              topic: custom/electricityPrice
              payload: '{"tariff":"day","rate":0.33}'
      - conditions:
          - condition: time
            after: '23:00:00'
            before: '08:00:00'
        sequence:
          - service: mqtt.publish
            data:
              topic: custom/electricityPrice
              payload: '{"tariff":"night","rate":0.25}'
    default: []
mode: single
```
