## How to Create Mixed-Mode Geometry CP/M Boot Disks From Scratch in IMG, IMD and HFE File Formats

Another name for this repository could be "Creating Exidy Sorcerer & Morrow DiskJockey 2D CP/M Boot + Data Disks from Scratch and for HxC Emulator".

I believe this is an important repository for anyone looking to create IMG/IMD/HFE CP/M boot and data disks from scratch from a collection of PC-based disk files, especially if the 2 system tracks use "mixed mode geometry". It actually took me several months to work out all of the issues even if you may consider this a simple "60 second operation".

<div style="text-align:center">
<img src="/Images/Snapshot.webp" alt="" style="width:75%; height:auto;">
</div>

In particular, this tutorial will explain:

- Automatically creating IMG, IMD and HFE formatted CP/M disk images from a directory of PC files of 8.3 characters. HFE files are used with the [HxC Floppy Emulator](<https://hxc2001.com>).

- How to deal with "mixed track geometry", including single-sided and double-sided disks, whereby track 0 is in single-density (128x16) and the remainder of the tracks are in double density (1024x8), as used with the Morrow DiskJockey 2D double-density disk format. 

- How to insert a stock system boot image into tracks 0 and 1, and in either single-sided or double-sided modes. The creation of those Morrow DJ2D system boot images is [outlined here](<https://github.com/rcl9/Morrow-DJ2D-CPM-22-Recompile-From-Source>).

- In particular, for my own usage scenario, this process was created to clone/mirror my physical Morrow DiskJockey 2D S-100 8" floppy disk system running on my Exidy Sorcerer and its S-100 expansion box. My intent was to be able to take my archived collection of files on my PC hard disk and automatically save them back to 640k (single sided) or 1.2MB (double sided) HFE files to be used with the HxC Floppy Emulator. This process would also allow me to "pick and choose" from a number of [pre-defined system boot images](</Boot Track System Images for DJ2D and Exidy Sorcerer>) that I had created in the near past. 

## An Overview of the Disk Image Creation Process

This section documents my [script files](</Conversion script>) automation process.  The script files are written in the "4DOS/4NT/TCE/Take-Command" scripting language by [JP Software](<https://jpsoft.com>) for which I was one of their inital users. They also depend on Unix-like commands like 'rm' and 'dd' which are defined by the CygWin toolset on my computer.

- First you need to make a directory of the files on your PC which are to be embedded in the CP/M file image. Note that due to the larger block sizes on a CP/M disk you will often have to keep the overall directory file size much less than the stated capacity of the CP/M diskette (ie. 500k files for a 640k single sided diskette). Also, the directory's name must be 8 character or less due to the restrictions of the CPMtools utility program. 

- The *_Make_Sorcerer_CPM_Disk_IMG_File.btm* script file will need to be modified in the following manner:
  
  - *ImageDisk_PATH* is to point to the 32-bit version of the ImageDisk utilities, maintained by Mark Ogden at <a href="https://github.com/ogdenpm/disktools">this URL</a>.
  
  - *HxC_PATH* is to point to the HxC floppy emulator's directory which contains the *hxcfe.exe* command line conversion utility.

  - The CPMTools directory needs to be appended to your computer's global PATH environment variable. 
  
  - If you are going to execute this script from a PC disk drive other than where CPMTools is located then you need to copy the "diskdefs" file to the directory *\cpmtools* on the disk drive holding this script file. Why is this necessary? Well, the older version of CPMTools (and not the "Modified" version with libDSK support) ignores the *CPMTOOLS* environment variable. 
  
  - The global environment variable *CPMTOOLS* needs to point your CPMTools directory. Given the comment in my prior paragraph, this might not be necessary on your computer. 
  
   - The two different references of the *IMAGEDISK_B2I_CONFIG_FILE* variable will need to be set up for your own CP/M file format. I have them defined to be "dj2d_SS.B2I" and "dj2d_DS.B2I" (for the Morrow DJ2D disk format running on the Exidy Sorcerer). I store these files in the 32-bit ImageDisk home directory. Creating these two custom ImageDisk B2I config files of my own making was one of trickier aspects of getting this scripted process to work. I've broken out my experience in a sub-section towards the end of this tutorial. 

- The 'diskdefs' file in your CPMTools directory will need to be augmented with the definition(s) for your own CP/M file format. Those for the DiskJockey 2D are provided in [this directory](</DJ2D diskdefs for CPMTools>).

- A short explanation is in order. Since the DJ2D file format uses "mixed track geometry" for tracks 0 (SD) and tracks 1 (DD), I could not use CPMTools itself to create a single unified image file from the get-go (with tracks 0 to 77) which accomodated this varying track geometry. Instead my script creates the data tracks first (tracks 2 through 77) and only later prepends the two system tracks. Because of this situation my **diskdef** file contains two specialized sub-definitions named *rclcpm_SS_nosystracks* and *rclcpm_DS_nosystracks*. These basically have the number of tracks reduced by 2. The script file itself makes reference to those two files depending on whether a single sided or double sided disk is being created:
  
  ```
  set FORMAT=rclcpm_SS_nosystracks
  set FORMAT=rclcpm_DS_nosystracks
  ```

- The *_Make_All_Sorcerer_CPM_Disk_IMG_Files.btm* script file calls the *_Make_Sorcerer_CPM_Disk_IMG_File.btm* script file with the name of a directory of PC-based files. It also defines whether a single-sided or double-sided disk is desired, and also the selection of [System image](</Boot Track System Images for DJ2D and Exidy Sorcerer>) for tracks 0 and 1. You can define multiple disks to be created at the same time in this top-level script file.

- The first part of the *_Make_Sorcerer_CPM_Disk_IMG_File.btm* script file determines the filename of the system image which is desired and stored in **INPUT_SYSTEM_IMAGE_NAME**.

- An empty CP/M disk file is then created using:
  
  ```
  mkfs.cpm.exe -f %FORMAT% %2
  ```
  
    whereby **FORMAT** is determined to be these two **diskdefs** needed by CPMTools:
  
  ```
  set FORMAT=rclcpm_SS_nosystracks
  set FORMAT=rclcpm_DS_nosystracks
  ```

- The PC-based list of files are copied into the CP/M disk file via:
  
  ```
  cpmcp -f %FORMAT% %2 %1 0:
  ```

- The contents of the new disk file is listed with:
  
  ```
  cpmls -f %FORMAT% %2
  ```

- And the file system checked with:
  
  ```
  fsck.cpm.exe -f %FORMAT% %2
  ```

- If the desired disk format is single sided then the selected "System boot image" is just pre-pended to the CP/M IMG file generated using the above processes (2 tracks - one SD and one DD). Simple!

- However, if the desired disk format is double sided then things get more involved. This is why I mention "Mixed Geometry" in the title of this tutorial. My stock "system images" are 2 tracks long (with track 0 in SD and track 1 in DD). However, for double sided disks we need to make the system image 4 tracks long. The script file basically plays some games to prepend this to our previously created CP/M Image file:

```
    Track 0 on side 0 (3328 bytes = 16 sectors of 128 bytes each = SD)
    3328 bytes for an empty SD track 0 on side 1
    Track 1 on side 0 (8192 bytes = 8 sector of 1024 bytes each == DD)
    8192 bytes for an empty DD track 1 on side 1
```

    This is explained in detail in the script file. 
  
    Note: the DJ2D controller card only reads the system tracks off of side 1 and not side 2 even though it interleaves track access between both sides. 

- The resulting IMG file is checked for the correct and expected file size with:
  
  ```
   fsutil file seteof %2 625920
  ```
  
    or
  
  ```
  fsutil file seteof %2 1251840
  ```

- The raw CP/M IMG file is converted to a DJ2D compatible ImageDisk IMD file using the 32-bit version of the bin2imd utility with:
  
  ```
  %ImageDisk_PATH%\bin2imd ^
      %2 ^
      %~n2.imd ^
      %ImageDisk_PATH%/%IMAGEDISK_B2I_CONFIG_FILE%
  ```
  
    The creation of the two B2I (binary-to-IMD) files are explained further below. 

- The IMD file is then converted to a HFE file to be used with the HxC Floppy Emulator with:
  
  ```
  %HxC_PATH%\hxcfe.exe ^
      -finput:"%~n2.IMD" ^
      -conv:HXC_HFE ^
      -foutput:"%~n2.hfe"
  ```

- You can then use the HFE file with your HxC Floppy Emulator and/or archive or use the IMD file with the common utilities that support the IMD disk archiving file format. 

## Creating the Custom "B2I" Config File for ImageDisk

As noted above one of the more "hair pulling" of experiences was to come up with my own functional custom definition for two "B2I" (binary-to-IMD) configuration files for the 32-bit version of ImageDisk. Why are these needed? Because ImageDisk has no idea of how to take a flat geometry-less binary IMG CP/M file and turn it back into an ImageDisk IMD file with the proper disk geometry encoded within it. That encoding comes about from the information in B2I file. 

It took a good bit of time to figure out how the B2I config file format + definitions work through trial and error. At this time B2I is not a well documented format. Why was it tricky? Because of the "Mixed geometry" format of DJ2D CP/M disks for which track 0 is in single density (FM) and tracks 1+ are in double density (MFM). It also gets more complicated if the disk is double sided as I note in my second B2I file. 

I am providing two versions, [one for single sided](</ImageDisk B2I Config Files/dj2d_SS.B2I>) disks and one for [double sided](</ImageDisk B2I Config Files/dj2d_DS.B2I>) disks. The conversion script file differentiates which one is to be used via the **FORMAT** script variable. These are defined for the Morrow DJ2D CP/M disk format running on my Exidy Sorcerer. 

## See also

[CPMTools](<https://www.moria.de/~michael/cpmtools>). I used [CPMTools for Windows](<http://cpmarchives.classiccmp.org/cpm/mirrors/www.cpm8680.com/cpmtools/index.htm>)

[ImageDisk 32-bit Version](<github.com/ogdenpm/disktools>)

[HxC Emulator software tools](https://hxc2001.com/download/floppy_drive_emulator/HxCFloppyEmulator_soft.zip)

[How to Boot CP/M-3 from the HxC Floppy Emulator on the Cypher Z80/68000 SBC](<https://github.com/rcl9/Cypher-Z80-68000-Single-Board-Computer---Running-CPM-3-via-the-HxC-Floppy-Emulator>)

[Morrow DISK JOCKEY 2D CP/M 2.2 "SYSGEN" Recompile From Source Files (For Exidy Sorcerer)](<https://github.com/rcl9/Morrow-DJ2D-CPM-22-Recompile-From-Source>)

[Morrow DISK JOCKEY 2D Floppy Disk Controller for S-100 Bus - A Historical Compendium & Snaphot of Technical Information](<https://github.com/rcl9/Morrow-DISK-JOCKEY-2D-Floppy-Disk-Controller-for-S100-Bus>)

[CPM-Floppy-Definitions & List of How-To's](<https://github.com/ldkraemer/CPM-Floppy-Definitions>)

[Access CP/M Floppy's via libdsk & cpmtools](<https://forums.debian.net/viewtopic.php?t=112244>)