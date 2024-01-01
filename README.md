# Work in progress - do not use for now !!!
##############################################

# Unite PiFinder & Stellarmate[PiFinder](https://www.pifinder.io/) is one of the coolest astronomy projects out there besides [Stellarmate](https://stellarmate.com). 
In this project I describe, how to add the PiFinder software to an existing Stellarmate.

Main sources for PiFinder are:
- Main page: https://www.pifinder.io/
- Github: https://github.com/brickbots/PiFinder
- Documentation: https://pifinder.readthedocs.io/en/release/index.html


# Install and use PiFinder with Stellarmate 

## Preface

As of writing this (December 2023), this project is done with
- Raspberry PI 4 (4GB) - tested with pure SD card only and PiFinder hat, but also with additional setup (see here: [YAACS](https://github.com/apos/case_system_stellarmate_astroberry) )
- Fresh install (! no upgrade path !) of stable branch of Stellarmate 1.8.0 (64 bit) which is based on debian bookworm
- PiFinder Software Version 1.7.4 (64 bit)

Please be aware, this procedure will NOT WORK WITH a 32bit version! 

Hint: If you updated Stellarmate from an early version (earlier than 1.70), it is very likely, that you are running a 32 bit version and an older system (buster). The architecture will not upgrade, just the software. You than have to get an actual version. You can simply copy over your KStars directory. 

STOP: at the end in the troubleshooting section I write, how to get this information. Please check, before you procede with to check your environment:

     sudo inxi -Fxz

NOW YOU CAN READ ON :-)

Stellarmate 1.8.0 is actually on debian bookworm (as of Dec 2023). I test with a fast Sandisk Extreme 64GB SD card an then copy that back to a 1TB NVME-

![image](https://github.com/apos/pifinder_tweaks/assets/456034/f9679771-bf34-4d27-b1da-b191dbb25dae)
![image](https://github.com/apos/pifinder_tweaks/assets/456034/d01bca60-bc64-4069-a82e-dc9eeb59a2a0)

There is also a [thread for this project on the Discord channel](https://discord.com/channels/1087556380724052059/1179949372847423489) of the PiFinder project. You''ll find the discord invitation on the [PiFinder homepage](https://www.pifinder.io).

This process might be something for the experienced user, but it is straight forward and probably you will learn a lot on how the underlying debian linux is working. 

Absolute mandatory skills therefore are the following ones. If you do not have these skills, have not fear, you can get them. The internet is full of help. Or you probable get real physical help from an experienced user. I know this is a long list, but better be prepared in advance and know what to expect, than go into a dead end and loose your Stellarmate installation and setup. 

1. a working internet connection and network cable (best, do not use wlan for this - you can, but it can be slow or troublesome)
1. know how to at least start and configure the network of stellarmate
1. work with the commandline in linux. There is not graphical interface for this procedure. Therefore having Putty or a terminal (WSL, Powershell) accessible. Working with VNC is not recommended, but possible though 
2. know how to ssh into stellarmate (knowing how to work with ssh). Therefore you need a network connections to stellarmate.
3. stellarmate must have an internet connection. Simmplest is, to connect a network cable from your router to stellarmate, so it will be automatically 
4. work with an editor and alter files with sudo and safe them (default editor is "nano" or, you can install vim)
5. know how to work with sudo and what this means
6. be able to navigate in dialogs with the keyboard (Tab, Enter, arrow keys, ...)
7. have a good quality and speedy second SD card (e.g. Sandisk Extreme 32GB). You do not want to tinker around with a faulty or full SD card. This is not fun and can lead into real trouble, believe me.
8. know how to make a copy of your sd card (stellarmate -> SD card copier). 
9. know how to shutdown and reboot the pi (sudo shutdown -P now, sudo reboot)
10. investate logs and restart services (using "less" for reading logs, do servicemanagament: sudo service xyz start|stop|status)

I also recommend to 
- label your SD card and sticks you create, so you know which one is which
- create an extra backup of your recent Stellarmate installation with the SD card copier. You should not not touch at all (can be USB stick or SD card).
- always have a bootable stellarmate SD card at hand. You might need it for troubleshooing (e.g. using the SD card copier, altering the Pi settings like booting from USB, mount something to alter files).


And last but not least: have no fear :-) Because we are working with a SD card copy or a fresh install, you can not damage anything. Just try it, I will explain everything as good as possible. But: I am not responsible for any damage on your system, these informations are provides without any support.  


Hint: Do not underestimate the most time consuming parts of this process:

- [10-40 minutes]: Doing an SD card copy of stellarmate. This depends on the amount of disk usage - mine was 50 GB of a configured stellarmate with local star catalogues and acutal taken pictures. The SD card you copy to has to have at least 5 GB space left (the more the better) and has to be bigger than the actual disk usage of your installation (use `du -h`)
- [1-2 hours) Updating stellarmate's debian from buster to bullseye. Depends heavily on your internet speed, the speed of your SD card, NVME or USB-stick, the amount or RAM you have or if you are using a Pi4 or a Pi5. If you are upgrading at your NVME or on a fast USB3 stick, this might be much faster. 



## Get standard stellarmate up and running

Downloaded stellarmate (SM) or use you actual on one

ssh into your actual stellarmate

    ssh -5624 stellarmate@stellarmate.local

I also recommend to install git and some tools, you will need it anyways and this saves time later on 

   sudo apt install git inxi

If you are used to work with vim, install also vim as an alternative editor
   sudo apt install git inxi vim vim-scripts

## Update your Stellarmate to the latest stable version

Update your Stellarmate installation. Do this within your Stellarmate App.

You can also use the Web-VNC and call the "StellarMate Tools" app from the desktop:

   http://stellarmate.local:6080  (default password is "smate")

Then open the StelarMate Tools app an ipdate to the lates Stellarmat version. 

Then reboot

   sudo reboot

### Boot into the copy of your Stellarmate

Boot your copy of stellarmate from the newly created SD card (remove the USB card reader or an NVME you used). After a few seconds, ssh into your stellarmate with your stellarmate user as usual. 

    ssh -p 5624 stellarmate@stellarmate.local # default password is smate or stellar@mate


### Test, if your Stellarmate still works - check the version

After reboot you should be able to successfully ssh into stellarmate

    ssh -p 5624 stellarmate@stellarmate.local

Also you should be able to VNC into your stellarmate with your browser. Do this

   http://stellarmate.local:6080  (default password is "smate")

Great job! You are almost done. 

## Create pifinder user

We have now to create a pifinder user and home directory and add it to the sudo group. 
We will act then as pifinder to get the same environment, as PiFinder expects.
    
    # create user with homedir
    sudo useradd -m pifinder
    sudo passwd pifinder

    # add pifinder to the sudo group
    sudo usermod -aG sudo pifinder


## Become pifinder user

### STOP: This step ist important! Change to the pifinder user

    su - pifinder

We now mainly do, what is described in the [docs]https://pifinder.readthedocs.io/en/release/software.html#build-from-scratch), but with a few differences within the pifinder setup.  

### Enable SPI and I2C
Like mentioned there, check that SPI an I2C are enabled. When you altered the settings, you have to reboot. 

     sudo raspi-config

- Update raspi-config tool

![image](https://github.com/apos/pifinder_tweaks/assets/456034/4134eb62-f3e5-43c3-9b6e-98ee6842e514)

- Enable I2C and SPI

![image](https://github.com/apos/pifinder_tweaks/assets/456034/08740526-f5ed-4670-862a-97f3de3c235b)


### Download and alter the pifinder setup scrip
Next we download the original PiFinder script like mentioned in the docs, but we DO NOT RUN IT, just download the script (no `| bash` at the end!)

So first download the script to a local file:

    wget -c https://raw.githubusercontent.com/brickbots/PiFinder/release/pifinder_setup.sh

We have to alter the script, because we do not want to alter or overwrite the setting, that stellarmate already had set and controls:

- Wlan (hostapp and other stuff)
- GPS
- Samba

We habe to alter the way starting the [pifinder service](https://www.tderflinger.com/en/using-systemd-to-start-a-python-application-with-virtualenv) via 

    nano pifinder_setup.sh

Here is the diff that shows, what to disable: 

    diff pifinder_setup.sh pifinder_setup.sh.stellarmate

    2c2,3
    < sudo apt-get install -y git python3-pip samba samba-common-bin dnsmasq hostapd dhcpd gpsd
    ---
    > # sudo apt-get install -y git python3-pip samba samba-common-bin dnsmasq hostapd dhcpd gpsd
    > sudo apt-get install -y git python3-pip # samba samba-common-bin dnsmasq hostapd dhcpd gpsd
    9,10c10,11
    < sudo dpkg-reconfigure -plow gpsd
    < sudo cp ~/PiFinder/pi_config_files/gpsd.conf /etc/default/gpsd
    ---
    > # sudo dpkg-reconfigure -plow gpsd
    > # sudo cp ~/PiFinder/pi_config_files/gpsd.conf /etc/default/gpsd
    22,25c23,26
    < sudo cp ~/PiFinder/pi_config_files/dhcpcd.* /etc
    < sudo cp ~/PiFinder/pi_config_files/dhcpcd.conf.sta /etc/dhcpcd.conf
    < sudo cp ~/PiFinder/pi_config_files/dnsmasq.conf /etc/dnsmasq.conf
    < sudo cp ~/PiFinder/pi_config_files/hostapd.conf /etc/hostapd/hostapd.conf
    ---
    > # sudo cp ~/PiFinder/pi_config_files/dhcpcd.* /etc
    > # sudo cp ~/PiFinder/pi_config_files/dhcpcd.conf.sta /etc/dhcpcd.conf
    > # sudo cp ~/PiFinder/pi_config_files/dnsmasq.conf /etc/dnsmasq.conf
    > # sudo cp ~/PiFinder/pi_config_files/hostapd.conf /etc/hostapd/hostapd.conf
    27c28
    < sudo systemctl unmask hostapd
    ---
    > # sudo systemctl unmask hostapd
    30c31
    < sudo chmod 666 /etc/wpa_supplicant/wpa_supplicant.conf
    ---
    > # sudo chmod 666 /etc/wpa_supplicant/wpa_supplicant.conf
    33c34
    < sudo cp ~/PiFinder/pi_config_files/smb.conf /etc/samba/smb.conf
    ---
    > # sudo cp ~/PiFinder/pi_config_files/smb.conf /etc/samba/smb.conf


I also had to install Adafruit-Blinka with the following

    sudo -H pip3 install Adafruit-Blinka

## Ready to go

The final step before we reboot is to install the pifinder script like on a fresh pifinder. Remember: we have to do this as the user pifinder, whcih 

    bash pifinder_setup.sh
    [sudo] password for pifinder:

    # script runs ...
    # [...]
    PiFinder setup complete, please restart the Pi

Whe you read the last line, the script finished and we can reboot stellarmate (fingers crossed :-) ).

    sudo reboot 

## Check stellarmate status after PiFinder install

After the reboot, you should see pifinder running.
When this is OK, then we first check, if Stellarmate is still running correctly. 

- Start the handy app an check, if you can connect. Check all functions (load setup)
- Can you reach the Stellarmate dashboard via your browser?: http://stellarmate.local:3000/#/diagnostics
- Can you Web-VNC into the device?: http://stellarmate.local:6080/
- Can you connect to the INDI-Webmanager?: http://stellarmate.local:8624

If PiFinder is not starting, you have to investigate.

Otherise: Crongratualions. You got it!!!


# Troubleshooing
You can report here in Gitlab (https://github.com/apos/pifinder_tweaks) under Issues. But it might take time, until I will answer, I am not a developer and will be on GitHub only every few weeks, sometimes months. I also will not give any linux beginners support or basic questions. I am not responsible for any damage you make (both hard- or software). And I don not answer questions which do not rely on a good problem description with log files added and so on. You should basically know, what you are doing. Also other might also help. 

## Check if you are on 32 or 64bit

Login into Stellarmate via SSH:

    ssh -p 5624 stellarmate@stellarmate.local   

Check your version with

    cat /etc/os-releasecat
    lsb_release -a

Get your architecture. If the result is 32, you have a 32bit, which will NOT work. Download an acutal version of Stellarmate on https://www.stellarmate.com.  64bit is what you need.

    getconf LONG_BIT

You can also install some tools to get much more informations (only, if you like - this will add some dependencies): 

    sudo apt install lshw inxi
    sudo inxi -Fxz


## Fast revert changes if something went wrong 

The savest, but slowest way is to reboot into your original Stellarmate SD-card and create a new backup copy with the SD card copier. Then start over booting from the created copy and run the procedure again. 

Don't be a fool and try to speed things up, this might destroy your Stellarmate setup. It is also very wise to create an extra backup with the SD card copier. Always use a SD card for this backup, you might not be able to boot from USB, if you are not an experienced user!



## Fix Error "EKOS server is down"

Stellarmate relies on a service calles ["EKOS live"](https://www.stellarmate.com/support/ekos-live.html) - please read the link first to know what EKOS live is and how it works!

This service can be offline or online. You need your credentials, which you registerd on stellarmate.com. I is worth mentioning, that you can register to ELOS live WITHOUT the need to buy stellarmate. The website is: 

- https://live.stellarmate.com/

The service can malfuncion also after an update of Stellarmate, e.g. to the beta versions (which you should not use, until you know what you are doing). I encountered this problem several times. The handy app than is not able to connect to the installation and Kstars throws this error, which is really not that cool. 


### Check the status and get a picture

You can check the status. Login into Stellarmate via SSH:

    ssh -p 5624 stellarmate@stellarmate.local   
    sudo service ekoslive status

This should look similiar like this. 

    stellarmate@stellarmate:~ $ sudo service ekoslive status
    ● ekoslive.service - EkosLive Offline
         Loaded: loaded (/etc/systemd/system/ekoslive.service; enabled; vendor preset: enabled)
         Active: active (running) since Mon 2024-01-01 13:06:03 CET; 168ms ago
       Main PID: 3371 (ekoslive)
          Tasks: 6 (limit: 4915)
            CPU: 149ms
         CGroup: /system.slice/ekoslive.service
                 └─3371 /opt/ekoslive/ekoslive

You see, that the Ekos Live status is "Offline".

You can also open the Stellarmate dashboard with your browser and check the logs on the Diagnostic page of the Stellarmate dashboard:

    http://192.168.0.15/#/diagnostics  # this has trouble with firefox, use chrome

You should also Web-VNC into Stellarmate for some further troubleshooging. 

    http://stellarmate.local:6080/



### Fix it - WIP
There are some things you should try to get this up and running again.

Open EKOS inside of KStars and tip onto the cloud symbol (see above inside of the 

Easeast solution 

1. If the status shows "Offline", try to reconnect. 


TO BE DONE













