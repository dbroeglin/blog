---
title: "Vagrant Based Windows Lab"
date: 2015-11-01
---
_Having a lab handy is a tremendous help when learning Windows 
 administration. This post goes through an easy way to setup your own
 lab with Vagrant and Packer._

**Update:** I used this lab quite a bit since this post and did quite a bit of 
improvements. Most things remain correct but you will, for instance, have to 
replace `eval-win2012r2-standard-ssh-nocm-1.0.4` with `eval-win2012r2-standard-nocm-1.0.4`
in the following examples for the lab to work. Please, refer to the github repository 
for details.

Even if, nowadays, it is fairly easy to setup a Windows Lab on various cloud 
providers, doing it by oneself is the best way to learn. However, it can 
quickly get time consuming. This post describes how Vagrant sets up 
automatically a new lab on my laptop each time I need to try something on
a windows infrastructure.

# Packer

[Packer][packer] by Hashicorp is a tool for automated creation of machines and
container images. It can do so for various target platforms. In the following 
lines we are going to create Windows machines targeted at VirtualBox. 

The aforementioned site extensively documents how Packer works. However, 
automatic creation of a Windows machine is not an easy task. For those interrested
the following post will walk you through the process of 
[creating a Windows base image][creating-windows-base-images].

Because creating the base images is not the subject of this post we will take a 
shortcut and use packer definitions from the [Boxcutter][boxcutter] project which
contains community-driven templates for most common OSes.
After installing Packer (on the Mac, with brew, you can use `brew install packer`)
execute the following commands:
 
{% highlight bash %}
git clone git@github.com:boxcutter/windows.git boxcutter-windows
cd boxcutter-windows
make virtualbox/eval-win2012r2-standard-ssh
{% endhighlight %}

This will take quite a bit of time: download the Windows evaluation ISO and then
prepare, cleanup and package a VirtualBox base image.

# Vagrant

After having installed Vagrant, go in the boxcutter directory mentioned above 
and execute:

{{< highlight bash >}}
vagrant box add --provider virtualbox \
       --name eval-win2012r2-standard-ssh-nocm-1.0.4 \
       -f box/virtualbox/eval-win2012r2-standard-ssh-nocm-1.0.4.box
{{< / highlight >}}

Then in your `Vagrantfile` add the following definitions:
{{< highlight ruby >}}
config.vm.define "dc" do |config|
  config.vm.box = "eval-win2012r2-standard-ssh-nocm-1.0.4"
  config.vm.network "private_network", ip: "192.168.100.10"
end

config.vm.define "client" do |config|
  config.vm.box = "eval-win2012r2-standard-ssh-nocm-1.0.4"
  config.vm.network "private_network", ip: "192.168.100.11"
end
{{< / highlight >}}

The purpose of the `private_network` interface is to allow
the boxes to both have fixed IP addresses (better for the _DC_) and to communicate
directly with the host system. The NATed interfaces will allow the VMs to communicate
with the rest of the world.

After executing: 
{{< highlight bash >}}
vagrant up
{{< / highlight >}}
and going for a lengthy coffee, you should have two VMs running called respectively
_dc_ and _client_. 

The _dc_ VM is provisionned as a _domain controller_. The provisioning part 
happens because of the following configuration in the `Vagrantfile`:

{{< highlight ruby >}}
  config.vm.define "dc" do |config|
    # ...
    config.vm.provision "shell", path: "provision/00_admin_password.ps1"
    config.vm.provision "shell", path: "provision/01_install_AD.ps1"
    config.vm.provision "shell", path: "provision/02_install_forest.ps1"
  end
{{< / highlight >}}

A similar block is used for the client:

{{< highlight ruby >}}
  config.vm.define "client" do |config|
    # ...
    config.vm.provision "shell", path: "provision/06_client_setup.ps1"
  end
{{< / highlight >}}

# Setting up Active Directory
	    
After setting up the VM with a blank Windows system we do the following: 

1. `00_admin_password.ps1`: set the administrator password to `Passw0rd`.
1. `01_install_AD.ps1` : install the various required features for Active Directory.
1. `02_install_forest.ps1` : actually install the AD Forest.

On the client side the `06_client_setup.ps1` provisioning script will just setup DNS
and join the `client` virtual machine to the newly created domain.
         
That is all we have for today, the actual Vagrant project can be found at 
[https://github.com/dbroeglin/windows-lab](https://github.com/dbroeglin/windows-lab)
         
[packer]: https://www.packer.io/
[creating-windows-base-images]: http://www.hurryupandwait.io/blog/creating-windows-base-images-for-virtualbox-and-hyper-v-using-packer-boxstarter-and-vagrant
[boxcutter]: https://atlas.hashicorp.com/boxcutter
[oobe]: https://technet.microsoft.com/library/ff716016#HideLocalAccountScreen
[sysprep-cli]: https://technet.microsoft.com/en-us/library/hh825033.aspx