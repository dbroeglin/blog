---
title: "Debian Squeeze 64 Vagrant Base Box Upgrade Recipe"
date: 2011-08-09
lastmod: 2012-05-08
categories:
 - Vagrant
 - Virtual machine
 - Linux
 - Debian
---
_After building a Vagrant base box, Vagrant 
 and Virtual Box updates incurred lots of upgrades. 
 This article gives a recipe to do that quickly and efficiently._

# Build a new environment

{{< highlight bash >}}
mkdir /tmp/upgrade
cd /tmp/upgrade
vagrant init squeeze64
vagrant up
vagrant halt
{{< / highlight >}}

At that point the VM is stoped. Launch it from the Virtual Box GUI and mount the guest tools (Host+D).

# Upgrade

Log into the VM through vagrant:

{{< highlight bash >}}
vagrant ssh
{{< / highlight >}}

Upgrade the OS and the tools:

{{< highlight bash >}}
sudo aptitude update
sudo aptitude dist-upgrade
sudo mount /dev/scd0 /mnt
sudo aptitude install build-essential linux-headers-$(uname -r)
sudo /mnt/VBoxLinuxAdditions.run 
sudo rm -rf /usr/src/vboxguest*
# insert whatever other upgrades you may need to do here.
sudo aptitude purge build-essential linux-headers-$(uname -r)
sudo aptitude clean
sudo init 1
{{< / highlight >}}

# Clean up

Re-log through the console and execute:

{{< highlight bash >}}
rm -r /var/cache/*/*
mount -o remount,ro /dev/sda1
zerofree /dev/sda1
halt
{{< / highlight >}}

Package the VM:

{{< highlight bash >}}
cd /tmp/upgrade
vagrant package
vagrant box remove squeeze64
vagrant box add squeeze64 /tmp/upgrade/package.box
{{< / highlight >}}

Destroy the upgrade environment:

{{< highlight bash >}}
cd /tmp/upgrade
vagrant destroy
{{< / highlight >}}

**Update (10/12/2011)**: The base box has been updated to _Virtual Box Guest Additions_ v4.1.4 and the latest _Debian Squeeze_ updates.

**Update (12/02/2011)**: The base box has been updated to _Virtual Box Guest Additions_ v4.1.6 and the latest _Debian Squeeze_ updates.

**Update (02/08/2012)**: The base box has been updated to _Virtual Box Guest Additions_ v4.1.8 and the latest _Debian Squeeze_ updates.

**Update (03/25/2012)**: The base box has been updated to _Virtual Box Guest Additions_ v4.1.10, and chef 0.10.8 and the latest _Debian Squeeze_ updates.

**Update (04/04/2012)**: The base box has been updated to _Virtual Box Guest Additions_ v4.1.12 and the latest _Debian Squeeze_ updates.

**Update (29/04/2012)**: The package has been updated to the latest _Debian Squeeze_ updates and the _nfs-common_ package was installed.

**Update (05/08/2012)**: The base box has been updated to _Virtual Box Guest Additions_ v4.1.14 and the latest _Debian Squeeze_ updates.
