# BaseBox

A repository containing instructions, scripts, etc., relating to creating a GitMachines Vagrant BaseBox

##### Tasks:

[x] Document steps required to build a BaseBox

[x] Build a 64-bit BaseBox using CEntOS 6 Minimal

[ ] Build a 32-bit BaseBox using CEntOS 6 Minimal

##### Note(s):

* These instructions have been used to build a BaseBox in CEntOS 6.4 Minimal. While it should be the case that they work for all flavors and most versions of CEntOS, we can't know until we try.

* All machines, as they are built and tested, will be inlcuded in Google Drive until we decide to release them as official Vagrant BaseBox machines.

* The following Yum Repositories are added to the box during the setup process:

	[RPMForge](http://repoforge.org/)

	[PuppetLabs](http://puppetlabs.com/)

##### Reference Materials:

* [Vagrant's BaseBox documentation](http://docs-v1.vagrantup.com/v1/docs/base_boxes.html) - Date: 2013-11-07
 
* [okfn/ckan on github](https://github.com/okfn/ckan/wiki/How-to-Create-a-CentOS-Vagrant-Base-Box) - Date: 2013-11-07

* [CEntOS's Guest Additions documentation](http://wiki.centos.org/HowTos/Virtualization/VirtualBox/CentOSguest) - Date: 2013-11-07

* [CEntOS's RPMForge documentation (Required for DKMS)](http://wiki.centos.org/AdditionalResources/Repositories/RPMForge) - Date: 2013-11-07

## Existing Vagrant BaseBoxes
---

1. [CEntOS - Minimal - 6.4 - Built: 2013-11-02](https://drive.google.com/a/gitmachines.com/file/d/0BzWZ9l36IG9hM2t5Wkw1R01xTTA/edit?usp=sharing) (Private, at the moment)

## Creating a BaseBox

### On host:
---

##### NOTE: This guide assumes you have Vagrant, VirtualBox, and network connectivity.

#### Create a VM in VirtualBox
1.   Give it a meaningful name

2.   Specify Linux for Type

3.   Specify Red Hat [(64 bit) if necessary] 

4.   Give the VM at least the recommened memory size (At least 512MB)

5.   Select "Create a virtual hard drive now"

   6a.   Select VDI as the type

   6b.   Select Dynamically allocated

   6c.   Specify at least 50 GBs, per Vagrant's recommendations.

7.   Disable Audio

8.   Disable the USB Controller

9.   Ensure the Network Adapter is enabled and it is using NAT

10.  In Storage, select your ISO as the image mounted on Controller: IDE

11.  Boot the VM and install the OS

   11a.   Skip the media test

   11b.   Select OK at welcome message

   11c.   Select English for the Installer language

   11d.   Select us as the keyboard model

   11e.   Select Re-initialize all

   11f.   Ensure System clock uses UTC is checked and select America/New York as the time zone

   11g.   Provie the root password 'vagrant' per Vagrant's recommendations. (Select Use Anyway)

   11h.   Select Use entire drive

   11i.   Select Write changes to disk

   11j.   Reboot ( Congradulations :) )

### On VM:
---

#### Fix networking (required for Minimal installs which do not have networking automatically enabled)

1.   Run the following commands:

```bash
	sed -ie 's/ONBOOT=no/ONBOOT=yes/g' /etc/sysconfig/network-scripts/ifcfg-eth0
	service network restart
```

#### Update the VM so that we can get the proper Kernel Headers (required for VirtualBox Guest Additions installation) and so other do not have to update as much

1.   Run the following commands:

```bash
	yum update -y
	yum install kernel-devel -y
```

2.   Reboot the VM (Required if the Kernel was updated during this process, better safe than sorry)

#### Install the needed services/tools/yum-repositories

1.   Run one of the following commands:

```bash
	rpm -i http://packages.sw.be/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.i686.rpm
		or
	rpm -i http://packages.sw.be/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm
```

2.   Run the following commands:

```bash
	rpm --import http://apt.sw.be/RPM-GPG-KEY.dag.txt
	rpm -i http://yum.puppetlabs.com/el/6/products/i386/puppetlabs-release-6-7.noarch.rpm
	yum install nano vim openssh-server wget gcc bzip2 make dkms puppet -y
```

#### Configure our system

1.   Run the following commands:

```bash
	service sshd start
	chkconfig sshd on
	groupadd admin # Per Vagrant's documentation
	groupadd vagrant 
	useradd -g vagrant -G admin vagrant # Create the user, and add them to both groups
	passwd vagrant
	   # specify **'vagrant'** for the password, per Vagrant's recommendations.
	# Results in the following added to the end of the sudoer's file
	#    # Added for Vagrant Support
	#    Defaults        env_keep += \"SSH_AUTH_SOCK\"
	#    %admin   ALL=NOPASSWD:ALL"
	echo -e "\n# Added for Vagrant Support\nDefaults	env_keep += \"SSH_AUTH_SOCK\"\n%admin   ALL=NOPASSWD:ALL" >> /etc/sudoers
	# Attempts to comment out requiretty, per Vagrant's documentation
	sed -ie 's/Defaults\s\+requiretty/#Defaults   requiretty/g' /etc/sudoers
	sed -ie 's/#UseDNS yes/UseDNS no/g' /etc/ssh/sshd_config
	mkdir ~vagrant/.ssh
	curl -k https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub > ~vagrant/.ssh/authorized_keys
	chmod 0700 ~vagrant/.ssh
	chmod 0600 ~vagrant/.ssh/authorized_keys
	chown -R vagrant:vagrant ~vagrant/.ssh
```

2.   Use the context menu to auto-mount the VirtualBox Guest Additions ISO.

3.   Run the following commands:

```bash
	mount /dev/dvd /mnt
	/mnt/VBoxLinuxAdditions.run # Ignore failure to find X.Org Window System, as per Vagrant's documentation
```

#### Set our VM's hostname

1.   Run the following commands:

```bash
	hostname GitMachines-BaseBox
	sed -ie 's/localhost.localdomain/GitMachines-BaseBox/g' /etc/sysconfig/network
```

#### Shutdown the VM, it is complete
1.   Run the following commands:

```bash
	yum clean all # To remove temporary files and fastestmirror checks
	shutdown -h now
```

### Back on the Host
---

1.   Run the following commands:

```bash
	vagrant package --output 'Name'.box --base 'VirtualBox_Name'
```

### If you wish to test your new BaseBox

1.   Run the following commands:

```bash
	vagrant box add 'VirtualBox_Name-2' 'Name'.box # VirtualBox_Name-2 should not already exist in VirtualBox
	vagrant init 'VirtualBox_Name-2'
	vagrant up
```

### Otherwise, add it to your Vagrant until we make the location public

1.   Run the following commands:

```bash
	vagrant box add 'VirtualBox_Name-2' 'Name'.box
```

2.   Modify any existing Vagrantfile's with:

```bash
	config.vm.box = "VirtualBox_Name-2"
	# config.vm.box_url = "" # Until we make the BaseBox public
```
