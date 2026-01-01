# Introduction to macOS

The modern macOS operating system came from the merger of two operating systems, NeXTSTEP and Mac OS 9. The merger began in 1997 when Apple acquired NeXT, a company owned by Steve Jobs, and took three years to complete. NeXTSTEP provided the core, the kernel, and the runtime. Mac OS provided the GUI, which was completely rewritten to work with the new core.

In 2001, the first version of OS X was released. It debuted with the name “Cheetah” and a version number of 10.0. Every year or two, Apple released a new version, and the OS significantly evolved over time. In 2016, OS X was renamed to macOS to align with the naming convention of Apple’s  other operating systems (e.g.: tvOS, iOS, ipadOS).

In this module we will focus on the introduction of macOS, its architecture, and key elements of the system. Once the fundamentals are laid, we will review standard executables on the system called Mach-O files. Lastly, we will talk about Objective-C, cover some of the basic syntax, and features that we will require to develop exploits and to better reverse engineer and understand applications.

These sections are not meant to take the place of a macOS internals or development course. Instead, we hope to provide a useful overview that will help us with the work ahead.

## macOS System Overview

To better understand how the OS is organized, we will explore the OS structure, its basic building blocks, and its architecture. Following that, we will learn how the Apple File System (APFS) works on a high-level. Specifically, how it organizes the file system and some important directories. Then we will explore the concept of bundles, which are a fundamental structure that macOS uses to organize applications. Finally, we will learn about property list (PLIST) files, which the entire  system uses extensively to store various configuration data.

## High-Level OS Architecture

macOS is a complex system built from multiple components. The following high level diagram illustrates some of the core components.

<img width="1280" height="1209" alt="image" src="https://github.com/user-attachments/assets/8624d1f7-e03d-4b41-9ab1-5ec79e3bcd6f" />

At the core of the OS, we have the XNU kernel. It’s a hybrid kernel comprised of the Mach microkernel, components from BSD, IOKit, and Kernel Extensions (KEXT). Let’s examine each of these.

XNU runs the Mach microkernel, which is only responsible for the most basic tasks, like task scheduling, managing threads, interfacing with the hardware, managing virtual memory, and passing messages between tasks.

Next, the BSD kernel component brings higher-level abstractions, like the POSIX process model. BSD is also responsible for the file system, user management, and networking.

Next, IOKit enables developers to create device drivers using object-oriented programming in C++. For example, Apple implements a class to handle USB thumb drives. As a result, developers are only responsible for implementing extensions unique to their device or possibly overloading existing functions as necessary.

This brings us to Kernel Extensions (KEXT), which provide additional functionality. Sandboxing, for example is implemented in Sandbox.kext.

The rest of the layers we will review are all implemented in user mode. Because they are typically interdependent, it can be helpful to think of them in a hierarchy, so we’ll continue to reference the structure described in image above.

On top of the kernel, we find the core and third-party libraries. These libraries implement fundamental functionalities. For example, they include libraries such as libmalloc, which is the memory allocation library, and libxpc, which implements the XPC cross-process communication 
protocol.

The components we have described so far, including the kernel, provide the building blocks for macOS’s core, Darwin. Many of Darwin’s components are open-sourced by Apple and can be downloaded from Apple’s website.

Technically, it is possible to download these components, compile them, and build the XNU kernel. This would give us a fully working kernel environment, but it would be missing some proprietary components built by Apple. One example would be Sandbox, which besides its application sandbox functionality also implements extra security features. Another example would be AppleMobileFileIntegrity, which is responsible for many of the code signing operations. 

On top of Darwin, we find the runtime environments. These support being able to run code written in different languages. The primary languages for developing apps on macOS are Objective-C and Swift. We will review Objective-C in more detail later in this module. In a later module, we will also learn more about how the Objective-C runtime works.

Above the runtime environments, we find the frameworks implemented by Apple in macOS. While some frameworks are public and designed to be used by developers, others are private and meant to be only used by Apple (although, as we will show in later modules, we can still use them). While developers still have access to the low-level C libraries, these frameworks provide higher-level APIs with more functionality.

Finally, at the top of the architecture, we have the applications provided by macOS. Some of these, like Spotlight, Terminal, and Finder, are commonly known. Others, like WindowServer, which provides the look and feel of the system, are less known.

Next, we will explore the key parts of the file system.

### Apple Proprietary File System (APFS)

With macOS 10.13, Apple replaced the HFS+ file system with the Apple Proprietary File System(APFS). Understanding some key features and properties of APFS will be very helpful. Specifically, it will be useful to understand how partitions and volumes work in APFS, compared to other file systems.

The figure below illustrates a typical, non-APFS disk layout.

<img width="581" height="381" alt="image" src="https://github.com/user-attachments/assets/7decc73c-dacd-4789-9b2a-8c668bed9a14" />

Traditionally, when we split a disk into multiple logical disks or partitions, we create a single file system volume on each partition. This files system takes up the entire space of the partition.

A volume is a logical unit of the file system that the operating system can access. The partition sizes are fixed, and the volumes on different partitions can’t consume each other’s space. Resizing a partition is possible, but it’s a common source of data loss if the operation isn’t performed properly. Increasing the size is less problematic than downsizing.

APFS, on the other hand, works a bit differently.

<img width="581" height="381" alt="image" src="https://github.com/user-attachments/assets/69305830-6c33-4d31-a2dd-73a1269c92e5" />

In APFS, a single partition will first contain an APFS container. Within the container, we find one or more APFS Volumes. Each APFS volume is still a single logical unit.

The benefit of this model is that all APFS volumes in a single container share the free space of the entire container. This means that we can have logically separated file systems, and the size of each file system can be dynamic. APFS allow for much better space utilization compared to the traditional approach.

Let’s connect to our bigsur1 virtual machine and explore the file system.

We can use the diskutil list command to get an overview of the APFS volumes on our disk

```
~ % diskutil list
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *251.0 GB   disk0
   1:             Apple_APFS_ISC Container disk1         524.3 MB   disk0s1
   2:                 Apple_APFS Container disk3         245.1 GB   disk0s2
   3:        Apple_APFS_Recovery Container disk2         5.4 GB     disk0s3

/dev/disk3 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +245.1 GB   disk3
                                 Physical Store disk0s2
   1:                APFS Volume Macintosh HD            12.2 GB    disk3s1
   2:              APFS Snapshot com.apple.os.update-... 12.2 GB    disk3s1s1
   3:                APFS Volume Preboot                 8.1 GB     disk3s2
   4:                APFS Volume Recovery                1.2 GB     disk3s3
   5:                APFS Volume Data                    177.9 GB   disk3s5
   6:                APFS Volume VM                      6.4 GB     disk3s6

/dev/disk4 (disk image):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        +102.1 MB   disk4
   1:                  Apple_HFS KeePassXC               102.1 MB   disk4s1
```

Here we observe the volume names and the sizes they consume. We can also see that the APFS container, disk1, contains multiple volumes, named disk1s1 through disk1s5.

The properties of the volumes can be different. For example, one can be encrypted, while another is not

> NOTE: For more on APFS and the layout, we recommend several of Howard Oakley’s blog posts.

Next, we will explore how macOS protects core system files through multiple layers of defense.

### System Volume Protections

macOS protects its core system files with System Integrity Protection (SIP), also known as rootless. While SIP has many responsibilities, its initial purpose was to prevent anyone, even the superuser, root, from modifying system files and directories.

The /System directory is the most well-known SIP protected directory, but there are many others. Using the ls command with the -lO switch on the root directory, we can find some SIP protected directories.

```
~ % ls -lO /
total 10
drwxrwxr-x  43 root  admin  sunlnk            1376 30 Dec 01:20 Applications
drwxr-xr-x@ 39 root  wheel  restricted,hidden 1248 29 Oct 01:21 bin
drwxr-xr-x   2 root  wheel  hidden              64  7 Dec  2024 cores
dr-xr-xr-x   4 root  wheel  hidden            4804 30 Nov 20:26 dev
lrwxr-xr-x@  1 root  wheel  restricted,hidden   11 29 Oct 01:21 etc -> private/etc
lrwxr-xr-x   1 root  wheel  hidden              25 30 Nov 20:27 home -> /System/Volumes/Data/home
drwxr-xr-x  69 root  wheel  sunlnk            2208 30 Nov 20:27 Library
drwxr-xr-x   3 root  wheel  hidden              96 21 Jun  2025 opt
drwxr-xr-x   6 root  wheel  sunlnk,hidden      192 30 Nov 20:27 private
drwxr-xr-x@ 76 root  wheel  restricted,hidden 2432 29 Oct 01:21 sbin
drwxr-xr-x@ 10 root  wheel  restricted         320 29 Oct 01:21 System
lrwxr-xr-x@  1 root  wheel  restricted,hidden   11 29 Oct 01:21 tmp -> private/tmp
drwxr-xr-x   5 root  admin  sunlnk             160 30 Nov 20:27 Users
drwxr-xr-x@ 11 root  wheel  restricted,hidden  352 29 Oct 01:21 usr
lrwxr-xr-x@  1 root  wheel  restricted,hidden   11 29 Oct 01:21 var -> private/var
drwxr-xr-x   4 root  wheel  hidden             128 23 Dec 21:31 Volumes
```

The restricted flag denotes a directory or file that is protected by SIP. Subdirectories within a SIP protected directory may be excluded.

```
~ % ls -lO /usr/ 
total 0
drwxr-xr-x  919 root  wheel  restricted 29408 29 Oct 01:21 bin
drwxr-xr-x   32 root  wheel  restricted  1024 29 Oct 01:21 lib
drwxr-xr-x  420 root  wheel  restricted 13440 29 Oct 01:21 libexec
drwxr-xr-x    3 root  wheel  sunlnk        96 30 Nov 20:27 local
drwxr-xr-x  230 root  wheel  restricted  7360 29 Oct 01:21 sbin
drwxr-xr-x   43 root  wheel  restricted  1376 29 Oct 01:21 share
drwxr-xr-x    5 root  wheel  restricted   160 29 Oct 01:21 standalone
lrwxr-xr-x    1 root  wheel  restricted    25 29 Oct 01:21 X11 -> ../private/var/select/X11
lrwxr-xr-x    1 root  wheel  restricted    25 29 Oct 01:21 X11R6 -> ../private/var/select/X11
```

When syntax the /usr directory, we find that the local subdirectory has the sunlnk(System No Unlink) flag. This means local can’t be deleted, but users are allowed to create or delete files and directories inside.

In addition to SIP, macOS has other protections in place for core system files. We can explore them by running diskutil apfs list. The command lists the details of the APFS volumes and their layout.

```
~ % diskutil apfs list
APFS Containers (3 found)
|
+-- Container disk3 02309B08-10E9-4C67-88EC-EFF7826A50CE
    ====================================================
    APFS Container Reference:     disk3
    Size (Capacity Ceiling):      245107195904 B (245.1 GB)
    Capacity In Use By Volumes:   206009921536 B (206.0 GB) (84.0% used)
    Capacity Not Allocated:       39097274368 B (39.1 GB) (16.0% free)
    |
    +-< Physical Store disk0s2 67FBEB05-4ED5-41AE-96F5-FB1E08013549
    |   -----------------------------------------------------------
    |   APFS Physical Store Disk:   disk0s2
    |   Size:                       245107195904 B (245.1 GB)
    |
    +-> Volume disk3s1 F8590D7B-5752-4271-BABE-30665839A8A7
    |   ---------------------------------------------------
    |   APFS Volume Disk (Role):   disk3s1 (System)
    |   Name:                      Macintosh HD (Case-insensitive)
    |   Mount Point:               Not Mounted
    |   Capacity Consumed:         12190003200 B (12.2 GB)
    |   Sealed:                    Yes
    |   FileVault:                 Yes (Unlocked)
    |   Encrypted:                 No
    |   |
    |   Snapshot:                  067913E9-6A57-426C-BA34-6E064B9C200D
    |   Snapshot Disk:             disk3s1s1
    |   Snapshot Mount Point:      /
    |   Snapshot Sealed:           Yes
    |
    +-> Volume disk3s2 70C8F5AE-906F-476B-A7F5-EE7E0AE36F73
    |   ---------------------------------------------------
    |   APFS Volume Disk (Role):   disk3s2 (Preboot)
    |   Name:                      Preboot (Case-insensitive)
    |   Mount Point:               /System/Volumes/Preboot
    |   Capacity Consumed:         8107560960 B (8.1 GB)
    |   Sealed:                    No
    |   FileVault:                 No
    |
    +-> Volume disk3s3 52181C7E-F697-4A45-9B74-A5938E58025B
    |   ---------------------------------------------------
    |   APFS Volume Disk (Role):   disk3s3 (Recovery)
    |   Name:                      Recovery (Case-insensitive)
    |   Mount Point:               Not Mounted
    |   Capacity Consumed:         1228455936 B (1.2 GB)
    |   Sealed:                    No
    |   FileVault:                 No
    |
    +-> Volume disk3s5 83AF7316-7778-460D-86BA-F1DD15107F7B
    |   ---------------------------------------------------
    |   APFS Volume Disk (Role):   disk3s5 (Data)
    |   Name:                      Data (Case-insensitive)
    |   Mount Point:               /System/Volumes/Data
    |   Capacity Consumed:         177902637056 B (177.9 GB)
    |   Sealed:                    No
    |   FileVault:                 Yes (Unlocked)
    |
    +-> Volume disk3s6 2F14DF40-550D-44CE-9242-1B4A3A0D2196
        ---------------------------------------------------
        APFS Volume Disk (Role):   disk3s6 (VM)
        Name:                      VM (Case-insensitive)
        Mount Point:               /System/Volumes/VM
        Capacity Consumed:         6442475520 B (6.4 GB)
        Sealed:                    No
        FileVault:                 No
```

Let’s review some of the information here.

Under Volume disk1s1 we note that user-accessible locations are mounted under /System/Volumes/Data. This is the APFS Data volume.

Next, under Volume disk1s5, macOS has a snapshot of the System APFS volume, and it mounts the snapshot at the root directory (/). Generally, APFS snapshot captures the state of the file system and enables recovery in case files are deleted or modified. Note that the System volume 
itself is not mounted–only its snapshot.

Next, we note that the snapshot is sealed. This means that the System volume snapshot is cryptographically signed by the OS. This is done as a security measure in case an attacker manages to bypass SIP. Any modification would invalidate the seal and would cause the OS not to boot anymore. This particular feature was introduced in Big Sur.

We can verify if the seal is enabled by running the csrutil authenticated-root status command.

```
~ % csrutil authenticated-root status 
Authenticated Root status: enabled
```

The output shows that the seal is enabled.

Let’s discuss an additional protection mechanism. To begin, we’ll check the output of the mount command.

```
~ % mount
/dev/disk3s1s1 on / (apfs, sealed, local, read-only, journaled)
devfs on /dev (devfs, local, nobrowse)
/dev/disk3s6 on /System/Volumes/VM (apfs, local, noexec, journaled, noatime, nobrowse)
/dev/disk3s2 on /System/Volumes/Preboot (apfs, local, journaled, nobrowse)
/dev/disk3s4 on /System/Volumes/Update (apfs, local, journaled, nobrowse)
/dev/disk1s2 on /System/Volumes/xarts (apfs, local, noexec, journaled, noatime, nobrowse)
/dev/disk1s1 on /System/Volumes/iSCPreboot (apfs, local, journaled, nobrowse)
/dev/disk1s3 on /System/Volumes/Hardware (apfs, local, journaled, nobrowse)
/dev/disk3s5 on /System/Volumes/Data (apfs, local, journaled, nobrowse, protect, root data)
map auto_home on /System/Volumes/Data/home (autofs, automounted, nobrowse)
/dev/disk4s1 on /Volumes/KeePassXC (hfs, local, nodev, nosuid, read-only, noowners, quarantine, mounted by victorsangwan)
```

The snapshot disk is mounted as read-only. Even if we could bypass SIP, we couldn’t write to the volume. This adds another layer of protection for core system files.

To summarize, the core system files have three layers of protection. They are protected by SIP, and the snapshot is both cryptographically signed and mounted as read-only.

### Firmlinks

There is one more APFS feature that we need to understand before we move forward. Previously, we discussed that the root folder is mounted as read-only and the Data volume is mounted readwrite at /System/Volumes/Data. Interestingly, if we check the /usr/local directory we find that most files are owned by our user, and we can freely write there.

```
~ % ls -l /usr/local
total 0
drwxr-xr-x  16 root  wheel  512 24 Sep 15:47 bin
````

Let’s look into why this is possible.

When macOS Catalina and APFS introduced the Data volume, it also introduced firmlinks. Firmlinks enable the OS to map directories in the Data volume, to directories on the System volume. Apple describes this as a “Bi-directional wormhole in path traversal.” Firmlinks are used on the System volume to point to the user data on the Data volume.

The list of firmlinks can be found in the /usr/share/firmlinks file.

```
~ % cat /usr/share/firmlinks
/AppleInternal	AppleInternal
/Applications	Applications
/Library	Library
/System/Library/Caches	System/Library/Caches
/System/Library/Assets	System/Library/Assets
/System/Library/PreinstalledAssets	System/Library/PreinstalledAssets
/System/Library/AssetsV2	System/Library/AssetsV2
/System/Library/PreinstalledAssetsV2	System/Library/PreinstalledAssetsV2
/System/Library/CoreServices/CoreTypes.bundle/Contents/Library	System/Library/CoreServices/CoreTypes.bundle/Contents/Library
/System/Library/Speech	System/Library/Speech
/Users	Users
/Volumes	Volumes
/cores	cores
/opt	opt
/private	private
/usr/local	usr/local
/usr/libexec/cups	usr/libexec/cups
/usr/share/snmp	usr/share/snmp
```

On the left, we have the directory path on the System volume, and on the right, the directory path where it maps on the Data volume. For example, /usr/local maps to /System/Volumes/Data/usr/local because /System/Volumes/Data is where the Data volume is mounted.

Let’s explore this a bit.

```
~ % ls -li /System/Volumes/Data/usr/
total 0
8255651 drwxr-xr-x  3 root  wheel  96 29 Oct 01:21 libexec
8255754 drwxr-xr-x  3 root  wheel  96 30 Nov 20:27 local
8255755 drwxr-xr-x  3 root  wheel  96 29 Oct 01:21 share

~ % ls -li /usr/
total 0
1152921500312559137 drwxr-xr-x  919 root  wheel  29408 29 Oct 01:21 bin
1152921500312560779 drwxr-xr-x   32 root  wheel   1024 29 Oct 01:21 lib
1152921500312561867 drwxr-xr-x  420 root  wheel  13440 29 Oct 01:21 libexec
            8255754 drwxr-xr-x    3 root  wheel     96 30 Nov 20:27 local
1152921500312563398 drwxr-xr-x  230 root  wheel   7360 29 Oct 01:21 sbin
1152921500312563841 drwxr-xr-x   43 root  wheel   1376 29 Oct 01:21 share
1152921500312596926 drwxr-xr-x    5 root  wheel    160 29 Oct 01:21 standalone
1152921500312559135 lrwxr-xr-x    1 root  wheel     25 29 Oct 01:21 X11 -> ../private/var/select/X11
```

The local directories are the same. We can confirm this with the identical inode number for both code blocks. Per Wikipedia, inode (index node) is a data structure in a Unix-style file system that describes a file-system object such as a file or a directory.

### Important Directories

In this section we will walk through some of the key directories of the file system.

#### POSIX Directories

Since macOS is a BSD compliant operating system, we find many of the same directories we find 
in other *nix (Linux, Unix) based systems. Let’s review some of the important ones.

- /bin, /usr/bin, /sbin and /usr/sbin hold core system binaries, essential for the operating system.
- /usr/lib holds all of the system dynamic libraries (dylibs).
- /tmp is the common temporary folder. It’s emptied upon reboot and has the sticky bit set, so users can’t delete each other’s files.
- /var contains logs, configuration files, and other data files, including the user’s temp folder.
- A user’s home directory will be under /Users except for root which is at /var/root. This is different from most *nix systems, where home directories are commonly found under the /home/ directory.

In addition to the standard *nix directories, macOS introduces other directories that may prove useful from a security perspective.

#### LaunchDaemons and LaunchAgents

Specially crafted files placed in the LaunchAgents or LaunchDaemons directory will autorun commands or applications upon startup. Anything placed in the LaunchAgents directory will run as the logged-in user, while anything placed in LaunchDaemons will run as root.

These directories can be found in multiple locations. Both LaunchAgents and LaunchDaemons can be found in the /System/Library directory, which contains the core system daemons and agents. Third-party application-specific launch daemons and agents can be found under /Library directory. Finally, user-specific agents can be found in ~/Library.

The LaunchDaemons directory is not located in ~/Library. LaunchDaemons would run as root. Since the Library directory is within the user’s home directory, where the user has full write access, including LaunchDaemons there would allow for privilege escalation.

Next, we will discuss how applications are organized.

#### Applications

Applications are typically installed via drag and drop or package installers and are placed under /Applications. Core system apps, like Calculator, are located under /System/Applications. Users also have an ~/Applications directory; however, it’s not widely used.

Application data can be found in multiple locations, but the most common directory is /Library/Application Support for the applications running as root and ~/Library/Application Support for applications running as the current user.

Sandboxed apps have very limited file system access and are mapped into the ~/Library/Containers directory. Each sandboxed app has a folder, which is named according to the application’s bundle ID. The bundle ID is typically a string in a reverse domain name notation,30 which identifies the application. For example, Safari has the bundle ID of com.apple.Safari.

The /Library/PrivilegedHelperTools/ directory is the typical location of third-party application daemons that need to run as root.

#### Frameworks

Various system frameworks can be found in /System/Library/Frameworks/. Private frameworks can be found in /System/Library/PrivateFrameworks/. Private frameworks shouldn’t be used by applications, and in fact, Apple will deny applications that use private frameworks from being listed in the Mac App Store

#### Kernel and Kernel Extensions (KEXTs)

The kernel is located at /System/Library/Kernels/kernel.

Apple’s own kernel extensions are found in /System/Library/Extensions, while third-party kexts are typically installed under /Library/Extensions.

Now that we have explored the file system, we will learn about property list files, which are used across the OS to store various data.

### Property List Files

Property List files are another concept inherited from NextSTEP. These files commonly store serialized data and are used across the entire OS for multiple purposes. Most often they are used to store configuration data or metadata. These files have the .plist extension and are commonly referred to as PLIST files.

PLIST files can be found in three different formats (in order of how common they are): XML, binary, and JSON. For human readability, XML and JSON are the best formats. Binary representation is better for machine processing. The structure of the binary format is undocumented, and there are at least three versions of it, 

marked by the file headers bplist00, bplist15, or bplist16. bplist00 is the most common. For example, Music’s Info.plist is in a binary format (bplist00), which is located at /System/Applications/Music.app/Contents/Info.plist. PLIST files can hold the following data types: string, integer, real, date, array, dictionary, Boolean, 

and binary data. Binary data is usually stored as base64 encoded. Let’s take a look at the /System/Library/LaunchDaemons/com.apple.tccd.system.plist file. This PLIST is the startup definition for the system-wide tccd daemon, which is responsible for systemwide privacy settings and control.

```
~ % cat /System/Library/LaunchDaemons/com.apple.tccd.system.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>com.apple.tccd.system</string>
	<key>ProgramArguments</key>
	<array>
		<string>/System/Library/PrivateFrameworks/TCC.framework/Support/tccd</string>
		<string>system</string>
	</array>
	<key>MachServices</key>
	<dict>
		<key>com.apple.tccd.system</key>
		<true/>
	</dict>
	<key>POSIXSpawnType</key>
	<string>Adaptive</string>
	<key>EnablePressuredExit</key>
	<true/>
	<key>PublishesEvents</key>
	<dict>
		<key>com.apple.tccd.events</key>
		<dict>
			<key>DomainInternal</key>
			<true/>
		</dict>
	</dict>
</dict>
</plist>
```

In order to understand it a little better, let’s break this PLIST file into smaller chunks.

We’ll start with the header

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
```

The header of a PLIST file is always the same, with the version number as “1.0” and the !DOCTYPE unchanged.

Following the header, we have various data objects serialized into XML inside the plist tag.

```
<dict>
	<key>Label</key>
	<string>com.apple.tccd.system</string>
...
...
...	
</dict>
```

We have a dictionary, which is specified by dict. The dictionary has various keys and values. The key name is always a string. The first key we find is the Label key, which has a string value of “com.apple.tccd.system”.

Reading binary PLIST files is not quite as easy. Let’s check out ~/Library/Preferences/com.apple.screensaver.plist, which is a binary PLIST file. If we print it out, we might find a few human-readable strings, but the majority is not human-readable.

```
~ % cat ~/Library/Preferences/com.apple.screensaver.plist
bplist00?_tokenRemovalAction
                            "%
```

Luckily, Apple provides plutil, a command-line utility tool that converts PLIST files into various formats.

Using plutil, we’ll convert our binary file. The -convert xml1 switch will convert the file to XML format and -o - will display it to the standard output, which is our shell.

```
~ % plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>tokenRemovalAction</key>
	<integer>0</integer>
</dict>
</plist>
```

We can use plutil to convert to JSON, binary, or code blocks for Objective-C and Swift. To do this we will replacing xml1 with the json, binary1, objc, or swift switches.

```
~ % plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -
{"tokenRemovalAction":0}%

~ % plutil -convert swift ~/Library/Preferences/com.apple.screensaver.plist -o -
/// Generated from com.apple.screensaver.plist
let com.apple.screensaver = [
    "tokenRemovalAction" : 0,
]

~ % plutil -convert objc ~/Library/Preferences/com.apple.screensaver.plist -o -
/// Generated from com.apple.screensaver.plist
__attribute__((visibility("hidden")))
NSDictionary * const com.apple.screensaver = @{
    @"tokenRemovalAction" : @0,
};
```

The above syntax shows the different formatting options for conversion with plutil.

Next, we will explore the concept of bundles, which is a key term and structure to understand, as that is how all frameworks, extensions, apps are organized.

### Bundles

The concept of bundles originated with NextSTEP. The idea behind this approach was to have all the resources the application needs in a single location. This can include the executables, resource files, metadata, and unique dylibs or frameworks (which are not present on the system by default). The most frequent bundle we will encounter is the .app bundle, but many other executables are also packaged as bundles, such as .framework and .systemextension.

Bundles are not unique to executable codes. For example, .photoslibrary is the bundle used by Photo.app to store and organize pictures. We will only cover the main items of the .app bundle, but Apple has very extensive documentation about bundles.

Before we dive in, we need to mention that not all executables have to be packed in a bundle. For example, the classic BSD command-line tools, like ls and cat, are single executable files.

### The Application Bundle

In this section, we will explore the Safari.app bundle, found under /Applications/Safari.app.

Every bundle is a directory, with subdirectories and files, that follow a specific structure. We will start by exploring the top level.

```
~ % ls -l /Applications/Safari.app
lrwxr-xr-x@ 1 root  wheel  54 29 Oct 01:21 /Applications/Safari.app -> ../System/Cryptexes/App/System/Applications/Safari.app
```

If we list the contents of the .app directory, we will find that it contains a single directory called Contents. Normally we won’t find anything else in the root directory of an application.

If we examine the Contents directory, we will find other directories and files.

```
~ % ls -l /Applications/Safari.app/Contents
total 24
drwxr-xr-x   3 root  wheel     96 24 Oct 06:12 _CodeSignature
drwxr-xr-x   4 root  wheel    128 24 Oct 06:12 Extensions
-rw-r--r--   1 root  wheel  22381 24 Oct 06:12 Info.plist
drwxr-xr-x   4 root  wheel    128 24 Oct 06:12 MacOS
-rw-r--r--   1 root  wheel      8 24 Oct 06:12 PkgInfo
drwxr-xr-x   5 root  wheel    160 24 Oct 06:12 PlugIns
drwxr-xr-x  96 root  wheel   3072 24 Oct 06:12 Resources
-rw-r--r--   4 root  wheel    467 24 Oct 06:12 version.plist
drwxr-xr-x   5 root  wheel    160 24 Oct 06:12 XPCServices
```

Let’s quickly review some of the most important items in the Contents directory.

Every bundle contains the Info.plist file, which contains metadata about the application.

The MacOS directory contains the main executable of the application.

In the PkgInfo file, we will find the eight-character identifier of package. For most apps, the value will be “APPL” followed by four additional characters.

The Resources directory typically contains media resources, like icons and localization strings. The localization strings are inside another [2 letter country code].lproj directory. This is shown in syntax bellow.

```
~ % ls -l /Applications/Safari.app/Contents/Resources
total 51232
-rw-r--r--   1 root  wheel    110546 24 Oct 06:12 Acknowledgments.html
-rwxr-xr-x   1 root  wheel    136512 24 Oct 06:12 appdiagnose
-rw-r--r--   1 root  wheel     75152 24 Oct 06:12 AppIcon.icns
drwxr-xr-x  29 root  wheel       928 24 Oct 06:12 ar.lproj
-rw-r--r--   1 root  wheel  25817032 24 Oct 06:12 Assets.car
drwxr-xr-x  12 root  wheel       384 24 Oct 06:12 Background Images
drwxr-xr-x  30 root  wheel       960 24 Oct 06:12 Base.lproj
...
...
...
```

In the Contents directory, we also find the version.plist file. This is a PLIST that contains version information about the application.

Next, we find the _CodeSignature directory, which contains a single file named CodeResources. This PLIST file, which does not have an extension, contains the hashes of all the files that do not contain an embedded code signature. Typically, these are non-executable files, for example resources. The hashes are encoded as base64 strings. code block bellow shows a snippet of the CodeResources file.

```
<key>Resources/Assets.car</key>
<data>
pdz/bJtNEtI+1XVTUojI7w08OcE=
</data>
```

The base64 encoded hash, pdz/bJtNEtI+1XVTUojI7w08OcE= is a SHA1 hash.

We can recreate this by calculating the hash ourselves and then using openssl to base64 encode it. We’ll provide the dgst option to calculate the hash, the -binary flag to generate output in binary format, and -sha1 to specify the type of hash we are interested in. Finally, we’ll pipe the output into openssl specifying the base64 option to encode the output appropriately.

```
openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64
pdz/bJtNEtI+1XVTUojI7w08OcE=
```

The result string shown in code block is equal to the one provided in the PLIST file, which confirms that the CodeResources file contains the correct hash of the file.

Returning to the Contents directory, we also find a PlugIns directory, which contains plugins specific to Safari. This is not a common entry.

When we examine the PlugIns directory, we will find that there are four additional directories with the extension of .appex and .wkbundle.

```
~ % ls -l /Applications/Safari.app/Contents/PlugIns
total 0
drwxr-xr-x  3 root  wheel  96 24 Oct 06:12 CacheDeleteExtension.appex
drwxr-xr-x  3 root  wheel  96 24 Oct 06:12 DiagnosticExtension.appex
drwxr-xr-x  3 root  wheel  96 24 Oct 06:12 SafariQuickLookPreview.appex
```

These directories are also bundles. If we inspect the CacheDeleteExtension.appex directory, the output shows that it has a very similar format to the .app bundle.

```
ls -lR /Applications/Safari.app/Contents/PlugIns/CacheDeleteExtension.appex
total 0
drwxr-xr-x  6 root  wheel  192 24 Oct 06:12 Contents

/Applications/Safari.app/Contents/PlugIns/CacheDeleteExtension.appex/Contents:
total 16
drwxr-xr-x  3 root  wheel    96 24 Oct 06:12 _CodeSignature
-rw-r--r--  1 root  wheel  1860 24 Oct 06:12 Info.plist
drwxr-xr-x  3 root  wheel    96 24 Oct 06:12 MacOS
-rw-r--r--  6 root  wheel   468 24 Oct 06:12 version.plist

/Applications/Safari.app/Contents/PlugIns/CacheDeleteExtension.appex/Contents/_CodeSignature:
total 8
-rw-r--r--  1 root  wheel  2428 24 Oct 06:12 CodeResources

/Applications/Safari.app/Contents/PlugIns/CacheDeleteExtension.appex/Contents/MacOS:
total 48
-rwxr-xr-x  1 root  wheel  137840 24 Oct 06:12 CacheDeleteExtension
```

This directory has an Info.plist file, a MacOS directory for the executable, _CodeSignature for code signing information, and finally a version.plist file for version information.

### Other Bundles

The framework bundle is another common one. It’s similar to the app bundle but also includes a Version directory. We can investigate the AVFoundation framework as an example. We’ll find it at /System/Library/Frameworks/AVFoundation.framework.

```
~ % ls -l /System/Library/Frameworks/AVFoundation.framework/Versions
total 0
drwxr-xr-x  5 root  wheel  160 29 Oct 01:21 A
lrwxr-xr-x  1 root  wheel    1 29 Oct 01:21 Current -> A
```

AVFoundation has a Version folder with a subfolder of A and a symlink, called Current pointing to A. This structure allows for multiple versions, but most commonly we only find one, “A”.

Inside A there are several directories. This is similar to what we noticed with app bundles.

```
~ % ls -l /System/Library/Frameworks/AVFoundation.framework/Versions/A
total 0
drwxr-xr-x  3 root  wheel   96 29 Oct 01:21 _CodeSignature
drwxr-xr-x  3 root  wheel   96 29 Oct 01:21 Frameworks
drwxr-xr-x  7 root  wheel  224 29 Oct 01:21 Resources
```

We will not find the executable here. There is no MacOS directory, nor an executable. Up until macOS Catalina, there was an actual framework binary here with the name AVFoundation. With the release of Big Sur, this was removed and embedded in the shared cache, which is a single binary with most frameworks and dylibs embedded.

In general, if we find a directory name with an “extension”, it means that we’re probably looking at a bundle. That’s the case here, with AVFoundation.framework. While framework bundles do not contain executables, other bundles, such as kext and systemextension, have executables and share a similar layout.

We will discuss the dyld shared cache next in this section.

### The dyld Shared Cache

dyld is the dynamic linker on macOS. When an application is loaded, it loads the shared libraries into memory.

On macOS (and iOS) all system shared libraries, like frameworks and dylibs, are combined into a single file, called the dyld shared cache. When the cache is created, it goes through a series of operations, which are normally done by dyld at load time. This results in improved performance, since code can be loaded faster.

When an application loads, this cache will be used instead of the native libraries. Since the individual libraries are effectively the same and including them would have increased storage requirements, Apple removed these files from the system in macOS Big Sur. On iOS, this removal happened much earlier, in iOS 3.

Similar to the dyld shared cache, the kernel and the kernel extensions are also compiled into a kernel cache, which is loaded at boot time. The individual kernel and the extension files are still in place.

On Big Sur, the dyld shared cache can be found at /System/Library/dyld/dyld_shared_cache_x86_64. It is a large file, usually around 2.5GB.

Since the system libraries are no longer on the filesystem, Apple provides an open-source tool, called dyld_shared_cache_util, which can extract individual files from the cache. Using these extracted files, we can load them into a disassembler34 for inspection.

At the time of this writing, Apple has not released an update to the dyld_shared_cache_util tool that can work with Big Sur.

Thankfully, Jeff Johnson35 and the MBSPlugins Team36 were able to create a version that can process the shared cache found on Big Sur. We can use this updated tool to extract the libraries from the cache by running the following command.

```
~ % dyld_shared_cache_util -extract ~/shared_cache/ /System/Library/dyld/dyld_shared_cache_x86_64
```

Once the extraction is complete, we can retrieve the AVFoundation framework executable.

```
~ % ls -l shared_cache/System/Library/Frameworks/AVFoundation.framework/Versions/A/AVFoundation
ls: shared_cache/System/Library/Frameworks/AVFoundation.framework/Versions/A/AVFoundation: No such file or directory
```

Although we can extract these files, we need to remember that these are not the original files used to build the shared cache. This is because not all code optimization, performed during the creation of the cache, is reversed by the tool.

In the next section, we will learn about Mach-O files, which is the executable file format used by macOS

## The Mach-O File Format

Mach-O, or Mach Object, is a file format for various program files on all Apple platforms, from macOS to tvOS. Mach-O file types range from standard executables to dylibs, frameworks, and even kernel extensions.

In this section, we will discuss the core concepts and buildout of Mach-O files. As we go along, we will use XMachOViewer, and the otool command-line utility to analyze the /bin/ls executable.

We will also use the source code of XNU version 7195.50.7.100.1 to understand the building blocks of Mach-O files. We can view it online or download it and view it locally.

The source code has been downloaded and placed in the /Users/offsec/source directory on the bigsur1 lab machine. This will allow us to quickly search through the code with the grep command.

Next, we will discuss universal binaries.

### Universal Binaries

Mach-O supports the concepts of universal binaries, also known as fat files. Fat files allow the grouping of multiple versions of a file into a single binary. The resulting binary has a large file size, which explains the origin of the nickname, fat. Essentially we can compile our binary for multiple CPU types, like Intel and ARM, and then combine them into a single executable.

This feature was useful when Apple migrated their devices from PowerPC CPUs to Intel CPUs. They could embed Mach-O files for both architectures. Up until macOS Catalina, the OS also used this format to group binaries for the x86 and x64 versions of the executables. Now, with Apple starting to move to ARM with macOS Big Sur, the FAT files contain x64 and ARM versions of the executables.

The only difference between universal binaries and standard Mach-O files is an extra “FAT” header containing information about the various embedded Mach-O files.

Let’s explore the fat file structure defined in xnu-7195.50.7.100.1/EXTERNAL_HEADERS/macho/fat.h.

```
1 #define FAT_MAGIC 0xcafebabe
2 #define FAT_CIGAM 0xbebafeca /* NXSwapLong(FAT_MAGIC) */
3 
4 struct fat_header {
5 uint32_t magic; /* FAT_MAGIC */
6 uint32_t nfat_arch; /* number of structs that follow */
7 };
8 
9 struct fat_arch {
10 cpu_type_t cputype; /* cpu specifier (int) */
11 cpu_subtype_t cpusubtype; /* machine specifier (int) */
12 uint32_t offset; /* file offset to this object file */
13 uint32_t size; /* size of this object file */
14 uint32_t align; /* alignment as a power of 2 */
15 };
```

A fat file header starts with a magic number (line 5), which is 0xcafebabe (defined in line 1). The magic number is followed by the number of architectures the file contains (in line 6). Each architecture will be defined in a fat_arch structure (line 9), which will contain information about the  CPU type and the placement of the binary within the file.

As an example, let’s analyze /bin/ls with the file command.

```
~ % file /bin/ls
/bin/ls: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit executable x86_64] [arm64e:Mach-O 64-bit executable arm64e]
/bin/ls (for architecture x86_64):	Mach-O 64-bit executable x86_64
/bin/ls (for architecture arm64e):	Mach-O 64-bit executable arm64e
```

The file command prints out basic information about the file type, and we note that it’s a universal binary.

Running the otool command with the -f switch gives us detailed information about the FAT header. Adding the -v switch will resolve numeric constants, like cputype, cpusubtype, and capabilities.

```
~ % otool -f -v /bin/ls
Fat headers
fat_magic FAT_MAGIC
nfat_arch 2
architecture x86_64
    cputype CPU_TYPE_X86_64
    cpusubtype CPU_SUBTYPE_X86_64_ALL
    capabilities 0x0
    offset 16384
    size 48112
    align 2^14 (16384)
architecture arm64e
    cputype CPU_TYPE_ARM64
    cpusubtype CPU_SUBTYPE_ARM64E
    capabilities PTR_AUTH_VERSION USERSPACE 0
    offset 65536
    size 89088
    align 2^14 (16384)
```

The otool command displays the sizes of the embedded files, the target CPU, and the offset. The offset indicates at what byte the embedded Mach-O file starts within the file.

### Mach-O Structure

Mach-O files consist of three main parts, as shown below

<img width="581" height="351" alt="image" src="https://github.com/user-attachments/assets/888166ad-720c-4a43-842e-9bc7164d06de" />

The header contains basic metadata about the file. For example, it may contain platform information. The load commands contain instructions on how to map the binary into memory. Finally, the data part holds the actual binary data that will be mapped to memory, like program code, and variables.

Next, let’s explore the Mach-O header.

### Mach-O Header

The Mach-O header is a short data blob. We can view its structure in the XNU kernel source code, at xnu-7195.50.7.100.1/EXTERNAL_HEADERS/mach-o/loader.h.

Here we find a header defined for 32-bit files and another for 64-bit files. Their definitions are included in code block bellow.

```
struct mach_header {
 uint32_t magic; /* mach magic number identifier */
 cpu_type_t cputype; /* cpu specifier */
 cpu_subtype_t cpusubtype; /* machine specifier */
 uint32_t filetype; /* type of file */
 uint32_t ncmds; /* number of load commands */
 uint32_t sizeofcmds; /* the size of all the load commands */
 uint32_t flags; /* flags */
};
struct mach_header_64 {
 uint32_t magic; /* mach magic number identifier */
 cpu_type_t cputype; /* cpu specifier */
 cpu_subtype_t cpusubtype; /* machine specifier */
 uint32_t filetype; /* type of file */
 uint32_t ncmds; /* number of load commands */
 uint32_t sizeofcmds; /* the size of all the load commands */
 uint32_t flags; /* flags */
 uint32_t reserved; /* reserved */
};
```

Both headers begin with a magic number (magic), followed by the CPU type definitions (cputype, cpusubtype), the filetype (filetype), the number and size of the load commands (ncmds, sizeofcmds), and finally flags.

The magic numbers are defined in the same header file as indicated in code block bellow.

```
/* Constant for the magic field of the mach_header (32-bit architectures) */
#define MH_MAGIC 0xfeedface /* the mach magic number */
#define MH_CIGAM 0xcefaedfe /* NXSwapInt(MH_MAGIC) */
/* Constant for the magic field of the mach_header_64 (64-bit architectures) */
#define MH_MAGIC_64 0xfeedfacf /* the 64-bit mach magic number */
#define MH_CIGAM_64 0xcffaedfe /* NXSwapInt(MH_MAGIC_64) */
```

We can check the Mach-O header information with otool and the -h flag. We’ll use -v to resolve numeric constants, CPU types, and the magic number.

Additionally, we’ll use the -arch option, which allows us to select the architecture we are interested in.

> NOTE: If we leave out the ‘-arch’ option, otool will use the first embedded Mach-O binary.

```
~ % otool -arch arm64e -hv /bin/ls
/bin/ls:
Mach header
      magic  cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
MH_MAGIC_64    ARM64          E USR00     EXECUTE    20       1712   NOUNDEFS DYLDLINK TWOLEVEL PIE

~ % otool -arch x86_64 -hv /bin/ls
/bin/ls:
Mach header
      magic  cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
MH_MAGIC_64   X86_64        ALL  0x00     EXECUTE    18       1736   NOUNDEFS DYLDLINK TWOLEVEL PIE
```

We can find the same results displayed in code block above in XMachOViewer as well.

<img width="1064" height="611" alt="Screenshot 2026-01-01 at 16 00 28" src="https://github.com/user-attachments/assets/88539746-9b88-41d0-953f-e706192c6739" />

We can confirm this also from the fact that the header’s last four-byte entry is at 001c, as indicated in screenshot above.

The various load commands have different structures, but each one begins with the same eight bytes, which define the type of command, and the total size of the command (in bytes). The structure definition is defined in the same xnu-7195.50.7.100.1/EXTERNAL_HEADERS/macho/loader.h file.

```
struct load_command {
	uint32_t cmd;			/* type of load command */
	uint32_t cmdsize;		/* total size of command in bytes */
};
```

There are about 50 different types of load commands that the system handles differently. We will briefly discuss the most common ones that we will encounter. These are LC_SEGMENT_64, LC_LOAD_DYLINKER, LC_MAIN, LC_LOAD_DYLIB, and LC_CODE_SIGNATURE.

### Load Commands

As a bit of review, load commands are structures that describe how to load different parts of the executable into memory.

LC_SEGMENT_64 (or LC_SEGMENT for x86 architecture) defines a segment that will be mapped into the process’s memory space. The segment might be a __TEXT segment, which contains the executable code, or a __DATA segment, which contains data for the process.

All segments can be found in the data portion of the Mach-O file. Each segment contains multiple sections, and the load command structure will contain information about each section inside the segment. The section information will directly follow the LC_SEGMENT_64 command. The structure of the command is shown in code block bellow.

```
struct segment_command_64 { 		/* for 64-bit architectures */
 	uint32_t cmd; 					/* LC_SEGMENT_64 */
 	uint32_t cmdsize; 				/* includes sizeof section_64 structs */
 	char segname[16]; 				/* segment name */
 	uint64_t vmaddr; 				/* memory address of this segment */
 	uint64_t vmsize; 				/* memory size of this segment */
 	uint64_t fileoff; 				/* file offset of this segment */
 	uint64_t filesize; 				/* amount to map from the file */
 	vm_prot_t maxprot; 				/* maximum VM protection */
 	vm_prot_t initprot; 			/* initial VM protection */
 	uint32_t nsects; 				/* number of sections in segment */
	uint32_t flags; 				/* flags */
};
```

The LC_SEGMENT_64 command defines the number of sections in the nsects member. After the LC_SEGMENT_64 command structure, we have the section information.

```
struct section_64 { 				/* for 64-bit architectures */
 	char sectname[16]; 				/* name of this section */
 	char segname[16]; 				/* segment this section goes in */
 	uint64_t addr; 					/* memory address of this section */
 	uint64_t size; 					/* size in bytes of this section */
 	uint32_t offset; 				/* file offset of this section */
 	uint32_t align; 				/* section alignment (power of 2) */
 	uint32_t reloff; 				/* file offset of relocation entries */
 	uint32_t nreloc; 				/* number of relocation entries */
 	uint32_t flags; 				/* flags (section type and attributes)*/
 	uint32_t reserved1; 			/* reserved (for offset or index) */
 	uint32_t reserved2; 			/* reserved (for count or sizeof) */
	uint32_t reserved3; 			/* reserved */
};
```

The section structure will hold the actual information about the section location in the file that is pointed to by the offset member.

To make sense of this, let’s take a look at the binary in XMachOViewer.
















