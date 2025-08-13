---
title: Exporting with MQTT
draft: false
---

MQTT serves as a simple, useful message queue for passing data between devices for various reasons. It's the core part of a lot of Home Assistant tech, allowing easy connectivity to services such as Frigate NVR, ESPresence.

MQTT works using a defined set of "Topics" that hold messages passing between services. Topics are formed in a tree structure, allowing for subtopics to organise messages for collection by other clients. Here, a local MQTT server shows where it's storing the current "other" energy usage for the building, and the history of it:

![[Pasted image 20250813071615.png]]

## Choosing Data to Export

Before looking at the config, the best option is to work out what to export. Typically for Home Assistant, this will be a set of sensors, for this example, this will be the power data for a house, and some other smaller pieces of data.
## **Configuring the Exports**

HA has some great docs for this, but they're a little large for simple use. The easiest integration to get the data out is "MQTT Statestream". This is installed, but not enabled if you install the MQTT Integration, but enabling and configuring is very straightforward.

[MQTT Statestream](https://www.home-assistant.io/integrations/mqtt_statestream/ "MQTT Statestream")

There's a lot in the docs there, but the config's pretty simple based on what you want to export, which break down as follows:

- domain - a whole class of entity - e.g. "sensor" will export all sensor data)
- entities - individual entity data (e.g. "sensor.outside_switch_battery" will export the exact value of the battery in a specific switch)
- entity_globs - essentially "a set of entities covered by a wildcard" - e.g. "sensor.*battery") will export anything that matches it (e.g. "sensor.outside_switch_battery", "sensor.tempsensor_battery", but not "sensor.kindle_batt")

There are also "include" and "exclude" groupings, which allow for even finer filtering, e.g. include all sensors, but exclude "sensor.tempsensor" will export everything but the tempsensor.

It's possible to get a super complex include/exclude set, but the above is pretty much all you need to know to get started. MQTT messages are tiny, get it accurate as can be within reason, I don't use all the power data I export, but as you can see, I'm pushing ~200 messages in 5 minutes, and using at most 3MB of RAM, this is not a place to send files through, at most a few lines of text.


![[Pasted image 20250813071704.png]]

## Configuring the Export

WIth a list of entities, there's two config parts needed:

1. Statestream in configuration.yaml

![[Pasted image 20250813071724.png]]

Here, a statement just tells the main configuration file that we're going to create a file "mqtt_statestream.yaml" and it needs to read that for data

2. Statestream Configuration

First, "mqtt_statestream.yaml" doesn't exist by default, so it needs to be created.

Next, add in the topic to export to, and the entities we're looking to export. For this usage, I want to export all of my power sensors, my electricity provider current usage and a couple of counters I keep for DNS:

![[Pasted image 20250813071736.png]]