# Welcome to my GitHub Page

The purpose of this page is to share my notes and howtos that may be useful to other sysadmins.


## Creating a MS-DOS boot disk USB for BIOS updates

I recently needed to create a DOS boot disk to update my HP ProLiant Microserver N54L bios, so I did what everyone does first, search on Google, but surprisingly, I was unable to find a guide that met all the requirements, which were the following:

- The boot disk has to be **real MS-DOS**. For some reason, the provided HP's bios update tool was not compatible with FreeDOS, it crashed at every flash attempt.
- The boot disk needs to be created under Ubuntu. I don't have any Windows workstation at home, so using Rufus as suggested by most guides, was not an option.
- It has to run from a USB stick, as I don't have floppy or CD-ROM units in my HP ProLiant Microserver.

After some trial and error, I was able to create a procedure that met the requirements and worked pretty good, so here it is:

### Download a real MS-DOS boot disk image

You can get the disk image from many places, but I recommend AllBootDisks: 
[https://www.allbootdisks.com/download/dos.html](https://www.allbootdisks.com/download/dos.html). 

There's many versions to choose, I guess any will do, but I chose the **Dos6.22.img** one.

### Install QEMU

QEMU is a really useful open source machine emulator, if you are using Ubuntu, just install it via apt:
```markdown
sudo apt-get install qemu
```

Check that the DOS image is working by booting it using QEMU:

```markdown
qemu-system-i386 -boot a -fda Dos6.22.img
```

It should open an emulator screen and boot into the classic MS-DOS prompt. You can close the window once you get the prompt, this step is just to check that QEMU and the DOS image work.

### Prepare the USB stick

The USB stick format doesn't really matter, as we will just write straight away to the block device. Just in case, I suggest using gparted to remove any partition on the USB stick, leave it partition-less and unformatted.


### Install DOS boot files into the USB stick

Now that we have all the necessary tools, it's time to put all of it together.

**IMPORTANT!** Be extra careful in the following step, we will write directly to a block device with root privileges, which means that you need to be sure you are pointing to the USB stick block device file, and not any other device like your hard drive which could cause data loss. Replace "/dev/sdX" with you USB stick device path, you can check which one is with 'lsblk' command.

Run QEMU with both DOS image and USB as emulated floppy devices:


```markdown
sudo qemu-system-i386 -boot a -fda Downloads/Dos6.22.img -fdb /dev/sdX
```

It should boot MS-DOS prompt. Now we have "A:" as the boot disk image, and "B:" as the USB stick.
At DOS prompt, use DOS format executable with "/s" option to format and copy DOS boot files on the USB stick:

```markdown
A:\>format /s b:
```

You can close the prompt window now. I suggest to check that the USB stick boots in QEMU. Again, replace '/dev/sdX' with your USB stick device path.

```markdown
sudo qemu-system-i386 -boot a -fda /dev/sdX
```

It will probably ask for a new date and time, just hit enter twice to skip it. This will happen at every boot, if it annoys you too much, you can prevent it creating an AUTOEXEC.BAT file with just the line "@ECHO OFF" on it.

You can close the prompt window.


### Copy your BIOS update tools

Now we have a bootable DOS USB stick, with the minimum files to be able to boot. You can remount the USB stick in Ubuntu using your file manager and copy the manufaturer's provided update BIOS tools on it (Usually a ROM or BIN file, and a FLASH.EXE or similar executable).

Your USB is ready now! Plug it in your workstation/server and remember to select to boot from USB during the boot process or in the BIOS settings ;)
