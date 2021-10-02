---
layout: post
title:  "Cloning 2 partitions from dying HDD to brand new SSD"
author: evermind
categories: [ windows, ddrescue, linux, ntfsclone, ntfsresize, OSFMount ]
---
I had to move two windows partitions from an aging 1TB HDD that was already
dying to a new 1TB SSD.

## Using Samsung Data Migration Software
My first attempt was in using the Samsung Data Migration Software on Windows as
I thought that they know their business of cloning partitions to their SSD's in
a fashion that honors correct alignment on the SDD.

But I was wrong. It only allowed me to just clone the first partition on to the
new SSD. There was no option like the the + button they advertised. Nevertheless
I succeeded to clone the first partition only to later discover with the help of
`cfdisk` that it was not aligned correctly.

## Creating a Manual Backup of Both Partitions
As I do not like Windows the natural step for me is using a GNU/Linux System.
I start using Rescuezilla from an USB-Stick. And started to clone the partitions
using `ddrescue` to a raw image on my NFS mounted external network storage.

```bash
ddrescue /dev/sda1 sda1.img log.sda1
```
After the first run I saw that are some parts that are not readable so I tried
the direct option. And after running it more than once all places with read errors
could be recovered.
```bash
ddrescue -d /dev/sda1 sda1.img log.sda1
```

## Resizing the partition's NTFS volume
As I wanted the partitions to be aligned correctly I had to shrink the first
partition. So here I run into some problems. I tried using `ntfsresize` but it
refused to work as it screamed that the partition had some data inconsistency
and I should run `chkdsk` on windows to fix it.

### Run chkdsk on the raw image file
I knew that the already dying HDD will likely take ages to run such a check and
may brake completely. So I had the idea using a kind of Linux loopback device on
Windows and too use the `ddrescue` created raw image file. But is there a tool
that let me do this? A quick search on the internet revealed `OSFMount` to be
the tool I needed and to my surprise it worked and I could mount the image and
run chkdsk successfully from the command line. `OSFMount` mounted the image to
the letter `E:` so I used below statement:
```bash
chkdsk /f e:
```

[OSFMount](https://www.osforensics.com/tools/mount-disk-images.html)

## Resize (exactly shrink) NTFS volume on Linux for real
I simple used `kpartx to 'losetup' the raw image file. 
```bash
kpartx -av /media/remote/sda1.img
```

Now the image file is accessible via the block device `/dev/loop0`
So obtain the information about what your options are for shrinking the
partition:
```bash
ntfsresize -i /dev/loop0
```

Here you get the information what is is smallest shrink size possible.
I needed only to shrink the NTFS volume for about 100MB.
```bash
ntfsresize -ns 179200M /dev/loop0 # run with no action
ntfsresize -s 179200M /dev/loop0 # run for real to resize the volume
```

## Clone Shrinked NTFS Volume to Real Disk Partition
Having a suitable first partition on the SSD I used the following command:
```bash
ntfsclone --overwrite /dev/sda1 /dev/loop0
```
`/dev/sda1` is the target partition and '`/dev/loop0` is the source (in
my case the raw image file.

## Conclusion
Long story short. It took a lot time to copy the data to the new SSD. But
it was successful and that's all what matters. Really? Not really I still
had trouble getting Windows 7 running. See posts:
 * [#2] [Windows 7 won't boot](../windows-7-wont-boot)
 * [#3] [Bluescreen aka BSOD STOP 0x0000007B INACCESSABLE_BOOT_DEVICE](../bluescreen-aka-bsod-stop-0x0000007b-inaccessable-boot-device)
