# Migrating Arch (legally)<br><br><br>


# Notes
### This guide is about migrating your Arch system from one disk to another without breaking it and thus the 'legally' part.<br>(Absolutely nothing the ICE should worry about).<br><br>

### In this guide we 'copy' our system into the new disk rather than 'clone' it, cloning is easier so you might want to look that up but copying allows you to **change the disk layout** of your system<br><br>

### The concepts are the same for every other distro but the tools would be different. **Refer to your distro's wiki.**<br><br>

### This guide covers GRUB + x64 UEFI / GPT configuration. Steps for other configurations may differ slightly, specifically in sections: [#Setting up the ESP](#setting-up-the-esp) and  [#Reinstalling the bootloader](#reinstalling-the-bootloader).
To check if you booted from legacy BIOS or UEFI and find out the bitness of your UEFI run:
```
# cat /sys/firmware/efi/fw_platform_size
```
 in the terminal it will display:
 ```
 64 
 ```
 for x64 UEFI<br><br>

 ```
 32
 ```
 for x32 UEFI<br><br>

 ```
 cat: /sys/firmware/efi/fw_platform_size: No such file or directory
 ```
 for legacy BIOS<br><br><br>


# 1 Start Arch in the live boot environment
Pretty straightforward. If you didn't enable booting from USB in BIOS and set it as first priority you should do that first in your BIOS settings.
When you boot you should see multiple options, select 'Arch Linux install medium (x86_64, UEFI)' and press Enter. Wait for everything to load. Finally you should see a command prompt. Congrats, you are in the live boot!<br><br><br>


# 2 Partitioning your disk
Note: Make sure the disk is empty or backed up. Partitioning the disk will erase all data on it<br><br>

The first step is to figure out which disk is the old one and which is the new one. You can run:
```
# lsblk
```
To display all block devices (e.g. disks, partitions), you should see something similar to:
``` bash
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda            8:0  0    500G 0 disk
├─sda1         8:1  0   512M  0  part
└─sda2         8:2  0  499.5G 0 part
sdb            8:16 1    16G  0  disk
├─sdb1         8:17 1    512M 0  part
└─sdb2        8:18  1   15.5G 0  part
nvme0n1       259:0 0   1.8T  0  disk
```
Note that this is just an **example**, don't panic if your output is different.
Figure out which is the new disk (destination). Usually it is the disk with no partitions, like nvme0n1 here <br>
**From now on, we will refer to 'new disk' as nvme0n1.**<br>
But how do we partition the disk? with a tool called fdisk.<br><br><br>


<a id="setting-up-the-esp"></a>
# 2.1 Setting up the ESP (not the one used in IoT)
ESP -which stands for "EFI System Partition"- is the partition where we will install the bootloader (more on that in section: [#Reinstalling the bootloader](#reinstalling-the-bootloader)) and other files required for booting, It is recommended to make the partition **1 GiB** in size to ensure it has adequate space for multiple kernels or unified kernel images, a boot loader, firmware updates files and any other operating system or OEM files. If still in doubt, 4 GiB ought to be enough for anybody.<br>
Note: some firmware expects the ESP to be first partition so do that to be safe.<br>
To create the partition run and give it the type 'EFI System':
```
# fdisk /dev/nvme0n1
```
This opens the fdisk interactive CLI, from there:
1. Type `n` to create a new partition  
2. Press Enter to accept the default partition number (which should be 1 if you haven't partitioned your disk before)
3. Press Enter to accept the default start sector  
4. Type `+4G` for the partition size (or `+1G` if you want a smaller ESP)
5. Type `t` to set the partition type
6. If it asks you for the partition number select the number you chose in step 2
7. Type `uefi`
8. Type `w` to write your changes to the disk

Run:
```
# lsblk
```
You should see that a new partition called nvme0n1p1 was added to nvme0n1<br>
BOOM, you have created your own ESP. Now let's format it!<br>
It is recommneded to use FAT32 because it is universal, to format your partition simply run:
```
# mkfs.fat -F 32 /dev/nvme0n1p1
```
You can also give it a label which is optional but future you will thank you when he is trying to figure out which partition is which. Run:
```
fatlabel /dev/nvme0n1p1 EFI
```
And that is it. The ESP is now set up.<br><br><br>


# 2.2 Setting up a swap parition
If you want swap, which is recommended especially on low RAM systems, run:
```
# fdisk /dev/nvme0n1
```
1. Type `n`
2. Press Enter to accept the default partition number (which should be 2)
3. Press Enter to accpet the default start sector
4. Type `+8G` to set partition size to 8GiB. If you have a potato computer with less than 8GiB of RAM you can set the size equal to your RAM
5. Type `t` to set the partition type
6. If it asks you for the partition number select the number you chose in step 2
7. Type `swap`
8. Type `w`
Now you should see the new partition (nvme0n1p2) when you run:
```
# lsblk
```
Run:
```
# mkswap /dev/nvme0n1p2
```
And to label the partition run:
```
# swaplabel /dev/nvme0n1p2 -L SWAP
```
<br><br><br>

# 2.3 Setting up data partitions
Note: Here I will setup only one data partition which, will be used as the **root (/) partition**. you can setup as many as you like (e.g. a separate partition for home, var etc...) with the same method provided here<br><br>

Run:
```
# fdisk /dev/nvme0n1
```
1. Type `n`
2. Press Enter to accept the default partition number (which should be 3 now)
3. Press Enter to accept the default start sector
4. Press Enter to accept the default end sector (or manually choose a size for the partition if you will add more data partitions)
5. Type `w`

There is no need to select a parition type becuase the default is Linux filesystem<br>
Now you should see the new partition (nvme0n1p3) when you run:
```
# lsblk
```

You can format the root partition (or any data partition) with any filesystem you want. This guide will cover ext4.<br>
Run:
```
# mkfs.ext4 /dev/nvme0n1p3
```
And finally to label the partition "ROOT" run:
```
# e2label /dev/nvme0n1p3 ROOT
```
<br><br>

# 3 Copying the system
This step takes from a few minutes to hours, depending on your system and the content you are copying. The good news though: you can leave it unattended.<br>
To start this step you must first identify the source disk that has your old system the same way you identified the destination disk. Run:
```
# lsblk
```
Figure out which disk and which data partition on that disk has your old system (ignore any boot or swap partitions). **From now on we will refer to the source disk as 'sda' and the source partition as sda2.**<br><br>

**Quick recap:** we now have the source disk **sda** that has the source partition **sda2**. We want to copy the contents to the destination disk **nvme0n1**, particularly the partition **nvme0n1p3**.<br><br>

Step one: Mount the destination and source partitions. Run:
```
# mount /dev/nvme0n1p3 /mnt
```
```
# mkdir /source
```
```
# mount /dev/sda2 /source
```
<br><br>
Step two: Copy the system. We exclude virtual filesystems (/dev, /proc, /sys), temporary folders (/tmp, /run), mounted disks (/mnt, /media), cache directories (/home/*/.cache), and system folders (/lost+found) to avoid copying unnecessary or ephemeral data. Run:
```
# rsync -aAXHv --exclude='/dev/*' --exclude='/proc/*' --exclude='/sys/*' --exclude='/tmp/*' --exclude='/run/*' --exclude='/mnt/*' --exclude='/media/*' --exclude='/home/*/.cache/' --exclude='/lost+found/' /source/ /mnt/
```
Step three: Go take an hour's rest while rsync finishes copying your system<br><br><br>


# 4 Updating fstab
When booting, how does your system know which partition gets mounted where? From the 'fstab' file.<br>
Now, when you copied the system to nvme0n1, you copied the old fstab file. We need to update it.<br><br>

First you need to mount your data partition to the root folder, you also need to mount swap, ESP and if you want to mount other partitions you should do so now (but again, this guide won't cover that)<br>
Run:
```
# mount /dev/nvme0n1p3 /mnt
```
```
# swapon /dev/nvme0n1p2
```
It is recommended to mount the ESP at /mnt/boot/efi

```
# mkdir -p /mnt/boot/efi
```

```
# mount /dev/nvme0n1p1 /mnt/boot/efi
```

Now, insert an arbitrary comment such as `#end of old fstab` at the end of your /mnt/etc/fstab. To do that run:
```
# echo '#end of old fstab' >> /mnt/etc/fstab
```
And to generate the new fstab run:
```
# genfstab -U /mnt >> /mnt/etc/fstab
```
You can then run:
```
# vim /mnt/etc/fstab
```
Do not leave duplicate /, swap, or ESP entries. The system may fail to boot if duplicates exist.<br>
Remember to save your changes to the fstab file.<br><br><br>

<a id="reinstalling-the-bootloader"></a>
# 5 reinstalling the bootloader
Now we need to install grub in nvme0n1. To do so follow these steps:<br>
Run:
```
# mount /dev/nvme0n1p3 /mnt
```
If the partition was already mounted then there is no need to run the previous command.<br>
Then run:
```
# arch-chroot /mnt
```
<br>
For this part, you need internet connection. See [Installation guide#Connect to the internet](https://wiki.archlinux.org/title/Installation_guide#Connect_to_the_internet)<br><br>

After connecting to the internet, install the required packages (grub, efibootmgr, linux, linux-firmware):
```
# pacman-key --init
```
```
# pacman-key --populate archlinux
```
```
# pacman -Sy grub efibootmgr linux linux-firmware
```
Finally to install grub, run:
```
# grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
```
One thing remains, generating the main configuration file. Run:
```
# grub-mkconfig -o /boot/grub/grub.cfg
```
<br><br>

# 6 Regenerate kernel image
Simply run:
```
# mkinitcpio -P
```
<br><br>

# Congrats, you have succefully copied your system.
All you need to do now is to reboot and set the new disk as first priority in your BIOS settings.<br><br><br>


# References:
[Migrate installation to new hardware](https://wiki.archlinux.org/title/Migrate_installation_to_new_hardware)
