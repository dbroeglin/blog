---
layout: post
comments: true
title: "Debian Wheezy 64 Vagrant Base Box"
datE: 2012-02-25
comments: true
categories:
 - Vagrant
 - Virtual machine
 - Linux
 - Debian
aliases:
  - /2012/02/25/wheezy-64-vagrant-base-box.html
---
In the same spirit as [Debian Squeeze 64 Vagrant Base Box](/2011/03/26/squeeze-64-vagrant-base-box.html) this post describes a new Vagrant base box for Debian Wheezy. Wheezy is currently the unstable version of Debian, meaning the box is a constant work in progress. I will update this post each time the box is upgraded.

## Quickstart

To get up and running you just need to:

{{< highlight bash >}}
gem install vagrant
vagrant box add wheezy64 http://dl.dropbox.com/u/937870/VMs/wheezy64.box
vagrant init wheezy64
vagrant up
{{< / highlight >}}

## Details

The base box has the following characteristics:

* 395Mb in it's packaged form;
* 8Gb hard drive;
* Debian Wheezy with a linux-image-3.2.0-1-amd64 kernel;
* fr_CH keyboard layout;
* Debian packaged ruby 1.8;
* Debian packaged puppet 2.7.10-1;
* Chef 0.9.14 installed as a system wide gem;
* VirtualBox Guest Tools 4.1.8r75467

## Recipe

For those interrested in the details, the box was built by
following [Vagrant's instructions](http://vagrantup.com/docs/base_boxes.html) for
creating base boxes to create a Squeeze box which was then upgraded to Wheezy.

## Cleanup

In order to reduce the packaged VM's size, a little bit of
cleaning up was required. The following steps where used:

Inside the VM:

{{< highlight bash >}}
sudo aptitude purge cpp-4.3 gcc-4.3-base
sudo aptitude purge cpp-4.4 gcc-4.4-base g++-4.4
sudo aptitude purge python2.6 python2.6-minimal
sudo debfoster
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

 * **Update (04/01/2012)**: The base box has been updated to _Virtual Box Guest Additions_ v4.1.10, chef 0.10.8 and the latest _Debian Wheezy updates.
 * **Update (04/04/2012)**: The base box has been updated to _Virtual Box Guest Additions_ v4.1.12 and the latest _Debian Wheezy updates.
 * **Update (29/04/2012)**: The package has been updated to the latest _Debian Wheezy updates and the _nfs-common_ package was installed.
 * **Update (05/07/2012)**: The base box has been updated to _Virtual Box Guest Additions_ v4.1.14 and the latest _Debian Wheezy updates.
