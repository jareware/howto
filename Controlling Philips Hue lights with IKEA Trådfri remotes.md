# Controlling Philips Hue lights with IKEA Trådfri remotes

There's various descriptions of this floating around, mostly in videos of varying quality. Given that the process is not entirely straightforward, thought I'd write it down.

## Use case

You have:

* Philips Hue Bridge (the new, rounded-square kind)
* Philips Hue bulb (suppose the process is the same for all kinds)
* IKEA Trådfri remote (the round, 5-button kind)

## Process

1. Install the official Hue app on iOS or Android
1. Add your bulb to the Hue app (I had to enter the serial number of the bulb), and assign it to a room
1. Test to see that you can control the bulb from the app
1. To ensure the correct device reacts to the pairing, physically power off the bulb you just added
1. Install the Hue Essentials app on Android (it seems an iOS version isn't available at the time of writing, so hope you have some old and busted Android phone laying around!)
1. Reset the Trådfri remote by pressing its internal pairing button 4 times
1. The remote's LED starts pulsating; you may not actually have to, but I always waited until it stopped pulsating before proceeding
1. In the Hue Essentials app, go Lights ➜ TODO TODO TODO
1. Move the remote very close to the Hue Bridge, and start holding down its internal pairing button
1. In the Hue Essentials app, press the Touchlink button
1. You should see the LED on the remote come on, then off (there's no other confirmation as to whether the process worked or not)
1. Physically power your bulb back on
1. Hold the remote close to the bulb, and start holding down its internal pairing button
1. The bulb should confirm a successful pairing by blinking 7 (or 10 `¯\_(ツ)_/¯`) times
1. Test to see that you can control the bulb from the remote, and also still from the app

## Gotchas

* It would have seemed the Android app part could have been avoided by using the physical button on the Hue Bridge, but this never seemed to work
* This process can be repeated for multiple bulbs, though it may be a good idea to physically power off the other bulbs while you do, lest the wrong things get paired (I had this happen and had to start over)
