---
title: Home Assistant
draft: false
---
# Home Assistant

Home Assistant serves mainly as an aggregator for IoT devices, making things vendor agnostic, allowing local control of a large number of devices and allows for the creation of automations across these devices

## Showcase

It's easier to show some of the examples of what can be achieved:
### Floorplans

![[ha-overview.mp4]]

### Dashboards

![[Energy-Overview.png]]

### Power Monitoring

![[Screenshot 2024-08-07 175019.png]]

The real power, however, comes from tying these things together.

For example:
- Disable alerting of people in the garden when I open the back door
- Monitor the power usage of the dishwasher and alert me when it drops back to 0
- Check if I've gone out for 15 minutes and turn off my PC monitors if I've left them on
- Power on/off my home lab depending on last time used

I run a pretty standard install across the board but as the saying goes "Data is Beautiful" and I can now use various bits of data for other things, including my favourite pet project - "ha-mqtt-display", some python I wrote which shows power usage, current internet usage and DNS  stats for my house, on an LED matrix.

This video is a placeholder, it's greatly improved since:

![[G-Unicorn.mp4]]

The code is all available here: 
https://github.com/markhenning/ha-mqtt-display

Docs, I'm backfilling, but at this time, if anything, the code is overly commented