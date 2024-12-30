# Debian install with disk encryption, BTRFS, timeshift and ZRAM swap

## Sources
- [https://medium.com/@inatagan/installing-debian-with-btrfs-snapper-backups-and-grub-btrfs-27212644175f](https://medium.com/@inatagan/installing-debian-with-btrfs-snapper-backups-and-grub-btrfs-27212644175f)
- [https://www.cyberciti.biz/security/how-to-change-luks-disk-encryption-passphrase-in-linux/](https://www.cyberciti.biz/security/how-to-change-luks-disk-encryption-passphrase-in-linux/)
- [https://www.youtube.com/watch?v=PQcIThqCXes](https://www.youtube.com/watch?v=PQcIThqCXes)
- [https://www.youtube.com/watch?v=RiBRw613mDI](https://www.youtube.com/watch?v=RiBRw613mDI)
- [https://youtu.be/_i_InuWyfQE](https://youtu.be/_i_InuWyfQE)
- [https://www.lorenzobettini.it/2022/10/timeshift-and-grub-btrfs-in-ubuntu/](https://www.lorenzobettini.it/2022/10/timeshift-and-grub-btrfs-in-ubuntu/)
- [https://github.com/Antynea/grub-btrfs](https://github.com/Antynea/grub-btrfs)
- [https://github.com/wmutschl/timeshift-autosnap-apt](https://github.com/wmutschl/timeshift-autosnap-apt)
- [https://mutschler.dev/linux/timeshift/](https://mutschler.dev/linux/timeshift/)
- [https://btrfs.readthedocs.io/en/latest/ch-mount-options.html](https://btrfs.readthedocs.io/en/latest/ch-mount-options.html)

## Debian installer part one
* Boot from debian netinst choose advanced options, expert install from the install menu.
  * if you get a grey screen inside a VM installation: add fb=false to the grub boot entry so it looks something like this: linux /install.amd/vmlinuz vga=788 fb=false --- quiet
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
* set root password, create normal user account, set user information and password
* Configure the clock, use NTP, ntp server: 0.debian.pool.ntp.org
* Select timezone: Europe/Brussels
* Detect disks
* Partition disks: manual, select the disk to use
* Create new partition table: yes, gpt
* Select free space, create new partition, size 1GB, beginning, use as "EFI System partition", Done setting up the partition
* Select free space, create new partition, size 1GB, beginning, use as "Ext4 journaling file system" and mount point "/boot", Done setting up the partition
* Select free space, create new partition, size maximum, use as "physical volume for encryption", Done setting up the partition
* Select "Configure encrypted volumes", select the device with "crypto" when asked for Devices to encrypt, Finnish, Erase data, Cancel to save time, set the encryption passphrase
* Back in the Partition disks window, select filesystem under the encrypted volume (should have filesystem ext4), change use ase to "btrfs journaling file system" and mount point to "/", Done setting up the partition
* Finish partitioning and write changes to disk, do not return to partitioning menu to setup swap because we will do this later on
* DO NOT install base system!

## BTRFS setup after partitioning
* Switch to a console using CTRL-ALT-F2 (ALT-F2 on a VMWare remote console or a Virtualbox VM, CTRL-ALT-FN-F2 on a Dell laptop)
``` bash
df -h
# remember which device was used for /target and substitue sdaX with this device in the following commands (probably sda3_crypt or nvme0n1p3_crypt)
# remember which device was used for /target/boot and substitue sdaY with this device in the following commands (probably sda2 or nvme0n1p2)
# remember which device was used for /target/boot/efi and substitue sdaZ with this device in the following commands (probably sda1 or nvme0n1p1)
umount /target/boot/efi
umount /target/boot
umount /target
# probably /dev/mapper/sda3_crypt or nvme0n1p3_crypt
mount /dev/mapper/sdaX /mnt/
cd /mnt
mv @rootfs/ @
btrfs subvolume create @home
btrfs subvolume create @log
btrfs subvolume create @cache
btrfs subvolume create @crash
btrfs subvolume create @tmp
btrfs subvolume create @spool
# probably /dev/mapper/sda3_crypt or nvme0n1p3_crypt
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@ /dev/mapper/sdaX /target 
cd /target
mkdir -p boot/efi
mkdir -p home
mkdir -p var/log
mkdir -p var/cache
mkdir -p var/crash
mkdir -p var/tmp
mkdir -p var/spool
# probably /dev/mapper/sda3_crypt or nvme0n1p3_crypt
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@home /dev/mapper/sdaX /target/home
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@log /dev/mapper/sdaX /target/var/log
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@cache /dev/mapper/sdaX /target/var/cache
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@crash /dev/mapper/sdaX /target/var/crash
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@tmp /dev/mapper/sdaX /target/var/tmp
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@spool /dev/mapper/sdaX /target/var/spool
# probably /dev/sda2 or nvme0n1p2
mount /dev/sdaY /target/boot
# probably /dev/sda1 or nvme0n1p1
mount /dev/sdaZ /target/boot/efi
nano /target/etc/fstab
```

```
UUID=blah1 /            btrfs	noatime,compress=zstd,ssd,discard=async,subvol=@      0 0
UUID=blah1 /home        btrfs	noatime,compress=zstd,ssd,discard=async,subvol=@home  0 0
UUID=blah1 /var/log     btrfs	noatime,compress=zstd,ssd,discard=async,subvol=@log   0 0
UUID=blah1 /var/cache   btrfs	noatime,compress=zstd,ssd,discard=async,subvol=@cache 0 0
UUID=blah1 /var/crash   btrfs	noatime,compress=zstd,ssd,discard=async,subvol=@crash 0 0
UUID=blah1 /var/tmp     btrfs	noatime,compress=zstd,ssd,discard=async,subvol=@tmp   0 0
UUID=blah1 /var/spool   btrfs	noatime,compress=zstd,ssd,discard=async,subvol=@spool 0 0
UUID=blah2 /boot        ext4	defaults    0 2
UUID=blah3 /boot/efi    vfat	umask=0077  0 1
```

```
cd /
unmount /mnt
exit
```
Switch back to the install console with CTRL-ALT-F1 (ALT-F1 on a VMWare remote console or a Virtualbox VM, CTRL-ALT-FN-F2 on a Dell laptop)
## Installing base system and completing the setup
* Install the base system
* Kernel to install: choose the default option
* Drivers to include in the initrd: generic
* Configure the package manager
* Scan extra instalation media: no
* Use a network mirror: yes, http, Belgium, deb.debian.org, no HTTP proxy
* Use non-free firmware: yes
* Use non-free software: yes
* Enable source repositories: no
* Services to use: security updates and release updates, no backported software
* Select and install software
* Update management on this system: no automatic updates
* Participate in package usage survey: no
* Choose software to install: KDE Plasma, SSH server and standard system utilities
* Install the GRUB loader, force GRUB installation to the EFI removable media path: no, update NVRAM variables: yes
* Run os-prober automatically: no
* Finish the installation
* Is the system clock set to UTC: yes

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

## Testing and changing the disk encryption key
#### Check your setup
```
cat /etc/crypttab
```
In my case, this tells me I have an sda3_crypt luks device.

#### Check the header of your device
```
cryptsetup luksDump /dev/sda3
```
It seems I only have slot 0, but on many systems, you may see up to 8 slots numbered from 0 to 7.

#### Determine the correct slot
This command will tell you the correct LUKS slot without any guesswork on your part:
```
cryptsetup --verbose open --test-passphrase /dev/sda3
```
Take not of the key slot in the output.
#### Change the encryption key
To change the encryption key for slot 0 in a luks encrypted device sda3:
```
cryptsetup luksChangeKey /dev/sda3 -S 0
```
