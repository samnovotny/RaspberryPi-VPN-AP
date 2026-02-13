# RaspberryPi-VPN-AP
Instructions to configure any old RaspberryPi (with ethernet and WiFi) to be an access point.

## Install clean OS

[Raspberry Pi documentation](https://www.raspberrypi.com/documentation/computers/getting-started.html)

## Update to latest versions

`sudo apt update`  
`sudo apt upgrade`

## Configure a static IP for the eth0 interface
### (Choose appropriate values, must match local lan, e.g.192.168.x.x)

Use **nmtui** to set the static addresses (ip, gateway and DNS server) **and** enable wifi radio

## Create hotspot (access point)

Use the following **nmcli** command to create a hotspot

`sudo nmcli d wifi hotspot ifname wlan0 ssid <SSID> password <PASSWORD>`  

Use **nmtui** to set hotspot to start automatically.

Restart NetworkManager

`sudo systemctl restart NetworkManager`  

## Install Mullvad VPN

Follow instructions for Linux from [Mullvad website](https://mullvad.net/en/help/easy-wireguard-mullvad-setup-linux/).

`sudo apt install openresolv wireguard`  

Secure copy (scp) files from Mac to Raspberry  

`scp -r mullvad_wireguard_linux_gb_all pi@192.168.1.123:/home/pi`  

Then copy vpn configuration file to the Wireguard directory

`sudo cp gb-lon-wg-002.conf /etc/wireguard`  
`sudo chown root:root -R /etc/wireguard && sudo chmod 600 -R /etc/wireguard`  

Check vpn working.

`sudo su`  
`cd /etc/wireguard`  
`wg-quick up gb-lon-wg-002`  
`curl https://am.i.mullvad.net/connected` or `wg`  
`wg-quick down gb-lon-wg-002`  

Start the service automatically

`sudo systemctl enable wg-quick@gb-lon-wg-002.service`  
`sudo systemctl daemon-reload`  
`sudo systemctl start wg-quick@gb-lon-wg-002`  

### Reboot

`sudo systemctl reboot`  
