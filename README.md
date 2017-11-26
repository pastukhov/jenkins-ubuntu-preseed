# Jenkins job to create unattended Ubuntu Sever 16.04 ISO #

## Requirements

These packages must be available on jenkins build node:

* 7zip - to extract ISO, mount -o loop needs root permissions:

	$ sudo apt-get install p7zip-full p7zip-rar

* genisoimage - mkisofs tool

	$ sudo apt-get install genisoimage

* fakeroot - for updating initrd image

	$ sudo apt-get install fakeroot

* xorriso - for making UEFI bootable iso

	$ sudo apt-get install xorriso

## Features

1. Configurable username, hostname, authorized public ssh key, domain, timezone and root disk device
2. Installed openssh-server and python packages (minimum requirements for ansible)

## Creating new job:

1. Create new 'Pipline' job
2. Choose 'Pipeline script from SCM'
3. Use github repository url: 'https://github.com/dariusbakunas/jenkins-ubuntu-preseed.git'
4. Click 'Save'
5. First build will populate new build parameters for the job, you will be able to set those on next build
6. Current installation uses the whole disk and creates single root partition
7. Check 'Configure' step for randomly generated password you can use to login to freshly installed system

### References

* [Automating-ubuntu-preseed](https://github.com/asniii/Automating-ubuntu-preseed/blob/master/64bit_system/files/15/u_preseed)
* [Ubuntu-16.04-Unattended-Install](https://github.com/dsgnr/Ubuntu-16.04-Unattended-Install)
* [Ubuntu-16.04-Unattended-Install for UEFI](https://github.com/dsgnr/Ubuntu-16.04-Unattended-Install)
* [ubuntu-unattended](https://github.com/netson/ubuntu-unattended)
* [example-preseed](https://help.ubuntu.com/lts/installation-guide/example-preseed.txt)