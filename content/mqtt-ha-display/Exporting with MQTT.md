---
title: Exporting with MQTT
draft: false
---

MQTT serves as a simple, useful message queue for passing data between devices for various reasons. It's the core part of a lot of Home Assistant tech, allowing easy connectivity to services such as Frigate NVR, ESPresence.

MQTT works using a defined set of "Topics" that hold messages passing between services.

Topics are formed in a tree structure, allowing for subtopics to organise messages for collection by other clients. 

Here, MQTT Explorer connected to an MQTT server shows where it's storing the current "other" energy usage for the building, and the history of it:

![[Pasted image 20250813071615.png]]

## Choosing Data to Export

Before looking at the config, the best option is to work out what to export. Typically for Home Assistant, this will be a set of sensors. At this point in the process, you will just need an idea of what you're after, and you can work them all out as you go.

### MQTT Statestream

HA has some great docs for this, but they're a little large for simple use. 

The easiest integration to get the data out is "MQTT Statestream". This is installed, but not enabled if you install the MQTT Integration. Doing the work to enable and configure is very straightforward.

Most likely, if you're using Frigate, Zigbee2MQTT etc, you've already running MQTT.

[Full MQTT Statestream Docs](https://www.home-assistant.io/integrations/mqtt_statestream/ "MQTT Statestream")

There's a lot in the docs there, but the following gives a brief overview of what we need:
## Enabling MQTT Statestream

Very simple one liner here. In your config.yaml file, just add a line like this at the bottom:

```
mqtt_statestream: !include mqtt_statestream.yaml
```

That tells Home Assistant that we're going to make an "mqtt_statestream.yaml" file and we'll fill in the details there for an export.

![[Pasted image 20250813071724.png]]

## Configuring the Exports

Next, create the mqtt_statestream.yaml file and you'll build a config that looks something like this as we go:

![[Pasted image 20250813071736.png]]

Looking at this line by line:

> base_topic: haexport

This is just the name of the MQTT topic that we're going to create. "haexport" is fine, but you can name it whatever you want.

> include:

There are two types of statement for filtering/grouping devices - **include** and **exclude**

Just as they sound:
* **include** will define a set of devices that we want to report to MQTT. 
* **exclude** filters out anything that's already included (e.g. we can include "all power" and exclude "current PC power")

## Entity/Globs/Domains

These are the actual sensors to export from Home Assistant. Every device has an "entity" on the back end, which you can find in the Dev tools, or if you click on the device itself. It'll be in the form of "sensor.device_info":

![[Pasted image 20251119164048.png]]

Includes can take up to three different options, depending on how you want to group your devices, most likely, you'll want entity_globs, and then fine tune with some additional entities

- **entities** - individual entity data (e.g. "sensor.outside_switch_battery" will export the exact value of the battery in a specific switch)
- **entity_globs** - essentially "a set of entities covered by a wildcard" 
	- e.g. "sensor.*battery") will export anything that matches it (e.g. "sensor.outside_switch_battery", "sensor.tempsensor_battery", but not "sensor.kindle_batt")
- **domain** - a whole class of entity - e.g. "sensor" will export all sensor data)

Next, if needed, fill in an exclude section, in exactly the same format as include.

> Note: It's possible to get a super complex include/exclude set, but the above is pretty much all you need to know to get started. 
> 
> MQTT messages are tiny, aim for as accurate as can be within reason, my current setup has about ~200 messages in 5 minutes, and using at most 3MB of RAM, this is not a place to send files through, at most a few lines of text in each message.


![[Pasted image 20250813071704.png]]

That's it!

Config check and restart Home Assistant, and data should start appearing in MQTT. If it doesn't, a configuration check should let you know what's gone wrong or check the home assistant MQTT logs.