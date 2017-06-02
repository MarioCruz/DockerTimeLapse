Here is the build and the steps we took to get this project up and running.

Materials:

- One (1) Raspberry Pi 3 Model B
- One (1) Raspberry Pi Camera Module v2
- One (1) [WD PiDrive BerryBoot Edition 1TB](https://www.wdc.com/products/wdlabs/wd-pidrive-berryboot-edition-1tb.html)
- One (1) [5V 3A adapter](https://www.amazon.com/gp/product/B00L88M8TE/ref=oh_aui_detailpage_o09_s00?ie=UTF8&amp;psc=1)

Steps:

1. Download [Raspbian Jessie Lite](https://www.raspberrypi.org/downloads/raspbian/)
2. Image the SD card and insert into your Raspberry Pi 3
3. Update your Raspberry Pi 3

```
 - sudo apt-get update -y
 - sudo apt-get upgrade –y
 - sudo apt-get install rpi-update
 - sudo reboot
 ```   
2. Enable SSH on the PI
  ```
  In a terminal window : sudo raspi-config
   Select Interfacing Options
   Navigate to and select SSH
   Choose Yes
   Select Ok
   Choose Finish
```
1. Get your Pi online, either via WiFi or Ethernet

-- For Wifi:
-- sudo nano /etc/wpa\_supplicant/wpa\_supplicant.conf
```
network={
ssid="YOUR_NETWORK_NAME"
  		psk="YOUR_NETWORK_PASSWORD"
key_mgmt=WPA-PSK
}

```
1. Prepare the PiDrive Volume
```
  sudo fdisk /dev/sda (in my case, the WD PiDrive is /dev/sda)
  sudo mkfs.ext4 /dev/sda1 -L &quot;data&quot;
  sudo mkdir /mnt/data (mount the partition)
  sudo mount -t ext4 -o &quot;noatime&quot; /dev/sda1 /mnt/data
  sudo chown -R 1000:1000 /mnt/data
  sudo chmod -R 775 /mnt/data
```
1. Auto mount the drive after every reboot
```
  sudo mkdir /mnt/data (if not already present)
  sudo nano /etc/fstab
```
 Add this entry in this file
  ```
  LABEL=Data /mnt/data ext4 defaults,noatime,nofail 0 2
  ```
  Save
```
<ctrl-x>, <y>, <enter>
```

1. Reboot

1. Make sure everything works and that you can see the data drive mount
```
  1. df -h
```
1. Follow steps 1-6 from [Build a Timelapse Rig with your Raspberry Pi](http://blog.alexellis.io/raspberry-pi-timelapse/)

1. Test the your camera is working

1. raspistill -o cam.jpg

1. If the output from raspistill gave you an error,

 
Check Step 1 in Alex's guide **Enable RPi camera**

Check that the cables the silver connectors are facing the HDMI port and the blue part faces the USB

1. Start Docker
```
docker run --restart=always --privileged -e TZ=&quot;US/Central&quot; -v `pwd`/phototimer/config.py:/root/images/config.py -v /mnt/data:/var/image --name cam -d alexellis2/phototimer
```
1. One of the things we noticed was by default the container runs By adding the -e TZ=&quot;US/Central&quot; parameter, we can change the timezone in the container to be what we want. *** Alex has updated his notes to fix this ***

1. Install ngrok. This will allow us to SSH into the box from anywhere, even if from behind a firewall.

  1. Create an account on Ngrok https://dashboard.ngrok.com/user/login
  2. sudo wget https://dl.ngrok.com/ngrok\_2.0.19\_linux\_arm.zip
  3. unzip ngrok\_2.0.19\_linux\_arm.zip
  4. ./ngrok &quot;yourauthtoken&quot; (from the online portal)
  5. ./ngrok tcp 221

1. Have ngrok load on reboot
  1. Sudo nano /home/pi/start.sh
  2. Add this to this new file:  /usr/local/bin/ngrok tcp 22
  3. Crontab -e
  4. Add to the bottom: @reboot /home/pi/start.sh
