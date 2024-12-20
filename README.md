# IBM-M1215-Crossflash
Crossflashing an IBM M1215 to P16 (ASUS)

# Overview
This guide is made as a reference for crossflashing an IBM M1215 Raid Controller, which is an OEM branded LSI 9340-8i based on the SAS3008 chip. This adventure was undertaken because I had bought an ASUS PIKE II 3008-8i raid controller. It was working fine, but one of the ports would not recognize drives. After troubleshooting with multiple drives, different cables, and so on, it was concluded that one of the ports was FUBAR. In need of a new card, it turned out these cards are difficult to find, if not impossible. After finding out that the IBM M1215 was functionally equivalent, I purchased one to experiment. 

The process was actually much more straightforward than one would expect. Here is my attempt at a run down.

1. First you will need a bootable DOS usb drive AND a bootable EFI usb drive. I used the same usb drive, just had to reformat when the time came.
2. Boot in to the DOS drive and run _megarec.exe_ and a few other commands.
3. Shutdown/Restart
4. Boot into the EFI drive to install the firmware and flash the bios
5. Few more commands
6. Reboot

Let's get started.

## DOS / EFI USB Drives

First we need a DOS usb drive and an EFI usb drive. The process of making them is simple.

` DOS `
1. Download Rufus
2. Set `Boot Selection` to FreeDOS
3. `File System`: FAT 32
4. `START`

`EFI `
1. Open Rufus
2. Set `Boot Selection` to Non bootable
3. `START`
4. Download `/EFI/boot/BOOTX64.efi` (No idea where I got it from.)
5. Copy the entire directory structure to the drive. Done.

## Prerequisites

- megarec and the other file in that folder
- sbrempty.bin
- Firmware - **Note** - In this guide I am flashing the ASUS Firmware, which the latest version at the time of writing is 16.00.13. ASUS provides a support package with this firmware and the bios rom files. I would assume that this would also work for the LSI/Broadcom firmware for the 9300-8i series, which use the same 3008 raid on chip (roc), but this I cannot confirm.
- Take note of the SAS Adress of you controller. It can be found on as sticker on the board. You'll need it later.
- Patience

## Quick Notes

Much of what I did was based on information I scavanged from other people's experiences flashing an IBM M1215 and LSI 93XX cards. There was no record of anyone attempting to flash the ASUS firmware to an OEM raid card, so I was in the dark. With the OEM M1215 I was working with, `sas3flash.efi` would not recognize the card. From what I've read, it only recognizes LSI branded cards. Therefore, `megarec.exe` is required for the initial purging of the OEM configuration. Oh and I recall seeing in some other guides that two pins on the raid card were jumpered before beginning the flashing process. The pins were on the J6 connector on the board. I did not do this, but it still worked. I don't have an explanantion.

## The Process
1. Boot into DOS. Make sure the megarec files are on the usb drive, including the `DOS4GW.EXE`
2. Make sure `megarec.exe` can see you card. When I ran this, it was listed as "3108." To be honest, I don't know if that number is related to the SAS ROC number (3008), but I said, "close enough" and moved on.
   ```
   megarec -adplist
   ```
3. Run the following commands with megarec. These will backup the current configuration, I think, which you can restore if needed. Now, the second command errored out on me. I ignored it and moved on.
   ```
    megarec -readsbr 0 origsbr.bin
   ```
   ```
   megarec -readspd 0 origspd.bin
   ```
4. Now we clear the current configuration.
   ```
   megarec -writesbr 0 sbrempty.bin
   ```
   ```
   megarec -cleanflash 0
   ```
5. Thats it, shut-er down.
6. Boot into EFI. Include the firmware/ bios files on the same usb drive.
7. `sas3flash.exe -list` should show the SAS3008 controller.
8. Clear the current firmware
    ```
    sas3flash.efi -o -e 6
    ```
9. Flashing the firmware.

   ** Note **  if you need to flash to another firmware, you're on your own. Good luck. But the steps would look like:
      ```
       sas3flash.efi -f 3008MCTP.fw # update the firmware
       sas3flash.efi -b mptsas3.rom # update the BIOS
       sas3flash.efi -b mpt3x64.rom # continue to update the BIOS
      ```
10. Finally, we need to reset the SAS Adress.
    ```
    sas3flash.efi -o -sasadd <Insert YOUR SAS Adress Here>
    ```
11. Verify.
    ```
    sas3flash.efi -listall
    ```
12. Congrats, you're done. `reset -s` to shutdown from the EFI shell.

## References

Megarec download from HPE (valid Nov. 2024)

   https://support.hpe.com/connect/s/softwaredetails?language=en_US&collectionId=MTX-63f4ca46c9424e7d&tab=releaseNotes
   
Information on flashing another IBM controller

   http://www.penlug.org/FreeNasHardwareInfoIbmM1015.html
   https://forums.freebsd.org/threads/flashing-my-m1015.52387/
   https://www.truenas.com/community/threads/where-is-the-megarec-utility-to-crossflash-m1015-to-it-mode.25130/
   https://www.truenas.com/community/threads/ibm-serveraid-m1015-and-no-lsi-sas-adapters-found.27445/
   https://www.servethehome.com/flash-lsi-sas-3008-hba-e-g-ibm-m1215-mode/
   
Megarec Command Usage

   https://forums.servethehome.com/index.php?threads/lsi-megacli-preboot-cli-storecli-megascu-megarec-sas2flash-and-megaoem-commands.457/

Another user's experience flashing back to IR mode

   https://forums.serverbuilds.net/t/flashing-sas2208-to-it-mode-when-sas2flsh-does-not-detect-it/2357

Thorough Flashing Guide(s) for LSI Variants

   https://www.truenas.com/community/resources/detailed-newcomers-guide-to-crossflashing-lsi-9211-9300-9305-9311-9400-94xx-hba-and-variants.54/
   https://geeksunlimitedinc.wordpress.com/2017/02/16/flashing-lsi-hba-to-it-mode-or-ir-mode/
   https://github.com/EverLand1/9300-8i_IT-Mode
   https://github.com/lrq3000/lsi_sas_hba_crossflash_guide?tab=readme-ov-file

LSI Raid Controller Listing and OEM Equivalents

   https://forums.servethehome.com/index.php?threads/lsi-raid-controller-and-hba-complete-listing-plus-oem-models.599/
