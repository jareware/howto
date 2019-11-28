# Logging values from a RuuviTag

## Reading values

To continuously read values from a [RuuviTag](https://tag.ruuvi.com/), place the following script onto your PATH (for example as `~/bin/read-lines-from-ruuvi`):

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

## Writing values

To keep a logfile of the values instead of sending them to stdout, you can simply:

```
$ read-lines-from-ruuvi >> my-tags.log
```

Or to prefix each line with a timestamp (make sure you've `apt-get install moreutils`):

```
$ read-lines-from-ruuvi | ts >> my-tags.log
```

## Writing values to InfluxDB

If you happen to have an InfluxDB instance running, the following script can be used to send [Line Protocol](https://docs.influxdata.com/influxdb/v1.4/write_protocols/line_protocol_tutorial/) data to it from stdin (save as e.g. `~/bin/send-lines-to-influx`):

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

## Surviving reboots

To make sure a random reboot of the Pi won't stop data collection, create a script (e.g. `~/bin/ruuvi2influx`), with:

```sh
#!/bin/bash

PATH="/home/pi/bin:$PATH"

read-lines-from-ruuvi | send-lines-to-influx
```

On Raspbian (and similar flavors), to make sure this command runs after each boot, add the following to your `/etc/rc.local`:

```sh
# Drop privileges and start collecting data
sudo -u pi /home/pi/bin/ruuvi2influx # can't detach with "&" due to https://github.com/ttu/ruuvitag-sensor/issues/32
```

## Automating reboots

If you leave a Pi (especially a tiny one like the Zero) running for long enough, it may grind to a halt on its own (e.g. by swapping). To reboot once per week, `sudo crontab -e` and:

```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                       7 is also Sunday on some systems)
# │ │ │ │ │
# │ │ │ │ │
# * * * * *  command to execute
  0 3 * * 0  /sbin/shutdown -r now
```

## Tagging values

By default, the `read-lines-from-ruuvi` script only tags the measurements with the MAC of the RuuviTag. To turn that into more useful tags, you can change the `ruuvi2influx` script to something like:

```sh
#!/bin/bash

PATH="/home/pi/bin:$PATH"

read-lines-from-ruuvi \
  | sed -u 's/\(CD.D3.*\) /\1,location=Balcony /g' \
  | sed -u 's/\(E2.F9.*\) /\1,location=Bathroom /g' \
  | sed -u 's/\(FC.C4.*\) /\1,location=Fridge /g' \
  | send-lines-to-influx
```

You can use more (or less) of the MAC length to identify your tags, as long as they don't collide.

Remember to update the script if you relocate the tags. :)

## Removing erroneous values from InfluxDB

Sometimes you end up with invalid data in InfluxDB; for example, an error in some script caused garbage data to be written, which you want to get rid of.

To remove values from the DB, first check the version of InfluxDB you're running (e.g. with `docker ps`, or whatever you use to run it) and use that to match your CLI version. Note that if your DB is accessible over the internet, you can run the following commands on any host, not just the one running your InfluxDB.

```terminal
$ read password # read the password into a variable, so it doesn't stick to your shell history
SECRET
$ docker run --rm -it influxdb:1.2 influx -host db.example.com -ssl -port 443 -username agent -password "$password" -precision rfc3339 # the "precision" gives you nicer formatting for timestamps
Connected to https://db.example.com:443 version 1.2.2
InfluxDB shell version: 1.2.4
> SHOW DATABASES
name: databases
name
----
_internal
ruuvi
telegraf
> USE ruuvi
Using database ruuvi
> SHOW MEASUREMENTS
name: measurements
name
----
ruuvitag
```

You can then use queries like:

* `SHOW SERIES` to see what kind of measurement/tag-combinations the DB has (in case you're interested)
* `SELECT * FROM ruuvitag WHERE time >= now() - 1m` to see what the most recent RuuviTag data looks like
* `SELECT * FROM ruuvitag WHERE time >= '2019-07-23T18:31:21Z' AND time <= '2019-07-27T15:00:00Z'` to narrow down the data range you want to get rid of
* `DELETE FROM ruuvitag WHERE time >= '2019-07-23T21:13:36.589362432Z' AND time <= '2019-07-27T11:58:27.331387442Z'` to perform the actual deletion
