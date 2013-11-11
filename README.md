asuswrt-notes
=============

* Currently testing with an Asus RT-N16, all should apply to the RT-(AC|N)[5,6]6U's
* See the [asuswrt-merlin](https://github.com/RMerl/asuswrt-merlin) and the awesome [wiki](https://github.com/RMerl/asuswrt-merlin/wiki) for more reference material
* Definitely make sure to extract (.zip -> .trx) and then rename (.trx -> .bin) before uploading to Flash Firmware (only a dummy would upload the .zip...)
* Follow the [Install](https://github.com/RMerl/asuswrt-merlin/wiki/Installation) instructions, pretty normal/basic

First Steps (web admin settings)
--------------------------------
Assumption is that the router is setup, networking, WiFi etc.
At this point this section is mostly personal preferences, but there are some really silly defaults that need addressed:

### General > AiCloud > Settings
Set the AiCloud Web access port to something other than 443, see below.
I'd prefer to have the admin panel on the normal https port, and I don't use AiCloud.

### Advanced Settings > WAN 
WAN DNS Settings are defaulted to Connect to DNS Server Automatically.
This sucks because most U.S. Internet providers will inject search results on failed lookups.
Using public DNS servers is putting a lot trust on those providers, do read Google's [security page](https://developers.google.com/speed/public-dns/docs/security) for a well written survey of the threats in this space

[OpenDNS](https://store.opendns.com/setup/device/asus_device): admire the uglyness of the firmware we aren't using
* 208.67.222.222
* 208.67.220.220

[Google DNS](https://developers.google.com/speed/public-dns/): 
* 8.8.8.8
* 8.8.4.4

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

Persistent Web Admin SSL Cert
-----------------------------
This is definitely something that bothers me.
If I choose to allow a self signed cert, all the major browsers all just say: 
"h'okay, we'll let you accept this cert, but we will always show a red X for it because it is self signed".
This makes sense, except for one thing, If that cert changes the acceptance behavior IS EXACTLY THE SAME.
IMO these certs should be stored locally in a browser cache and give a super bad alert if the cert changes for the same host 
(in the same way that openssh-client handles host identities).

The only real way to fix this is to go out of our way and add these certs to the OS level trusted certs.
Also, the router makes new certs every boot, so we need to make them persistent:

#### Create a persistent self signed key/cert for https web interface
* Create the key (replace the {{}}'ed names) 
        
         openssl req -x509 -newkey rsa:2048 -days 365 -nodes \
          -keyout /jffs/keys/key.pem \
          -out /jffs/keys/cert.pem \
          -subj '/CN={{networkname}}/O={{org}}/C=US'

* Add the below lines to /jffs/scripts/services-start (via [forum](http://forums.smallnetbuilder.com/showthread.php?t=10176)).
This adds the created keys to the working directory from the jffs partition which will survive reboots.
It is also probably a good idea to back up these scripts to the usb device (left as an exercise for the student).

        mv /tmp/etc/key.pem /tmp/etc/key.pem.bak
        mv /tmp/etc/cert.pem /tmp/etc/cert.pem.bak
        cp /jffs/keys/key.pem /tmp/etc/key.pem
        cp /jffs/keys/cert.pem /tmp/etc/cert.pem
        service restart_httpd

#### Add the self signed cert to OS trusted certs
In order to make this work correctly you need to add the self signed cert into your trusted certs (OS X, [more info](http://www.robpeck.com/2010/10/google-chrome-mac-os-x-and-self-signed-ssl-certificates/#.Un_4R2RDuiU)): 
* Drag the cert to the desktop (to save it)
* Keychain Access > File > Import Item > (router cert).pem

entware (optware alternative)
-----------------------------
The first step is to grab a mediocre usb drive that is laying around and [format it](http://www.itechlounge.net/2012/01/linux-partition-and-format-external-hard-drive-as-ext3-filesystem/).
ext3 or ext2 are what are supported.
Then plug it in and ssh into the router and run `df -hm`.
The drive should show up and be mounted at something like `/tmp/mnt/sda1`.
Follow the [Entware wiki](https://github.com/RMerl/asuswrt-merlin/wiki/Entware) ssetup guide.
From here I either just install the things I usually want, or get ansible to do it for me ([opkg module](http://www.ansibleworks.com/docs/modules.html#opkg)).

* `opkg install python git openssh-keygen python-requests` # 'required'
* `opkg install vim htop tcpdump openssh-client` # optional but nice

Email on WAN up
---------------
Follow the [wiki](https://github.com/RMerl/asuswrt-merlin/wiki/Sending-Email).
Make sure to use an [application specific password](https://support.google.com/accounts/answer/185833?hl=en) if you are using gmail

dynhover
--------
Work in progress, currently entware's pyhton is not compiled with ssl ([issue](https://code.google.com/p/wl500g-repo/issues/detail?id=268)).
See ([dynhover](https://github.com/bryfry/dynhover))

sshuttle
--------
Once python is installed and ssh is enabled your router is primed for using [sshuttle](https://github.com/apenwarr/sshuttle).
No other changes need to be made to the router at this point.
An example use case would be if your router is set up as an open WiFi network but want to keep your internet connections private and secure to/through the router, this would work pretty well.

You will need to tell sshuttle where the python binary is on the target, in this case: `/opt/bin/python`

e.g.: `sshuttle --dns --python /opt/bin/python -r {{user}}@{{networkname}} 0/0`
