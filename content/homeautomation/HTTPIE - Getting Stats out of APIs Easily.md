---
title: Home Automation
draft: true
---

Sometimes, people just say to use the API for something, but the hardest part can be working out the initial connection, auth and setup. Browsing the data and transforming it can really be the easy part later.

So, here's my guide to getting started with APIs, finding the data you want, and making it usable. Here, we'll set a task of getting the last minute DNS stats for the server so that we can pull it into Home Assistant later. 

To do this, we'll need three things:
1. An API URL to talk to
2. Some form of authentication
3. (Optional) A way to filter the results we get back (no point pulling 24 hours of stats if we don't have to)

First step, let's get an API to query. From the other pages you can see, I'm a massive fan of Technitium, small, powerful and the API is one of the easiest to get to grips with, I'd recommend it as a great place to start. 

Let's cover API calls first. Technitium publish their API docs at: https://github.com/TechnitiumSoftware/DnsServer/blob/master/APIDOCS.md so we'll use this an their examples.
## Understanding an API URL

Most URLs of an API have a format similar to the following, with three sections:

![[Screenshot_20250703_210915-1 1.png]]
These break down to :
1. URL
2. Path
3. Parameters (Optional) - note this is just "key=value", and tacked together with & signs

The parameters can either be not present, or imposingly large and difficult to read in a browser, but we can find the parameters for whatever we want in the API documentation.




