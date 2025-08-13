---
title: WakeOnLan and Sleeping Remote Servers
draft: false
---

With a home lab on 24/7, even a small amount of power usage can add up fairly quickly. Whilst not normally more than 20p/day, it's not exactly something I need to spend just to keep servers that aren't getting used for anything.

 Here, we'll configure Home Assistant to allow for the following Dashboard entries, so we can just toggle servers on/off as needed:

![[Pasted image 20250714221047.png]]

WakeOnLan solves one problem here, automatically turning on servers, but it doesn't handle shutdown/sleep, for that, another method is required.

## WakeOnLAN Home Assistant Configuration

WakeOnLAN is supported natively in Home Assistant, but adding in scripts to allow for shutdowns isn't.

**Setting Up Target Servers for Shutdown**

Wake on LAN is simple, spam the MAC address and wake the server, suspend takes a fair but more work, but it's not overly complicated. If you're just after wake on lan, skip this part.

Four things are needed for this:
1) SSH keys allowing access
2) A homeassistant user on the target server we want to sleep
3) sudo rules to allow the very specific suspend command we want to run
4) Home Assistant Shell Command integration installed

SSH Keys
It's best here to just make keys that home assistant can reference rather than trying to integrate them into the underlying docker/default users on the server.

Therefore, using a terminal on HA, make a directory for them, and make a key pair for us to use:

```
mkdir -p /config/ssh_keys
ssh-keygen -f /config/ssh_keys/ha_id -t ecdsa -b 512
```

Mine look like this (names are different on the files)

![[Pasted image 20250714221755.png]]
You'll need a copy of the .pub file so we can put it on the target server

**Target Server User Setup**

On each server to suspend, you'll need a user account (I just used "homeassistant") with the .pub file in authorized_keys

Next, a sudo rule is needed to allow nopasswd for the command "/bin/systemctl suspend"
No other commands should be needed, I hate anything nopasswd, but I can justify it here.

For my environment, I just make a "wakeonlan" group in ansible, with an ha_suspend role to create the user, keys and set the specific sudo command. The code's at the bottom of this page

## Home Assistant Configuration

### Shell Command Integration

Docs [Here](https://www.home-assistant.io/integrations/shell_command/)

This is a really simple integration that allows shell commands to be called by home assistant.
Installation's really simple, just the basic integration, and we'll configure it later.

### Config File Setup

As with everything like this, it's best to make a dedicated config files -  "wake-on-lan.yaml", "shell_commands/shell_commands.yaml" and just include them back into the main config.

(Note: a dedicated directory for shell commands isn't normally what you'd see in HA, but here, you can end up with lots of individual files, so separating them out here is good)

configuration.yaml
`switch: !include wake-on-lan.yaml`
`shell_command: !include shell_commands/shell_commands.yaml`

Then we can write definitions for each server we want to control in the wake on lan file:

wake-on-lan.yaml
`- platform: wake_on_lan`
    `mac: "d8-cb-8a-22-6f-40"`
    `host: 192.168.8.150`
    `name: "VC01"`
    `turn_off:`
       `service: shell_command.turn_off_server_vc01`

Here, we can see the normally expected MAC address and name for the server along with some additional info:
* Host - the IP address of the server, Home Assistant will ping this to check on server up/down status
* "turn_off" - this gives a command to allow Home Assistant to turn off/sleep the server. Note I've got individual ones for each sever

shell_commands/shell_commands.yaml
`turn_off_server_vc01: "ssh -i /config/ssh_keys/ha_id -o 'StrictHostKeyChecking=no' homeassistant@<<hostname>> sudo systemctl suspend"`

Repeat this line for each server, changing the hostname in the alias (the part before the ":") and where needed in the command:

Once complete, check the config is valid (Developer Tools -> Check Configuration) and then restart HA. You'll then have new switch entities for the servers
### Ansible playbook for servers HA can suspend

```
---  
# tasks file for ha_suspend  
  
- name: Create homeassistant User  
 ansible.builtin.user:  
   name: homeassistant  
   shell: /bin/bash  
   create_home: yes  
   password: '{{ homeassistant_enc_pw }}'  
   append: yes  
  
- name: Authorized_keys  
 ansible.posix.authorized_key:  
   user: homeassistant  
   state: present  
   key: '{{ item }}'  
 with_file:  
   - ../files/homeassistant_id_rsa.pub  
  
- name: sudo rules  
 community.general.sudoers:  
   name: homeassistant-suspend  
   user: homeassistant  
   commands:  
     - /bin/systemctl suspend  
   nopassword: yes
```
``