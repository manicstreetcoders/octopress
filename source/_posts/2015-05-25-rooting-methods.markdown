---
layout: post
title: "Rooting Methods"
date: 2015-05-25 22:20:02 +0900
comments: true
categories: 
---

### HackingTeam 이 사용하는 exploits

* framaroot
* [towelroot](https://github.com/android-rooting-tools)
* [putuser](https://github.com/fi01/libput_user_exploit/blob/master/put_user.c)

### Gingerbreak

The vulnerability exploited by Gingerbreak exists in the Volume Manager (vold) on Android version 2.2 (Froyo) - and 3.0 (Honeycomb). Vold manages the mounting of external storage volumes on Android. The vulnerability was an out-of-bounds array access that allowed the exploit author to overwrite entries in the Global Offset Table (GOT) to trick the system into executing a copy of the sh binary as root.

[here](http://c-skills.blogspot.com/2011/04/yummy-yummy-gingerbreak.html)

### Exynos Abuse - Exploiting Custom Drivers

[here](http://forum.xda-developers.com/showthread.php?t=2048511)

The post detailed that a block device located at /dev/exynos-mem allowed the mapping of kernel memory into user space by any user. The exploitation technique used was to patch a comparison made in the `setresuid()` function. This comparison is normally `cmp r0,#0` and was altered to `cmp r0,#1` as a result of having complete access to the memory space, which meant that when sysresuid(0) was called later on the code, access was granted to change to root context. This exploit also elegantly bypassed the `kptr_restrict` memory protection, which does not allow applications to read `/proc/kallsyms` and obtain kernel pointers. It did so by changing the enforcing flag of this check in live memory.

### Samsung Admire - Abusing File Permissions with Symlinks

Permissive file permissions on files used by the system on Android devices can sometimes be used to obtain root. This method may sound obscure but consider the following classic example from Dan Rosenberg in his exploit for the Samsung Admire: [here](http://vulnfactory.org/blog/2011/09/12/rooting-the-samsung-admire/)

He discovered that when an application crashes, a dump file was created at `/data/log/dumpState_app_native.log` on the filesystem by root with the world-writable file permission. In addition, the `/data/log` parent directory was also world-writable. Therefore, placing a symbolic link named `dumpState_app_native.log` in this directory and causing an application to crash would cause a file to be written somewhere else on the filesystem as world-writable. There existed a file in older versions of Android at `/data/local.prop`, which was used to (among other things) determine what privilege level ADB should run under. This file was not present on this device and so Dan exploited this vulnerability to create the `/data/local/prop` file as wrold-writable and then insert a command in this file stating that ADB should run as root. This attribute is `ro.kernel.qemu=1` on this particular device. From there the exploit uses ADB as root, place the su binary, and installs the root manager application.

### Acer Iconia - Exploiting SUID binaries

A SUID binary that is owned by root and world-executable is very high-value target for root exploit developers. If they discover any vulnerabilities in this binary that allow the execution of arbitary code, they will have gained root access on the device.
This particular issue was discovered by an XDA Developers user named sc2k on the Acer Iconia A1000, which had a pre-installed SUID binary named cmdclient that was vulnerable to command injection. See the original post at [here](http://forum.xda-developers.com/showthread.php?t=1138228).

### Master Key Bugs - Exploiting Android AOSP System Code

[here](http://www.saurik.com/id/17)
[Cydia Impactor](http://www.cydiaimpactor.com)

### TowelRoot

[here](http://www.all-things-android.com/content/android-and-linux-kernel-towelroot-exploit)

### Wunderbar/asroot

This bug was discovered by Tavis Ormandy and Julien Tinnes of the Google Security Team and was assigned CVE-2009-2692.

The Linux kernel 2.6.0 through 2.6.30.4 and 2.4.4 through 2.4.37.4, does not initialize all function pointers for socket operations in proto_ops structures, which allows local users to trigger a NULL pointer dereference and gain privileges by using mmap to map page zero, placing arbitrary code on this page, and then invoking an unavailable operation, as demonstrated by the sendpage operation (sock_sendpage function) on a PF_PPPOX socket.

[here](http://www.zenthought.org/content/file/android-root-2009-08-16-source)

### Recovery: Volez

A typographical error in the signature verifier used in Android 2.0 and 2.0.1 recovery images caused the recovery to incorrectly detect the End of Central Directory (EOCD) record inside a signed update.zip file. This issue resulted in the ability to modify the contents of a signed OTA recovery image.

[here](http://zenthought.org/content/project/volez)

### Udev: Exploid

* CVE-2009-1185

### Adbd: RageAgainstTheCage

[here](http://stealth.openwall.net/xSports/RageAgainstTheCage.tgz)

### Zygote: Zimperlich and Zyysploit

[here](http://c-skills.blogspot.com.es/2011/02/zimperlich-sources.html)
[here](https://github.com/unrevoked/zysploit)

### Ashmem: KillingTheNameOf and psneuter

[here](http://c-skills.blogspot.com/2011/01/adb-trickery-again.html)
[here](https://github.com/tmzt/g2root-kmod/tree/scotty2/scotty2/psneuter)

### PowerVR: Ievitator

[here](http://jon.oberheide.org/files/levitator.c)

### Libsysutils: zergRush

[here](https://github.com/revolutionary/zergRush)

### Kernel: mempodroid

[here](https://github.com/saurik/mempoj

### File Permission and Symbolic Link-Related Attacks

[here](http://vulnfactory.org/blog/)

Initial versions of Android 4.0 had a bug in the init functions for do_chmod, mkdir, and do_chown
that applied the ownership and file permissions specified even if the last element of their target
path was a symbolic link. Some Android devices have the following line in their init.rc script.

`mkdir /data/local/tmp 0771 shell shell`

As you can guess now, if the /data/local folder is writeable by the user or group shell, you can
exploit this flaw to make the /data folder writeable by replacing /data/local/tmp with a symbolic
link to /data and rebooting the device. After rebooting, you can create or modify the /data/local.prop
file to set the property `ro.kernel.qemu` to 1.

The commands to exploit this flaw are as follows:

```
adb shell rm -r /data/local/tmp
adb shell ln -s /data/ /data/local/tmp
adb reboot
adb shell "echo 'ro.kernel.qemu=1' > /data/local.prop"
adb reboot
```

Another popular variant of this vulnerability links /data/local/tmp to the system partition and then
uses debugfs to write the su binary and make it set-uid root. For example, the ASUS Transformer Prime
running Android 4.0.3 is vulnerable to this variant.

The init scripts in Android 4.2 apply `O_NOFOLLOW` semantics to prevent this calss of symbolic link attacks.

### Adb Restore Race Condition

Android 4.0 introduced the ability to do full device backups through the adb backup command.
This command backs up all data and applications into the file backup.ab, which is a compressed
TAR file with a prepended header. The `adb restore` command is used to restore the data.

There were two security issues in the initial implementation of the restore process that were
fixed in Android 4.1.1. The first issue allowed creating files and directories accessible by other
applications. The second issue allowed restoring file sets from packages that run under a special UID,
such as system, without a special backup agent to handle the restore process.

To exploit these issues, Andreas Makris (Bin4ry) created a specially crafted backup file with a world
readble/writeable/executable directory containing 100 files with the content `ro.kernel.qemu=1` and 
`ro.secure=0` inside it. When the contents of this file are written to `/data/local.prop`, it makes
`adbd` run with root privieges on boot.

[here](http://forum.xda-developers.com/showthread.php?t=1886460)

The following one-liner, if executed while the `adb restore` command is running, causes a race between
the restore process in the backup manager service and the while loop run by the `shell` user:

```
adb shell "while ! ln -s /data/local.prop /data/data/com.android.settings/a/file99; do :; done"
```

If the loop creates the symbolic link in file99 before the restore process restores it, the restore
process follows the symbolic link and writes the read-only system properties to /data/local.prop, making
adbd run as root in the next reboot.

### Exynos4: exynos-abuse

* [here](http://forum.xda-developers.com/showthread.php?t=2048511)
* [Framaroot](http://blog.azimuthsecurity.com/2013/02/revisiting-exynos-memory-mapping-bug.html)

### Diag: lit / diaggetroot

This vulnerability was discovered by giantpune and was assigned CVE identifier CVE-2012-4220:

```
diagchar_core.c in the Qualcomm Innovation Center (QuIC) Diagnostics (aka DIAG) kernel-mode driver
for Android 2.3 through 4.2 allows attackers to execute arbitrary code or cause a 
denial of service (incorrect pointer dereference) via an application that uses crafted arguments in a
local diagchar_ioctl call.
```

[here](https://docs.google.com/file/d/oB8LDObFOpzZqQzducmxjRExXNnM/edit?pli=1)
