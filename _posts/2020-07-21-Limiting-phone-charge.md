---
layout: post
title:  "Limiting phone charge using Home Assistant"
excerpt: "Limiting phone charge using Home Assistant"
date:   2020-07-21 21:20:00
---

One of the biggest annoyances I have with my Samsung [S10e](https://rianoc.github.io/2019/09/21/SamsungS10E/) is it's idle power draw. Overnight when not being used it's drains close to 30% of it battery capacity. As a result I have to leave the phone plugged in overnight. This was a habit I lost when I had my Oneplus phones. Having some concerns about the long term effects on the phones battery I thought I could use Home Assistant to help.

Items needed:
* [Home Assistant](https://www.home-assistant.io/)
* Smart plug. I used a [Sonoff S26](https://sonoff.tech/product/wifi-smart-plugs/s26) but any will work.
* Home Assistant [app](https://play.google.com/store/apps/details?id=io.homeassistant.companion.android&hl=en_GB) installed on the phone you are charging.

The app is needed because it reports your phones battery level to Home Assistant as a sensor input.

The key tool to complete the task is [Node-RED](https://nodered.org/) which makes it easy to draw up the logic needed.

![node red]({{ site.url }}/assets/images/node.JPG)

The nodes are:

1. Battery Level - a `poll state` node which checks the phones battery level every 5 minutes.
1. Check battery level - a `switch` node.
   * If the phone has less than 60% battery output `1` (On) is called.
   * If the phone has more than 80% battery output `2` (Off) is called.
1. On - a `call service` node which turns the smart plug on.
1. Off - a `call service` node which turns the smart plug off.

Now my phone will charge to 80%, then the plug will turn off. If the phone drops below 60% the plug will turn back on again.

I have a similar plan to do this for my laptop. As I am currently working from home it is constantly plugged in at 100%. To achieve the same logic I  need to investigate how to publish the battery level to Home Assistant as there is no native app like there is for a smartphone.

![node red]({{ site.url }}/assets/images/charge.JPG)

After running overnight the graph shows the automation in action:

* 22:39 - 38% - Phone put charging
* 23:15 - 85% - Automation turns off plug to stop charging
* 08:55 - 58% - Charge dropped to less than 60%, plug turned on
* 09:40 - 90% - Automation turns off plug again

This is the JSON of the flow which can be imported:

```json
[
    {
        "id": "80777ff5.e80f",
        "type": "tab",
        "label": "Control  charging",
        "disabled": false,
        "info": ""
    },
    {
        "id": "3cf91d3c.f6ae72",
        "type": "poll-state",
        "z": "80777ff5.e80f",
        "name": "Battery Level",
        "server": "81a8849.12fa778",
        "version": 1,
        "exposeToHomeAssistant": false,
        "haConfig": [
            {
                "property": "name",
                "value": ""
            },
            {
                "property": "icon",
                "value": ""
            }
        ],
        "updateinterval": "5",
        "updateIntervalUnits": "minutes",
        "outputinitially": false,
        "outputonchanged": false,
        "entity_id": "sensor.sm_g970f_battery_level",
        "state_type": "num",
        "halt_if": "",
        "halt_if_type": "str",
        "halt_if_compare": "is",
        "outputs": 1,
        "x": 160,
        "y": 200,
        "wires": [
            [
                "c6305754.9fdb18"
            ]
        ]
    },
    {
        "id": "c6305754.9fdb18",
        "type": "switch",
        "z": "80777ff5.e80f",
        "name": "Check battery level",
        "property": "payload",
        "propertyType": "msg",
        "rules": [
            {
                "t": "lt",
                "v": "60",
                "vt": "str"
            },
            {
                "t": "gt",
                "v": "80",
                "vt": "str"
            }
        ],
        "checkall": "true",
        "repair": false,
        "outputs": 2,
        "x": 410,
        "y": 200,
        "wires": [
            [
                "7be1508f.e2ddf"
            ],
            [
                "e55ee01f.6beed"
            ]
        ]
    },
    {
        "id": "7be1508f.e2ddf",
        "type": "api-call-service",
        "z": "80777ff5.e80f",
        "name": "On",
        "server": "81a8849.12fa778",
        "version": 1,
        "debugenabled": false,
        "service_domain": "switch",
        "service": "turn_on",
        "entityId": "switch.sonoff_100065c641",
        "data": "{\"entity_id\":\"switch.sonoff_100065c641\"}",
        "dataType": "json",
        "mergecontext": "",
        "output_location": "",
        "output_location_type": "none",
        "mustacheAltTags": false,
        "x": 650,
        "y": 160,
        "wires": [
            []
        ]
    },
    {
        "id": "e55ee01f.6beed",
        "type": "api-call-service",
        "z": "80777ff5.e80f",
        "name": "Off",
        "server": "81a8849.12fa778",
        "version": 1,
        "debugenabled": false,
        "service_domain": "switch",
        "service": "turn_off",
        "entityId": "switch.sonoff_100065c641",
        "data": "{\"entity_id\":\"switch.sonoff_100065c641\"}",
        "dataType": "json",
        "mergecontext": "",
        "output_location": "",
        "output_location_type": "none",
        "mustacheAltTags": false,
        "x": 650,
        "y": 220,
        "wires": [
            []
        ]
    },
    {
        "id": "81a8849.12fa778",
        "type": "server",
        "z": "",
        "name": "Home Assistant",
        "addon": true
    }
]
```
