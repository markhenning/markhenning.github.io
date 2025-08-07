---
title: WakeOnLan and Sleeping Remote Servers
draft: true
---

With a home lab on 24/7, even a small amount of power usage can add up fairly quickly. Whilst not normally more than 20p/day, it's not exactly something I need to spend just to keep servers that aren't getting used for anything.

 Here, we'll configure Home Assistant to allow for the following Dashboard entries, so we can just toggle servers on/off as needed:

![[Pasted image 20250714221047.png]]

WakeOnLan solves one problem here, automatically turning on servers, but it doesn't handle shutdown/sleep, for that, another method is required.

## WakeOnLAN Home Assistant Configuration

WakeOnLAN is supported natively in Home Assistant, but we'll need some data for each server:

`- platform: wake_on_lan`
    `mac: "d8-cb-8a-22-6f-40"`
    `host: 192.168.8.150`
    `name: "VC01"`
    `turn_off:`
       `service: shell_command.turn_off_server_vc01`

Here, we can see the normally expected MAC address and name for the server along with some additional info:
* Host - the IP address of the server, Home Assistant will ping this to check on server up/down status
* "turn_off" - this gives a command to allow Home Assistant to turn off/sleep the server


`turn_off_server_vc01: "ssh -i /config/ssh_keys/ha_id_rsa -o 'StrictHostKeyChecking=no' homeassistant@vc01.home.mcsmiggins.com sudo systemctl suspend"`

![[Pasted image 20250714221755.png]]

Sleep/Shutdown