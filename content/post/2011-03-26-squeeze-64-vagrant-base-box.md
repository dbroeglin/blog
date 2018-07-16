---
title: "Debian Squeeze 64 Vagrant Base Box"
date: 2011-03-26
lastmod: 2012-05-08
categories:
 - Vagrant
 - Virtual machine
 - Linux
 - Debian
---
_Lately I needed to test a project under Debian Squeeze 64 bit. I found several 32 bit boxes but no 64 bits. So I decided to build one of my own._

# Quickstart

To get up and running you just need to:

{{< highlight sh >}}
gem install vagrant
vagrant box add squeeze64 http://dl.dropbox.com/u/937870/VMs/squeeze64.box
vagrant init squeeze64
vagrant up
{{< / highlight >}}

# Details

The base box has the following characteristics:

* 275Mb in it's packaged form;
* 8Gb hard drive;
* Debian Squeeze with a linux-image-2.6.32-5-amd64 kernel;
* fr_CH keyboard layout;
* Debian packaged ruby 1.8;
* Debian packaged puppet 2.6.2-4;
* Chef 0.9.14 installed as a system wide gem;
* VirtualBox Guest Tools 4.1.0r73009;

# Recipe

For those interrested in the details, the box was built by
following "Vagrant's instructions for creating base boxes": https://www.vagrantup.com/docs/boxes/base.html

# Cleanup

In order to reduce the packaged VM's size, a little bit of
cleaning up was required. The following steps where used:

Inside the VM:

{{< highlight bash >}}
sudo aptitude install zerofree
sudo apt-get clean
sudo rm -rf /usr/src/vboxguest*
sudo rm -rf /usr/share/doc
sudo find /var/cache -type f -exec rm -rf {} \;
{{< / highlight >}}

Then halt the VM with vagrant:

{{< highlight bash >}}
vagrant halt
{{< / highlight >}}

Restart the VM with VirtualBox and log in as root (the root password was set to 'vagrant' during previous steps):

{{< highlight bash >}}
init 1
{{< / highlight >}}

Type the root password and then:

{{< highlight bash >}}
mount -o remount,ro /dev/sda1
zerofree /dev/sda1
{{< / highlight >}}

**Update (10/12/2011)**: The base box has been updated to _Virtual Box Guest Additions_ v4.1.4 and the latest _Debian Squeeze_ updates.

**Update (12/02/2011)**: The base box has been updated to _Virtual Box Guest Additions_ v4.1.6 and the latest _Debian Squeeze_ updates.

**Update (02/08/2012)**: The base box has been updated to _Virtual Box Guest Additions_ v4.1.8 and the latest _Debian Squeeze_ updates.

**Update (03/25/2012)**: The base box has been updated to _Virtual Box Guest Additions_ v4.1.10, chef 0.10.8 and the latest _Debian Squeeze_ updates.

**Update (04/04/2012)**: The base box has been updated to _Virtual Box Guest Additions_ v4.1.12 and the latest _Debian Squeeze_ updates.

**Update (29/04/2012)**: The package has been updated to the latest _Debian Squeeze_ updates and the _nfs-common_ package was installed.

**Update (05/08/2012)**: The base box has been updated to _Virtual Box Guest Additions_ v4.1.14 and the latest _Debian Squeeze_ updates.
