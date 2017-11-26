# Jenkins job to create unattended Ubuntu installation ISO #

## Requirements

These packages must be available on jenkins build node:

* 7zip - to extract ISO, mount -o loop needs root permissions:

	$ sudo apt-get install p7zip-full p7zip-rar

* genisoimage - mkisofs tool

	$ sudo apt-get install genisoimage

* fakeroot - for updating initrd image

	$ sudo apt-get install fakeroot

## Creating new job:

1. Create new 'Pipline' job
2. Choose 'Pipeline script from SCM'
3. Use github repository url: 'https://github.com/dariusbakunas/jenkins-ubuntu-preseed.git'
4. Click 'Save'
5. First build will populate new build parameters for the job, you will be able to set those on next build