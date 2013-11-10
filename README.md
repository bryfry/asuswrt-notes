asuswrt-notes
=============

Notes on asuswrt fun

Context and References
--------------------

* Currently testing with an Asus RT-N16, all should apply to the RT-(AC|N)[5,6]6U's
* See the [asuswrt-merlin](https://github.com/RMerl/asuswrt-merlin) and the awesome [wiki](https://github.com/RMerl/asuswrt-merlin/wiki) for more reference material
* Note: definitely make sure to extract (.zip -> .trx) and then rename (.trx -> .bin) before uploading to Flash Firmware (only a dummy would upload the .zip...)
* Follow the [Install](https://github.com/RMerl/asuswrt-merlin/wiki/Installation) instructions, pretty normal/basic

Setup and First Steps
---------------------
Assumption at this point is that the router is setup, networking, WiFi etc. At this point this section is mostly personal preferences, but there are some really silly defaults that need addressed:

### Advanced Settings > WAN 
WAN DNS Settings are defaulted to Connect to DNS Server Automatically.  This sucks because most U.S. Internet providers will inject search results on failed lookups.  Using public DNS servers is putting a lot trust on those providers, do read Google's [security page](https://developers.google.com/speed/public-dns/docs/security) for a well written survey of the threats in this space

[OpenDNS](https://store.opendns.com/setup/device/asus_device): admire the uglyness of the firmware we aren't using
* 208.67.222.222
* 208.67.220.220

[Google DNS](https://developers.google.com/speed/public-dns/): 
* 8.8.8.8
* 8.8.4.4

### General > AiCloud > Settings
Set the AiCloud Web access port to something other than 443, see below.  I'd prefer to have the admin panel on the normal https port, and I don't use AiCloud.  


### Advanced Settings > Administration
Most of the tweaks and additions in the merlin version of asuswrt are on the System tab
* Enable JFFS Partition (Yes) - needed for wan-start scripts that persist on reboot ([see wiki](https://github.com/RMerl/asuswrt-merlin/wiki/JFFS))
* WPS Button Behavior (Toggle Radio) - WPS is vulnerable [cert](http://www.us-cert.gov/ncas/alerts/TA12-006A) [wiki](http://en.wikipedia.org/wiki/Wi-Fi_Protected_Setup#Security) not having a button on the router that turns it on is a good idea ;)
* Enable Telnet (No) - this is the default, just check it to make sure
* Enable SSH (Yes)
* Allow SSH Port Forwarding (Yes) - useful feature
* Allow SSH access from WAN (No) - this is the default, just check it to make sure
* Allow SSH password login (No) - don't do it, see Auth Key
* Enable SSH Brute Force Protection (Yes)
* SSH Authentication key - past your .pub contents here
* Authentication Method (HTTPS) - No idea why it is default to HTTP 
* HTTPS Lan Port - make sure Cloud Disk's port doesn't conflict, see above / [see issue](https://github.com/RMerl/asuswrt-merlin/issues/454)
* Enable Web Access from WAN (No)
