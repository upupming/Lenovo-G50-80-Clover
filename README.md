# Hackintosh for Lenovo G50-80 (Broadwell 5200U version)

Please read this file carefully. Simply copying my configuration file is very likely to not work, although you can still play around. You should apply the same steps as mine to get your Hackintosh working on your laptop.

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

> BIOS update and specs can be found at [here](https://pcsupport.lenovo.com/us/en/products/laptops-and-netbooks/lenovo-g-series-laptops/g50-80/80e5/80e5007ccd/pf06sm0m/downloads?linkTrack=Caps%3ABody_SearchProduct&searchType=6&keyWordSearch=PF06SM0M), or simply use this link https://download.lenovo.com/consumer/mobiles/b0cna0ww.exe to download the BIOS upadte.

## DSDT patches

See [patches.txt](./DSDT-patching/patches.txt). I only patched `DSDT.aml` and ignored all `SSDT-*.aml`.

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

The ACPI debug version of my EFI configuration can be found at [EFI_with_ACPI_DEBUG](./EFI_with_ACPI_DEBUG). I basically added the `ACPIDebug.kext` and then patched the DSDT with `Add DSDT Debug Methods` and `Instrument EC Queries` from https://github.com/RehabMan/OS-X-ACPI-Debug.

## Kexts

All kexts used can be found at [EFI/CLOVER/kexts/Other](EFI/CLOVER/kexts/Other).

- Keyboard and trackpad: `VoodooPS2Controller.kext` (debug version for testing keys are PS2 or ACPI), and I have to enable single tap on trackpad in `System Preferences -> Trackpad -> Tap to click`.
- Audio: `VoodooPS2Controller.kext`
- Ethernet: `RealtekRTL8111.kext`
- Graphics: `Lilu.kext` + `WhateverGreen.kext`, note you will need use [config_HD5300_5500_6000.plist](https://github.com/RehabMan/OS-X-Clover-Laptop-Config/blob/master/config_HD5300_5500_6000.plist) by RehabMan to set `stolenmem` to 19 MB and `cursormem` to 9 MB, see [this](https://www.tonymacx86.com/threads/guide-alternative-to-the-minstolensize-patch-with-32mb-dvmt-prealloc.221506/) and [this](https://www.tonymacx86.com/threads/guide-intel-framebuffer-patching-using-whatevergreen.256490/). Also, you should enable 'Legacy support' in BIOS's boot tab to avoid glitches, see [this post](https://www.tonymacx86.com/threads/guide-intel-hd-graphics-5500-on-os-x-yosemite-10-10-3.162062/).

## Further working

At present, everything is working except:

1. ~~Brightness adjustment~~ -- **Done**
2. Wake from sleep

## Links

1. [tonymacx86 FAQ](https://www.tonymacx86.com/threads/faq-read-first-laptop-frequent-questions.164990/)
2. [The same model guide on tonymacx86](https://www.tonymacx86.com/threads/guide-lenovo-g50-80-el-capitan.171080/)
3. [Solve error when patching DSDTs](https://www.tonymacx86.com/threads/fixing-a-couple-of-errors-in-dsdt.259284/)
4. [Guide for i3 version of Lenovo G50-80 in tonymacx86](https://www.tonymacx86.com/threads/guide-lenovo-g50-80-80l0-and-high-sierra-10-13-4-updated-to-10-13-5.254285/)
5. [Video guide for i3 version](https://youtu.be/Th_G7BMNiSI)