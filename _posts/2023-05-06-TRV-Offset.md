---
layout: post
title:  "Automating TRV temperature offsets"
excerpt: "Automating TRV temperature offsets"
date:   2023-05-06 15:25:00
---

When we moved in to our house we discovered that the kitchen extension was a much colder room than the rest of the house. Unfortunately the heating only had a single zone which made it difficult to get a balance of heat across the house.

Most radiators in the house came with Thermostatic Radiator Valves (TRVs) attached which is theory allow per room control. I have found in most cases these are nearly useless, I don't think I've ever seen anyone have them on anything other than the max setting. THen even if you do try to use them they have a fatal flaw, they measure temperature in only one place which is right beside the heat source. This means in many cases they close too early leaving rooms cold.

To try to improve the situation we had a plumber add a TRV valve to the 2 remaining radiators in the house which oddly had been excluded from the addition of a TRV. We did not add them to the 2 bathroom radiators as it is recommended against for a few reasons.

I wanted smart TRVs with more control and the option to use an external temperature sensor for accurate operation. I chose the [Moes BRT-100](https://www.zigbee2mqtt.io/devices/BRT-100-TRV.html) models due to their looks, price, and Zigbee2MQTT compatibility. Installing was very easy, I unscrewed the old TRV heads and popped on the BRT-100s in their place.

They took some getting use to but with Home Assistant and the information gathered the needed tweaks were possible. The remaining issue was controlling the offset. I tried a few blueprints but had no success.

My solution was to create a sensor for each TRV in my `configuration.yaml`:

{% raw %}
```yaml
  - sensor:
      - name: Living Room TRV Offset
        state_class: measurement
        unit_of_measurement: °C
        device_class: temperature
        state: >
          {% set TRV_name  = 'climate.living_room_trv' %}
          {% set TRV_temp  = state_attr(TRV_name,'local_temperature') | float(default=0) %}
          {% set TRV_offset  = state_attr(TRV_name,'local_temperature_calibration') | float(default=0) %}
          {% set room_temp  = states('sensor.living_room_temperature_temperature') | float(default=0) %}
          {{ ([-9,(room_temp - (TRV_temp-TRV_offset)) |int,9]|sort)[1] }}
```
{% endraw %}

Then I created an automation for each to trigger the update of offset to be sent to the TRV:

{% raw %}
```yaml
alias: "Heating: TRV Living Room Offset"
description: ""
trigger:
  - platform: state
    entity_id:
      - sensor.living_room_trv_offset
condition: []
action:
  - service: number.set_value
    data:
      value: "{{ states('sensor.living_room_trv_offset')}}"
    target:
      entity_id: number.living_room_trv_local_temperature_calibration
mode: single
```
{% endraw %}

This has worked very well and means the rooms heat to the correct target temperature.