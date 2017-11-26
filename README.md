# Jenkins job to create unattended Ubuntu installation ISO #

## Requirements

* 7zip - to extract ISO, mount -o loop needs root permissions:

	$ sudo apt-get install p7zip-full p7zip-rar

* genisoimage - mkisofs tool

	$ sudo apt-get install genisoimage

* fakeroot - for updating initrd image

	$ sudo apt-get install fakeroot