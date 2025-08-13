---
title: Site Selective VPN
draft: false
---

So the UK launched the OSA, and everything got id checks... fun

The solution everyone's falling back on is "VPN, VPN, VPN". This works fine for sites, but it's pretty much all or nothing. I don't want to play about with clients on all my devices and toggling on/off all the time. There's also browser plugins and apps that will select which processes to route over the VPN, honestly these are great solutions.

However, I really don't want extra stuff where I can avoid it, I don't like syncing browser settings etc across machines. So, let's set up a method where we can pick individual sites to route over the VPN.

The only real hard part is that those pesky websites keep changing what other sites they call, and moving their DNS records about. This is especially hard as most firewalls only really care about IP addresses, so whenever a site moves, we need to adjust on our end.

Since I don't own the internet, I need a way that works within the rules of the internet. So we're going to need two things:
1. A site to Site VPN Tunnel
2. A way of routing traffic that adapts to changes

**The Easy Part - OPNsense**

OPNsense, and most routers, will provide a method to make a site to site VPN.
They've also got great documentation on setting up the tunnel on their site

[Opnsense Mullvad Road-Warrior Setup Guide](https://docs.opnsense.org/manual/how-tos/wireguard-client-mullvad.html)

The only problem at the end of that doc it configured routing everything through the tunnel, which isn't ideal. We've essentially got two gateways "Normal" and "VPN" and rather than routing everything over the VPN

This is where Firewall Aliases come in, and forms the lynch pin of the whole process.
Firewall Aliases allow for one configuration item that contains a list of IPs or subnets that are treated as a firewall object, so we can use one to adjust what goes where on the fly, we just need to have a method to update the contents of the Firewall Alias. More on this later.

(These live under Firewall -> Alias)

We'll just need a new "Hosts" alias, I've called mine "VPN_Websites"

![[Screenshot_20250813_010341-50.png]]

For now, we're going to adjust the NAT rules so that that anything targetting "VPN_Websites" routes out over the VPN:

![[Screenshot_20250813_010819.png]]

So, we're there on the VPN sites of things, we just need to adjust "VPN_Websites" somehow...

**A Note on Firewall Aliases**

You can skip this bit for setup, but anyone who know's OPNsense is already thinking about the Alias type I used.

I set up a "hosts" alias here, there's a load of sensible options here, the most obvious being "URL (IPs)" and "URL (IPs) in JSON formatting". These can pull from a webserver on a schedule, but after testing, the minimum that can be specified and actually work for me here is 1 hour.

You can set up a cron job to refresh downloaded lists like this on any cron schedule, however, this tasks updates every list that's been downloaded. Since my OPNsense box is pulling a list of crowd sourced IPs to block for security reasons, these would get updated as well, and since I want to update every 5 minutes, I'd probably end up on those lists for abuse, so we need to avoid that and be a good internet citizen.

**Adjusting VPN Websites**

Let's start with an overall plan for what we need to do:

![[DNS-Full-Flow.png]]

(We're only going to go "one deep" on the website crawl, because it'll mostly cover most domains and I really don't want to crawl entire sites)

OPNsense's API is pretty robust and works well.

We can get the JSON of an alias content with some basic python

``` 
alias = "VPN_Websites"
proto = "https"
router = "router.home...."
api_key = "32faf....."
api_secret = "rwef23f3...."

def get_alias_json(alias):  
   path = 'api/firewall/alias_util/list'  
   url = proto + "://" + router + "/" + path + "/" + alias  
   r = requests.get(url,auth=(api_key,api_secret))  
   if r.status_code == 200:  
       return r.json()  
   else:  
       return {}
```

### Getting the List of IPs

**DNS and Avoiding Some Pitfalls**

The trick to all of this is controlling what the clients see in terms of DNS. There's no point in 

If we're doing this we need to be incredibly respectful of TTLs, sites can change IP at any moment (especially if someone's loadbalancing using DNS), and typically provide a TTL of how long a cached entry is valid.

I run two DNS servers at home, so we're just going to loop over them, for each domain we're going to need lookups for, we need to clear the current entries out of the cache, and then query the server again so that it re-populates the cache. So we can be sure that whatever happens, if something asks for a DNS entry, we know what it'll get as a reply and that entry is ready to go.

### Future Steps

These aren't implemented at this time, but the ground work's there, when these become issues, I'll put the python code in:

International DNS

At this point in time, DNS requests all come from my home IP rather than the country the VPNs exiting. It's not causing any problems for me right now, but if we need DNS from the other side of the tunnel there's a solution we can use.

VPN providers will publish their own DNS servers for clients using the tunnel to use. I don't want to redirect the clients to talk to there, but we can add conditional forwarders to the DNS servers so if we're looking up "site xxx", it'll resolve as if the DNS lookup occurred in the destination country.

CDNs and IP Blocks

Sites with large amounts of subdomains can be a real pain to navigate. E.g some sites will user "server234342.domain", "server32432432.domain" etc, so pulling them and using DNS doesn't really work as we could get any of those servers referred to from a site.

The solution here is ASNs. The internet's IPv4 address space is divided up into blocks, each block is assigned to a specific owner, or ASN. These are unique IDs for each company, and you can find who owns what IPs using a whois lookup.

![[Pasted image 20250813020753.png]]

That shows the block of IPs that they own, and their name.

Since it's common for providers to own more than one block, we need to find them all. This turns out to be easy to do.

The database is collected every 8 hours from running devices on the internet and is available to download.

This can be done with the python module "pyasn" which includes download utilities in it's package:

`pyasn_util_download.py --latest`
`pyasn_util_convert.py --single <<downloaded_file>> dbfile`

Will give us a text file mapping subnets to ASNs:
![[Pasted image 20250813021048.png]]

So once that's done, we just collect all of the subnets allocated to an ASN, and put them all into "VPN_Websites"

Whilst it's possible that this could eventually move half the internet to the VPN, it's best we don't just let the script decide ASNs on the fly. So we'll add another worker script that reads a set list of ASNs, and adds them to the IPs for the firewall alias.