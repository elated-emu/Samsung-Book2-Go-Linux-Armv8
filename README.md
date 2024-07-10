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

Here is another resource for compiling DTB Files (https://github.com/mykhani/device-tree-guide)

I followed this guide to compile the DTS to with other DTSI modules to DTB (https://stackoverflow.com/questions/50658326/device-tree-compiler-not-recognizes-c-syntax-for-include-files)

The only thing I did was take the Samsung-Book.dts file from hexdump0815 and modified it to include the sc7280.dtsi instead of the sc7180.dtsi file, since I figure these two laptops are similar enough.

```
ERROR (duplicate_label): /reserved-memory/memory@86000000: Duplicate label 'mpss_mem' on /reserved-memory/memory@86000000 and /reserved-memory/mpss@8b800000
```
Okay, maybe the samsung DTS file conflicts. I will just remove that section from the samsung DTS.

That seemed to go through with a myriad of warnings
```
arch/arm64/boot/dts/qcom/sc7280.dtsi:4488.26-4559.6: Warning (avoid_unnecessary_addr_size): /soc@0/display-subsystem@ae00000/dsi@ae94000: unnecessary #address-cells/#size-cells without "ranges", "dma-ranges" or child "reg" property
arch/arm64/boot/dts/qcom/sc7280.dtsi:1098.21-1119.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/i2c@980000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/spi@980000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1098.21-1119.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/i2c@980000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/serial@980000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1121.21-1140.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/spi@980000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/serial@980000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1158.21-1179.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/i2c@984000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/spi@984000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1158.21-1179.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/i2c@984000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/serial@984000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1181.21-1200.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/spi@984000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/serial@984000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1218.21-1239.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/i2c@988000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/spi@988000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1218.21-1239.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/i2c@988000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/serial@988000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1241.21-1260.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/spi@988000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/serial@988000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1278.21-1299.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/i2c@98c000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/spi@98c000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1278.21-1299.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/i2c@98c000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/serial@98c000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1301.21-1320.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/spi@98c000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/serial@98c000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1338.21-1359.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/i2c@990000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/spi@990000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1338.21-1359.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/i2c@990000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/serial@990000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1361.21-1380.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/spi@990000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/serial@990000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1398.21-1419.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/i2c@994000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/spi@994000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1398.21-1419.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/i2c@994000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/serial@994000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1421.21-1440.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/spi@994000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/serial@994000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1458.21-1479.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/i2c@998000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/spi@998000)
  also defined at arch/arm64/boot/dts/qcom/sc7280-samsung-galaxy-book2-go.dts:148.7-161.3
arch/arm64/boot/dts/qcom/sc7280.dtsi:1458.21-1479.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/i2c@998000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/serial@998000)
  also defined at arch/arm64/boot/dts/qcom/sc7280-samsung-galaxy-book2-go.dts:148.7-161.3
arch/arm64/boot/dts/qcom/sc7280.dtsi:1481.21-1500.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/spi@998000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/serial@998000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1518.21-1539.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/i2c@99c000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/spi@99c000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1518.21-1539.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/i2c@99c000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/serial@99c000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1541.21-1560.6: Warning (unique_unit_address): /soc@0/geniqup@9c0000/spi@99c000: duplicate unit-address (also used in node /soc@0/geniqup@9c0000/serial@99c000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1613.21-1634.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/i2c@a80000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/spi@a80000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1613.21-1634.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/i2c@a80000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/serial@a80000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1636.21-1655.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/spi@a80000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/serial@a80000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1673.21-1694.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/i2c@a84000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/spi@a84000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1673.21-1694.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/i2c@a84000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/serial@a84000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1696.21-1715.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/spi@a84000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/serial@a84000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1733.22-1754.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/i2c@a88000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/spi@a88000)
  also defined at arch/arm64/boot/dts/qcom/sc7280-samsung-galaxy-book2-go.dts:163.8-180.3
arch/arm64/boot/dts/qcom/sc7280.dtsi:1733.22-1754.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/i2c@a88000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/serial@a88000)
  also defined at arch/arm64/boot/dts/qcom/sc7280-samsung-galaxy-book2-go.dts:163.8-180.3
arch/arm64/boot/dts/qcom/sc7280.dtsi:1756.22-1775.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/spi@a88000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/serial@a88000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1793.22-1814.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/i2c@a8c000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/spi@a8c000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1793.22-1814.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/i2c@a8c000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/serial@a8c000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1816.22-1835.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/spi@a8c000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/serial@a8c000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1853.22-1874.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/i2c@a90000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/spi@a90000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1853.22-1874.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/i2c@a90000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/serial@a90000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1876.22-1895.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/spi@a90000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/serial@a90000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1913.22-1934.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/i2c@a94000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/spi@a94000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1913.22-1934.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/i2c@a94000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/serial@a94000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1936.22-1955.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/spi@a94000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/serial@a94000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1973.22-1994.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/i2c@a98000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/spi@a98000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1973.22-1994.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/i2c@a98000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/serial@a98000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:1996.22-2015.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/spi@a98000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/serial@a98000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:2033.22-2054.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/i2c@a9c000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/spi@a9c000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:2033.22-2054.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/i2c@a9c000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/serial@a9c000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:2056.22-2075.6: Warning (unique_unit_address): /soc@0/geniqup@ac0000/spi@a9c000: duplicate unit-address (also used in node /soc@0/geniqup@ac0000/serial@a9c000)
arch/arm64/boot/dts/qcom/sc7280.dtsi:3942.10-3952.6: Warning (graph_child_address): /soc@0/eud@88e0000/ports: graph node has single child node 'port@0', #address-cells/#size-cells are not necessary
```

---

## First Boot
I think the kernel actually loaded. I am very proud to have gotten this far so quickly, I have never touched anything related to kernel level stuff and haven't even heard of a device tree until starting this project.

Unfortunately, it wasn't all working but luckily the computer is speaking clearly. Here are the error logs.
```
[0.000000] OF: reserved mem: OVERLAP DETECTED!
[0.000000] mpss@8b800000 (0x000000008b800000--0x000000009ae00000) overlaps with memory@8f600000 (0x000000008f600000--0x000000008fb00000)
[1.816666] qcom-rpmh-regulator 18200000.rsc:pm6150-rpmh-regulators: ldo10: could not find RPMh address for resource ldoa10
```
It looks like the first error is saying we've allocated the same memory to multiple things. (Lines 1 & 2). I don't think that is what caused the computer to hang though. It seemed more of a warning, I will focus on the error on line 3. It looks like a resource is not being located at all.

### What is RPMh?
- [Article](https://lwn.net/Articles/759856/)
It seems like the changelog mentions changing the naming convention (- Renamed qcom_rpmh-regulator.c to be qcom-rpmh-regulator.c)
It could be possible that the new name is causing this error? Especially since the sc7180-samsung-galaxy-book-go.dts file is pretty old, it could predate the change.
I still don't know what RPMh is, my best understanding: its a specific part of the cpu.

### What is pm6150?

### What is ldo10?

### What is ldoa10?

Those must be specific modules within the cpu?

## Trying Precompiled DTB files
I am using the pre-compiled DTB files from the arch project, maybe I will have more success.

| DTB | ISO | Result |
| --- | -------------- | --------------- |
| sc7280-crd-r3.dtb | snapdragon_7c_woa-aarch64-bookworm | Lost connection to USB |
| sc7280-herobrine-crd.dtb | snapdragon_7c_woa-aarch64-bookworm | same as above |
| sc7280-herobrine-evoker.dtb | snapdragon_7c_woa-aarch64-bookworm | same as above |
| sc7280-herobrine-herobrine.dtb | snapdragon_7c_woa-aarch64-bookworm | same as above |

I am beginning to think that I need to work on the DTS file and manually add the USB ports.

I think I can get these references from the DSDT.aml or DSDT.dsl files. No clue how to do it though.
Getting late so i'll get to it tomorrow


