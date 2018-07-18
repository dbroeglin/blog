---
title: "Automated NetScaler VirtualBox Setup with PowerShell"
date: 2017-02-19
---
_For a few months now, I have been working a lot with NetScaler VPX instances. As most of my personal work happens on a MacBook Pro, I was very eager to run NetScaler on VirtualBox. Here are my findings._

As documented by Citrix, NetScaler is not directly available on VirtualBox and my previous attempts to make it work were unsuccessful. That was until a few days ago when I found this [post][netscaler_virtualbox] by Esther Barthel. Basically, the trick is to convince NetScaler it is running on ESX by customizing the VirtualBox DMI BIOS settings. Obviously, this way of doing things is not at all supported by Citrix. But that is not an issue in my case as I use VirtualBox only for development and testing.

As I'm currently working hard at setting up _system tests_ for the [NetScaler PowerShell Module][netscaler_module]. So, I wanted to see if I could move one step further and create a [Vagrant base box][vagrant_base_box] that would then neatly integrate with my [Vagrant based Windows Lab][windows_lab]. I created a VirtualBox guest from the NetScaler VPX image for ESX and setup a network configuration compatible with what Vagrant sets up by default:

* NSIP: 10.0.2.15
* Netmask: 255.255.255.0
* Default gateway: 10.0.2.2
* DNS: 10.0.2.3

The `Vagrantfile` I packaged in the base box is: 

{{< highlight ruby >}}
Vagrant.configure("2") do |config|
  # use password based authentication for SSH
  config.ssh.username = "nsroot"
  config.ssh.password = "nsroot"

  config.vm.provider "virtualbox" do |vb|
    # Netscaler requires a minimum of 2 cores and 2G of RAM.
    vb.cpus   = "2"
    vb.memory = "2048"

    vb.check_guest_additions = false
    vb.functional_vboxsf = false

    # Forward the NetScaler GUI to 8080
    config.vm.network "forwarded_port", guest: 80, host: 8080

    vb.customize ["setextradata", :id, "VBoxInternal/Devices/pcbios/0/Config/DmiBIOSVendor", "Phoenix Technologies LTD"]
    vb.customize ["setextradata", :id, "VBoxInternal/Devices/pcbios/0/Config/DmiBIOSVersion", "6.00"]
    vb.customize ["setextradata", :id, "VBoxInternal/Devices/pcbios/0/Config/DmiBIOSReleaseDate", "07/31/2013"]
    vb.customize ["setextradata", :id, "VBoxInternal/Devices/pcbios/0/Config/DmiBIOSReleaseMajor", 6]
    vb.customize ["setextradata", :id, "VBoxInternal/Devices/pcbios/0/Config/DmiBIOSReleaseMinor", 0]
    vb.customize ["setextradata", :id, "VBoxInternal/Devices/pcbios/0/Config/DmiBIOSFirmwareMajor", 6]
    vb.customize ["setextradata", :id, "VBoxInternal/Devices/pcbios/0/Config/DmiBIOSFirmwareMinor", 0]
    vb.customize ["setextradata", :id, "VBoxInternal/Devices/pcbios/0/Config/DmiSystemVendor", "VMware, Inc."]
    vb.customize ["setextradata", :id, "VBoxInternal/Devices/pcbios/0/Config/DmiSystemProduct", "VMware Virtual Platform"]
  end
end
{{< / highlight >}}

The result is a base box of about 350mb that boots:

{{< highlight bash >}}
vagrant init nsvpx-esx
vagrant up
{{< / highlight >}}

However, after booting the image, Vagrant tries to connect to it through SSH and fails. My hypothesis is that vagrant's guest detection fails when it logs in and encounters the NetScaler CLI. I guess that implementing a new `netscaler` guest type for Vagrant could solve the issue. Unfortunately, I do not currently have enough time to try that out. I thought, though, that I would put what I found so far out there if someone else wants to give it a try.

In the mean time, I settled for a PowerShell script that automatically provisions a NetScaler virtual machine from an ESX VPX package.

The script can be used like this (please, have a look at the inline help for more parameters):

{{< highlight powershell >}}
./New-NSVirtualBoxInstance.ps1 -Package ~/NSVPX-ESX-11.1-49.16_nc.zip `
    -VMName NSVPX-11-1 -Force -Verbose
{{< / highlight >}}

It will open the package and create a new `NSVPX-11-1` VirtualBox virtual machine. By default the virtual machine's MAC address will be `00:15:5D:7E:31:00` on a NAT type VirtualBox virtual switch. host ports 2022, 2080, and 2443 will be forwarded respectively to ports 22, 80 and 443 on the NetScaler virtual machine.

The resulting script can be found here : [New-NSVirtualBoxInstance.ps1][gist]. Everything works neatly in my [Vagrant based Windows Lab][windows_lab]. 

A next step, moving forward, could be to enable automatic NSIP provisioning. To be continued...

*Update 2017-03-07:* Netscaler boots thinking that it runs on top of an ESX. Which means that, before NTP is configured, it will try to rely on the clock provided by the VMWare Guest tools... which do not work. Make sure
that NTP is setup _before_ you install the license file and reboot or the license will probably not work.

[gist]: https://gist.github.com/dbroeglin/0c9e512fc4640f2ceda22005650cd417
[netscaler_module]: https://github.com/devblackops/Netscaler
[netscaler_virtualbox]: http://www.virtues.it/2016/08/howto-netscaler-vpx-on-virtualbox/
[windows_lab]: {{< ref "2015-11-01-vagrant-windows-lab.md" >}}
[vagrant_base_box]: https://www.vagrantup.com/docs/boxes/base.html
