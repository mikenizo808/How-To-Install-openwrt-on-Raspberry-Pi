# How-To-Install-openwrt-on-Raspberry-Pi

## Intro
In this write-up we discuss using a `Raspberry Pi` to run a network router for your home or lab using the open source router `openwrt`.  The router os will get installed like a regular `Raspberry Pi` image onto the SD card. Then, you can login to the `openwrt` web interface and enable WIFI to reach your ISP.

Also, we configure our ethernet connection as well, since all of your internal devices that connect to that router (i.e. via a "5 port netgear hub"), will then get an IP Address from DHCP (issued by `openwrt`). Once a client has an address it can surf the net across your Pi router!

*Note: For this setup we need to connect to our ISP across WIFI so our ethernet can be dedicated to routing our client traffic (this happens automatically when a device gets DHCP from openwrt).*

## Your ISP
You can use any ISP or hardware. For reference though, I am using Verizon as my ISP, and connect my `Raspberry Pi` (running `openwrt`) via WIFI, to a hardware device called `MIFI` (a mobile hotspot available from Verizon).

## About Configuring `openwrt` (GUI or CLI)
You can use the GUI to do everything and that is the recommendation for this setup. We do discuss the location of config files if interested, but this is not required.


## Motivation
The motivation for using `openwrt` came from "Network Chuck" on YouTube and his video "my SUPER secure Raspberry Pi Router (wifi VPN travel router)"

	https://youtu.be/jlHWnKVpygw

## What is `openwrt`
`OpenWrt` is an open source operating system that is designed to be a network router. This gets installed onto an SD card just like we normally would for a `Raspberry Pi`.

  https://openwrt.org
  
## Downloading
To get the bits for the `Rapberry Pi` you will need to know your model (i.e. 2, 3 or 4) and then select the `ext` image from the "firmware selector" page:

  https://firmware-selector.openwrt.org/
  
*Note: DO not place the bits onto your SD card yet, until reading the next section.*

## Optional - Firmware Update your `Raspberry Pi`
Before preparing your SD with `openwrt` you can optionally update the `Raspberry Pi` firmware by running a regular `Raspberry Pi` operating system first.

The `Raspberry Pi` firmware is applied during our regular `apt` updates, but only when running an official `Raspberry Pi` release such as `bullseye` or similar.

	## update package list
	sudo apt-get update

	## update the packages (this also does firmware)
	sudo apt-get -u upgrade


## Optional - Firmware Only
If you only want the `Raspberry Pi` firmware perform the following:

	## update firmware
  	sudo rpi-eeprom-update
  	
	## show cpu details
  	cat /proc/cpuinfo 


## Create an `openwrt` image on SD
Now that the Raspberry Pi firmware is up to date, we can wipe that SD and install `openwrt` onto the SD just like we do for a `Raspberry Pi`.

If you have not already, navigate to `https://openwrt.org` to get the desired download for your `Raspberry Pi`.

Then, use the `Raspberry Pi Imager` to write the image to SD (using the Custom option at the bottom).


## Login to `openwrt`
You can go right to the web interface (`https://192.168.0.1` or you can optionally use `ssh`.

If using `ssh` from terminal, then press "Enter" on your keyboard to bring up the `openwrt` splash screen. Prior to hitting Enter, it may appear like the system is doing something, but it's simply waiting for you to press Enter.


## Default Password
You will probably be logged in already on the first boot, but if needed the default login is `root` with no password.

*Tip: You can optionally set a password now, but do not use any "real" or important password until we add `https` later. Remember that the default of `http` is totally plain text, so your roommate could capture the flag on you.*


## Login to `openwrt` via Web Interface
First, try to connect using `https` which is secure.  It that works, you will get some errors about unsigned certs, but that is expected.  If you cannot reach the page with `https`, then try `http`, which is not secure (but we fix that later).

	#try this first
	https://192.168.1.1

	#use plain text if needed (http)
	http://192.168.1.1


## System Time
When your system boots it will have the date/time of the build itself. This means that your date/time will likely be days off by default.  Once connected to the internet, `ntp` will handle time for us without any work on our part.  However, until we are on the internet, our time will likely be wrong.

This becomes important when we need to get updates for `openwrt` packages (i.e. using `opkg`, the package manager or the web interface).  The problem is that "downloads" will fail via `opkg` if our system time if not correct.  This date/time problem is especially observable on Raspberry Pi, which does not have a CMOS battery to keep the date/time after power off.

The way the `Raspberry Pi` team handled this is they read the last time/date known from the logs, and consider that the system time.  The `openwrt` team handles this the same way.  As you would expect with these limitations, the date/time can easily be multiple days off. Technically, we want to be within 5 minutes of "real" time for best results.


## Sync with Browser
To mitigate the problem of bad time, when using the web interface we can click the button to `sync time with browser`.  However, be sure that the time on your own device (i.e. your desktop/laptop, or another Raspberry Pi) has the correct time itself.  Specifically, if you use the browser time, but your time is wrong, then the `openwrt` device will have the wrong time, and updates for `openwrt` may fail.


## Optional - use `date` command
In `openwrt` (via `ssh`) we can issue the the `date` command to see the current date and optionally set it with `-s`.

	#get date
	date
	
	#set date
	date -s "2022-01-11 10:58:00"


## Web Interface vs.Configuration Files
You can do everything via the web interface (recommended) without even knowing about the config files. However, you might prefer to set the `eth0` manually (`/etc/config/network`) just so the web interface does not freak out at the end of the configuration.


## Configuration Files
Again, not needed since you can do everything in the web interface, but this is the location of the configuration files.

	/etc/config/network
	/etc/config/wireless
	/etc/config/firewall


## Connect to web interface
The default address for `openwrt` is `192.168.1.1`, until you have updated that in the web interface or in the `/etc/config/network` file.  Instead of the default, you will likely configure an address on the `10.x.x.x/24` network or similar.

	https://10.100.0.1  #or http if needed


## Connect to your WIFI
From the web interface, navigate to `Network > Wireless` and click `Scan`.


## Confirm WIFI Connection
Once configured, you can see your WIFI connection at:

	Network > Wireless
	
	-or-
	
	Status > Overview
	

## Enable https
The following enables `https` for the for the web interface of `openwrt`.

	# confirm internet access
	ping google.com
	
	#update package list
	opkg update

	#install https
	opkg install luci-ssl

	#optional sync
	sync; sync
	
	#reboot the system
	reboot


## Set Final Password
Now that we have `ssl` installed, we can use a "real" password when logging into the web interface. We can set the password in the web interface itself, or using the `passwd` command from terminal.

	passwd


## Secure Login to Web Interface
Open a browser and navigate to `https://<your router ip>` and agree to the security warning for using self-signed certs (we can always replace those later). The important bit is that we are now `https` instead of `http` when using the web interface. This means we can safely enter our "real" password now.


## About Updating `openwrt` Packages
We can updates packages in the web interface or via terminal. In both cases, the recommendation is to update one at a time. When using terminal, the `opkg` command is similar to `apt` in that it can download, list, update and install packages.


## Checking for Updates
Once connected to WIFI, we can check for updates using the web interface, or with `opkg update` from terminal.

	opkg update

*Note: If `opkg update` fails, check your system time.*


## Show what can be updated

	opkg list-upgradable


##  Example Output

	root@OpenWrt:~# opkg list-upgradable
	luci-app-opkg - git-21.079.58598-6639e31 - git-21.312.69848-4745991
	iw - 5.9-8fab0c9e-1 - 5.9-8fab0c9e-3
	busybox - 1.33.1-6 - 1.33.2-1
	luci-mod-system - git-21.295.66903-8acd0d7 - git-21.305.74567-388dae9
	luci-theme-bootstrap - git-21.298.68362-d24760e - git-21.320.44446-0cee46b
	wpad-basic-wolfssl - 2020-06-08-5a8b3662-35 - 2020-06-08-5a8b3662-37
	netifd - 2021-07-26-440eb064-1 - 2021-10-30-8f82742c-1
	luci-app-firewall - git-21.295.66767-8eceb63 - git-21.312.70727-3eac573
	luci-base - git-21.295.67054-13df80d - git-21.340.48972-61cc3b1
	luci-mod-network - git-21.295.67048-4d3de0e - git-21.335.80427-5091496
	hostapd-common - 2020-06-08-5a8b3662-35 - 2020-06-08-5a8b3662-37
	wireless-regdb - 2021.04.21-1 - 2021.08.28-1
	root@OpenWrt:~# 


## Apply Updates
We can use the web interface to do updates, or use the terminal.  If using the web interface, we need to click updates one at a time, and be okay with getting no feedback. Or do it from the terminal like boss.

	opkg upgrade <name of package>


## Yolo Mode - Update all packages
You should really update things one at a time, but if you want to go all out perform the following.

This is not recommended, but you can optionally run the following until failure and then reboot. Eventually you will have all packages.

	#Update package list (required after any reboot)
	opkg update
	
	#Yolo Mode - Install All Updates (until failure)
	opkg list-upgradable | cut -f 1 -d ' ' | xargs -r opkg upgrade

	#After failure
	sync; sync; reboot

*Note: Credit for yolo mode https://unix.stackexchange.com/questions/400231/how-do-i-upgrade-all-of-my-installed-packages-in-openwrt*

## Handling `opkg update` Problems
To update with `opks` requires good name resolution and good time sync.  Assuming you have those done right and a package is still failing to download a signature during `opkg update` then you can try to `wget` the package manually just to wake the network up so to speak.  You will not use the download, but if it succeeds, then you can try your `opkg update` again.

## Example Error
All other packages (not shown) were fine, except for this one.

	Collected errors:
 	* opkg_download: Failed to download https://downloads.openwrt.org/releases/21.02.1/packages/aarch64_cortex-a72/routing/Packages.gz, wget returned 4.
 	* opkg_download: Check your network settings and connectivity.

## Example download "wake-up"

	root@DesktopUSB:~# wget -O package.gz https://downloads.openwrt.org/releases/21.02.1/packages/aarch64_cortex-a72/routing/Packages.gz
	Downloading 'https://downloads.openwrt.org/releases/21.02.1/packages/aarch64_cortex-a72/routing/Packages.gz'
	Connecting to 168.119.138.211:443
	Writing to 'package.gz'
	package.gz           100% |*******************************| 12295   0:00:00 ETA
	Download completed (12295 bytes)
	root@DesktopUSB:~# 

## Finally

	opkg update

*Note: Once you have `opkg update` working, scroll up in this guide and learn how to install the available packages, if you have not already.*

## Optional - Install `open-vpn`
You can run without using vpn, but this would be a nice to have. Using VPN may require more work depending on what vendor (if any) you use for your VPN service.

	## optional - install openvpn
	opkg install openvpn-openssl

	## optional - install luci gui app for vpn
	opkg install luci-app-openvpn

## Summary
In this write-up we got you up and running with the `openwrt` router on `Raspberry Pi` hardware. Big thanks to Network Chuck for the motivation. See his excellent video first linked above for a guided tour of `openwrt`. The only thing we do different is we do not need any extra WIFI drivers since we use the on-board `Rapsberry Pi` WIFI to connect to our ISP, and then our "lab" devices connect to `openwrt` via ethernet (by simply being connected to the same physical hub).
