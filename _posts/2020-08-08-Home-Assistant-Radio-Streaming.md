---
layout: post
title:  "Home Assistant Radio Streaming"
excerpt: "Home Assistant Radio Streaming"
date:   2020-08-08 16:48:00
---

Historically I would listen to the radio by saying _"Ok Google - Stream RTÉ Radio One"_ and my Nest Home Mini would oblige. Recently though, TuneIn has refused stating that RTÉ One is not available in the region I am in (The UK). I initially found [Simple Radio](https://play.google.com/store/apps/details?id=com.streema.simpleradio&hl=en_GB) which has Chromecast support but when using it was forced to start the stream from my phone rather than by voice as TuneIn is the only radio app supported natively.

I started to search for ways to use Home Assistant to try and come up with a work around. I narrowed it down to three promising posts on the Home Assistant [community forum](https://community.home-assistant.io/):

* [Lovelace chromecast radio jukebox](https://community.home-assistant.io/t/lovelace-chromecast-radio-jukebox/83867)
* [Chromecast radio with station and player selection](https://community.home-assistant.io/t/chromecast-radio-with-station-and-player-selection/12732)
* [Streaming radio on hassio](https://community.home-assistant.io/t/streaming-radio-on-hassio/58619/7)

I started on trying to implement the [first](https://community.home-assistant.io/t/lovelace-chromecast-radio-jukebox/83867) link. Quickly I ran in to what I have found to be common difficulties when trying to work with Home Assistant:

1. Instructions written by a long time fan of Home Assistant, missing some non obvious pieces for a beginner like myself.
2. Instructions and code out of date, Home Assistant has evolved fast and there have been changes in the UI, config, and codebase. Making it difficult to get things up and running without modifications.
3. Debugging in Home Assistant is difficult. I have been getting a lot better at this but do find it's not obvious often how to proceed.

I moved on to the [third](https://community.home-assistant.io/t/streaming-radio-on-hassio/58619/7) link and with some lessons learnt from the first failure was able to get it working for me.

The steps then were to:

1. Create a `packages` folder inside the `config` folder.
2. Add an include instruction in the `homeassistant` section of `configuration.yaml`

    ```yaml
    homeassistant:
    customize: !include customize.yaml
    packages: !include_dir_named packages #<-- This line
    ```

3. Paste in the code from the [link](https://community.home-assistant.io/t/streaming-radio-on-hassio/58619/7) to `packages/chromecast_radio.yaml`
4. I used [Configuration Validation](https://www.home-assistant.io/getting-started/configuration/#:~:text=Do%20this%20by%20clicking%20on,Mode%E2%80%9D%20on%20your%20user%20profile.) which is found in `Configuration -> Server Controls` in the UI. This flagged that I needed to delete a line with a deprecated setting from the code: `hide_entity: True`.
5. I then added a card to my dashboard with the new entities. These were available after reloading my configuration again from the `Server Controls` tab. The card yaml is below:

    ```yaml
    type: entities
    title: Radio
    entities:
    - entity: input_select.radio_station
    - entity: input_select.chromecast_radio
    - entity: script.radio
    - entity: input_number.volume_radio
    ```

    ![Radio card]({{ site.url }}/assets/images/radio.JPG)

6. I edited `chromecast_radio.yaml` to have the correct names of my Chromecast devices. I tested on of the existing stations to prove all was working correctly.
7. I searched [radio-browser.info](http://www.radio-browser.info/) to find links for the radio stations I wanted and edited the file to include include them:

    * [RTÉ Radio 1](http://icecast2.rte.ie/radio1)
    * [RTÉ Raidió na Gaeltachta](http://icecast1.rte.ie/rnag)

8. I removed the `Listen Radio` automation as I wanted to click the `EXECUTE` button after I chose station & speakers rather than it triggering after I edited station alone.

My full [chromecast_radio.yaml]({{ site.url }}/assets/code/chromecast_radio.yaml) is available. Originally I had embedded it in a code block in this post but github pages build failed with an unknown [Liquid tag](https://docs.github.com/en/github/working-with-github-pages/troubleshooting-jekyll-build-errors-for-github-pages-sites#unknown-tag-error) error.

Now I can control radio from the UI in Home Assistant but this still requires a device with a web browser to operate. I will cover how I automated starting a stream with a single button press in my next post.
