# Wifi Repeater
## A DIY wifi repeater using a Raspberry Pi zero

#This is a test change to check if updates are being committed.

The goal of this project was to understand a bit more about how wifi routers work. There are some great tutorials of how to build a router using a raspi but I struggled to find a functional **router** using a raspberry pi zero. This would be handy and in principle quite affordable to since the rapi zero is just $5 and a cheap wifi card is about £10.

The raspi has an onboard wifi antenna and it is [apparently very good]() with nanostructured antennae to add directionality. You will need to buy an extra usb wifi card and I bought an XXX? SO one wifi acts as the receiver and another the transmitter. 

## Build: Copy the image

I made a disk image and all you need to do is clone this repo. 

`git clone https://github.com/tmopencell/wifirepeater.git`  

`cd wifirepeater` 

## For Mac Users 

Type `diskutil list` and note the names of the drives in the form `/dev/disk**X**`. Then write down the size of the SD card you will be using. Insert the SD card into your reader on your laptop. Now type `diskutil list` and look for the drive that was not there previously, for example: `/dev/disk3`. Look at the file size, does it roughly match the size that you recorded previously? If so you are good to proceed. Copy the drive number.  

**NOTE:** this may seem a bit overkill but you want to make sure you are sure you are overwriting the right drive.  

Now unmount the drive by typing: 

`diskutil unmountDisk /dev/disk**X**`  where **X** is the drive number you recorder previously.

Now type `sudo dd if=rasppizero_wifirepeater.dmg of=/dev/disk**X** bs=1m`  (insert the drive number for **X**). This will take a bit of time, maybe 10-15 mins and there will be nothing displayed meanwhile. At the end of the process you will see something like this: 

```
dd: /dev/disk3: Resource busy
4087880+0 records in
4087880+0 records out
2092994560 bytes transferred in 1332.643448 secs (1570559 bytes/sec)
```

This is good, you were successful! Now pop the SD card into your raspi and insert a usb wifi adaptor.

## Startup your Raspi

The system can be accessed normally by logging in with a screen and entering your password etc. 

However, you can also use **headless** login. The system is set up to launch a wifi network right away! **SSID: Pi-Extender** with **passcode: raspberry**. It also has fixed IP addresses the Pi-Extender network is **192.168.220.1** and for the network you attempt to connect to it is 192.168.0.2 (**NOTE** you may need to edit this depending on your network). 

To login connect to the Pi-Extender network. Once connected type: `ssh pi@192.168.220.1` and enter the username+password:
```
username: pi
password: raspberry
```

Then you will need to set up your wifi credentials for your current network.

Type `sudo nano /etc/wpa_supplicant/wpa_supplicant.conf` and you will see:


```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=GB

network={
        ssid=""
        psk=""
} 
```

Put your details into the quotation marks, it should look like this:

```

ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=GB

network={
        ssid="yourwifiname"
        psk="yourwifipassword"
}
```


To check if oyu are connected type `sudo ifconfig`

You should see something like this:
```
~ sudo ifconfig
lo: flags=7XX<UP,LOOPBACK,RUNNING>  mtu XXXXX
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 2079  bytes 195277 (190.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2079  bytes 195277 (190.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan0: flags=XXXX<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.220.1  netmask 255.255.255.0  broadcast 192.168.220.255
        inet6 XXXX::XXXX:XXXX:XXXX:XXXX  prefixlen 64  scopeid 0x20<link>
        ether XX:XX:XX:XX:XX:XX  txqueuelen 1000  (Ethernet)
        RX packets 1033126  bytes 1447522803 (1.3 GiB)
        RX errors 0  dropped 3  overruns 0  frame 0
        TX packets 592482  bytes 75293703 (71.8 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.2  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 XXXX::XXXX:XXXX:XXXX:XXXX  prefixlen 64  scopeid 0x20<link>
        ether XX:XX:XX:XX:XX:XX  txqueuelen 1000  (Ethernet)
        RX packets 614913  bytes 63994832 (61.0 MiB)
        RX errors 0  dropped 9  overruns 0  frame 0
        TX packets 1028656  bytes 1481624145 (1.3 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```  



 If you type `ping google.com` and you should see something like this:  

```
ping google.com
PING google.com (XXX.XX.XX.XX) 56(84) bytes of data.
64 bytes from XX-in-YY.net (XXX.XX.XXX.XX): icmp_seq=1 ttl=52 time=30.4 ms

^X64 bytes from XX-in-YY.net (XXX.XX.XXX.XX): icmp_seq=5 ttl=52 time=268 ms

--- google.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4008ms
rtt min/avg/max/mdev = 9.316/65.574/268.818/101.945 ms
```  

hold the `ctrl` and `c` buttons to exit this.

If you are not connected then restart the dnsmasqing and networking (NOTE: this will kick you out of the system and you will need to login again). Type:

`sudo service dnsmasq restart`
`sudo service networking restart`  

Reconnect to the wifi network **Pi-Extender** and check if you are connected to the net!  


## (Optional)

You may want to edit the IP address(es) and if you choose to then type: 

`sudo nano /etc/network/interfaces`

## Basic Security: 
I am not an expert so please don't take my word for this. The default login is:
```
username: pi
password: raspberry
```

Please change the password by typing `sudo raspi-config` and select **change user password**

I also like to use an ssh key for login and to disable password login altogher. You can read about how to do this [here](https://raspi.tv/2012/how-to-set-up-keys-and-disable-password-login-for-ssh-on-your-raspberry-pi)