---
layout: post
title:  "Kdb+ MQTT Serial Publisher"
excerpt: "Publishing serial data to MQTT with Kdb+"
date:   2020-05-16 12:35:00
---

My [Arduino Environmental Monitoring]({{ site.url }}/2020/04/14/Arduino/) system publishes it's sensor readings to [Home Assistant MQTT Sensors]({{ site.url }}/2020/05/16/Home-Assistant-MQTT-Sensors/) using python.

Last week a new [MQTT interface](https://code.kx.com/q/interfaces/mqtt/) for kdb+ was released. I decided to replicate my python publisher in q.

## Install and setup

Setup was quick using the Github [notes](https://github.com/KxSystems/mqtt#installation). My notes are for a 64bit x86 Linux install (on my [Chromebox]({{ site.url }}/2020/04/19/Linux-Chromebox/)).

First grab the latest release of [paho.mqtt.c](https://github.com/eclipse/paho.mqtt.c/releases):

```bash
cd
wget https://github.com/eclipse/paho.mqtt.c/releases/download/v1.3.2/Eclipse-Paho-MQTT-C-1.3.2-Linux.tar.gz
tar -xzf Eclipse-Paho-MQTT-C-1.3.2-Linux.tar.gz
```

Then make sure you have all needed environment variables in your `.bashrc`:

```bash
export QHOME=/home/rian/q
export PATH="$PATH:/home/rian/q/l64"
export PAHO_HOME="/home/rian/Eclipse-Paho-MQTT-C-1.3.2-Linux"
export LD_LIBRARY_PATH="$PAHO_HOME/lib/:$LD_LIBRARY_PATH"
```

Finally download the latest release of [mqtt](https://github.com/KxSystems/mqtt/releases) and install it:

```bash
cd
wget https://github.com/KxSystems/mqtt/releases/download/1.0.0-rc/mqtt-linux-1.0.0-rc.tar.gz
tar -xzf mqtt-linux-1.0.0-rc.tar.gz
cd mqtt
chmod +x install.sh && ./install.sh
```

## Publishing serial data to MQTT

Creating [serToMQTT.q](https://github.com/rianoc/Arduino/blob/master/EnvironmentalMonitor/serToMQTT.q) was very straight forward. The only setting change I made from the publish [example](https://github.com/KxSystems/mqtt/blob/master/examples/producer.q) was to not print from the `msgsent` call-back by simply redefining the function to do nothing:

```q
.mqtt.msgsent:{}
```

To read from the serial port no library is needed as kdb+ natively can read from [named pipes](https://code.kx.com/q/kb/named-pipes/):

```q
ser:hopen`$":fifo://",COM
rawdata:last read0 ser;
```

I currently only have a stub for a `crc16` checksum function and have commented out the check:

```q
crc16:{}

pyCRC:crc16 #[;x] 1+last where x=",";
data:"," vs x;
arduinoCRC:"I"$last data;
/if[not pyCRC=arduinoCRC;'"Failed checksum check"];
```

I will be writing that soon and will update here once complete.
