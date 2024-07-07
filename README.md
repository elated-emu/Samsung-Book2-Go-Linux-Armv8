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

### Turn off Secure Boot
The key to enter UEFI settings is F12.

### Boot off a USB Drive
The Key to boot from a USB Drive is F10.
> [!NOTE]
> Many people had issues getting the drive to be recognized. I haven't had any issues with it once Secure Boot was disabled. They also mentioned that it had something to do with the flashdrive model. I am using my Rooted OnePlus 7 Pro with [DriveDroid](https://play.google.com/store/apps/details?id=com.softwarebakery.drivedroid&hl=en_US) as the FlashDrive.

### Boot into GRUB
**Booting the [ARM64 Debian ISO](https://cdimage.debian.org/cdimage/release/current/arm64/iso-dvd/)**

