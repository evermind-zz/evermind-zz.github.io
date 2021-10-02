---
layout: post
title:  "Windows 7 won't boot"
author: evermind
categories: [ windows, bootmanager, bootrec, bcdboot ]
---
As mentioned in [Cloning 2 partitions from dying HDD to brand new SSD](../cloning-2-partitions-from-dying-HDD-to-brand-new-SSD/)
I tried to copy NTFS over and then used several tools like `bootrec`
`bcdboot` etc. using Windows 7 DVD and also tried various Linux tools
but with no success. Still getting a message like:
```bash
A disk read error occurred
Press Ctrl + Alt + Del to restart
```

So what did I do to fix that?

I deleted the first partition that contained the Windows installation and than
installed Windows 7 again. Yes you heard right I installed it again - but that
is not the end of the story. The Windows installer created actually 2 partitions
in the free space where I had just one. One small about 100M that actual contained
bootloader files etc. and the second partition with the windows OS itself. To my
pleasant surprise after restarting the newly installed Windows 7 it booted.

As I still did not want to have a new installation and want to restore the one from
the raw image file I overwrote the freshly installed Windows 7 (of course I did a
backup before - in case I need it for real) with the image file using Linux and
`ntfsclone`.

After `ntfsclone` was finished I rebooted and dang it stopped again with another message.
Not sure if it was exactly like the message below but similar.
```bash
Windows failed to start. A recent hardware or software change might be the cause. To fix the problem:

1. Insert your Windows installation disc and restart your computer.
2. Choose your language settings, and then click "Next."
3. Click "Repair your computer."

If you do not have this disc, contact your system administrator or computer manufacturer for assistance.

Status: 0xc000000f

Info: An error occurred while attempting to read the boot configuration data.
```

So trying to solve that 
## change the serial to NTFS volume of the freshly installed NTFS volume.
I tried to change the serial of the NTFS volume to the one that I wrote down
before overwriting it with the NTFS volume from the raw file. (Of course using Linux)
```bash
ntfslabel --new-serial=<SERIAL_FROM_WINDOWS_NTFS_VOLUME_FRESHLY_INSTALLED_BEFORE> /dev/sda2
```

Still no success!! Same error message as before. So I don't think this step
is necessary to succeed.

### Solution to that problem
I again search the internet and found a solution that finally worked.
Source: [boot bcd status 0xc000000f error](https://superuser.com/questions/628038/boot-bcd-status-0xc000000f-error)

Boot your Windows installation DVD and a right before Installation press
`shift+F10` get a console.
Type `c:` to get to the boot partition drive's root folder. (c: should be the drive letter of boot partition)
I am not sure if that is really necessary but nevertheless I did it this way.

Than type do the following
```bash
type bootrec /fixmbr
type bootrec /fixboot
```
Both command should give something like this messages:
`The operation completed successfully` or `element not found`
Hopefully the will complete successfully!!

Restart the computer back into recovery mode/repair mode and load CMD prompt. I also
don't know if this step is necessary but again do it or not.

Switch to your boot partition directory if you are not already there (usually c:)
type bcdboot c:\Windows where 'c:\Windows' is the exact path to your windows folder.
Keep in mind that in recovery mode, the path of the boot partition and path to
windows may be different. In my case it was **e:**\Windows.

This finally worked out and I could boot the system -- NO! I can't but I got a different
error message it was a Bluescreen!!!
 * [#3] See next related post: [Bluescreen aka BSOD STOP 0x0000007B INACCESSABLE_BOOT_DEVICE](../bluescreen-aka-bsod-stop-0x0000007b-inaccessable-boot-device)
 * [#2] Previous related post: [Cloning 2 partitions from dying HDD to brand new SSD](../cloning-2-partitions-from-dying-HDD-to-brand-new-SSD/)

