---
layout: post
title:  "Making a washing machine smart"
excerpt: "Making a washing machine smart with Home Assistant and Switchbot"
date:   2022-05-19 13:20:00
---

## Adding delay functionality

Our washing machine which was in our house already when we arrived is a [Bosch Classixx 1400 Express](https://www.bosch-home.co.uk/supportdetail/product/WFO2867GB/15). Unfortunately it does not have a delay function allowing us to schedule when it runs. The reason for wanting this:

* Convenience to schedule so if we are out of the house the clothes are not sitting wet waiting to be removed.
* Economy to schedule when the [electricity tariff]({{ site.url }}/2022/02/10/Electricity_price/) is cheapest.
* Efficiency to schedule when the grid is using the most renewable power sources.

I looked for smart product which could press the start button to add the delay ability to the machine. Initially I was looking for Zigbee devices. The only item I could find was a [Third Reality Smart Switch Gen3](https://www.3reality.com/online-store/Third-Reality-Smart-Switch-Gen3-Zigbee-Version-p381658008) but it was not going to be suitable for a number of reasons:

* It's design covers the existing button meaning if it fails or battery run out you need to remove it to operate the machine.
* The mechanism slides as it is made for US style light switches. This would have needed some adaptation to press the machine button.
* They only sell in the US (I am in EU)

This left me with the [SwitchBot Bot](https://www.switch-bot.com/pages/switchbot-bot) which is a Bluetooth device. I bought it and easily stuck it on to the machine.

![SwitchBot on washing machine]({{ site.url }}/assets/images/switchbot.jpg)

Then in the app a schedule can be set to press the button at a specific time.

![SwitchBot schedule]({{ site.url }}/assets/images/switchbotApp.jpg)

This works perfectly with one small annoyance. In the app if I try and set the button to press using an existing schedule it does not save correctly. Instead I find it works if I edit the schedule (I just move it by 1 minute) and save before turning it on.

I saw in the app that there is some NFC support but it requires their hub which I was not interested in buying. I was also temporarily tempted by [SwitchBot-MQTT-BLE-ESP32](https://github.com/devWaves/SwitchBot-MQTT-BLE-ESP32) on Github which would use an ESP32 to communicate with the SwitchBot over Bluetooth and talk to Home Assistant over MQTT. Ultimately I decided extra hardware using more electricity goes against my goal of being green. Using the app works fine and there is no need to overcomplicate the project.

## Monitoring power

I added [SmartThing plug]({{ site.url }}/2021/08/25/Samsung-Energy/) to the washing machine which would monitor it's power usage and extend the Zigbee network.

![Washing machine power usage]({{ site.url }}/assets/images/washingMachinePower.PNG)

## Detecting a failure

The washing machine works for 30C cycles but when trying a hotter wash we found the machine would seem to run for a long time. Sometimes it would stay on 1 minute remaining seemingly indefinitely.

Looking deeper at the Home Assistant statistics something interesting showed in the graph:

![Washing machine power usage]({{ site.url }}/assets/images/washingMaching.PNG)

The washing machine which is rated for 2000W never used above 250W. This pointed to the heating element being the possible failure. I ordered the element and replaced the old one. Using the continuity tester on my multimeter showed that the old element had failed. After installation the machine now has the correct power profile in Home Assistant.

![Washing machine power usage]({{ site.url }}/assets/images/washingMachingFixed.PNG)
