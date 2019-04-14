# Mojave Hackintosh for Lenovo G50-80 (Broadwell 5200U version)

Warning: Please read this guide carefully. Simply copying my configuration file is very likely to not work (because there are many different Lenovo G50-80 according to [PSREF](http://psref.lenovo.com/Product/Lenovo_Laptops/Lenovo_G50)), although you can still play around. You should apply the same steps as mine to get your Hackintosh working on your laptop. If you still want to try, please download it [here](https://codeload.github.com/upupming/Lenovo-G50-80-Clover/zip/master).

Happy hacking :wink:

## Table of Contents

- [Mojave Hackintosh for Lenovo G50-80 (Broadwell 5200U version)](#mojave-hackintosh-for-lenovo-g50-80-broadwell-5200u-version)
  - [Table of Contents](#table-of-contents)
  - [Specs](#specs)
  - [DSDT patches](#dsdt-patches)
    - [Brightness adjustment keys](#brightness-adjustment-keys)
  - [Kexts](#kexts)
  - [Bluetooth](#bluetooth)
  - [Further working](#further-working)
  - [Links](#links)

## Specs

```txt
Model: Lenovo G50-80
Bios version: B0CNA0WW
CPU: i5 5200U, HD 5500 Graphic
RAM: 8GB
Hard drive: Kingston 256 GB SSD + 1 TB Seagate HDD
Audio: Conexant CX20752
Ethernet: RTL8111
```

> BIOS update and specs can be found at [here](https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/lenovo-g-series-laptops/g50-80/80e5/80e5007ccd/pf06sm0m/downloads?linkTrack=Caps%3ABody_SearchProduct&searchType=6&keyWordSearch=PF06SM0M), or simply use this link https://download.lenovo.com/consumer/mobiles/b0cna0ww.exe to download the BIOS update.

## DSDT patches

See [patches.txt](https://github.com/upupming/Lenovo-G50-80-Clover/blob/master/DSDT-patching/patches.txt). I only patched `DSDT.aml` and ignored all `SSDT-*.aml`.

Please follow the [DSDT patching tutorial by RehabMan](https://www.tonymacx86.com/threads/guide-patching-laptop-dsdt-ssdts.152573/) carefully, and you may find [this video](https://www.youtube.com/watch?v=RVMrwMW3jOY) very helpful.

The patched results can be found at [EFI/CLOVER/ACPI/patched](EFI/CLOVER/ACPI/patched).

### Brightness adjustment keys

For me, my laptop use F11/F12 for brightness down/up, so I have to map these keys to brightness adjust methods. By following [Patching DSDT/SSDT for LAPTOP backlight control](https://www.tonymacx86.com/threads/guide-patching-dsdt-ssdt-for-laptop-backlight-control.152659/), I found my laptop use EC query methods (ACPI) for those keys. (This is the case with most modern laptops.) The `Consloe.log` shows they are `_Q11` and `_Q12`. So the patch I used for mapping these keys is (see also in `patches.txt`):

```txt
into method label _Q11 replace_content
begin
// Brightness Down\n
    Notify(\_SB.PCI0.LPCB.PS2K, 0x0405)\n
end;
into method label _Q12 replace_content
begin
// Brightness Up\n
    Notify(\_SB.PCI0.LPCB.PS2K, 0x0406)\n
end;
```

The ACPI debug version of my EFI configuration can be found at [EFI_with_ACPI_DEBUG](https://github.com/upupming/Lenovo-G50-80-Clover/blob/master/EFI_with_ACPI_DEBUG). I basically added the `ACPIDebug.kext` and then patched the DSDT with `Add DSDT Debug Methods` and `Instrument EC Queries` from https://github.com/RehabMan/OS-X-ACPI-Debug (add the source to `MaciASL`).

## Kexts

Note: All kexts used can be found at [EFI/CLOVER/kexts/Other](EFI/CLOVER/kexts/Other), and I prefer to only install kexts to EFI folder instead of /S/L/E or /L/E for better update experience.

- Keyboard and trackpad: `VoodooPS2Controller.kext` (I used the debug version for testing keys are PS2 or ACPI). I personally would like to enable single tap on trackpad in `System Preferences -> Trackpad -> Tap to click`, also enable one-finger tap & drag in `System Preferences -> Accessibility -> Mouse & Trackpad -> Trackpad options -> Enable dragging without drag lock`.
- Audio: `AppleALC.kext` + `layout-id=3`, HDMI audio fixed with `WhateverGreen` framebuffer patching, see [here](https://www.tonymacx86.com/threads/guide-intel-igpu-hdmi-dp-audio-all-sandy-bridge-kaby-lake-and-likely-later.189495/).
- Ethernet: `RealtekRTL8111.kext`
- Graphics: `Lilu.kext` + `WhateverGreen.kext`, note you will need use [config_HD5300_5500_6000.plist](https://github.com/RehabMan/OS-X-Clover-Laptop-Config/blob/master/config_HD5300_5500_6000.plist) by RehabMan to set `stolenmem` to 19 MB and `cursormem` to 9 MB, see [this](https://www.tonymacx86.com/threads/guide-alternative-to-the-minstolensize-patch-with-32mb-dvmt-prealloc.221506/) and [this](https://www.tonymacx86.com/threads/guide-intel-framebuffer-patching-using-whatevergreen.256490/). Also, you should enable 'Legacy support' in BIOS's boot tab to avoid glitches, see [this post](https://www.tonymacx86.com/threads/guide-intel-hd-graphics-5500-on-os-x-yosemite-10-10-3.162062/).
- Battery status: `ACPIBatteryManager.kext` + the `[bat] Lenovo G50-70` DSDT patch

## Bluetooth

Go to Windows device manager and find your bluetooth's vid and pid information:

```txt
USB\VID_105B&PID_E065&REV_0112
USB\VID_105B&PID_E065
```

Convert the hex to decimal using google search or any tool you prefer:

```txt
hex E065 == decimal 57445
hex 105B == decimal 4187
```

Add the following to `S/L/E/IOBluetoothFamily.kext/Contents/PlugIns/BroadcomBluetoothHostControllerUSBTransport.kext/Contents/Info.plist` -> `IOKitPersonalities`:

```plist
<dict>
	<key>Lenovo G50 80 Bluetooth</key>
	<string>com.apple.iokit.BroadcomBluetoothHostControllerUSBTransport</string>
	<key>IOClass</key>
	<string>BroadcomBluetoothHostControllerUSBTransport</string>
	<key>IOProviderClass</key>
	<string>IOUSBHostDevice</string>
	<key>idProduct</key>
	<integer>57445</integer>
	<key>idVendor</key>
	<integer>4187</integer>
</dict>
```

A demo patched `Info.plist` can be found at [Bluetooth/Info.plist](Bluetooth/Info.plist).

Note that in order to let the native Bluetooth work, we have to boot into Windows or Linux first, and then restart into macOS. See [here](https://github.com/daliansky/XiaoMi-Pro/issues/50). This is known as **RAMUSB device firmware update problem**. If you find any solution or want to discuss/fix this, please open an issue.

<details><summary>Test of transferring files to my Pixel phone using bluetooth</summary>

![20190307162150.png](https://i.loli.net/2019/03/07/5c80d4a1b8c69.png)

</details>

## Further working

At present, everything is working except:

1. ~~Brightness adjustment~~ -- **Done**
2. ~~Wake from sleep~~ -- **Unsolvable** Disable sleep as a workaround, see [my post on tonymacx86](https://www.tonymacx86.com/threads/unsolvable-black-screen-when-waking-from-sleep-on-mojave-10-14-3-lenovo-g50-80.271315/). Discussions are welcome there!

## Links

1. [tonymacx86 FAQ](https://www.tonymacx86.com/threads/faq-read-first-laptop-frequent-questions.164990/)
2. [The same model guide on tonymacx86](https://www.tonymacx86.com/threads/guide-lenovo-g50-80-el-capitan.171080/)
3. [Solve error when patching DSDTs](https://www.tonymacx86.com/threads/fixing-a-couple-of-errors-in-dsdt.259284/)
4. [Guide for i3 version of Lenovo G50-80 in tonymacx86](https://www.tonymacx86.com/threads/guide-lenovo-g50-80-80l0-and-high-sierra-10-13-4-updated-to-10-13-5.254285/)
5. [Video guide for i3 version](https://youtu.be/Th_G7BMNiSI)
6. [Get bluetooth vid & pid from windows](http://bbs.memacx.com/thread-5209-1-1.html)
7. [Inject bluetooth kext](http://www.yekki.me/how-to-make-a-bt-injector/)
