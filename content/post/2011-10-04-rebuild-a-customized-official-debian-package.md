---
title: "Rebuild a Customized Official Debian Package"
date: 2011-10-04T22:53:00
categories: 
  - Debian
  - Linux
  - Network
  - QuickTips
---
Today, I had a question about Debian package building from a member of my 
team which was not familiar with
the Debian build system. I thought I might as well turn this into a quick 
tip to anybody wanting to rebuild an official Debian package with a slight
change.

To rebuild the package I'm using a pristine Vagrant environment like the one 
I described in [Debian Squeeze 64 Vagrant Base Box](/2011/03/26/squeeze-64-vagrant-base-box.html).
However, if you do that often you might want to prepare a build box with all the
usual Debian development tools already installed in order to save the initial
setup steps.

First we need to build a new Vagrant VM:

{{< highlight bash >}}
mkdir builder
cd builder
vagrant init squeeze64
vagrant up
vagrant ssh
{{< / highlight >}}

From this point on we are in the virtual builder box. First we need to setup a
build environment:

{{< highlight bash >}}
export DEBEMAIL="my@emailaddress.com"
export DEBFULLNAME="My Full Name"

sudo aptitude install devscripts
{{< / highlight >}}

Now, as an example, let us compile the _quagga_ package with SNMP enabled:

{{< highlight bash >}}
apt-get source quagga
sudo aptitude build-dep quagga
cd quagga-0.99.17/
dch --local +custom1 "Activating SNMP"
WANT_SNMP=1 fakeroot dpkg-buildpackage -us -uc 
sudo dpkg --install ../quagga_0.99.17-2+squeeze2+custom1.deb 
{{< / highlight >}}

That's it ! Quite simple and straightforward once you know what to do. Have fun 
customizing your Debian packages.
