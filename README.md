# Samsung-Book2-Go-Linux-Armv8

This is a project that I wanted to document. It is my first GitHub repo! It is more so for sharing my personal references.

I am taking on this project since it is very niche, and seems like others have done something similar before (notably with the Samsung Galaxy Book Go 1 [GitHub Page](https://github.com/merckhung/linux_on_arm64_laptop)). I am jumping into something I don't know at all, so take everything here with a grain of salt.

## About the Samsung Galaxy Book2 Go 5G
This device is an Arm device, so I think it would be fun to play around with it until the prices of the [Snapdragon X Elite](https://www.qualcomm.com/products/mobile/snapdragon/pcs-and-tablets/snapdragon-x-elite) laptops fall in price.

There are multiple Samsung Galaxy Book models, all cleverly named the same or with slight variations. So there is a lot of confusion online, and little documentation.

My Device Specs: 
- Qualcomm Snapdragon 7c+ Gen 3
- 128GB eUFS
- 4GB Ram

## What is the goal
Let's get Linux to boot on the Samsung Galaxy Book2 Go 5G

- [X] 1. Turn off Secure Boot
- [X] 2. Boot off a USB Drive
- [x] 3. Boot into GRUB
- [ ] Load the Linux kernel? 

### 1. Turn off Secure Boot
The key to enter UEFI settings is F12.

### 2. Boot off a USB Drive
The Key to boot from a USB Drive is F10.
> [!NOTE]
 Many people had issues getting the drive to be recognized. I haven't had any issues with it once Secure Boot was disabled. They also mentioned that it had something to do with the flashdrive model. I am using my Rooted OnePlus 7 Pro with [DriveDroid](https://play.google.com/store/apps/details?id=com.softwarebakery.drivedroid&hl=en_US) as the FlashDrive. ISO files that were not compiled for arm seem to be completely ignored, and don't show up under boot devices.

### 3. Boot into GRUB

**DEBIAN** (12.6.0-arm64-netinst.iso)
I am pretty surprised this worked first try. The [ARM64 Debian ISO](https://cdimage.debian.org/cdimage/release/current/arm64/iso-dvd/) booted into grub. However, all of the preconfigured boot options I tried seemed to just restart the computer with no visible logs or output. (I tried graphical install, install, expert install, and rescue)

---

**ARCH** (archboot-2024.07.07.00.43-6.9.7-1-aarch64)
Similar results to debian, however we get a crumb of information outputted before rebooting.

```
EFI stub: Booting Linux Kernel...
EFI stub: Loaded initrd from LINUX_EFI_INITRD_MEDIA_GUID
EFI stub: Measured initrd data into PCR 9
EFI stub: Generating empty DTB
EFI stub: Exiting boot services...
```

I am not sure exactly what this means, but the DTB (Or device tree blob) entry catches my eye because [aarch64-laptops](https://github.com/aarch64-laptops/build) and [merckhung](https://github.com/merckhung) mentioned it.

#### What the heck is DTB
Yesterday I was up at 2AM asking myself this. Today I think I grasp it a little bit. Next year I will probably still be asking myself this.

DTB is a blob of information that translates the device hardware references into readable input for the kernel. It is very low level and designed to escape the limitations of compiling a kernel for each motherboard and their specific components.

If the DTB is not readable, I assume the empty DTB is generated as a template. Maybe it does a lot of guesswork to try to boot, however in our case it is not booting. We need to give it an accurate DTB.

Now we have the **DTS** file. I think this file is a decompiled version of the DTB. They can be converted back and forth from DTB to DTS (thank you whoever allowed that to happen). I think this makes our work a lot easier, but I am still going in blind.

#### Okay but how do we get the compiled DTB?
I don't know.

[hexdump0815](https://github.com/hexdump0815/linux-mainline-qcom-kernel/tree/main/misc.qc7/misc) has many DTS Files here, but they are for the wrong chipset.
The DTS is for the SC7180 (Qualcomm 7C Gen 2). We need the DTS for SC7280 [Qualcomm chipset names here](https://en.wikipedia.org/wiki/List_of_Qualcomm_Snapdragon_systems_on_chips)
Here are some more DTS Files for the SC7280 from the man himself [Torvalds](https://github.com/torvalds/linux/tree/master/arch/arm64/boot/dts/qcom) , but they seem to be named after ??minecraft mobs??
I am not sure about the repercussions for using random DTS files to boot with. Let's try it.

I am using the [sc7280.dtsi file](https://github.com/torvalds/linux/blob/master/arch/arm64/boot/dts/qcom/sc7280.dtsi)
Now I want to convert this to DTB so that we can select it in our grub menu. Here is some references to converting it over (DTSI to DTB)[https://developer.toradex.com/software/linux-resources/device-tree/first-steps-with-device-trees/]

**DTSI** is a DTS File that includes other DTS modules. We need to grab the other modules before converting them to DTB.

