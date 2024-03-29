# Armbian

Armbian is based on Debian or Ubuntu and has nice support for a lot of ARM-based SBC's.

## Get image

Their images for Odroid XU4 also work for HC2, as it's essentially the same hardware.

[Armbian page for the HC2](https://www.armbian.com/odroid-xu4/)

Download an image from the armbian page listed above. Get a version without desktop, currently called "CLI".

## Write the image to your SD-card

Get a tool to write images. I like Balena Etcher [Balena Etcher](https://www.balena.io/etcher/).

Write the image to the SD-card.

Then insert the SD-Card and plug in the power to start the machine.

## Login for the first time

The defaults are:

* Hostname: odroidxu4
* Log in: root
* Password: 1234

You can login with:

    ssh root@odroidxu4

...and use the password above. As soon as you login, you are prompted to change this password (remember, it's still using US-Keyboard setting at this point, so avoid characters like æøå).

### Create normal/non-root user

Right after changing the password, you are asked to create a normal user-account for your everyday tasks. It will have *sudo* rights. I created a user called "sv"... you should probably choose something else and replace it in the rest of this document, where I use it here and there.

### Set public key for the root user

Add your own public key, in my case:

    mkdir $HOME/.ssh
    chmod 700 $HOME/.ssh
    echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEOUWK5GhGu42n434IH2e6wQXrP5SzZLROdSZpEyalB6 myemail@domain.com' >> $HOME/.ssh/authorized_keys
    chmod -R 600 $HOME/.ssh/*

### Change hostname

To change the hostname, run these commands:

    sudo hostnamectl set-hostname <hostname>
    sudo apt install -y avahi-daemon

You can use another hostname, of course. just replace the hostname above with something else.

You can optionally change the name in /etc/hosts to set the hostname to 127.0.0.1, if you want to use it locally:

    sudo nano /etc/hosts

### Update and upgrade

Run these commands to update the OS:

    sudo apt update && sudo apt upgrade -y && sudo apt-get dist-upgrade && sudo apt autoremove -y

### Reboot

Reboot for the new hostname to take effect:

    sudo reboot

Give it a couple of minutes for the reboot.

### Login

SSH login as the new user, in my case 'sv':

    ssh sv@<hostname>

### Set public key for the regular user

Add your own public key, in my case:

    mkdir $HOME/.ssh
    chmod 700 $HOME/.ssh
    echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEOUWK5GhGu42n434IH2e6wQXrP5SzZLROdSZpEyalB6 myemail@domain.com' >> $HOME/.ssh/authorized_keys
    chmod -R 600 $HOME/.ssh/*

### Add sudo without a password

With great power comes great responsibility.
If you feel lazy and don't care about security, you set the user to not need a password for sudo. This is not recommended, but here's the recipe if you like living dangerously:

    echo "$USER ALL=(ALL:ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/$USER

## Initial setup of the OS

Here comes a few steps that are good to do in the beginning, to get a good start on a freshly installed OS.

### Install some tools

Install some of my favourite tools and utilities. Avahi announces hostname on the network. Glances is a good alternative to htop.

    sudo apt install -y git nano htop mc build-essential glances curl pigz unattended-upgrades

### Unattended upgrades

You can configure unattended-upgrades on ubuntu/debian systems to also update normal packages, instead of just security packages.

It's not required and it's not a good idea for production systems, but nice for something like your personal jenkins or plex server (see later sections)

Here is the guide: [How to Enable Unattended Upgrades on Ubuntu/Debian](https://haydenjames.io/how-to-enable-unattended-upgrades-on-ubuntu-debian/)

Quick bits from the article. First edit

    sudo nano /etc/apt/apt.conf.d/50unattended-upgrades

Uncomment the line that says

    //"${distro_id}:${distro_codename}-updates";

Then edit:

    sudo nano /etc/apt/apt.conf.d/20auto-upgrades

Add this line:

    APT::Periodic::AutocleanInterval "7";

You're done with configuring unattended-upgrades

### OPTIONAL: Install HDD/SSD

You *can* install the root file system to the HDD/SSD (but you don't have to). That will save wear and tear on the SD-card, probably give you much more space in "/" and maybe enven make it faster to run. The boot partition will remain on the SD-card, so you'll still need to tleave it in the machine. The Odroid-HC2 cannot boot purely from the HDD/SSD.

Here are some guides, but you can skip them and just read the steps below.

* Delete partitions, see how here <https://phoenixnap.com/kb/delete-partition-linux>
* Create a single partition, ext4. See <https://linuxize.com/post/fdisk-command-in-linux/>
* [Install to harddisk/SSD using this guide](https://docs.armbian.com/User-Guide_Getting-Started/#how-to-install-to-emmc-nand-sata-usb)

To wipe all partitions and create a new one on the drive, first run:

    sudo fdisk /dev/sda

* Then use the **"d"** command in the fdisk tool to delete the partions, one-by-one if you have several. You'll need to run the **"d"** command once for each partition
* Use the **"g"** command to create a new GPT partition table
* Use the **"n"** command to create a new partion. Accept the defaults.
* Use the **"w"** command to write changes to disk and exit.

You're now done with fdisk.

Then run:

    sudo nand-sata-install

...and:

* Choose the "Boot from SD..." option
* Choose your **/dev/sda1** as the destination.
* Choose the filesystem. I prefer **ext4** over btrfs, for maximum compatibility.

Reboot when it asks you to.

### All done with OS

Now the OS is configured and you are ready to start using the server.

### OPTIONAL: Disable ZRAM swap

If you don't like ZRAM as swap, you can disable it.
From https://serverfault.com/a/1094015

    sudo nano /etc/default/armbian-zram-config

A few lines down the file, uncomment the line that says SWAP=false:

    # Zram swap enabled by default, unless set to disabled
    SWAP=false

### OPTIONAL: Add swap on disk

From https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-20-04

For some reason, you have to run these one at a time:

    sudo fallocate -l 6G /swapfile
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile
    sudo cp /etc/fstab /etc/fstab.bak
    echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab    
