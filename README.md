# CanaryPiE
IDS for internal networks

The goal of the CanaryPiE project is to design a low-cost intrusion detection system (IDS) for internal (firewalled / protected) networks. It will alert (currently by email) on detecting reconnaissance techniques triggered by an intruder who has already gained access to the network. Based on the MITRE ATT&CK framework CanaryPiE reacts to the following TTPs:

Reconnaissance (on internal network):

* Active Scanning
    - Scanning IP Blocks (e.g. nmap) 
    - Vulnerability Scanning (e.g. Nessus, nmap)
* Gather Victim Host Information
    - Software (e.g. nmap -sV)

It will also attempt to lure attackers (virtual trap aka honeypot) by providing services on well known ports 22 (ssh), 80 & 443 (http & https) and 445 (smb). Any connection attempts to these services will be logged and alerted.

It is based on a design and deployment model, intended to look like a legitimate and possibly vulnerable system to attract as many classes of cybercriminals as possible.

<pre>
nmap 192.168.xx.xx

Starting Nmap 7.94 ( https://nmap.org ) at 202x-xx-xx 12:58
Nmap scan report for 192.168.xx.xx
Host is up (0.020s latency).
Not shown: 995 closed tcp ports (reset)
PORT    STATE    SERVICE
22/tcp  open     ssh
25/tcp  filtered smtp
80/tcp  open     http
443/tcp open     https
445/tcp open     microsoft-ds
MAC Address: D8:3A:DD:xx:xx:xx (Unknown)
</pre>


| Port | State | Service | Process
| ---- | ----- | ------- | -------
22/tcp | open | ssh | Cowrie
25/tcp | filtered | smtp | Exim4 MTA
80/tcp | open | httpd | lighttpd
443/tcp | open | https | lighttpd
445/tcp | open | smb | Impacket

In its initial version CanaryPiE is pure honeypot that monitors attacks through bug taps. Next steps will to be to move to a high-interaction honeypot that will behave like real production infrastructure and will not restrict the level of activity of a cybercriminal. It will also be populated with fake information classified as sensitive to track potential data leaks.

## Honeypot Resources
- https://github.com/paralax/awesome-honeypots
- https://dingtoffee.medium.com/creating-a-honeypot-on-raspberry-pi-475858a2ba88
- https://medium.com/@alt3kx/build-an-easy-rdp-honeypot-with-raspberry-pi-3-and-observe-the-infamous-attacks-as-bluekeep-29a167f78cc1
- https://www.researchgate.net/publication/325952854_Raspberry_Pi_based_intrusion_detection_system


## Hardware Requirements
The project was installed on (will probably run on less):

- Raspberry Pi 5 Model 8GB
- with a SanDisk microSDHC-Karte Ultra UHS-I A1 32 GB
- running Raspberry Pi OS [Debian GNU/Linux 12 (bookworm)]


## Disable IPv6 (optional)
As we don't run any applications or services that rely on IPv6 let's disable it. But there is no harm to leave it enabled.

Edit /boot/firmware/cmdline.txt

<pre>
Add ipv6.disable=1 to the end of the line

e.g. console=serial0,115200 console=tty1 root=PARTUUID=7cd76f4e-02 rootfstype=ext4 fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles cfg80211.ieee80211_regdom=CH ipv6.disable=1
</pre>

Edit /etc/sysctl.conf
<pre>
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
</pre>

sudo sysctl -p

You will see this in the terminal:
<pre>
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
</pre>
Reboot

and then:

cat /proc/sys/net/ipv6/conf/all/disable_ipv6

If you see 1, ipv6 has been successfully disabled.

## macchanger (optional)

The reasoning behind spoofing the mac address is to potentially hide the fact that you have a single Raspberry Pi in your network (it could be a hint that it's acting as a honeypot). Pick a mac address which blends into your network. 

https://pimylifeup.com/raspberry-pi-mac-address-spoofing/

ARP Discovery using nmap (layer 2)
nmap -PR -sn 192.168.xx.xx

Starting Nmap 7.94 ( https://nmap.org ) at 2024-03-31 10:21 W. Europe Daylight Time
Nmap scan report for 192.168.xx.xx
Host is up (0.018s latency).
MAC Address: D8:3A:DD:xx:xx:xx (Unknown)
Nmap done: 1 IP address (1 host up) scanned in 0.29 seconds

https://www.macvendorlookup.com/

Company:	Raspberry Pi Trading Ltd
Address:	Maurice Wilkes Building, Cowley Road
Cambridge CB4 0DS
GB
Range	D8:3A:DD:00:00:00 - D8:3A:DD:FF:FF:FF
Type 	IEEE MA-L

## nftables (optional)

Configure the nftables firewall to start by default upon system boot:
sudo systemctl enable nftables.service

Start the service:
sudo systemctl start nftables.service
Configuration
https://medium.com/@diyar.parwana/nftables-for-uslinux-administrators-a-simple-guide-d13c5f0cf40f
Edit the ruleset
sudo nano /etc/nftables.conf
Restart nftables to reload the ruleset
sudo systemctl restart nftables
Check ruleset
sudo nft list ruleset

## scanlogd
scanlogd is a TCP port scan detection tool. For more information see the following article.
http://phrack.org/issues/53/13.html#article


## Supervisor
Supervisor is a client/server system that allows its users to monitor and control a number of processes on UNIX-like operating systems.
http://supervisord.org/
http://supervisord.org/introduction.html

Used for Cowrie and Impacket processes.

sudo supervisorctl
- status
- stop / start / restart [component]

/etc/supervisor/conf.d/*.conf

## Cowrie (SSH port 22 and optional telnet port 23)
https://github.com/cowrie/cowrie

Customizing:
https://cryptax.medium.com/customizing-your-cowrie-honeypot-8542c888ca49

## Impacket (SMB port 445)
Impacket is a collection of Python classes for working with network protocols. Impacket is focused on providing low-level programmatic access to the packets and for some protocols (e.g. SMB1-3 and MSRPC) the protocol implementation itself. Packets can be constructed from scratch, as well as parsed from raw data, and the object-oriented API makes it simple to work with deep hierarchies of protocols. The library provides a set of tools as examples of what can be done within the context of this library (from README.md).
https://github.com/fortra/impacket
Add to supervisor:

## Lighttpd (HTTP/S ports 80 & 443)

## Logcheck

- https://manpages.ubuntu.com/manpages/trusty/man8/logcheck.8.html
- https://linux.die.net/man/8/logcheck
- https://fcerbell.github.io/Debian113Server110Logchecktonotifyaboutanyunknownactivit-en/

/etc/logcheck/logcheck.logfiles.d/

sudo -u logcheck logcheck -o -t

## Exim4 (Mail)

dpkg-reconfigure exim4-config

sudo -u logcheck logcheck -o -t | mail -s "subject" -aFrom:"email-addr"\<name@any.com\> email-addr

<pre>
#!/bin/sh
string=$(sudo -u logcheck logcheck -o)
len=${#string}
if [ "$len" -gt "0" ]
   then echo "$string" | mail -s "subject" -aFrom:"email-addr"\<name@any.com\> email-addr
fi
</pre>




