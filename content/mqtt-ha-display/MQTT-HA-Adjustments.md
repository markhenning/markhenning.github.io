---
title: MQTT-HA-Display-Adjustments
draft: true
---
There's quite a few bespoke bits in there that depend on the data you're exporting.

This page shows the key variables, text strings etc and where to find them.

## Clock Hour

Micropython NTP doesn't really do timezones and daylight savings without a massive amount of extra work.

During operation, you can hold the A button on the rear of the  display to advance the clock by an hour for every second.

Also, for a more permanent fix, in the clock module, the following variable useful to change:

| File          | Variable      |
| ------------- | ------------- |
| display_clock | hr_offset = 1 |

## MQTT Topics and Strings


This is going to be the fiddly part, depending on how HA writes them out/what your topic/messages are, it's likely that indivdual setups will have different topic names for each value

In order to watch for these, you can easily adjust the strings to watch for for various purposes

These all live in **settings.py**, and fall into two categories
### 1. Main Message Topic Strings

These are lists of strings, if any of the words appear in the message topic, the relevant handler is called:

| List          | Defaults          | Notes                      |
| ------------- | ----------------- | -------------------------- |
| topic_power   | 'power', 'energy' | Triggers handle_energy()   |
| topic_network | 'router_current'  | Triggers handle_network(), |
| topic_dns     | 'tdns'            | Triggers handle_dns(),     |
### 2. Individual Handler Strings

These are used inside each handler to further classify data and use it appropriately. 

E.g "router_current_upload" will trigger handle_network(), the "upload" will be used to mark it as the upload data.

Again, in settings.py, the relevant entries are:
### Power String/References

|         | Variable             | Default          | Notes |
| ------- | -------------------- | ---------------- | ----- |
| Power   | topic_power_total    | "current_demand" |       |
| Network | topic_network_upload | "upload"         |       |
|         | topic_network_upload | "download"       |       |

## Changing the Colours

In order to change the colour of something, we need to do two things:
1. Add the new colour to the program
2. Update the references that link "clock hours" to "greens"

#### Adding a new Colour
Colour generation, the fading values etc are all calculated from one place - 

colours.py - rgbs dict

To add a new colour, you'll need the RGB value of the colour you want, and then add a line in there like this:

`'spring_greens' : [20,225,148],`

Don't worry about needing to remove anything, just add your new colour in.

#### Updating the Colour References

Everything that links "this part is this colour" is in settings.py

Here's some of the locations that you might want to update:

| Display Section | Sub Section           | Variable         | Example                          | Notes                                      |
| --------------- | --------------------- | ---------------- | -------------------------------- | ------------------------------------------ |
| Power           | Overall Power         | power_map        | 500 : 'yellow',                  | Sets power bar when above 500W to "Yellow" |
|                 | Individual power bars | power_bar_fg     | pens['plum']                     |                                            |
| Network         | Download              | net_colour_map   | 'download': 'spring_greens',<br> |                                            |
|                 | Upload                | net_colour_map   | 'upload': 'honolulu'             |                                            |
| DNS Blinks      | Queries               | dns_colour_map   | 'no_error' : 'honolulu'          | Update "dot_maxes" and mins below          |
|                 | Blocked               | dns_colour_map   | 'blocked' : 'mango'              | See above                                  |
|                 | Servfail              | dns_colour_map   | 'servfail' : 'red'               | See above                                  |
| Clock           | Time                  | clock_time_fg    | pens['lgrey']                    |                                            |
|                 | Hours Bar             | clock_colour_map | hrs': 'spring_greens',           |                                            |
|                 | Mins Bar              | clock_colour_map | 'min': 'sea_greens',             |                                            |
|                 | SecBar                | clock_colour_map | 'sec': 'blues',                  |                                            |

That should do it, save, upload your changes and restart the device.

## Adjusting Network Bar Heights

For the network, everyone has different connections, my home connection is 900/100Mbit. Obviously I'd like this to scale differently than if I had 50/10.

In the settings.py file, there's net_download_scale and net_upload_scale, where you can set the number of dots to be drawn at each bandwidth usage level:

e.g.
`
```
net_download_scale = {
    1 : 0,
    2 : 2000,
    3 : 5000,
    .....
    9 : 4000000,
    10 : 8000000,
}
```

As as suggestion, don't scale these linearly, instead step up in powers of two or something similar, it makes for a much nicer effect and you'll probably see more fluctuations that look nicer.