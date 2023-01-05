---
layout: post
title:  "Scheduling Dishwasher"
excerpt: "Scheduling an older dishwasher to run at a set time"
date:   2023-01-05 21:45:00
---

Previously I wrote about masking my [washing machine]({{ site.url }}/2022/05/19/Smart-Washing-Machine/) smart. It uses a [SwitchBot Bot](https://www.switch-bot.com/pages/switchbot-bot) to physically press it's smart button at a scheduled time.

Initially for my dishwasher I had less use for scheduling. The night rate cheaper electricity started at 11pm, I would mostly be awake at that time to start it myself. The other difficulty ruling out the use of a Switchbot is that the dishwasher is an integrated model, it's buttons can only be interacted with when the door is open. Now I am on a new electricity tariff with an even cheaper rate 2am-4am and want to take advantage of this making scheduling more important.

My work around was to use the [SmartThings plug]({{ site.url }}/2021/08/25/Samsung-Energy/) I already had the dishwasher plugged in to for energy monitoring. I exploited the power outage recovery behaviour of the washing machine.

The workflow goes like this:

1. I start the dishwasher running as if I wanted it to run it immediately
2. After a few seconds I scan a Home Assistant [NFC tag](https://www.home-assistant.io/integrations/tag/) with my phone
3. The tag triggers an Automation which turns off the dishwasher at the plug
4. At 2am the Automation turns the dishwasher plug back on
5. The dishwasher thinks power has been restored after an outage and continues to complete it's cycle

I was able to create the automation easily in the UI. In YAML format it looks like:

```yaml
alias: " Dishwasher Tag 2am"
description: ""
trigger:
  - platform: tag
    tag_id: 07b43ddd-a737-41cd-9898-5d7d38e5b661
condition: []
action:
  - type: turn_off
    device_id: 1c462a1734665d968810e819324b37cd
    entity_id: switch.dishwasher
    domain: switch
  - wait_for_trigger:
      - platform: time
        at: "02:00:00"
  - type: turn_on
    device_id: 1c462a1734665d968810e819324b37cd
    entity_id: switch.dishwasher
    domain: switch
mode: single
```

Overall I'm happy with how this is turned out. The small NFC tag is placed under the sink where I need to reach for a dishwasher tablet anyway so the workflow feels natural.
I am next going to try this same approach with my washing machine. This will free up the Switchbot for use elsewhere, likely for use on a dehumidifier.
