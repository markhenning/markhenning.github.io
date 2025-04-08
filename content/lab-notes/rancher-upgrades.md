---
title: Rancher Upgrades
draft: false
---
## Symptoms 

Triggering a cluster version upgrade on the non-Local Cluster will see the first node start it's upgrade. It will complete most components, but then stall out reporting "Waiting for kubelet to update"

## Cause

During restarts etc, if Rancher is running in a container it can move IP address (rare, but it happens if something else changes in the network etc). This causes the check-in process to timeout, stalling the upgrade. 

## Fix

To fix the issue, we need to update the "rancher" Endpoint with the new IP of the rancher container, so the upgrade reports back to the right place.

#### Check the Rancher container IP address

```
docker inspect <container_name> | grep 172
```
#### Update the Endpoint IP
- Log into Rancher
- Select the Local Cluster
- Search for "Endpoints"
![[Pasted image 20250409004758.png]]
- Look for the "rancher" Endpoint
- Update the IP Address to match the docker container

![[Pasted image 20250409004853.png]]
- From here, it should recover on it's own, give 5-10 mins