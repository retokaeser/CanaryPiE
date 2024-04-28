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

```
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
```

## Disable IPv6
Edit /boot/firmware/cmdline.txt
```
Add ipv6.disable=1 to the end of the line

e.g. console=serial0,115200 console=tty1 root=PARTUUID=7cd76f4e-02 rootfstype=ext4 fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles cfg80211.ieee80211_regdom=CH ipv6.disable=1
```

Edit /etc/sysctl.conf
```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

sudo sysctl -p

You will see this in the terminal:
```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```
Reboot

and then:

cat /proc/sys/net/ipv6/conf/all/disable_ipv6

If you see 1, ipv6 has been successfully disabled.

## macchanger
https://pimylifeup.com/raspberry-pi-mac-address-spoofing/
ARP Discovery using nmap (layer 2)
nmap -PR -sn 192.168.xx.xx

ICMP Ping using nmap (layer 3)
nmap -PE -sn 192.168.xx.xx

TCP ACK Ping using nmap (layer 4)
nmap -PA80 -sn 192.168.xx.xx

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

## nftables
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
