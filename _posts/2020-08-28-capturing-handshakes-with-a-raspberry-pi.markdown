---
title:  "Capturing handshakes with a Raspberry Pi"
date:   2020-08-28 22:25:35 
categories:
  - Blog
tags: 
  - Security 
  - Homelab
---

# Intro to pwnagotchi
This summer I worked at a ISP and was talking about different Raspberry Pi projects with a collegue who introduced me to this amazing project called [pwnagotchi](https://pwnagotchi.ai/). The project is a small tamagotchi based on a Rasperry Pi Zero W to capture handshakes between a router and wireless clients connected to it. By capturing these handshakes one can then crack the password for the Wi-Fi. Later that afternoon after reading up on it we both ordered the parts needed, as we both only had the Pi Zero itself and really wanted a screen for the cute ASCII face. 

![Pwnagotchi with temporary back case](https://dvardoo.github.io/images/pwnagotchi/pi2.jpeg "Pwnagotchi with temporary back case")
{: .full}

## Thoughts on project
The project is a great way to learn about the basics of hacking Wi-Fi and cracking passwords while also being forced to take som walks for finding new SSID's. 

One can use the pwnagotchi both for learning, competing with you're collegues who can capture the most handshakes or to recon interesting handshakes in pentesting scenarios.  

I actually learned quite a bit about hacking Wi-Fi and cracking passwords with some hands on experience instead of theoretical studies. This was also my first time soldering, when I soldered the headers for the GPIO (and burned myself badly haha) which in itself was a lesson!

## Parts needed for the project
1. Raspberry Pi Zero W.
2. [Inky pHAT eInk display](https://shop.pimoroni.com/products/inky-phat?variant=12549254938707)
3. [Case for the Pi and display](https://thepihut.com/collections/raspberry-pi-cases/products/pi-zero-case-for-waveshare-2-13-eink-display)
4. Random powerbank or battery pack (just make sure that it's compatible with the case if it's a battery pack). 

## Steps taken

1. Download and burn the image.
- Create file for SSH.
- Edit config.txt for ethernet over USB.
	* Append "*dtoverlay=dwc2*" to file.  

2. To enable inky screen one can edit the config file over SSH, or simply enable webconfig and then insert the correct value depending on the screen.
3. For fixing IP address and to SSH to the USB interface download the [linux_connection_share.sh](https://github.com/evilsocket/pwnagotchi/blob/master/scripts/linux_connection_share.sh) and edit it depending on enviroment.
- Connect the pwnagotchi via USB.
	* Run `dmesg` to find the new USB interface.
	* Edit the script with the name of the interface.
	* Run the script.
	* Connect to the SSID of the pwnagotchi.
	* SSH to the pwnagotchi

3. If no screen is available or if it's just more discreet one can hook up the pwnagotchi via bluetooth tethering to watch the UI on a smartphone screen (as it looks sort of suspicious staring at a PCB with a screen connected to a power bank). 
* Check the MAC address of the smartphone.
* SSH to the pwnagotchi and follow these steps:
	* Run `bluetoothcli show` to find nearby bluetooth enabled devices and copy the MAC address of the phone.
	* Run `bluetoothcli pair [MACADDRESS]` to pair the smartphone and pwnagotchi.
	* Run `bluetoothcli trust [MACADRESS]` to trust the device for future connections.
	* Run `bluetoothcli connect` to finnaly connect to the smartphone.
* Start tethering from the smartphone and wait for the pwnagotchi to connect.
* Use a browser to navigate to the webUI.

![The webUI of the pwnagotchi](https://dvardoo.github.io/images/pwnagotchi/pi3.jpg "The webUI of the pwnagotchi")
{: .full}

4. For easier configuration instead of a shaky SSH connection one can make changes via the webUI. 
* Connect to the IP address and port of the pwnagotchi.
	* Standard credentials are "changeme" and "changeme" (which you actually should change).
* Click on plugins.
	* Enable webconf with the slider.
	* Reboot the pwnagotchi.

5. I had to wait for the parts to start assembly as I just had the Pi Zero initially and ran it headless for the first weeks. And these steps are just for looks and pretty self explanatory.
* Connect the inky PHAT to the GPIO pins.
* Assemble the case (which is harder than it looks if you are eager and careless). 

6. **Take some walks and feed that poor soul some handshakes.**

![The final case for the pwnagotchi](https://dvardoo.github.io/images/pwnagotchi/pi1.jpeg "The final case for the pwnagotchi")
{: .full}
