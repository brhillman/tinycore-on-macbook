# tinycore-on-macbook
Notes and tools for running tinycore linux on a 2012 Macbook Air

This will be some rough notes to be cleaned up lated...

## Overview
Installed TinyCore to a removable USB thumbdrive to test compatibility. First created a bootable image on a smaller removable drive using the CorePlus image. Boot from this with extensions and installer, and install to a second removable drive in “frugal” mode. Can now boot from the second drive, and save configuration and extensions and data there. 

## Wireless support
The Broadcom wifi card does not work natively without additional firmware. After too much reading, this actually turned out to be as easy as adding the firmware-broadcom_bcm43xx.tcz extension for the the broadcom wifi card, which can be found at 

http://tinycorelinux.net/15.x/x86_64/tcz/firmware-broadcom_bcm43xx.tcz

This would easily be installable from within the system with an ethernet connection, but I didn’t have this hooked up at the time. Rather, I downloaded the extension from my OS X install, and copied to yet another USB drive. Then, booted into TinyCore, and mounted the drive via

```
sudo mount /dev/sdc2 /mnt/sdc2
```

I am not sure if it is necessary, but most advice seemed to recommend removing b43 from loaded modules, so first
```
Sudo modprobe -r b43
```
And then loading the firmware extension:
```
tce-load -i <path-to-firmware-broadcom_bcm43xx.tcz>
```
And then running the `wifi.sh` script with `sudo wifi.sh`. This seems to get everything working. To make persistent, we need to copy the firmware extension to our bootable drive at `/mnt/sdb1/tce/boot/optional/`, and then edit `/mnt/sdb1/tce/boot/onboot.lst` to include the name of the firmware extension. I then also added the `blacklist=b43` argument to the boot options by editing the `extlinux.conf` file and appending to the `APPEND` line.

## PIV-C certificate reading in Firefox 
Need to install OpenSC. There is no extension package in TCL for this, so need to build from source (and will ultimately need to create a tce package). This required a handful of dependencies, including pcsc-lite which also needed to be built from source, with additional dependencies available from the tce repos.

Once OpenSC is compiled and installed, the opensc-pksc11.so module is supposed to be loaded into Firefox, but this was giving me an error saying that the module could not be loaded. I tried in the standard Firefox install from the tce packages, and also with the esr version which is loaded by running firefox_getLatest -e. This wasn’t either to load the module either.

One suggestion found on the internet: try loading pcscd before loading pksc11 module in Firefox?

Another idea: try without betterfox profile settings?

## Yubikey certificate support
Yubico-piv-tool dependencies:
Check
Gengetopt
ccid (I think needed for driver support?)

Build and package these as extensions:
my-check-0.15.2.tcz
my-gengetopt-0.23

This didn’t work initially. Some searching suggested running the pcscd daemon in the foreground with debugging turned on. This can be done via:

```
pcscd -dfe
```

The options mean `debug`, `foreground`, and `error`. Running this and then trying to use the `yubico-piv-tool` shows that `pcsc` is looking for drivers in `/usr/` rather than in `/usr/local/`. This is controlled by a flag during configuration for the `pcsc` build, `usbdropdir`, which defaults to `/usr/lib/pcsc/drivers`. I just had to change this to `/usr/local/lib/pcsc/drivers` via:

```
meson setup builddir -Dlibsystemd=false -Dusbdropdir=/usr/local/lib/pcsc/drivers --prefix=/usr/local
```

and then rebuilding and repackaging the extension. Fresh reboot and loading, and now yubico-piv-tool is able to see the yubikey. So now, after loading the ykcs11 module into firefox, authentication works as expected!

## TODO
  - webcam
  - extension recipes / upload to repo
  - proper sleep / suspend
  - document xorg settings and drivers (xorg over xvesa)
  - mouse stuff
  - copy and paste not working between terminal and browser
