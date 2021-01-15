# lglaf-h872
Fork of https://gitlab.com/runningnak3d/lglaf specifically the `h872-miscwrite` branch to capture my
perspective of [runningnak3d's xda guide](https://forum.xda-developers.com/t/root-h872-up-to-and-including-20g.3775518/)
and then installing LineageOS 17.1 and root.

## Disclaimer
This is specifically for ***H87220g***. This process worked for me but use at your own risk.
As always this kind of thing will void your waranty and could brick your device. There is no implied
support or help of any kind here. I'm simply documented the steps I took for my own reference. You
should assume that you will ***loose all your data*** during this proces, so back it up first if it
is important.

## LG G6 h872 <a name="lg-g6-h872"/></a>
The LG G6 launched February 26, 2017. The phone came out of the box with Android 7.0 and later
upgraded to Android 8.0. Until now I've only ever used Nexus or Pixel devices which make it super
simple to unlock and flash images via `fastboot` to the point where although I'd flash TWRP to them I
only ever used it to clear the cache occasionally and just used `fastboot` for all my flashing needs.

So I was a bit shocked to learn that non Google approved devices frequently are either missing
`fastboot` or have crippled vesions that can't flash. Such is the case with the `h872`.

### Quick links
* [Step by step process](#step-by-step-process)
  * [1. Arch Linux requirements](#arch-linux-requirements)
  * [2. Unlock the bootloader](#unlock-the-bootloader)
  * [3. Fix Download mode](#fix-download-mode)
  * [4. Flash TWRP](#flash-twrp)
  * [5. Flash LineageOS](#flash-lineageos)
* [Supplemental guides](#supplemental-guides)
  * [Overview](#overview)
    * [Downlod mode](#download-mode)
    * [Flashing](#flashing)
  * [TWRP Recovery Guides](#twrp-recovery-guides)
  * [SkyHawk Recovery Guides](#skyhawk-recovery-guides)

# Step by step process <a name="step-by-step-process"/></a>
Step by step guide to take an LG H872 running stock H87220g to a rooted LineageOS 17.1. The steps
should be followed sequentially as called out by their numbering and order. I'll use the term
`device` to refer to the LG H872 and `host` to refer to an `Arch Linux` machine and `Guest VM` to
refer to a Windows 10 Virtual Box VM hosted on the Arch Linux `host`.

I'm using `>` to delineate loosly the area to focus on or actions to take e.g. navigate to 
`Settings >General >About phone` would indicate you should find the devices `Settings` then tap
`General` then tap `About phone`. Also when describing a proces I'll use `run` for shell commands
that should be executed on the host and `tap` for actions to take on the device.

## 1. Arch Linux requirements <a name="arch-linux-requirements"/></a>
Unfortunately this isn't a complete list as I set this machine up for this task a long time ago but
I'll try to capture as much as I can here.

**Install dependencies:**
```bash
$ sudo pacman -Sy android-tools gvfs-mtp libmtp virtualbox virtualbox-ext-oracle \
    virtualbox-guest-iso virtualbox-guest-utils virtualbox-host-modules-arch
```

**Groups your user should be part of:**
* `adbusers`
* `uucp`
* `vboxusers`

## 2. Unlock the bootloader <a name="unlock-the-bootloader"/></a>

### 2.a Enable developer mode <a name="enable-developer-mode"/></a>
Developer mode simply enables the hidden `Developer options`

1. On your device navigate to `Settings >General >About phone >Software info`
2. Tap the `Build number` entry 7-8 times until you see the pop up `You are now a developer!`

### 2.b Allow OEM unlock <a name="allow-oem-unlock"/></a>
1. On your device navigate to `Settings >General >Developer options`
2. The first time you access the `Developer options` you'll likely need to accept a warning
3. Toggle the switch to enable the `Enable OEM unlock` option

### 2.c Enable USB debugging <a name="enable-usb-debugging"/></a>
1. On your device navigate to `Settings >General >Developer options`
2. Toggle the switch to enable the `USB debugging` option

### 2.d Actually unlock the bootloader <a name="actually-unlock-the-bootloader"/></a>
Now that the prerequisites are out of the way we can unlock the bootloader.

***WARNING***: this will factory reset your device so make sure you've backed up anything you want to
keep from the device.

1. Plug your USB cable from your host into the device
2. From the host run `adb devices`
3. On the device you now need to tap `Always allow from this computer` and `OK` to authorize the host
   to be able to interact with the device over USB.
4. Reboot into fastboot mode, from host run: `adb reboot bootloader`
5. Validate the bootloader is locked, from host run: `fastboot getvar unlocked`
6. Unlock the bootloader, from host run: `fastboot oem unlock`
7. Reboot the device, from host run: `fastboot reboot`

### 2.e Walk through the first run wizard <a name="walk-through-the-first-run-wizard"/></a>
Because unlocking reset the device to its factory default state once its boots from the previous step
you'll need to tap through the first run wizard to get back to a runnning system we can work with.

## 3. Fix Download mode <a name="fix-download-mode"/></a>
Because LG shipped a crippled version of `fastboot` we'll need to work around this limitation by
flashing an alternate `Download mode` from `H91810p` which won't fix `fastboot` but will provide a
different entry point for flashing.

### 3.a Setup Windows 10 Guest VM <a name="setup-windows-10--guest-vm"/></a>
I originally tried an older version of Windows but driver support required Windows 10

1. Validate `virtualbox-ext-oracle` extension is installed by navigating to `File >Preferences…>Extensions`
2. Add your user to the vboxusers group: `sudo usermod -a -G vboxusers <USER>`  
3. Reboot your host system
4. Plugin the USB cable from your host into your device
5. Before starting the Guest VM changes the settings to capture the phone as a USB device
6. Navigate to the Guest VM's `Settings >USB` and click the add icon on the right and select `LGE LG-H872`
7. Reboot the device into the bootloader: `adb reboot bootloader`
8. Now that the device is in fastboot mode it will have a differnt USB id
9. Repeat step 6 to capture the USB but this time it will be called a generic Google Android I think
10. Boot the VM

### 3.b Install JRE on Guest VM <a name="install-jre-on-guest-vm"/></a>
Install JRE on Windows 10 VM which is a dependency of `adb`

1. Navigate to https://www.oracle.com/java/technologies/javase-jre8-downloads.html
2. Click `jre-8u271-windows-x64` and check the box to agree to terms
3. Copy redirection download link https://www.oracle.com/webapps/redirect/signon?nexturl=https://download.oracle.com/otn/java/jdk/8u271-b09/61ae65e088624f5aaa0b1d2d801acb16/jre-8u271-windows-x64.exe
4. Now copy out the `nexturl` portion and change `otn` to `otn-pub` to give you
   `https://download.oracle.com/otn-pub/java/jdk/8u271-b09/61ae65e088624f5aaa0b1d2d801acb16/jre-8u271-windows-x64.exe`
5. Download and install `jre-8u271-windows-x64.exe` on the VM

### 3.c Install platform-tools on Guest VM <a name="install-platform-tools-on-gutest-vm"/></a>
1. Download [platform-tool](https://www.xda-developers.com/install-adb-windows-macos-linux/) on Guest VM
2. Extract to `C:\` which ends up being `C:\platform-tools`

### 3.d Install LG USB Universal Driver on Guest VM<a name="install-lg-usb-universal-driver-on-guest-vm"/></a>
1. Navigate to https://www.lg.com/us/support/help-library/lg-mobile-drivers-and-software-CT10000027-20150179827560
2. Click `Windows Universal Mobile Drivers Download` to download on the VM
3. Install `LGMobileDriver_WHQL_Ver_4.5.0.exe` to the VM
4. Now adb should work
5. Open a shell on the VM and navigate to: `cd C:\platform-tools`
6. Run: `adb devices`

### 3.e Install Microsoft Visual C++ Redistributable 2010 <a name="install-microsoft-visual-redistributable"/></a>
1. Navigate to https://www.microsoft.com/en-us/download/details.aspx?id=5555
2. Select `Microsoft Visual C++ 2010 Redistributable Package (x86)`
3. Download copy to VM and install

### 3.f Install patched LGUP tool <a name="install-patched-lgup-tool"/></a>
1. Navigate to https://forum.xda-developers.com/t/guide-unbrick-patch-lgup-to-unlock-features-unbrick-v20-variant-mismatch-fix.3652222/
2. Download and copy `LGUP&V20dll_Patched.zip` to VM
3. Extract to `C:\Program Files (x86)\LG Electronics` which should already exist from the USB driver
   install
4. Ensure that the binary is at `C:\Program Files (x86)\LG Electronics\LGUP\LGUP.exe`

### 3.g Use LGUP tool to flash working download mode <a name="use-lgup-tool-to-flash-working-download-mode"/></a>
1. Download firmware for H918 10p to get a laf partition with COPY opcode
   a. Navigate https://sourceforge.net/projects/t-mobile-lg-v20-firmware/files/h918/10p/H91810p_00_0717.kdz/download  
   b. Download and copy `H91810p_00_0717.kdz` to the VM  

2. Download your devices firmware
   a. Navigate to https://lg-firmwares.com/downloads-file/17788/H87220g_00_1228  
   b. Download the corresponding firmware `H87220g_00_1228.kdz` and copy it to the VM  

3. Launch LGUP and flash laf
   a. Choose `PARTITION DL` leave `BIN File` checked  
   b. Click `…` in the `File Path` section to browse to `H91810p_00_0717.kdz`  
   c. Click `Start` which will then show you `Partitions list` to flash  
   d. Choose `laf only` and click `OK`  
   e. The phone will reboot into Firmware Update mode and will get again change USB id values  
   f. To allow USB in Virtual Box click `Devices >USB` check the `LG Electronics…` one  
   g. Now you'll get a warning about partition changes click `Yes`  
   h. Next to `Progress` in LGUP's UI is the `Step` section once it moves past laf pull USB cord  
   i. Note the device will be stuck in the Firmware Update mode on some partition after laf  
   j. Click `Exit` in LGUP and close out  
   k. Now plug the USB cable back in  

4. Relaunch LGUP and flash all partitions except laf
   a. Allow USB in Virtual Box click `Devices >USB` check `LG Electronics…` one  
   b. Choose `PARTITION DL` leave `BIN File` checked  
   c. Click `…` in the File Path section to browse to `H87220g_00_1228.kdz`  
   d. Click `Start` which will then show you `Partitions list` to flash  
   e. Choose everything `except laf` and click `OK`  
   f. It will stay in Firmware Update mode and install the correct firmware on everything except laf
   except laf then reboot. It should just boot back to what it looked like before with USB debug
   still enabled
5. Shutdown the VM we shouldn't need it anymore

## 4. Flash TWRP <a name="flash-twrp"/></a>
Now that we have a working `Download mode` from `H918` in the `laf` we can proceed to install TWRP

### 4.a Flash TWRP to laf <a name="flash-twrp-to-laf"/></a>
1. Boot your `h872` into the new `laf` base `Download mode`
   a. Power off your device  
   b. Hold down `Vol up` and plug in the USB cable  
   c. Your device will boot into ***Download mode***  

2. Install TWRP to `laf` from your Arch Linux host
   ```bash
   # Clone the repo
   $ git clone https://github.com/phR0ze/lglaf-h872
   
   # Change directory to the repo
   $ cd lglaf-h872
   
   # Execute step1.sh
   $ ./step1.sh
   ```

3. Follow instructions in the output of `./step1.sh` to unplug USB once complete
4. Your device will poweroff
5. Hold down `Vol up` and plug in the USB cable to boot into TWRP. Note first you'll see that it goes
   into `Download mode` then will eventually boot TWRP. It takes awhile

### 4.b Wipe encryption <a name="wipe-encryption"/></a>
1. From TWRP hit the `WIPE`
2. Navigate back to the root menu  
3. Hit `Reboot` and select `Poweroff`  

### 4.c Install TWRP to recovery <a name="install-twrp-to-recovery"/></a>

1. Install TWRP to `recovery`
   ```bash
   $ ./step2.sh
   ```
2. Once complete the device will reboot into TWRP and run post scripts

3. Once complete it will reboot once more but this time into the OS

4. Work throught the first run wizard

## 5. Flash LineageOS <a name="flash-lineageos"/></a>
To get LineageOS 17.1 installed on the H872 I'll be following [this XDA guide from bernardobas](https://forum.xda-developers.com/t/rom-a10-h870-h870ds-h872-us997-lineageos-17-1-for-lg-g6-unofficial.4137809/)
From what I can tell reading through the post it is exactly what I'm looking for in a custom ROM.
I'll be using the nightly builds as they have all the extra apps stripped out. I prefer the Google
apps anyway.

### 5.a Prerequisites <a name="prerequisites"/></a>
Continuing where we left off means the bootloader is unlocked and TWRP is on the laf and recovery

1. [Enable developer mode](#enable-developer-mode)
2. [Enable USB debugging](#enable-usb-debugging)

### 5.b Flash latest SkyHawk Recovery <a name="flash-latest-skyhawk-recovery"/></a>
1. Navigate to https://sourceforge.net/projects/pa-g6/files/Releases/SHRP/
2. Click through and download latest `20.07.2020 >H872 >SHRP_v2.3.2_h872-233920072020.zip`
3. [Reboot into TRWP Recovery](#reboot-into-twrp-recovery)
4. From device tap `Advanced >ADB Sideload` then swipe to activate
5. From host run: `adb sideload SHRP_v2.3.2_h872-233920072020.zip`

### 5.c Flash latest bootloader  <a name="flash-latest-bootloader"/></a>
1. Navigate to https://sourceforge.net/projects/lg-g6/files/Bootloaders/
2. Click through to download latest `H872 >LG-H872_Oreo-20a_Bootloader_B2.zip`
3. [Reboot into recovery from TWRP](#reboot-into-recovery-from-twrp)
4. First thing you see is a `Mount` screen with `Decrypt Data` just hit the close button
5. [Wipe before flashing ROM](#wipe-before-flashing-rom)
6. From SkyHawk main menu after wipe tap `Advanced >Sideload` then accept
7. From host, run: `adb sideload LG-H872_Oreo-20a_Bootloader_B2.zip`

### 5.d Flash LineageOS 17.1 <a name="flash-lineageos-171"/></a>
Nightlies didn't include something core to the dialer so trying full OS

1. Navigate to https://sourceforge.net/projects/lg-g6/files/LineageOS%2017.1/
2. Click through to download latest `H872 >lineage-17.1-20210112-UNOFFICIAL-h872.zip`
3. [Reboot into SkyHawk Recovery from SkyHawk](#reboot-into-skyhawk-recovery-from-skyhawk)
4. [Wipe before flashing ROM](#wipe-before-flashing-rom)
5. [Reboot into SkyHawk Recovery from SkyHawk](#reboot-into-skyhawk-recovery-from-skyhawk)
6. From SkyHawk main menu after wipe tap `Advanced >Sideload` check both cache options then accept
7. From host, run: `adb sideload lineage-17.1-20210112-UNOFFICIAL-h872.zip`

### 5.e Flash GApps <a name="flash-gapps"/></a>
1. Navigate to https://opengapps.org/?api=10.0&variant=pico
2. Double check `ARM64`, `Android 10` and `pico` selected as anything more than pico will include
   Google Now and Google Search.
3. [Reboot into SkyHawk Recovery from SkyHawk](#reboot-into-skyhawk-recovery-from-skyhawk)
4. From SkyHawk main menu tap `Advanced >Sideload` check both cache options then accept
5. From host, run: `adb sideload open_gapps-arm64-10.0-pico-20210115.zip`

### 5.e Flash Magisk <a name="flash-magisk"/></a>
1. Navigate to https://github.com/topjohnwu/Magisk/releases
2. Click through to download `Magisk v21.2.zip`
3. [Reboot into SkyHawk Recovery from SkyHawk](#reboot-into-skyhawk-recovery-from-skyhawk)
4. From SkyHawk main menu tap `Advanced >Sideload` check both cache options then accept
5. From host, run: `adb sideload Magisk-v21.2.zip`

### 5.f Reboot <a name="reboot"/></a>
Finally lets reboot from SkyHawk into real system

1. From SkyHawk tap `Reboot >Reboot`
2. Walk through the first run wizard
3. Launch `Magisk` entry to upgrade to `Magisk Manager`
4. Allow installing apps from `Magisk Manager` source

# Supplemental guides <a name="supplemental-guides"/></a>
Sometimes simply saying flash this or reboot into recovery or reboot into download mode is not
sufficient for those new to the scene so I'm capturing here some guides for those processes using the
H872 device.

## Overview <a name="overview"/></a>

### Download mode <a name="download-mode"/></a>
Download mode is a somewhat nebulous term as there are about three implementation that I'm aware of
in this space. Essentially it means a mode that allows you to flash factory images to your device via
an official process. The simplest and most straight forward form is `fastboot`.  Unfortunately
`fastboot` is mostly only fully implemented on Google approved devices like the Nexus or Pixel lines.
Other vendors tend to have a crippled implementation. Samsung devices have `odin` which I know little
about and still others like LG have a generic lower level `Download mode` they call it.

I'll define it as a vendor approved mode requiring a USB cord to flash factory images.

### Flashing <a name="flashing"/></a>
Flashing is also a somewhat generic term as there are at least three different common forms:

1. ***fastboot*** best simplest way but not supported by `H872` due to LG's crippled `fastboot`
2. ***download mode*** a harder lower level mode we used to get TWRP on the `H872`
3. ***recovery*** common across vendors is to use a recovery like TWRP or SkyHawk

## TWRP Recovery Guides <a name="twrp-recovery-guides"/></a>

### Reboot into TRWP Recovery <a name="reboot-into-twrp-recovery"/></a>
**adb method**:
1. [Enable developer mode](#enable-developer-mode)
2. [Enable USB debugging](#enable-usb-debugging)
3. From host run: `adb reboot recovery`

### Reboot into recovery from TWRP <a name="reboot-into-recovery-from-twrp"/></a>
Some operations require reloading the recovery after a recovery operation without booting to the OS

1. From TWRP navigate using the `Back` widget or button until you get to the root level menu
2. Tap `Reboot >Recovery`

### Flash another recovery <a name="flash-another-recovery"/></a>
TRWP is the most common recovery and probably the one you'll end up with if you follow any guides out
on the net. However in some cases specific forks of TWRP like SkyHawk might be required to support
specific custom operations. To switch over to the new recovery you'll need to:

1. Download the target recovery e.g. `SHRP_v2.3.2_h872-233920072020.zip`
2. [Reboot into TRWP Recovery](#reboot-into-twrp-recovery)
3. From device tap `Advanced >ADB Sideload` then swipe to activate
4. From host run: `adb sideload SHRP_v2.3.2_h872-233920072020.zip`

## SkyHawk Recovery Guides <a name="skyhawk-recovery-guides"/></a>

### Integrated Magisk Manager <a name="integrated-magisk-manager"/></a>
SkyHawk will allow you to flash Magisk and perform Magisk manager type operations directly from the
recovery.

### Reboot into SkyHawk Recovery <a name="reboot-into-skyhawk-recovery"/></a>
Identical to TWRP's flow [Reboot into TRWP Recovery](#reboot-into-twrp-recovery)

### Reboot into SkyHawk Recovery from SkyHawk <a name="reboot-into-skyhawk-recovery-from-skyhawk"/></a>
Note the constant rebooting back into the container is necessary to ensure data is setup properly
after flashes

1. Typically operations end with a check button tap that to return the main menu
2. Tap `Reboot >Adavanced >Recovery`

### Wipe before flashing ROM <a name="wipe-before-flashing-rom"/></a>
***Warning*** this will delete all your data. The first thing I do when I get a new device is flash a
custom ROM onto so I don't think much about data on it.

1. From SkyHawk tap `Wipe >Format Data`
2. Type out `yes` to confirm wiping `Internal Storage` and remove encryption
3. From SkyHawk tap `Wipe >Advanced Wipe`
4. Select `Dalvik/ART Cache`, `System`, `data`, `Internal Storage`, `Cache`
5. Tap the accept button

### Remove an app from system partition <a name="remove-an-app-from-system-partition"/></a>
Example removing `AudioFX`

1. reboot into recovery
2. mount system partition
3. go to `system_root/system/priv-app/` and remove the AudioFX folder
4. wipe dalvik and cache
5. reboot

## License <a name="license"/></a>
See the [LICENSE](LICENSE) file for the license (MIT).
