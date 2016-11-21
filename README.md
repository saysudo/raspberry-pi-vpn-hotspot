# Raspberry Pi 3 Model B as VPN Hotspot - Simple Tutorial

Using TP-Link Wi-Fi adapter to connect to my router.
You can use ethernet instead.
Raspbian Jessie With Pixel - 9/16

Let's start! (May seem long but... it ain't that much!)


**Download first:**
~~~
sudo apt-get update
sudo apt-get install openvpn hostapd isc-dhcp-server
~~~


**VPN Setup:**

I can not help much on this part and you will have to contact your VPN provider for getting the setup ready!
Why? Cause each provider has different set of configurations.

But... some things you should remember:
Once you have your config and login details, all you need to do (in most cases), copy them to openvpn
directory. Then it's easy!

Example:
~~~
sudo cp ABC.conf /etc/openvpn
~~~

Test:
~~~
sudo service openvpn start
sudo service openvpn status
~~~
Status should read "active"


**DHCP Service:**
~~~
sudo nano /etc/dhcp/dhcpd.conf
~~~
Add # to below lines:
~~~
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;
~~~

Remove # from this line:
~~~
#authoritative;
~~~

Scroll down to the bottom and add below:
~~~
subnet 192.168.42.0 netmask 255.255.255.0 {
    range 192.168.42.10 192.168.42.50;
    option broadcast-address 192.168.42.255;
    option routers 192.168.42.1;
    default-lease-time 600;
    max-lease-time 7200;
    option domain-name "local";
    option domain-name-servers 8.8.8.8, 8.8.4.4;
}
~~~
~~~
sudo nano /etc/default/isc-dhcp-server
~~~
Add wlan0 like this:
~~~
INTERFACES="wlan0"
~~~


**Network Interfaces:**
~~~
sudo ifdown wlan0
sudo nano /etc/network/interfaces
~~~
Configure like this:
~~~
auto lo

iface lo inet loopback
iface eth0 inet dhcp

allow-hotplug wlan0

iface wlan0 inet static
  address 192.168.42.1
  netmask 255.255.255.0
  post-up iw dev $IFACE set power_save off

allow-hotplug wlan1

iface wlan1 inet manual
  wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf

up iptables-restore < /etc/iptables.nat.vpn.secure
~~~


**Assign Static IP to wlan0:**
~~~
sudo ifconfig wlan0 192.168.42.1
~~~


**Hostapd:**

~~~
sudo nano /etc/hostapd/hostapd.conf
~~~
Then fill it like this:
~~~
interface=wlan0
ssid=EXAMPLE
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=STRONG_PASSWORD
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
~~~
~~~
sudo nano /etc/default/hostapd
~~~
Remove # and add /etc/hostapd/hostapd.conf between quotes:
~~~
DAEMON_CONF="/etc/hostapd/hostapd.conf"
~~~


**Network Address Translation:**
~~~
sudo nano /etc/sysctl.conf
~~~
Look with care here and remove # from below or add it without # 
to the very bottom of the file:
~~~
# net.ipv4.ip_forward=1
~~~

Get it ready now:
~~~
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
~~~


**IPTables Rules:**
Pay attention here!
Copy and paste below:
~~~
sudo iptables -A INPUT -i tun0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -i tun0 -j DROP
sudo iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE
sudo sh -c "iptables-save > /etc/iptables.nat.vpn.secure"
~~~


**Time to test and start them:**

Hotspot:
~~~
sudo /usr/sbin/hostapd /etc/hostapd/hostapd.conf
~~~
Hit Ctrl+C to cancel the test!

DHCP and Hostapd Test:
~~~
sudo service hostapd start
sudo service isc-dhcp-server start
~~~
Check their status now:
~~~
sudo service hostapd status
sudo service isc-dhcp-server status
~~~
 Make them Auto-Boot on startup:
~~~
 sudo update-rc.d hostapd enable
 sudo update-rc.d isc-dhcp-server enable
~~~


**What are you waiting for?**
~~~
sudo reboot
~~~  
