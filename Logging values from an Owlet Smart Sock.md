# Logging values from an Owlet Smart Sock

The [Owlet Smart Sock](https://owletbabycare.co.uk/) monitors a baby's heart rate, oxygen saturation levels and movement, and reports them on their app. But the app only shows instantaneous values, and of course seeing longer-term trends is interesting. Owlet offers [an additional app for that](https://blog.owletcare.com/start-connected-care/), but it's (at the time of writing) US-only, and has per-month subscription pricing. The device itself is pretty expensive, and people should have the right to their own data, so let's pull it out.

## Reading values

Luckily, [a kind soul on GitHub](https://github.com/mbevand/owlet_monitor) has already solved the hard part: pulling values in real-time from Owlet's API.

```bash
$ OWLET_USER=xxxxxxxxxx OWLET_PASS=xxxxxxxxxx ./owlet_monitor
Logging in
Auth token xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Getting DSN
Found Owlet monitor device serial number xxxxxxxxxx
1551451163, 145, 95, still
Status: 1551451163, 145, 95, still
1551451174, 142, 94, wiggling
Status: 1551451174, 142, 94, wiggling
^C
```

## Writing values to InfluxDB

To make the values more useful, you can ship them to InfluxDB & look at them with Grafana. Place the following script onto your PATH (for example as `~/bin/read-lines-from-owlet`):

```bash
#!/bin/bash

# REQUIRED: git clone https://github.com/mbevand/owlet_monitor.git
# REQUIRED: sudo pip3 install requests
# SEE: https://github.com/mbevand/owlet_monitor
# EXAMPLE: ./read-lines-from-owlet | ./send-lines-to-influx

OWLET_BIN=/home/pi/owlet_monitor/owlet_monitor

export OWLET_USER=xxxxxxxxxx
export OWLET_PASS=xxxxxxxxxx

"$OWLET_BIN" 2> /dev/null \
  | sed -E --unbuffered 's/(.+), (.+), (.+), (.+)/owlet heart_rate=\2,oxygen_level=\3,movement="\4"/' \
  | sed -E --unbuffered 's/movement="wiggling"/is_moving=true/' \
  | sed -E --unbuffered 's/movement="still"/is_moving=false/'
```

Running it will give you lines in the InfluxDB [Line Protocol](https://docs.influxdata.com/influxdb/v1.4/write_protocols/line_protocol_tutorial/), such as:

```
owlet heart_rate=150,oxygen_level=99,is_moving=false
owlet heart_rate=150,oxygen_level=98,is_moving=true
owlet heart_rate=150,oxygen_level=98,is_moving=true
```

Then, set up a [`send-lines-to-influx`](https://github.com/jareware/howto/blob/master/Logging%20values%20from%20a%20RuuviTag.md#writing-values-to-influxdb) script, and use it like:

```bash
$ read-lines-from-owlet | send-lines-to-influx
```

## Running continuously

See [this gist](https://github.com/jareware/howto/blob/master/Logging%20values%20from%20a%20RuuviTag.md#surviving-reboots) for ideas on how to set this up so it runs continuously, and survives reboots.
