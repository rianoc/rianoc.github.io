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

Creating [serToMQTT.q](https://github.com/rianoc/EnvironmentalMonitor/blob/master/serToMQTT.q) was very straight forward. The only setting change I made from the publish [example](https://github.com/KxSystems/mqtt/blob/master/examples/producer.q) was to not print from the `msgsent` call-back by simply redefining the function to do nothing:

```q
.mqtt.msgsent:{}
```

To read from the serial port no library is needed as kdb+ natively can read from [named pipes](https://code.kx.com/q/kb/named-pipes/):

```q
ser:hopen`$":fifo://",COM
rawdata:last read0 ser;
```

## CRC16 checksum in q

I created needed operators which are not native to `q`:

```q
rs:{0b sv y xprev 0b vs x} /Right shift
ls:{0b sv neg[y] xprev 0b vs x} /Left shift
xor:{0b sv (<>/)vs[0b] each(x;y)} /XOR
land:{0b sv (.q.and). vs[0b] each(x;y)} /Logical AND
```

Creating `crc16` was then a case of matching the logic of [crc16_update](https://www.nongnu.org/avr-libc/user-manual/group__util__crc.html#ga95371c87f25b0a2497d9cba13190847f) and [calcCRC](https://github.com/rianoc/EnvironmentalMonitor/blob/044d07ffdbb8c651c4f53b3846a8949b74beffbb/EnvironmentalMonitor.ino#L58). The [over](https://code.kx.com/q/ref/over/) accumulator was used in place of `for` loops:

```q
crc16:{
 crc:0;
 {x:xor[x;y];
  {[x;y] $[(land[x;1])>0;xor[rs[x;1];40961];rs[x;1]]} over x,til 8
 } over crc,`long$x
 };
 ```
