# DockerTimeLapse
Here is the build and the steps we took to get this project up and running.

Materials:
One (1) Raspberry Pi 3 Model B
One (1) Raspberry Pi Camera Module v2 
One (1) WD PiDrive BerryBoot Edition 1TB
One (1) 5V 3A adapter 
Steps:
 
Download Raspbian Jessie Lite
Image the SD card and insert into your Raspberry Pi 3
Update your Raspberry Pi 3
sudo apt-get update -y
sudo apt-get upgrade –y
sudo apt-get install rpi-update
sudo reboot
Enable SSH on the PI
In a terminal window : sudo raspi-config
Select Interfacing Options
Navigate to and select SSH
Choose Yes
Select Ok
Choose Finish
Get your Pi online, either via WiFi or Ethernet 
For Wifi:
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
		network={
ssid="YOUR_NETWORK_NAME"
  		psk="YOUR_NETWORK_PASSWORD"
key_mgmt=WPA-PSK
}

Prepare the PiDrive Volume
sudo fdisk /dev/sda (in my case, the WD PiDrive is /dev/sda)
sudo mkfs.ext4 /dev/sda1 -L "data"
sudo mkdir /mnt/data (mount the partition)
sudo mount -t ext4 -o "noatime" /dev/sda1 /mnt/data
sudo chown -R 1000:1000 /mnt/data
sudo chmod -R 775 /mnt/data

Auto mount the drive after every reboot
sudo mkdir /mnt/data (if not already present)
sudo nano /etc/fstab
Add this entry in this file
LABEL=Data /mnt/data ext4 defaults,noatime,nofail 0 2
Save
<ctrl-x>, <y>, <enter>
Reboot
Make sure everything works and that you can see the data drive mount
df -h
Follow steps 1-6 from Build a Timelapse Rig with your Raspberry Pi
Test the your camera is working
raspistill -o cam.jpg
If the output from raspistill gave you an error,
Check Step 1 Enable RPi camera
Check that the cables the silver connectors are facing the HDMI port and the blue part faces the USB
Start Docker
docker run --restart=always --privileged -e TZ="US/Central" -v `pwd`/phototimer/config.py:/root/images/config.py -v /mnt/data:/var/image --name cam -d alexellis2/phototimer 
One of the things we noticed was by default the container runs UTC. By adding the -e TZ="US/Central" parameter, we can change the timezone in the container to be what we want.






Install ngrok. This will allow us to SSH into the box from anywhere, even if from behind a firewall.
Create an account on Ngrok https://dashboard.ngrok.com/user/login 
sudo wget https://dl.ngrok.com/ngrok_2.0.19_linux_arm.zip 
unzip ngrok_2.0.19_linux_arm.zip
./ngrok “yourauthtoken” (from the online portal)
./ngrok tcp 221
       
Have ngrok load on reboot
Sudo nano /home/pi/start.sh
Add this to this new file:  /usr/local/bin/ngrok tcp 22 
Crontab -e 
Add to the bottom: @reboot /home/pi/start.sh
