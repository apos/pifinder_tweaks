# Unite PiFinder & Stellarmate
[PiFinder](https://www.pifinder.io/) is one of the coolest astronomy projects out there besides [Stellarmate](https://stellarmate.com). 
In this project I describe, how to add the PiFinder software to an existing Stellarmate.

Main sources for PiFinder are:
- Main page: https://www.pifinder.io/
- Github: https://github.com/brickbots/PiFinder
- Documentation: https://pifinder.readthedocs.io/en/release/index.html


# Install and use PiFinder with Stellarmate 

## Preface

As of writing this (December 2023), I am worknig with
- Raspberry PI 4 (8GB) 
- Stable branch of Stellarmate 1.8.0 (64 bit)

which is based on debian buster. Please be aware, this procedure will NOT WORK WITH a 32bit version! If you updated from an early version of Stellarmate it it very likely, that you are running a 32 bit version (the architecture will not upgrade, just the software). 

STOP: at the end in the troubleshooting section I write, how to get this information. Please check, before you procede with

     sudo inxi -Fxz

Stellarmate Beta is actually on debian bullseye, but I do not like to go on the beta (nightly) stellarmate. I test with an Sandisk Extreme 64GB SD card an then copy that back to a 1TB NVME (see at github at my [YAACS](https://github.com/apos/case_system_stellarmate_astroberry) for specifications).

The only problem I had, was fixing the connection to the EKOS live server. The fix is to reconnect to the acccount. I explain this in the troubleshooting section at the end. 


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

   sudo apt install git lshw inxi

If you are used to work with vim, install also vim as an alternative editor
   sudo apt install git lshw inxi vim vim-scripts

## Update your Stellarmate to the latest stable version

Update the stellarmates debian to the newest version
    sudo apt update && sudo apt-dist upgrade

Update your Stellarmate installation. Do this within your Stellarmate App or Web-VNC into stellarmate and call the StellarMate Tools 

   http://stellarmate.local:6080  (default password is "smate")

Open the StelarMate Tools app an Update to the lates Stellarmat version. 


Then reboot

   sudo reboot

## Upgrade debian - Caution - create and use a copy of stellarmate 

You should first copy your stellarmate to a second sd card and then test it thorougly. Then you can copy it back onto your actual installation. Or, if this is your first stellarmate, it is no problem, because you can simply install it from scratch. Nevertheless, I would prefer to work with a copy. And, do not tinker with stellarmate, if you are new to it. Concentrate on the job here, to install PiFinder. You should not do two things at a time. 

Hint: You should have at least approximataly 3-5 GB space left on the card, because we update stellarmate. So best use a 32 GB card at least. It is always wise, to not use all the space on a sd card, this can reduce the lifetime of the card drastically - 30-40% space left is best. 

Therefore make a copy of your stellarmate onto a spare SDcard with "SD Card Copier" which is part of any raspian or stellarmate. Go to the menu and you'll find it under the submenu "accessories". Put a new SD-card into an USB-SD-card reader and copy things over (do not mix source and target - think twice!). This can take a while like 10-20 minutes depending of the speed of your SD-card.



### Boot into the copy of your Stellarmate

Boot your copy of stellarmate from the newly created SD card (remove the USB card reader or an NVME you used). After a few seconds, ssh into your stellarmate with your stellarmate user as usual. 

    ssh -p 5624 stellarmate@stellarmate.local # default password is smate or stellar@mate


First of alle you should update all packages and reboot 

    sudo apt update
    sudo apt dist-upgrade
    sudo reboot


### Upgrade debian from buster to bullseye

Because stellarmate runs on buster (at least my version 1.8), we have to upgrade stellarmate to "bullseye", which is possible due to the super cow power of debian :-) This process will take a lot of time, because this downloads 2-2,5 GB of new archives that have to be installed. So you also need to have that space. You also need to be at place to interact with some infos, warnings and so on. 


##################
The following process takes a bunch of time and the update process is interacive! 
##################

Just alter the apt sources in /etc/apt

    sudo nano /etc/apt/sources.list.d/smos.list
    > deb https://ppa.stellarmate.com/repos/apt/stable bullseye main

    sudo nano /etc/apt/sources.list.d/raspi.list
    > deb http://archive.raspberrypi.org/debian/ bullseye main

    sudo nano /etc/apt/sources.list
    > deb http://raspbian.raspberrypi.org/raspbian/ bullseye main contrib non-free rpi 

Then update. 

You will be prompted with a handful of warnings and informations. If you are unsure, what to do, just take the preselected values:

- Selections can be eather "yes" or "no" or simply "OK" followed by pressing the Enter key. Select between selections with the "Tab" key and press "Enter". 
- Or just a "q" to leave for informations. 
- If you ask about altering the package information always use "keep the local version currently installed" (press Tab to switch to OK, then press Enter)

    sudo apt update

Check the output message of this command, which should state "bullseye". There should not be any "buster" at all:

    Hit:1 http://raspbian.raspberrypi.org/raspbian bullseye InRelease
    Hit:2 http://archive.raspberrypi.org/debian bullseye InRelease
    Get:3 https://ppa.stellarmate.com/repos/apt/stable bullseye InRelease [2,186 B]
    Fetched 2,186 B in 1s (1,966 B/s)
    Reading package lists... Done
    Building dependency tree... Done
    Reading state information... Done


Then upgrade the system from debian buster to debian bullseye

    sudo apt dist-upgrade

As of my experience, it might be useful, to check again und upgrade again. It might be, that you get the message "The following packages have been kept back:" followed by a list of packages. This is no problem.

    sudo apt update && sudo apt dist-upgrade


After that, we remove the package cache (do NOT remove packages with "autoremove" as suggested), which deletes all downloaded packages, which saves a lot of space. This speeds up the copy process later on. 

    sudo apt clean

When everything finished, reboot

    sudo reboot

### Test, if your Stellarmate still works - check the version

After reboot you should be able to successfully ssh into stellarmate

    ssh -p 5624 stellarmate@stellarmate.local

Also you should be able to VNC into your stellarmate with your browser. Do this

   http://stellarmate.local:6080  (default password is "smate")

You might get an error, that the stellarmate server ist not accessable, after Kstars is running. Therefore start the Stellarmate Update, 



Great job! You are almost done. 
If you experience any problems, 


## Create pifinder user



We have now to create a pifinder user and home directory and add it to the sudo group. 
We will act then as pifinder to get the same environment, as PiFinder expects.
    
    # create user with homedir
    sudo useradd -m pifinder
    sudo passwd pifinder

    # add pifinder to the sudo group
    sudo usermod -aG sudo pifinder


## Become pifinder user

Now we can change the user This step important!

    su - pifinder

We mainly do, what is described in the [docs]https://pifinder.readthedocs.io/en/release/software.html#build-from-scratch), but with a few differences. 


### Enable SPI and I2C
Like mentioned there, check that SPI an I2C are enabled. When you altered the settings, you have to reboot. 

### Download and alter the pifinder setup scrip
Next we download the original PiFinder script like mentioned in the docs, but we DO NOT RUN IT, just download the script (no `| bash` at the end!)

So first download the script to a local file:

    wget -c https://raw.githubusercontent.com/brickbots/PiFinder/release/pifinder_setup.sh

We have to alter the script, because we do not want to alter or overwrite the setting, that stellarmate already had set and controls:

- Wlan (hostapp and other stuff)
- GPS
- Samba

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

You can also install some tools to get more informateon

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

1. If the status shows "Offline", try to reconnect. 


TO BE DONE













