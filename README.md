# raspberry_router

## how to set up a Raspberry Pi as an access point(Router)

## if you can't understand what I had saying, you can use the google translation translate the README.CN.md 

***

#### this program is a preview of change a raspberry as a router and a ntp server.

>from https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md and https://vitux.com/how-to-install-ntp-server-and-client-on-ubuntu/ 

>And I could install nextcloud and wordpress to change the raspberry as a smart route
---
### It depanded raspbian-2019-07

1. First we should install these bags
```shell
sudo apt update
sudo apt -y upgrade
sudo apt -y install dnsmasq hostapd
sudo systemctl stop dnsmasq
sudo systemctl stop hostapd
```
We will use the nano as editer,if you like to use vi/vim or other, change it and use it.

2. Configuring a static IP
>We are configuring a standalone network to act as a server, so the Raspberry Pi needs to have a static IP address assigned to the wireless port. 

```shell
sudo nano /etc/dhcpcd.conf
```
>Go to the end of the file and edit it so that it looks like the following:
```shell
interface wlan0
    static ip_address=192.168.4.1/24
    nohook wpa_supplicant
```
We shoud know that the "ip_address" shoud be set as different as the address on the eth0. And you can change the number of the ip_address, but it should be access the rule of the address.

**Someone maybe tell you that you should change the file */etc/network/interfaces* but this file has been abandoned. So you shoud change the file */etc/dhcpcd.conf***

>Now restart the **dhcpcd** daemon and set up the new wlan0 configuration:

```shell
sudo service dhcpcd restart
```

3. Configuring the DHCP server

Rename this configuration file, and edit a new one:

```shell
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
sudo nano /etc/dnsmasq.conf
```

And add these on the end

```shell
interface=wlan0      
dhcp-range=192.168.4.2,192.168.4.249,255.255.255.0,24h
```

When we set the wlan0 ip address, we use the *"192.168.4.1/24"*,so we set the dhcp-range is the *"192.168.4.2,192.168.4.249"* and the *"/24"* main as *"255.255.255.0"* .

Reload **dnsmasq** to use the updated configuration:

```shell
sudo systemctl reload dnsmasq
```

4. Configuring the access point host software 

>You need to edit the hostapd configuration file, located at /etc/hostapd/hostapd.conf, to add the various parameters for your wireless network. After initial install, this will be a new/empty file.

```shell
sudo nano /etc/hostapd/hostapd.conf
```
and change it as down

```shell
interface=wlan0
# use the wlan0 as ap
driver=nl80211
# use the drive nl802.11
ssid=NameOfNetwork
# your wireless ssid
hw_mode=g
# the mode you could use
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=AardvarkBadgerHedgehog
# your ssid password
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

defferent hw_mode can use different HZ

 * a = IEEE 802.11a (5 GHz)
 * b = IEEE 802.11b (2.4 GHz)
 * g = IEEE 802.11g (2.4 GHz)
 * ad = IEEE 802.11ad (60 GHz)

We now need to tell the system where to find this configuration file.
```shell
sudo nano /etc/default/hostapd
```

Find the line with #DAEMON_CONF, and replace it with this:
```shell
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```
5. Start it up
We should start the service of hostapd and dnsmasq
```shell
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo systemctl start hostapd
sudo systemctl restart dnsmasq
```
And we should check the status of these service
```shell
sudo systemctl status hostapd
sudo systemctl status dnsmasq
```
6. Add routing and masquerade

***Before we start, you maybe should reboot your raspberry pi***
```shell
sudo reboot
```
Edit /etc/sysctl.conf and uncomment this line:
```shell
net.ipv4.ip_forward=1
```
Add a masquerade for outbound traffic on eth0:
```shell
sudo iptables -t nat -A  POSTROUTING -o eth0 -j MASQUERADE
```
Save the iptables rule.
```shell
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```
Edit /etc/rc.local 
```shell
sudo nano /etc/rc.local
```
and add this just above "exit 0" to install these rules on boot.
```shell
iptables-restore < /etc/iptables.ipv4.nat
```
Reboot and ensure it still functions.

Then we install the ntp server

7. update and upgrade
```shell
sudo apt update && sudo apt -y up grade
```
8. install ntp server
```shell
sudo apt install ntp
```
9. Switch to an NTP server pool closest to your location

```shell
sudo nano /etc/ntp.conf
```
and you can find NTPPools on this site 
>https://support.ntp.org/bin/view/Servers/NTPPoolServers

You can choose a NTPPool near your location.

10. Restart the NTP server and Verify that the NTP Server is running
```shell
sudo service ntp restart
sudo service ntp status
```
## if you can't understand what I had saying, you can use the google translation translate the README.CN.md 