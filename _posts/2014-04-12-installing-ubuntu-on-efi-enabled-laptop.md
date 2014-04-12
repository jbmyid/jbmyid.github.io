---
layout: post
title: "Installing Ubuntu 13.10 in dual boot with windows 8.1"
quote: "Installing ubuntu 13.10 on efi enabled laptop"
image: /media/2014-04-12-installing-ubuntu-on-efi-enabled-laptop/warty-final-ubuntud.jpg
video: false
comments: true
logo: /media/2014-04-12-installing-ubuntu-on-efi-enabled-laptop/ubuntu.jpeg
---
#Installing Ubuntu 13.10 in dual boot with windows 8.1

Recently I bought Lenovo G510 which have efi feature for boot options. I have struglled a lot to install ubuntu and windows both in dual boot.
After a long strugle I have learned how to install windows 8.1 and ubuntu 13.10 in dual boot.

Here are steps I followed to work it out.

1. Before installing windows 8.1 make sure you are installing it in EFI mode.
	* Go to boot menu and disable Legacy mode and enable EFI mode only
	* Turn of Secure Boot
2. After enabling EFI install windows 8.1
3.	Now create bootable usb of ubuntu 13.10 
	* You can use linux startup disk creator to create bootable usb.
	* Or use universal usb installer for windows.

	{% include image.html url="/media/2014-04-12-installing-ubuntu-on-efi-enabled-laptop/Create-a-USB-stick.png" width="100%" %}


4. Go to disk manager, and shrink the partition to free up space to install Ubuntu.
	* Shrink about 10 - 20 Gb whichever suits you.
5.	From windows TURN OFF fast startup from Power option.
	
	{% include image.html url="/media/2014-04-12-installing-ubuntu-on-efi-enabled-laptop/power.png" width="100%" %}

6. Now plug ubuntu bootable usb and click on `restart` holding `SHIFT` button.
7. Now blue screen will appear, look for `Use Device` option and select and press `enter`
	* It will show `EFI USB Device`, select and press `enter` 
8. Now the Ubuntu boot screen will appear.
	* You can now try Ubuntu without installing or install directly.
9. If you see blank screen after selecting any option follow
	* Select any of the option from boot screen but dont press enter

	{% include image.html url="/media/2014-04-12-installing-ubuntu-on-efi-enabled-laptop/boot.png" width="100%" %}

	* Press `e` and add `acpi_osi=Linux acpi_backlight=vendor` in boot options before `quiet splash`
	* Press F10 to boot
	<pre>Note: This step will be required each time you boot.</pre>
10. If you see a message "System is running low graphics" kind of thing from where you can not prceed further.
	* Add `nomodeset` in boot options before `quiet splash`
	* And follow 9 th step
11. Now moving to installation of ubuntu 
	* Install from try ubuntu or Install unbuntu option
	* Select `Something Else` option
	* Now when the partition selection screen appear choose free space which was freed from windows of 10/20 GB
	* click `+` button and type 2 GB(i.e 2048 MB) and select swap area option from drop down and press create button
	* select free space again and click `+` button and select `/` (root) from dropdown and press create.
12. Continue installation
13. Now while booting ubuntu if you encounter blank screen with blinking cursor then enable Legacy Support from boot menu (I had to do it. Without enabling Legacy support i was not able to see ubuntu ui). 

# Afer installation 
If you used point 9. or 10. to boot you will need to add it in boot default options. To do so follow these steps.

* Open terminal 
* Type `sudo vi /etc/defaults/grub` and press enter
* Add `acpi_osi=Linux acpi_backlight=vendor` before `quiet splash` in `GRUB_CMDLINE_LINUX_DEFAULT` option and save the file
* Type `sudo update-grub` in terminal and press enter
* Now try rebooting you wont need adding these options further while booting.

<pre>Yes we are done</pre>

{% include image.html url="/media/2014-04-12-installing-ubuntu-on-efi-enabled-laptop/yes.jpeg" width="100%" %}


