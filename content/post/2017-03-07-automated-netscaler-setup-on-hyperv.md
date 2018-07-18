---
title: "Automated NetScaler Hyper-V Setup with PowerShell"
date: 2017-03-07
---
_Lately, I set up quite a few NetScaler instances on my Windows 10 laptop. Usually my hypervisor of choice is VirtualBox. However, running both VirtualBox and Hyper-V is not possible on Windows 10. As, I needed Hyper-V for other reasons, I had to switch my NetScaler instances to it too._

In my last post about [setting up NetScaler on VirtualBox][netscaler_virtualbox], I described how NetScaler can be tricked into working on VirtualBox (which is not an hypervisor supported by Citrix!). This time, I did not need such hacks, Hyper-V is fully supported. There is even a cherry on top: on Hyper-V, the NSIP can be automatically provisioned, thus removing the need for manual input when the instance boots the first time :-)

But first things first. How do I move from a VPX Hyper-V package to a running virtual machine? The script will basically follow the same steps as previously (with auto-provisioning at the end):

1. Unzip the package (download a VPX Hyper-V package [here][download_netscaler]),
1. Copy the disk image to the destination location,
1. Setup the virtual machine (2Gb of RAM, 2 cores),
1. Add a network interface with a fixed MAC address,
1. Automatically provision the NSIP.

The last step, automated provisioning, can be done by following the [instructions provided by Citrix][netscaler_hyperv_instructions]. However, at the time, there was a typo that prevented the XML example from working. If you plan on rolling you own auto-provisioning, use this one instead:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<Environment xmlns:oe="http://schemas.dmtf.org/ovf/environment/1"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
oe:id=""
xmlns="http://schemas.dmtf.org/ovf/environment/1">
<PlatformSection>
<Kind>HYPER-V</Kind>
<Version>2013.1</Version>
<Vendor>CISCO</Vendor>
<Locale>en</Locale>
</PlatformSection>
<PropertySection>
<Property oe:key="com.citrix.netscaler.ovf.version" oe:value="1.0"/>
<Property oe:key="com.citrix.netscaler.platform" oe:value="NS1000V"/>
<Property oe:key="com.citrix.netscaler.orch_env" oe:value="cisco-orch-env"/>
<Property oe:key="com.citrix.netscaler.mgmt.ip" oe:value="10.102.100.122"/>
<Property oe:key="com.citrix.netscaler.mgmt.netmask" oe:value="255.255.255.128"/>
<Property oe:key="com.citrix.netscaler.mgmt.gateway" oe:value="10.102.100.67"/>
</PropertySection>
</Environment>
```

This XML has to be saved in a file named `userdata`. This file will then be packaged in an ISO9660 image that will be exposed as a CDROM to the NetScaler virtual machine. At first boot, NetScaler will read this file and automatically set up the NSIP, network mask and default gateway.

Generating an ISO file in PowerShell is possible thanks to this [script][ps_iso] by Chris Wu. I then cobbled together a script that provisions a Hyper-V NetScaler instance from a VPX ZIP file. The script is available in my GitHub repository as  [New-NSHyperVInstance.ps1][ps_hyperv_provision_script]. I hope it will be merged into the [NetScaler][ps_netscaler] PowerShell module soon.

The following command will create a new NetScaler Hyper-V VM named `NSVPX-11-1` in directory  `c:\temp\NSVPX-11-1` from the given VPX ZIP package. It will also auto-provision it with NSIP `10.0.0.30` and default gateway `10.0.0.254` connected to the `Labnet` switch. If the VM already exists it will first be removed. The MAC address and netmask will respectively use default value `00155D7E3100` and `255.255.255.0`.

```powershell
    New-NSHyperVInstance.ps1 -Verbose -Package C:\temp\NSVPX-HyperV-11.1-50.10_nc.zip `
        -VMName NSVPX-11-1 `
        -SwitchName Labnet `
        -NSIP 10.0.0.30 -DefaultGateway 10.0.0.254 `
        -Path C:\temp\NSVPX-11-1 `
        -Force
```

Feel free to send me any remarks or improvement requests. The next post, in this NetScaler series, will be about automatic system setup with the [NetScaler][ps_netscaler] PowerShell module. Stay tuned!

[netscaler_virtualbox]: {{< ref "2017-02-19-automated-netscaler-setup-on-virtualbox.md" >}}
[netscaler_hyperv_instructions]: https://docs.citrix.com/en-us/netscaler/11/getting-started-with-vpx/install-vpx-on-hyper-v.html
[ps_iso]: https://gallery.technet.microsoft.com/scriptcenter/New-ISOFile-function-a8deeffd
[ps_hyperv_provision_script]: https://github.com/dbroeglin/NetScaler/blob/exp/hyper-auto-provision/Contrib/New-NSHyperVInstance.ps1
[ps_netscaler]: https://github.com/devblackops/NetScaler
[download_netscaler]: https://www.citrix.com/lp/try/netscaler-vpx-platinum.html