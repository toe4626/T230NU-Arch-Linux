# T230NU Arch Linux
This guide details how to get arch linux up and running on the Samsung Galaxy Tab 4 T230NU. The device must be rooted first, thankfully that is pretty easy. It is expected that the device has an external SD Card (4gb or more). This technically can be done on the internal SdCard, though that greatly limits the space you have to work with.

## Special Thanks to
- [Tomodoro](https://github.com/Tomodoro) for verifying this guide works and getting the VNC graphical environment running

## Disclaimer
This has only been tested using the T230NU, and as such can't be garaunteed to work on any other device. That said, this isn't the device's intended original purpose, and as such it can be expected that you could either successfully install Arch on your old tablet, or you could brick your device. Whatever you do, do it responsibly.

## Rooting The T230NU
First we must download some software to begin rooting:

- Download [Samsung Galaxy Tab 4 7.0 T230NU Driver](https://org.downloadcenter.samsung.com/downloadfile/ContentsFile.aspx?CDSite=US&CttFileID=6968032&CDCttType=SW&ModelType=C&ModelName=SM-T230NZWAXAR&VPath=SW/201801/20180105081701718/SAMSUNG_USB_Driver_for_Mobile_Phones_ver_1.5.65.0.exe) so that Windows recognizes the tablet as a device
- Download [Odin v3.0.9](https://odindownload.com/download/Odin_v3.09.zip) zip archive, and unzip.
- Download the latest [degas TWRP ROM](https://eu.dl.twrp.me/degas/) zip archive.
- Download the latest [SuperSU APK](http://supersuroot.org/downloads/SuperSU-v2.82-201705271822.zip) zip archive.

Install the Samsung driver. After the driver is installed, ensure that USB debugging is enabled. To do this, go to Settings -> General -> Developer Options -> Debugging -> USB Debugging. If 'Developer Options' is not available in the General panel of Settings, then you will need to enable the developer options by going to Settings -> General -> About Device -> Build Number, and tap 'Build Number' repeatedly until it confirms that developer options are enabled.

Next, run Odin v3.0.9. and connect the tablet to the pc using a microUSB data cable. Hold the tablet's Home, Volume Down, and Power button until the device restarts into Download Mode. Press volume up to confirm. In the "Files[Download]" section of Odin's UI, select "AP". When prompted with the file chooser dialogue, select the degas TWRP ROM that you downloaded and choose ok. Un-select "Auto Reboot" in the "Options" section of the UI. Click "Start" and wait for the rooting process to complete.

With the device still connected, copy the SuperSU APK into the tablet's extSdCard. Hold the Home, Volume Up, and Power button to enter recovery mode (This should be the TWRP recovery mode. If you let the device reboot normally before doing this step, the device will overwrite the TWRP recovery with the original Samsung recovery mode, and you will need to go through the above paragraph again). If you have done this correctly, you should see options such as "Install, Wipe, Backup..." etc. Select Install, and then find the SuperSU zip file in the extSdCard. After it is finished installing, reboot the device normally.

Once the device is booted, find the SuperSU apk that should now be available in the app drawer and follow the instructions it provides. Once complete, you should have a rooted device. To verify this, download 'Root Checker' from the app store and try to verify root. Hopefully if all went well, you have a rooted T230NU

## Quick Preperations
Before moving forward, I think there are some integral apps that you should download before moving forward, that allow you much greater control of your device. Some of these apps may or may not require purchases, I can't remember. I have purchased them because I use them frequently.
- DiskUsage - Gives you a visual representation of your storage a la WinDirStat
- Root Explorer - No-frills root-capable file explorer.
- Terminal Emulator - Bare-bones terminal emulator.
- Titanium Backup - Backup/application manager. Great if you want to save some space/CPU/RAM by disabling all google products.
- BusyBox Free - Necessary for enabling necessary linux commands in the terminal.
- Hacker's Keyboard - Soft keyboard that gives you all the bells and whistles of a PC keyboard.
- AndroidVNC - VNC viewer for old android devices

Before moving forward, it would be good to use Titanium Backup to create a backup of your apps or device. Hopefully you won't need it.

## Installing Arch

The T230NU tablet uses the Marvell PXA1088 chipset with ARMv7 architecture. It seems though, that a specialised Arch release for ARMv7 architecture with this specific chipset doesn't exist, so you should download the [ARMv7 Multi-platform](http://os.archlinuxarm.org/os/ArchLinuxARM-armv7-latest.tar.gz) Arch release (I am not saying other releases don't work, I just have not tested them, and know this one works). If you havent already opened BusyBox and allowed it to install its Linux tools, go ahead and do so before moving forward.

### A Note Regarding Storage

This guide was originally created and tested so that the arch image and chroot environment would exist on an external SD card so that you can have a much larger disk image. The internal sdcard can be used also but since it has such little space to work with, it can make it difficult to create the disk image without running out of memory, and if the disk image is created but is too small then you wont be able to unpack the arch tar. If you need to use internal memory, change the line 
```
# export STOR=/storage/extSdCard
```
to
```
# export STOR=/storage/emulated/0
```
### Create the disk image
```
# su
# export STOR=/storage/extSdCard
# dd if=/dev/zero of=$STOR/arch.img bs=1048576 count=1024
# mke2fs -F -T ext4 $STOR/arch.img
```
### Create chroot environment
```
# mkdir $STOR/chroot
# losetup /dev/block/loop7 $STOR/arch.img
# mount -t ext4 /dev/block/loop7 $STOR/chroot
# tar -xzf $STOR/arch.tar.gz -C $STOR/chroot
```
### Configure chroot environment
```
# rm $STOR/chroot/etc/resolv.conf
# echo "nameserver 8.8.8.8" > $STOR/chroot/etc/resolv.conf
# echo "export HOME=/root" >> $STOR/chroot/etc/profile
# echo "unset LD_PRELOAD" >> $STOR/chroot/etc/profile
# echo "cd /root" >> $STOR/chroot/etc/profile
```
### Mount and enter chroot environment
```
# mount -o bind /dev/ $STOR/chroot/dev/
# mount -t proc proc $STOR/chroot/proc/
# mount -t sysfs sysfs $STOR/chroot/sys/
# mount -t tmpfs tmpfs $STOR/chroot/tmp/
# mount -o gid=5,mode=620 -t devpts devpts $STOR/chroot/dev/pts/
# chroot $STOR/chroot/ /bin/bash -l 
```

You will need to preform the mounts once per terminal session, meaning every time you open up the terminal for the first time after booting, you will need to perform the last section (Mount and enter chroot environment) every time. Currently, Terminal Emulator opens to the non-root user and you have to mount and chroot into Arch. To always open the terminal directly into your Arch environment, you can create a shell script like the one below and save it to your sdCard (NOT EXTERNAL) and set Terminal Emulator's 'Initial Command' to the path of the mounting shell script (be sure to change the storage directory if you need to). 

```
#!/system/xbin/sh
export STOR=/storage/extSdCard
su -c losetup /dev/block/loop7 $STOR/arch.img
su -c mount -o bind /dev/ $STOR/chroot/dev/
su -c mount -t proc proc $STOR/chroot/proc/
su -c mount -t sysfs sysfs $STOR/chroot/sys/
su -c mount -t tmpfs tmpfs $STOR/chroot/tmp/
su -c chroot $STOR/chroot/ /bin/bash -l 
```

Also, if you have installed your busybox tools to a directory that ISN'T /system/xbin, then you will need to change the first line of the shell script to match your custom install location.

Et Voila! Arch is installed!

## Moving Foreward

### Updating Arch

Before galavanting off into the land of bugs and stack overflow light reading, you will probably need/want to do some things first:
- You will want to update the Arch system, but you will probably get an error in regards to certificates or the pacman keychain. This is because it is trying to use the x86 certs, and will need the proper ARM certs. You will want to run:
```
# pacman-key --init && pacman-key --populate archlinuxarm && pacman-key --refresh-keys
```
- Update the system:
```
# pacman -Syu
```
- Because Linux is a kernel, and the linux kernel is already on the android device on its lowest level, you can remove the Linux kernel and filesystem that was unzipped into the chroot environment to free up a good chunk of oh-so-valuable storage. To do this run:
```
# pacman -Rs linux-armv7
```

### VNC Graphical Environment

To run a VNC environment we will need to install some things:
```
# pacman -S tigervnc xorg-server openbox xterm ttf-dejavu ttf-liberation
```
Create a password using ```vncpasswd``` which will store the hashed password in ```~/.vnc/passwd```, then create the config file ```~/.vnc/config``` and set the correct settings:
```
session=openbox
geometry=1280x720
localhost
alwaysshared
```
If you want to be able to access the device using a VNC viewer on a seperate device, you will need to omit ```localhost```. Also, if you are using another window manager, such as DWM or something, change the session name accordingly.

We can create our first new user quickly by running:
```
# echo ":1=SomeUsername" >> /etc/tigervnc/vncserver.users
```
The number ':1' in the file corresponds to a TCP port. By default, :1 is TCP port 5901 (5900+1). If you want to have more than one VNC server going at a time, a second instance can then run on the next highest, free port, i.e 5902 (5900+2).

Now, if you have set up VNC before on Arch, you will be tempted to try and test the VNC server the systemd way, but since we are inside a chroot environment, this simply won't work. We have to start VNC with: 
```
# vncserver :1
```
You should now be able to minimize the terminal emulator, open AndroidVNC, and connect to ```127.0.0.1:590X``` where X is the number we provided to vncserver, in this case ```1```, to log in as ```SomeUsername```. Your VNC password will be the password you created with ```vncpasswd```.

### Errors
Starting VNC server, in the terminal you will see a lot of error messages regarding XKEYBOARD, and these can be ignored. These errors refer to Android-specific actions that are not found in Linux systems, and therefore XKEYBOARD is unsure how to map them to the Arch system.

If you figure out how to address or remove the errors:
```
xinit: XFree86_VT property unexpectedly has 0 items instead of 1
```
or 
```
dbus-update-activation-environment: error: unable to connect to D-Bus: Using X11 for dbus-daemon autolaunch was disabled at compile time, set yout DBUS_SESSION_BUS_ADDRESS instead
```
Then please let me know!

### Parting Words
Thanks for making it through the guide! If you find this guide useful and actually get it working, please let me know, I always love to see what people can do with these kinds of things. Feel free to make pull requests if there is more information you find to be relevant in implementing and customizing this Arch install.

#### Sources

- https://archlinuxarm.org/forum/viewtopic.php?f=3&t=12797
- https://technohackerblog.blogspot.com/2016/07/running-arch-linux-in-chroot-on-android.html?m=1
- https://blog.flexispy.com/how-to-root-the-samsung-galaxy-tab-4-sm-t230nu/
