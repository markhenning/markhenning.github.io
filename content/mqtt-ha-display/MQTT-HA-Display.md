---
title: Exporting with MQTT
draft: true
---

The code for this with asyncs and callbacks can look a lot scarier than it is.

Before you start, make sure you've got some [Exporting Data to Use](<Exporting with MQTT.md>)

## Technical Overview
## Overview of the Process

## Handling the MQTT Data

Before looking at what we do with the MQTT Data, let's start with how it gets into the program.

This is all achieved through "callbacks"

When the display starts up, it connects to wifi, finds the MQTT server and will subscribe to the MQTT Topic specified in connectivity.py

There some other startup stuff (we'll deal with that later), but it will then wait for the MQTT server to have new message in the topic.

It then delivers the message and topic to the "callback" function.

Which we then decode() into something we can deal with, and we'll have two text strings that we can compare/use the values of to call the relevant functions, e.g. we look for "tdns_" in the string topic and know to send that to the function to handle a DNS stat update.

### Processing the Data

There's two types of handler we use for the data here:
1. Processes we start that loop when the program starts, and we adjust the data for the loop to adjust to(e.g the clock and DNS blinks)
2. Processes that we trigger once that cause a section of the screen to update directly (e.g the power distribution)

Looking at a better example of each type:

### DNS Blinks

#### Start and Setup:
When the display starts up for the first time, it creates a 10x11 array holding "Dots". 

Dots have a brightness level (0-5) and a colour based on what they're displaying - read from settings, by  default "query" - blue, "ad block" - yellow, "error" - red )

There a values for max and min of each dot  type, these are copied into the "desired" array, which will hold values for how many of each dot type we want to display.

Next,  Dots are chosen at random in the array and the "desired" of each Dot type is created (initially 20 blue, 0 yellow, 0 red), with a brightness of 5.

The "blanks_queue" is then calculated, a list of all 0 brightness dots on the display.

Finally, with the initial display state created, it's rendered on the device.

### The Display Loop

It process then starts to loop (update_dotgrid_display) and roughly:
* Scan through the array
* For each active Dot, reduce it's brightness.
* If it's brightness moved to zero:
	* Put an entry for it in the "blanks_queue"
	* Check if the current number of that colour dot after removal is less than "desired" number for that colour. 
		* If it is, add a Dot at a random location from the blanks queue for that colour back in, brightness max
		* If it's not, let it go, that way we gracefully adjust the amount of dots on the display
		* Both of these checks also take into account min and max for each colour type, so we never have a blank or completely full display.
* Next, check if "desired" is more than the current count for each colour, add Dots at random locations to make desired match the count.
* Finally, paint the new dotgrid to the display
* Sleep for 0.4 seconds, and go again

As you can see, the loop is self sustaining, and self adjusting, always within min and max, and we just need to tweak the "desired" value for each colour, and the loop will pick it up for display. It also allows for a nice "burst" effect should the number rise dramatically.

## Updating the Data

As mentioned above, the MQTT server data gets to the program through a callback which then calls "handle_dns" with the new number of queries etc.

At that point, all we have to do is scale the number appropriately. E.g, I normally get 100-700 DNS requests a minute, so I just have number of dots = dns queries from mqtt / 10 as the number of dots.

There's "dns_scale_factors" in settings which holds the scaling number for easy tweaking.

### Power Draw Data

This is probably the most bespoke part. In terms of processing the data, there's two main topics we need: total power and individual device usage.

Home Assistant puts both of these in an MQTT message with topics "power" and "energy". These get used to call "handle_energy", it then looks at the data, and does one of two things:
* If the string topic contains "current_energy", it's my providers total power stat, which arrive roughly every 15 seconds. This triggers a display update.
* If not, it's a device power usage, so it gets stored in "energy_stats" in a key-value pair for device and energy, and the function ends.

When the "current_energy" turns up and the display needs re-drawing, the program just sorts the "energy_stats" and works out appropriate bar lengths for the new stats.

Then there's some padding/"window" movement that essentially makes an array of 10 strings (one per line), adds the new stats, some padding, and then the old stats, so we've got a line of 30 pixels, with a window of the last 10 pixels on display.

Then the display animation triggers, and we move the window "backwards", so what starts as chars 21-30 on display moves to 20-29, waits for 0.1 seconds then shows 19-28 etc, which makes the animation.

Once it's done, it works out the power bar color (green/yellow/red) from the settings.power_map dict and the function ends.















