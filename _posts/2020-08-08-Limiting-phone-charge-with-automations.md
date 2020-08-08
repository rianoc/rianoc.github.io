---
layout: post
title:  "Limiting phone charge using Home Assistant Automations"
excerpt: "Limiting phone charge using Home Assistant Automations"
date:   2020-08-08 10:30:00
---

In a previous [post](https://rianoc.github.io/2020/07/21/Limiting-phone-charge-with-Node-RED/) I outlined how I used Node-RED to automate my phone to stop charging at 80% to protect it's battery.

Since then I have implemented the same task using Home Assistant Automations out of curiosity. I was able to configure it all from the [UI editor](https://www.home-assistant.io/docs/automation/editor/). I needed to create 2 separate automations to complete the task, one to stop charging at 80% and one to start charging at 60% or lower. If I edited the `yaml` code directly I would be able to complete in one automation but for something this simple I didn't see the need.

Below are the auto generated entries added to `automations.yaml` by the UI:

```yaml
- id: '1596807'
  alias: Phone Charger - On
  description: ''
  trigger:
  - above: '0'
    below: '60'
    entity_id: sensor.sm_g970f_battery_level
    for: 00:00:01
    platform: numeric_state
  condition: []
  action:
  - data: {}
    entity_id: switch.sonoff_100065d446
    service: switch.turn_on
  mode: single
```

```yaml  
- id: '1596808'
  alias: Phone Charger - Off
  description: ''
  trigger:
  - above: '80'
    below: '100'
    entity_id: sensor.sm_g970f_battery_level
    for: 00:00:01
    platform: numeric_state
  condition: []
  action:
  - data: {}
    entity_id: switch.sonoff_100065d446
    service: switch.turn_off
  mode: single
```
