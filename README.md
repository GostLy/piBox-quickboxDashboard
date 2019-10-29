# piBox (Raspberry Pi 4 seedbox using the automation script from https://swizzin.ltd)

#### Customized pages for the swizzin quickbox panel

First of all I am using a Raspberry Pi 4 (4GB version) as a seedbox running Raspbian Buster. So, I edited the swizzin setup.sh script to include Raspbian as an acceptable OS. The setup took about 50 minutes but ran with only a couple of minor errors that I was able to fix/clean up after the setup finished. For the most part the seedbox has been running pretty smoothly for over 1 week now.

## If you would like to run the swizzin automation script on your Raspberry Pi (Raspbian OS) then you will need to do the following:
### download the swizzin "setup.sh" file and edit "setup.sh":
```
sudo wget https://raw.githubusercontent.com/liaralabs/swizzin/master/setup.sh
sudo chmod +x setup.sh
sudo nano setup.sh
```
#### find the following line:
```
if [[ ! $distribution =~ ("Debian"|"Ubuntu") ]]; then
```
#### replace with the following:
```
 if [[ ! $distribution =~ ("Debian"|"Ubuntu"|"Raspbian") ]]; then
```

After doing this you should elevate to root by typing "sudo su" and then running the automated install by typing "./setup.sh", install takes a while on the Raspberry Pi and does require some user interaction. Good luck!

#### Purpose of this repo

Since the Raspberry Pi runs the OS from an SD Card. The root/home directory is on the SD card, so the quickbox panel shows the size of the SD card. Although I have a storage device mounted that holds all of my torrent files and downloads, so I would like to have the panel tell me the disk space of the mounted device. 

I created a file: widgets/disk_datam.php which is an edited version of widgets/disk_data.php (only lines 69-74) to add the mounted storage device directory. (/mnt/piStorage)

I then had to modify the "box panel fix-disk" command by adding "mnt" as an option which will then use my custom "widgets/disk_datam.php" file:

### "/etc/swizzin/scripts/fix-disk" file:
```
#!/bin/bash
#Disk Widget Switcher mnt <->  root <-> home
if [[ -z $1 ]]; then
  echo "You must specify mnt, root or home"
  exit 1
fi

if [[ $1 == "home" ]]; then
  rm -f /srv/panel/widgets/disk_data.php
  wget -O /srv/panel/widgets/disk_data.php https://raw.githubusercontent.com/liaralabs/quickbox_dashboard/master/widgets/disk_datah.php > /dev/null 2>&1
  chown www-data: /srv/panel/widgets/disk_data.php
elif [[ $1 == "root" ]]; then
  rm -f /srv/panel/widgets/disk_data.php
  wget -O /srv/panel/widgets/disk_data.php https://raw.githubusercontent.com/liaralabs/quickbox_dashboard/master/widgets/disk_data.php > /dev/null 2>&1
  chown www-data: /srv/panel/widgets/disk_data.php
elif [[ $1 == "mnt" ]]; then
  rm -f /srv/panel/widgets/disk_data.php
  wget -O /srv/panel/widgets/disk_data.php https://raw.githubusercontent.com/GostLy/piBox-quickboxDashboard/master/widgets/disk_datam.php > /dev/null 2>&1
  chown www-data: /srv/panel/widgets/disk_data.php
else
  echo "You must specify mnt, root or home"
  exit 1
fi

service nginx reload
/usr/local/bin/swizzin/php-fpm-cli -r 'opcache_reset();'
```