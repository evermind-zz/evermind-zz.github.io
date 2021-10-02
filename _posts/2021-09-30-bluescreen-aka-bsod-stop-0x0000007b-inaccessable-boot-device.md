---
layout: post
title: "Bluescreen aka BSOD STOP 0x0000007B INACCESSABLE_BOOT_DEVICE"
author: evermind
categories: [ windows, bootmanager, bsod, bluescreen, ahci, ide, bios ]
---
So here we go again a bluescreen with saying something about disk changed etc.
I searched again as I had something in my head about AHCI and found a
solution. It was regarding how to switch to AHCI.

I knew on the system with the old HDD the BIOS was set to `IDE` as protocol
or whatever it is. But as I knew SSD like `AHCI` more and I also switched
the whole computer and where the setting was already set to `AHCI`. So
therefore I got that problem. So for testing I switched back to `IDE` and
to my surprise Windows 7 was booting for the first time from the new SSD
on a new Computer.

Wow!!! I totally enjoyed the speed of the SSD booting the system much faster
than on the dying HDD device.

So now I installed some AHCI drivers on booted Windows 7 system. And fired
up the regedit tool to switch Windows to `AHCI`

Locate and then click one of the following registry subkeys:
```bash
HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Msahci
HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\IastorV
```

And in the pane on the right side, right-click Start in the Name
column, and then click Modify.
In the Value data box, type 0, and then click OK.

I did this for both subkeys and than rebooted the system. Entered
the BIOS and set the Sata mode to `AHCI` and exit the BIOS and
started the computer again and finally Windows 7 worked the way
I wanted with AHCI enabled.

This was a long ride and also a long write-up that splitted into 3 blog posts:
 * [#1] [Cloning 2 partitions from dying HDD to brand new SSD](../cloning-2-partitions-from-dying-HDD-to-brand-new-SSD/)
 * [#2] [Windows 7 won't boot](../windows-7-wont-boot)


## sources
 1. [Error message occurs after you change the SATA mode of the boot drive](https://support.microsoft.com/en-us/topic/error-message-occurs-after-you-change-the-sata-mode-of-the-boot-drive-0e3ab9b8-99aa-c2b3-1f27-dc9eaf58a14b)
 1. [How to fix stop 0x0000007B](https://www.lifewire.com/how-to-fix-stop-0x0000007b-errors-2624109)
