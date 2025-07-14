---
title: "Authoring Bootable Meeting"
categories:
  - Blog
tags:
  - Grub2
  - El Torito
  - RockRidge
  - go-diskfs
  - EFI
  - UEFI

---

# Authoring  bootable media

I have found better results using the term "authoring" when searching for how
to create the files that are written, usually via `dd`, to a drive in order to
boot a computer. This post is a reminder for me later of the sites I found,
tools, and commands to author bootable media. In this post, serves as my notes
for this topic. 

## All the commands, none of the context

```bash 

image_file_name=my_disk.img 

size_of_disk=512 
dd if=/dev/zero of=$image_file_name bs=1M count=$size_of_disk status=progress
losetup --show --find $image_file_name
 ```



## Partition Table

There are two main partition table types, `mbr` and `gpt`. `gpt` is the newer
type which is used with UEFI systems.


## Making a disk image

The process overall is to create a file that has the size you want your disk to
be on your local file system. Use the various partitioning and grub to format
and install grub to the file.


### Create a file and loop mount it 

Using [fallocate](https://www.man7.org/linux/man-pages/man1/fallocate.1.html)
to make the file will be quick, but since `fallocate` doesn't write zeros to
the destination, you could have some garbage in your file system. For that
reason it is better to use `dd` to make the file that will be used as the disk
image.

#### Using dd

Use the command: `dd if=/dev/zero of=$image_file_name bs=1M count=$size_of_disk
status=progress`. The `bs=1M` is how many mebibytes, aka powers of two, to make
the disk image.  Beware that most storage media is sold in unites of Megabytes,
aka base 10. If you use `bs=1MB` that will be megabytes in base 10.


#### Mounting the image

A lot of the tools that are used to author bootable media expect that the
target of the operations are a disk like device. For the file to appear as a
disk use the `losetup` command to mount the disk image to the system using a
loopback mount.

`sudo losetup --find --show $image_file_name`

This command will find an unused loop device, attach the file to it, and return
the name. Use this device name as the argument for the remaining operations.


## Populating the disk

We have a file, mounted to the system via loop back, but it is empty. To
actually populate it we need to make a partition table, and a couple of
partitions. There are a lot of tools out there to use: `parted`, `gparted`,
`sgdisk`, `gdisk`,`fdisk`.

This guide will use gdisk.


### Making partitions


### Marking them as bootable

### UUID Codes

### Sizing Paritions 


