---
layout: post
title:  "Home Assistant MQTT Sensors"
excerpt: "Capturing sensor data from MQTT in Home Assistant"
date:   2020-05-16 12:05:00
---

## Adding MQTT sensors to Home Assistant

Previously I posted about the [Arduino Environmental Monitoring]({{ site.url }}/2020/04/14/Arduino/) system I made. It published it's data to MQTT.
Adding the sensors to Home Assistant was quick. Adding the following to `configuration.yaml`:

```yaml
sensor:
  platform: mqtt
  name: "Living Room Temperature"
  state_topic: "hassio/livingroom/temperature"
  qos: 0
  unit_of_measurement: "ºC"

sensor 2:
  platform: mqtt
  name: "Living Room Humidity"
  state_topic: "hassio/livingroom/humidity"
  qos: 0
  unit_of_measurement: "%"

sensor 3:
  platform: mqtt
  name: "Living Room Pressure"
  state_topic: "hassio/livingroom/pressure"
  qos: 0
  unit_of_measurement: "hPa"

sensor 4:
  platform: mqtt
  name: "Living Room Light"
  state_topic: "hassio/livingroom/light"
  qos: 0
  unit_of_measurement: "/1024"
  ```

## Adding sensors to Lovelace Dashboard

Then to add to a Lovelace dashboard:

```yaml
entities:
  - entity: sensor.living_room_temperature
  - entity: sensor.living_room_humidity
  - entity: sensor.living_room_pressure
  - entity: sensor.living_room_light
show_icon: true
show_name: false
show_state: true
title: Living Room
type: glance
```

The result looked well:

![Living Room sensors]({{ site.url }}/assets/images/LivingRoom.JPG)

However when I clicked in to the graph I could see some wild readings:

![Incorrect temp readings]({{ site.url }}/assets/images/bug.PNG)

## Adding checksum to serial data

The bad readings were due to the unreliability of the serial data. A missing decimal place would cause `22.30` to become `2230`.

I updated the project to send a `crc16` checksum.
The Arduino environment had a utility for this:

```ino
#include <util/crc16.h>

uint16_t calcCRC(char* str)
{
  uint16_t crc=0; // starting value as you like, must be the same before each calculation
  for (int i=0;i<strlen(str);i++) // for each character in the string
  {
    crc= _crc16_update (crc, str[i]); // update the crc value
  }
  return crc;
}
```

I took my readings `serString` and added the `crc16` checksum to the end:

```ino
 int str_len = serString.length() + 1;
 char char_array[str_len];
 serString.toCharArray(char_array, str_len);
 Serial.println(serString + "," + String(calcCRC(char_array)));
```

On the python side I then would verify the checksum before accepting the data. I used the [crcmod](https://pypi.org/project/crcmod/) module for this. This is installed with `pip`:

```bash
pip install crcmod
```

Then imported and told to use `crc16`:

```python
import crcmod.predefined
crc16 = crcmod.predefined.mkCrcFun('crc-16')
```

Finally each Arduino checksum is verified against one created in python:

```python
 pyCRC = hex(crc16(rawdata[0:rawdata.rfind(',')].encode('utf-8')))
 data = rawdata.split(',')
 arduinoCRC = hex(int(data[-1]))
 if not pyCRC == arduinoCRC:
    raise Exception("Failed checksum check")
```

I can then see in the process logs bad data being correctly rejected. Here `23.10` for temperature is missing the `2` and becomes `3.10`:

```txt
Error with data
3.10,37,704,1021,-69.04,42153
Failed checksum check
```

The graphed data now looks much better:

![0 temp readings]({{ site.url }}/assets/images/0drop.JPG)

## Checksum != Holy Grail

You will see however there was still an issue with bad readings. Rare, roughly once a day with the value always being `0`. I tracked this down to the serial data arriving as `"0"`. I've commented through the code to show how this was able to slip through:

```python
#rawdata = "0"
#No ',' is found, the empty string "" is passed to crc16 which returns 0x00
pyCRC = hex(crc16(rawdata[0:rawdata.rfind(',')].encode('utf-8')))
#Split results in ['0']
data = rawdata.split(',')
#arduinoCRC defined as 0x00
arduinoCRC = hex(int(data[-1]))
#No exception rasied 0x00 == 0x00
if not pyCRC == arduinoCRC:
  raise Exception("Failed checksum check")
#slice notation does not error out of bounds. data now equals [0.0]
data = list(map(float, data[0:4]))
#temperature incorrectly published as 0.0
client.publish("hassio/"+room+"/temperature",data[0])
#this will raise an IndexError: list index out of range
client.publish("hassio/"+room+"/humidity",data[1])
```

This passed the checksum test as `0x00 == 0x00`. This is a good example of relying too much on the checksum calculation. Changing the project from the ground up to send fixed width encoded data would be the robust method to add. In this case though I want to keep the data human readable so I resolved by adding back in one additional validity check to cover this edge case:

```python
data = rawdata.split(',')
if not 6 == len(data):
  raise Exception("Incomplete message")
```
