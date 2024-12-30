# Debian install with BTRFS, timeshift and ZRAM swap

## Sources
- [https://www.youtube.com/watch?v=PQcIThqCXes]()
- [https://www.youtube.com/watch?v=RiBRw613mDI]()
- [https://youtu.be/_i_InuWyfQE]()
- [https://www.lorenzobettini.it/2022/10/timeshift-and-grub-btrfs-in-ubuntu/]()
- [https://github.com/Antynea/grub-btrfs]()
- [https://github.com/wmutschl/timeshift-autosnap-apt]()
- [https://mutschler.dev/linux/timeshift/]()
- [https://btrfs.readthedocs.io/en/latest/ch-mount-options.html]()


## Debian installer part one
* Boot from debian netinst choose advanced options, expert install from the install menu.
* Choose language: English
* Country: other, Europe, Belgium
* Default locale: en_US.UTF-8
* Additional locales: none
* Configure the keyboard, keymap: Belgian
* Detect and mount installation media
* Load installer components from installation media
* Detect network hardware
* Configure the network, autoconfigure: yes
* Hostname: <redacted>
* Domain: <redacted>
* Set up users and passwords, enable shadow passwords: yes, allow login as root: yes
* Add user information
* Configure the clock, use NTP, ntp server: ntp.telenet.be
* Select timezone: Europe/Brussels
* Detect disks
* Partition disks: manual, select the disk to use
* Create new partition table: yes, gpt
* Select free space, create new partition, size 1GB, beginning, use as "EFI System partition", Done
* Select free space, create new partition, size maximum, use as "btrfs journaling file system", Done
* Finish partitioning and write changes to disk, do not return to partitioning menu to setup swap because we will do this later on
* DO NOT install base system!

## BTRFS setup after partitioning
* Switch to a console using CTRL-ALT-F2 (ALT-F2 on a VMWare remote console, CTRL-ALT-FN-F2 on a Dell laptop)
``` bash
df -h
# remember which device was used for /target and substitue sdaX with this device in the following commands (probably sda2 or nvme0n1p2)
# remember which device was used for /target/boot/efi and substitue sdaY with this device in the following commands (probably sda1 or nvme0n1p1)
umount /target/boot/efi
umount /target
mount /dev/sdaX /mnt/ # probably /dev/sda2 or nvme0n1p2
cd /mnt
mv @rootfs/ @
btrfs subvolume create @home
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@ /dev/sdaX /target
mkdir -p /target/boot/efi
mkdir /target/home
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@home /dev/sdaX /target/home
mount /dev/sdaY /target/boot/efi # probably /dev/sda1 or nvme0n1p1
nano /target/etc/fstab
```

```
UUID=blahdieblahdieblah		/			btrfs	noatime,compress=zstd,ssd,discard=async,subvol=@		0	0
UUID=blahdieblahdieblah		/home		btrfs	noatime,compress=zstd,ssd,discard=async,subvol=@home	0	0
UUID=otherblaaaaaah			/boot/efi	vfat	umask=0077															0	1
```
Switch back to the install console with CTRL-ALT-F1 (ALT-F1 on a VMWare remote console, CTRL-ALT-FN-F2 on a Dell laptop)
## Installing base system and completing the setup
* Install the base system
* Configure the package manager
* Scan extra instalation media: no
* Use a network mirror: yes, http, Belgium, deb.debian.org, no HTTP proxy
* Use non-free firmware: yes
* Use non-free software: yes
* Enable source repositories: no
* Services to use: security updates and release updates, no backported software
* Select and install software
* Update management on this system: no automatic updates
* Choose software to install: KDE Plasma, SSH server and standard system utilities
* Install the GRUB loader, force GRUB installation to the EFI removable media path: no
* Finish the installation

## Enabling ZRAM swap
``` bash
apt install zram-tools
vi /etc/default/zramswap
```

```
ALGO=zstd
PERCENT=25
```

``` bash
lsblk
reboot
lsblk
```

## Timeshift install and usage
``` bash
apt install timeshift
timeshift --btrfs

vi /etc/timeshift/timeshift.json
```

``` json
{
  "backup_device_uuid" : "b3c1b989-9ae1-407d-9ca1-646f54ae8054",
  "parent_device_uuid" : "",
  "do_first_run" : "false",
  "btrfs_mode" : "true",
  "include_btrfs_home_for_backup" : "false",
  "include_btrfs_home_for_restore" : "false",
  "stop_cron_emails" : "true",
  "schedule_monthly" : "false",
  "schedule_weekly" : "false",
  "schedule_daily" : "true",
  "schedule_hourly" : "false",
  "schedule_boot" : "false",
  "count_monthly" : "2",
  "count_weekly" : "3",
  "count_daily" : "3",
  "count_hourly" : "6",
  "count_boot" : "5",
  "snapshot_size" : "0",
  "snapshot_count" : "0",
  "date_format" : "%Y-%m-%d %H:%M:%S",
  "exclude" : [],
  "exclude-apps" : []
}
```

``` bash
timeshift --list
timeshift --create --comment "base install including timeshift"
timeshift --list
timeshift --restore # select 0, continue: yes
reboot
```

## Installing grub-btrfs
``` bash
apt install inotify-tools make git
git clone https://github.com/Antynea/grub-btrfs.git ~/grub-btrfs
cd ~/grub-btrfs
make install
update-grub
```

## Installing timeshift-autosnap-apt
``` bash
git clone https://github.com/wmutschl/timeshift-autosnap-apt.git ~/timeshift-autosnap-apt
cd ~/timeshift-autosnap-apt
make install
vi /etc/timeshift-autosnap-apt.conf
```

```
# /etc/timeshift-autosnap.conf
#

# snapshotBoot defines if /boot folder should be cloned into /boot.backup before the call to timeshift.
# Default value is true.
snapshotBoot=true

# snapshotEFI defines if /boot/efi folder should be cloned into /boot.backup/efi before the call to timeshift.
# Default value is true.
snapshotEFI=true

# skipAutosnap defines if timeshift-autosnap execution should be skipped.
# Default value is false.
skipAutosnap=false

# deleteSnapshots defines if old snapshots should be deleted.
# Default value is true.
deleteSnapshots=true

# maxSnapshots defines how much old snapshots script should left.
# Only positive whole numbers can be used.
# Default value is 3.
maxSnapshots=3

# updateGrub defines if grub entries should be auto-generated.
# If grub-btrfs package is not installed grub won't be generated.
# Default value is true.
updateGrub=true

# snapshotDescription defines value used to distinguish snapshots created using timeshift-autosnap
# Default value is "{timeshift-autosnap} {created before upgrade}".
snapshotDescription={timeshift-autosnap-apt}
```
