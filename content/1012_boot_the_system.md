Title: 101.2 Boot the System
Date: 2010-12-03 10:20
Category: LPIC1
Tags: System Architecture, LPIC1, LPIC1-101-500
Authors: Jadi
Summary: Candidates should be able to guide the system through the booting process.
Topic: System Architecture
sortorder: 040

*Weight: 3*

Candidates should be able to guide the system through the booting process.

## Key Knowledge Areas

* Provide common commands to the boot loader and options to the kernel at boot time
* Demonstrate knowledge of the boot sequence from BIOS/UEFI to boot completion
* Understanding of `sysVinit` and `systemd`
* Awareness of `Upstart`
* Check boot events in the log files

## Terms

* `dmesg`
* `journalctl`
* BIOS
* UEFI
* bootloader
* kernel
* `initramfs`
* `init`
* `sysVinit`
* `systemd`

<iframe width="560" height="315" src="https://www.youtube.com/embed/Zn_IGnNMHvc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## The Boot Process
It is important to understand because at this stage, you have very little control over the system and you can not issue commands to troubleshoot much. You should have a good understanding of what is happening.

1. Motherboard Firmware does a PowerOnSelfTest(POST)
2. Motherboard loads the bootloader
3. Bootloader loads the Linux Kernel-based on its configs/commands
4. The Kernel loads and prepares the system (root filesystem) and runs the initialization program
5. The `Init` program start the service such as web server, graphical interface, networking, etc.

As we discussed in the previous section (101.1), the Firmware on the motherboard can be BIOS or UEFI. 

### BIOS
Basic Input Output System

- Older
- Limited to one sector of the disk and needs a multi-stage bootloader
- Can start the bootloader from internal/external HDD, CD/DVD, USB Flash drive, Network server
- If booting from the HDD, the Master Boot Record will be used (1 sector)

### UEFI
Unified Extensible Firmware Interface. 

- Modern and fancy
- Specifies a special disk partition for the bootloader. Called EFI System Partition (ESP)
- ESP is FAT and mounted on `/boot/efi` and bootloader files has `.efi` extensions

> You can check `/sys/firmware/efi` to see if you are using a UEFI system or not

### Bootloader
Bootloader initializes the minimum hardware needed to boot the system. then, it locates and runs the OS.

Technically you can point your UEFI to run anything you want but typically under GNU/Linux systems, we use GRUB. GRUB can be used to run any specific program you need but it generally runs the OS. 

### Kernel

The Kernel is the core of your operating system. it basically is LINUX itself. Your bootloader loads the kernel in the memory and runs it. But the kernel needs some initial info to start; Things like drivers which are necessary to work with the hardware. Those are stored in `initrd` or `initramfs` alongside the kernel and used during the boot. 

You can also send parameters to the kernel during the boot using the Grub configs. For example, sending a `1` or `S` will result the system booting in single-user mode (recovery). Or you can force your graphics to work in 1024×768x24 mode by passing `vga=792` to the Kernel during the boot. 

---

<iframe width="560" height="315" src="https://www.youtube.com/embed/Sn20NKM0hbw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


#### `dmesg`

Linux will show you the boot process logs during the boot. Some desktop systems hide this behind a fancy splash screen which you can hide using the `Esc` key or press `Ctrl+Alt+F1`.

due to several reasons which are beyond the scope of this course, the kernel saves it's own logs into the "Kernel Ring Buffer". after the compilation of the boot process, the syslog daemon collects the *boot logs* and stores them in `/var/log/dmesg`.

to view all the logs including what has been logged after the boot process we use the `dmesg` command.

We can also use `journalctl -k` to check Kernel logs or use `journalctl -b` to check for boot logs (or even use `journalctl -b -n` to see all previous logs too).

In addition to these, most systems keep the boot logs in a text-like file too and they can be found in `/var/log/boot` or `/var/log/boot.log` in Debian or Red-Hat based systems, respectively.

#### `/var/log/messages`

After the init process comes up, `syslog` daemon will log messages. It has timestamps and will persist during restarts.

* The Kernel is still logging its messages in Kernel Buffer Ring
* in some systems, it might be called `/var/log/syslog`
* there are many other logs at `/var/log`

## init

When the Kernel initialization is finished, its time to start other programs. To do so, the Kernel runs the Initialization Daemon process, and it takes care of starting other daemons, services, subsystems and programs. Using the init system one can say "I need service A and then service B. Then I need C and D and E but do not start D unless the A and B are running". The system admin can use the init system to stop and start the services later.

There are different init systems:

- **`SysVinit`** is based on Unix System V. Not being used much anymore but people loved it because it followed Unix philosophies. you may see it on older machines or even on recently installed ones.
- **`upstart`** was an event-based replacement for the traditional init daemon developed by Canonical (The people behind Ubuntu). The goal of the project was to build a replacement for SysV when it got released in 2007. eventually, the project got discontinued due to the wide adoption of `Systemd`. Even Ubuntu uses `Systemd` these days, but upstart still can be found in google's ChromeOS.
- **`Systemd`** is the new replacement. It is hated by Linux elitists for not following Unix principles but it's widely adopted by major distros. It can start services in parallel and do lots of fancy stuff! 

The init process had the ID of 1 and you can find it by running the 

```
# which init
/sbin/init
# readlink -f /sbin/init
/usr/lib/systemd/systemd
# ps -p 1
PID TTY TIME     CMD
1   ?   00:00:06 systemd
```
 
You can check the hierarchy of processes using the `pstree` command.

```text
pstree
```

## `systemd`
Is new, loved, and hated. Lots of new ideas but not following some of the beloved UNIX principles (say.. not saving logs in a text file or trying to help you too much but asking for the root password when you are not running commands with sudo). It lets us run services if the hardware is connected, in time intervals, if another service is started, and ...

The systemd is made around **units**. A unit can be a service, group of services, or an action. Units do have a name, a type, and a configuration file. There are 12 unit types: `automount`, `device`, `mount`, `path`, `scope`, `service`, `slice`, `snapshot`, `socket`, `swap`, `target` & `timer`.

We use `systemctl` to work with these units and `journalctl` to see the logs.

```
# systemctl list-units
# systemctl list-units --type=target
# systemctl get-default # default target (groups of services are started via target unit files)
```

The units can be found in these places (sorted by priority):

1. `/etc/systemd/system/`
2. `/run/systemd/system/`
3. `/usr/lib/systemd/system`

```
# systemctl list-unit-files
# systemctl cat ntpd.service
# systemctl cat graphical.target
```

We can use these commands to work with services:

```
# systemctl stop sshd
# systemctl start sshd
# systemctl status sshd
# systemctl is-active sshd
# systemctl is-failed sshd
# systemctl restart sshd
# systemctl reload sshd # re-reads the configuration of the service configs
# systemctl daemon-reload sshd # re-reads the configuration of the systemd configs of this service
# systemctl enable sshd
# systemctl disable sshd
```

there are other commands too:

```
# systemctl is-system-running # running, degraded, maintenance, initializing, starting, stopping
# systemctl --failed
```

to check the logs, we have to use the `journalctl` utility:

```
# journalctl # show all journal
# journalctl --no-pager # do not use less
# journalctl -n 10 # only 10 lines
# journalctl -S -1d # last 1 day
# journalctl -xe # last few logs
# journalctl -u ntp # only ntp unit
# journalctl _PID=1234
```

## `SysV`
Is the older init system. Still can be used on many systems. The control files are located at `/etc/init.d/` and are closer to the general bash scripts. In many cases you can call like:

```
/etc/init.d/ntpd status
/etc/init.d/ntpd stop
/etc/init.d/ntpd start
/etc/init.d/ntpd restart
```

We will speak more about runlevels on 101.3.
