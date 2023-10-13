# RaspberryPi-VPN-AP
Instructions to configure any old RaspberryPi (with ethernet and WiFi) to be an access point.

## Install clean OS

[Raspberry Pi documentation](https://www.raspberrypi.com/documentation/computers/getting-started.html)

## Update to latest versions

`sudo apt-get update`  
`sudo apt-get upgrade`

## Configure a static IP for the eth0 interface
### (Choose appropriate values, must match local lan)

`sudo nano /etc/dhcpcd.conf`

And set the following:

`# Static IP eth0 configuration:`  
`interface eth0`  
`static ip_address=192.168.0.10/24`  
`static routers=192.168.0.1`  
`static domain_name_servers=192.168.0.1`  

## Install Mullvad VPN

Follow instructions for Linux from [Mullvad website](https://mullvad.net/en/help/easy-wireguard-mullvad-setup-linux/).

## Configure wireless access point routing

Note: remember the name of vpn channel created above, eg: **gb-lon-wg-002**

`sudo iptables -t nat -A POSTROUTING -o gb-lon-wg-002 -j MASQUERADE`  
`sudo iptables -A FORWARD -i gb-lon-wg-002 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT`  
`sudo iptables -A FORWARD -i wlan0 -o gb-lon-wg-002 -j ACCEPT`  
`sudo apt-get install -y iptables-persistent`  

## Make wireless access point

`sudo apt install hostapd`  
`sudo systemctl unmask hostapd`  
`sudo systemctl enable hostapd`  
`sudo apt install dnsmasq`  

### Edit DHCP configuration:

`sudo nano /etc/dhcpcd.conf`  

Add the following at the end:

`interface wlan0`  
`static ip_address=192.168.4.1/24`  
`nohook wpa_supplicant`  
    
### Create routing file:

`sudo nano /etc/sysctl.d/routed-ap.conf`  

Add:

`# Enable IPv4 routing`  
`net.ipv4.ip_forward=1`  

### One more firewall change:

`sudo iptables -t nat -A POSTROUTING -o gb-lon-wg-002 -j MASQUERADE`  
`sudo netfilter-persistent save`  

### Configure DNS

`sudo nano /etc/dnsmasq.conf`  

Adding: 

`interface=wlan0 # Listening interface`  
`dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h`  
                `# Pool of IP addresses served via DHCP`  
`domain=wlan     # Local wireless DNS domain`  
`address=/gw.wlan/192.168.4.1`  
                `# Alias for this router`  

### Ensure WiFi radio is unblocked:

`sudo rfkill unblock wlan`  

### Create config for the access point

`sudo nano /etc/hostapd/hostapd.conf`  

Add the following, choosing your own SSID and password:

`interface=wlan0`  
`ssid=`**My SSID**  
`hw_mode=g`  
`channel=7`  
`macaddr_acl=0`  
`auth_algs=1`  
`ignore_broadcast_ssid=0`  
`wpa=2`  
`wpa_passphrase=`**My Password**  
`wpa_key_mgmt=WPA-PSK`  
`wpa_pairwise=TKIP`  
`rsn_pairwise=CCMP`  

### Reboot

`sudo systemctl reboot`  
