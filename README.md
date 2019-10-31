## piBox (Raspberry Pi 4 seedbox using the automation script from https://swizzin.ltd)

#### Add mounted device as option for disk status displayed on the swizzin web panel

First of all I am using a Raspberry Pi 4 (4GB version) as a seedbox running Raspbian Buster. So, I edited the swizzin setup.sh script to include Raspbian as an acceptable OS. The setup took about 50 minutes and ran with only a couple of minor errors. Nothing that broke the apps from working, at least not any of the app options that I chose.

## If you would like to run the swizzin automation script on your Raspberry Pi (Raspbian OS) then you will need to do the following:
#### download the swizzin "setup.sh" file and edit "setup.sh":
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

After doing this you should elevate to root by typing "sudo su" and then run the automated install by typing "./setup.sh". Be aware that the install takes a while on the Raspberry Pi and does require some user interaction. Good luck!

### Purpose of this repository

For the most part this is a clone of https://github.com/liaralabs/quickbox_dashboard which is the web UI for swizzin. I may decide to do more customization to the web panel but as of right now I have only added the option for a mounted device to be shown in the Disk Status portion of the swizzin web UI.

Now, if you're using a Raspberry Pi like I am then you're aware that the Pi runs the OS from an SD Card. The root/home directory is on the SD card, so the swizzin panel shows the size of directories/partitions that are only on the SD card. I made this because I wanted the disk status to show me an external storage device that I use for my seedbox and I made it so that anyone who wanted to see their mounted device could use it for the same purpose.

I created a file: widgets/disk_datam.php which is the file that is to be remotely downloaded by the fix-disk command that will essentially fix the disk status in the web ui to show Total/Free/Used disk space of a mounted device.

So, first we have to modify the box command so that it will read another variable.

#### Edit "/etc/swizzin/scripts/box" file so that your panel function looks like this:
```
function _panel () {
  if [[ $2 != "fix-disk" ]]; then
    echo "fix-disk is the only acceptable panel argument at this time"
    exit 1
  fi
  /usr/local/bin/swizzin/panel/fix-disk $3 $4
}
```
Next we need to modify the fix-disk command to add "mnt /devicePath/" as an available option.

#### Edit "/etc/swizzin/scripts/panel/fix-disk" file and replace all the text with the following:
```
#!/bin/bash
#Disk Widget Switcher mnt <->  root <-> home
if [[ -z $1 ]]; then
  echo "You must specify mnt, root or home"
  exit 1
fi

if [[ $1 == "home" ]] && [[ -z $2 ]]; then
  rm -f /srv/panel/widgets/disk_data.php
  wget -O /srv/panel/widgets/disk_data.php https://raw.githubusercontent.com/liaralabs/quickbox_dashboard/master/widgets/disk_datah.php > /dev/null 2>&1
  chown www-data: /srv/panel/widgets/disk_data.php
elif [[ $1 == "root" ]] && [[ -z $2 ]]; then
  rm -f /srv/panel/widgets/disk_data.php
  wget -O /srv/panel/widgets/disk_data.php https://raw.githubusercontent.com/liaralabs/quickbox_dashboard/master/widgets/disk_data.php > /dev/null 2>&1
  chown www-data: /srv/panel/widgets/disk_data.php
elif [[ $1 == "mnt" ]] && [[ ! -z $2 ]]; then
  mntPth=$2
  echo "mntDevicePath=${mntPth}" > /srv/panel/inc/diskStatus.cfg
  rm -f /srv/panel/widgets/disk_data.php
  wget -O /srv/panel/widgets/disk_data.php https://raw.githubusercontent.com/GostLy/piBox-quickboxDashboard/master/widgets/disk_datam.php > /dev/null 2>&1
  chown www-data: /srv/panel/widgets/disk_data.php
  chown www-data: /srv/panel/inc/diskStatus.cfg
else
 # echo "1: ${1} 2: ${2}"
  echo "You must specify root, home, or mnt. When the option is mnt, be sure to include the mounted storage device path as well. (ex: box panel fix-disk mnt \"/mnt/piStorage\")"
  exit 1
fi

service nginx reload
/usr/local/bin/swizzin/php-fpm-cli -r 'opcache_reset();'
```

#### After making the above change to box and fix-disk, elevate to root by typing 'sudo su' in a terminal
#### Then run the box command like this:
```
box panel fix-disk mnt /mnt/piStorage
```
#### "/mnt/piStorage" is where you should replace with your mounted device folder

### Now from the swizzin web panel it should show you the disk status of your mounted device!