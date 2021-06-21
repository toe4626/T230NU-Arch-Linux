# T230NU Arch Linux
This guide details how to get arch linux up and running on the Samsung Galaxy Tab 4 T230NU. The device must be rooted first, thankfully that is pretty easy. It is expected that the device has an external SD Card (4gb or more). This technically can be done on the internal SdCard, though that greatly limits the space you have to work with.

## Disclaimer
This has only been tested using the T230NU, and as such can't be garaunteed to work on any other device. That said, this has only been tested thus far by myself, and as such it can be expected that you could either successfully install Arch on your old tablet, or you could brick your device. Whatever you do, do it responsibly.

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

Before moving forward, it would be good to use Titanium Backup to create a backup of your apps or device. Hopefully you won't need it.

## Installing Arch

The T230NU tablet uses the Marvell PXA1088 chipset with ARMv7 architecture. It seems though, that a specialised Arch release for ARMv7 architecture with this specific chipset doesn't exist, so you should download the [ARMv7 Multi-platform](http://os.archlinuxarm.org/os/ArchLinuxARM-armv7-latest.tar.gz) Arch release (I am not saying other releases don't work, I just have not tested them, and know this one works). If you havent already opened BusyBox and allowed it to install its Linux tools, go ahead and do so before moving forward.

### Create the disk image
```
# su
# dd if=/dev/zero of=/storage/extSdCard/arch.img bs=1048576 count=1024
# mke2fs -F -T ext4 /storage/extSdCard/arch.img
```
### Create chroot environment
```
# mkdir /storage/extSdCard/chroot
# losetup /dev/block/loop7 /storage/extSdCard/arch.img
# mount -t ext4 /dev/block/loop7 /storage/extSdCard/chroot
# tar -xzf -C /storage/extSdCard/chroot /storage/extSdCard/arch.tar.gz
```
### Configure chroot environment
```
# rm /storage/extSdCard/chroot/etc/resolv.conf
# echo "nameserver 8.8.8.8" > /storage/extSdCard/chroot/etc/resolv.conf
# echo "export HOME=/root" >> /storage/extSdCard/chroot/etc/profile
# echo "unset LD_PRELOAD" >> /storage/extSdCard/chroot/etc/profile
# echo "cd /root" >> /storage/extSdCard/chroot/etc/profile
```
### Mount and enter chroot environment
```
# mount -o bind /dev/ /storage/extSdCard/chroot/dev/
# mount -t proc proc /storage/extSdCard/chroot/proc/
# mount -t sysfs sysfs /storage/extSdCard/chroot/sys/
# mount -t tmpfs tmpfs /storage/extSdCard/chroot/tmp/
# mount -t devpts devpts /storage/extSdCard/chroot/dev/pts/
# chroot /storage/extSdCard/chroot/ /bin/bash -l 
```

You can also use [this shell script]() that will do all of this for you. You will need to preform the mounts once per terminal session, meaning every time you open up the terminal for the first time after booting, you will need to perform the last section (Mount and enter chroot environment) every time. Currently, Terminal Emulator opens to the non-root user and you have to mount and chroot into Arch. To always open the terminal directly into your Arch environment, you can copy [this other shell script]() into your sdCard (NOT EXTERNAL) and set Terminal Emulator's 'Initial Command' to the path of the mounting shell script. 

```
#!/system/xbin/sh
su -c losetup /dev/block/loop7 /storage/extSdCard/arch.img
su -c mount -o bind /dev/ /storage/extSdCard/chroot/dev/
su -c mount -t proc proc /storage/extSdCard/chroot/proc/
su -c mount -t sysfs sysfs /storage/extSdCard/chroot/sys/
su -c mount -t tmpfs tmpfs /storage/extSdCard/chroot/tmp/
su -c chroot /storage/extSdCard/chroot/ /bin/bash -l 
```

If you have installed your busybox tools to a directory that ISN'T /system/xbin, then you will need to change the first line of the shell script to match your custom install location.

Et voila! Arch is installed on your device!

### Here There Be Monsters
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

### Parting Words
So far, I havent been able to get xorg to work on this device because there is no /dev/tty0 for it to use. This could be a simple fix, or an issue inherent in the architecture of the device, I have no clue. The idea is to be able to run something lightweight like DWM or i3 through a VNC Viewer application on the tablet (such as 'androidVNC' in the Play Store). If you find this guide useful and actually get it figured out, please let me know. Feel free to make pull requests if there is more information you find to be relevant in implementing and customizing this Arch install.

