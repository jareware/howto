# Logging values from a RuuviTag

To continuously read values from a [RuuviTag](https://tag.ruuvi.com/), place the following script onto your PATH as `read-lines-from-ruuvi`:

```py
#!/usr/bin/python3

# REQUIRED: pip3 install --user ruuvitag-sensor
# SEE: https://docs.influxdata.com/influxdb/v1.4/write_protocols/line_protocol_tutorial/
# EXAMPLE: read-lines-from-ruuvi | sed --unbuffered 's/ /,name=kitchen /' | send-lines-to-influx

import sys
import time
from ruuvitag_sensor.ruuvi import RuuviTagSensor

MEASUREMENT = "ruuvitag"
TAG_MAC = "mac"
READ_EVERY = 30 # seconds

def convert_to_influx(mac, payload):
    line = []
    for key, value in payload.items():
        line.append(key + "=" + str(value))
    return MEASUREMENT + "," + TAG_MAC + "=" + mac + " " + ",".join(line)

while True:
    datas = RuuviTagSensor.get_data_for_sensors()
    lines = [convert_to_influx(mac, payload) for mac, payload in datas.items()]
    for line in lines:
        print(line)
    sys.stdout.flush()
    time.sleep(READ_EVERY)

```

Running this will keep printing lines like this:

```
ruuvitag,mac=FC:E4:33:8F:C7:24 pressure=1009.63,acceleration=1028.0155640845132,acceleration_y=-4,temperature=25.12,battery=2815,acceleration_x=-4,humidity=29.0,acceleration_z=1028
ruuvitag,mac=FF:A7:6F:D4:A1:B0 pressure=1009.61,acceleration=1024.9058493344644,acceleration_y=-40,temperature=24.91,battery=3175,acceleration_x=-16,humidity=28.0,acceleration_z=1024
```

To keep a logfile of the values instead of sending them to stdout, you can simply:

```
$ read-lines-from-ruuvi >> my-tags.log
```

Or to prefix each line with a timestamp (make sure you've `apt-get install moreutils`):

```
$ read-lines-from-ruuvi | ts >> my-tags.log
```

If you happen to have an InfluxDB instance running, the following script can be used to send [Line Protocol](https://docs.influxdata.com/influxdb/v1.4/write_protocols/line_protocol_tutorial/) data to it from stdin:

```sh
#!/bin/bash

# SEE: https://docs.influxdata.com/influxdb/v1.4/write_protocols/line_protocol_tutorial/

HOST=https://db.example.com
DB=ruuvi
USER=agent
PASS=SECRET

while read line
do
  echo "$(basename "$0"): $line"
  curl -XPOST --user "$USER:$PASS" --silent --show-error "$HOST/write?db=$DB" --data-binary "$line"
done < "${1:-/dev/stdin}"
```

Then, a simple:

```
$ read-lines-from-ruuvi | send-lines-to-influx
```

will keep reading the RuuviTag data and sending it to InfluxDB.

Note that the same `send-lines-to-influx` utility can be used to send arbitrary Line Protocol formatted data to your InfluxDB, if you happen to have other interesting stats to collect on the same host:

```
$ echo random_values value=$RANDOM | send-lines-to-influx
send-lines-to-influx: random_values value=32542
```
