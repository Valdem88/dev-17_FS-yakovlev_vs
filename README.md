# Домашнее задание к занятости "3.5. Файловые системы" - yakovlev_vs

#### 1. Узнайте о `sparse` (разряженных) файлах.

Решение 

Изучил. Было полезно.

Разрежённый файл (англ. sparse file) — файл, в котором последовательности нулевых байтов[1] заменены на информацию об этих последовательностях (список дыр).

Дыра (англ. hole) — последовательность нулевых байт внутри файла, не записанная на диск. Информация о дырах (смещение от начала файла в байтах и количество байт) хранится в метаданных ФС.
https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB

#### 2. Возможны ли файлы, являющиеся жесткой ссылкой на один объект, имеют разные права доступа и владельца? Почему?

Решение 

Не могут. Жесткая ссылка`(hardlink)` по факту является зеркальной копией объекта. Наследующая его права, владелец и группа. 
Имеет тот же inode, что и оригинальный файл.
```bash
root@vagrant:~# touch hardlink_fle
root@vagrant:~# ln hardlink_fle link
root@vagrant:~# ls -l
total 4
-rw-r--r-- 2 root root    0 May 28 11:05 hardlink_fle
-rw-r--r-- 2 root root    0 May 28 11:05 link
drwxr-xr-x 3 root root 4096 Dec 19 19:42 snap
root@vagrant:~# chmod 0755 hardlink_fle
root@vagrant:~# ls -ilh
total 4.0K
1703954 -rwxr-xr-x 2 root root    0 May 28 11:05 hardlink_fle
1703954 -rwxr-xr-x 2 root root    0 May 28 11:05 link
1703942 drwxr-xr-x 3 root root 4.0K Dec 19 19:42 snap
root@vagrant:~# chown vagrant:vagrant hardlink_fle
root@vagrant:~# ls -ilh
total 4.0K
1703954 -rwxr-xr-x 2 vagrant vagrant    0 May 28 11:05 hardlink_fle
1703954 -rwxr-xr-x 2 vagrant vagrant    0 May 28 11:05 link
1703942 drwxr-xr-x 3 root    root    4.0K Dec 19 19:42 snap
```
#### 3. Сделайте `vagrant destroy` на существующем инстанс Ubuntu. Заменить содержимое Vagrantfile
```bash
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.04"
  config.vm.provider :virtualbox do |vb|
    lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
    lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
    vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
    vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
    vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
    vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
  end
end
```
Решение 

Этот vagrantfile создает новую виртуальную машину с видимыми неразмеченными дисками по 2,5 Гб.

Виртуальная машина создана. Проверяем диски.
```bash
root@vagrant:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 55.5M  1 loop /snap/core18/2409
loop1                       7:1    0 55.4M  1 loop /snap/core18/2128
loop2                       7:2    0 61.9M  1 loop /snap/core20/1434
loop3                       7:3    0 70.3M  1 loop /snap/lxd/21029
loop4                       7:4    0 44.7M  1 loop /snap/snapd/15904
loop5                       7:5    0 67.8M  1 loop /snap/lxd/22753
loop6                       7:6    0 61.9M  1 loop /snap/core20/1494
sda                         8:0    0   64G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part /boot
└─sda3                      8:3    0   63G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm  /
sdb                         8:16   0  2.5G  0 disk
sdc                         8:32   0  2.5G  0 disk
root@vagrant:~#
```
#### 4. номер, разбейте fdiskпервый диск на 2 раздела: 2 Гб, оставшееся пространство.

Решение 
```bash
root@vagrant:~# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x5dbecad0.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (1-4, default 1):
First sector (2048-5242879, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242879, default 5242879): +2g

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (2-4, default 2):
First sector (4196352-5242879, default 4196352):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879):

Created a new partition 2 of type 'Linux' and of size 511 MiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
Проверяем
```bash
root@vagrant:~# fdisk -l
Disk /dev/sdb: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x5dbecad0

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdb1          2048 4196351 4194304    2G 83 Linux
/dev/sdb2       4196352 5242879 1046528  511M 83 Linux


Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/ubuntu--vg-ubuntu--lv: 31.51 GiB, 33822867456 bytes, 66060288 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```
#### 5. Используя sfdisk, перенесите данную таблицу разделов на второй диск.

Решение
```bash
root@vagrant:~# sfdisk -d /dev/sdb > part_table
root@vagrant:~# sfdisk /dev/sdc < part_table
Checking that no-one is using this disk right now ... OK

Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Created a new DOS disklabel with disk identifier 0x5dbecad0.
/dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.
/dev/sdc2: Created a new partition 2 of type 'Linux' and of size 511 MiB.
/dev/sdc3: Done.

New situation:
Disklabel type: dos
Disk identifier: 0x5dbecad0

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1          2048 4196351 4194304    2G 83 Linux
/dev/sdc2       4196352 5242879 1046528  511M 83 Linux

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
#### 6. Соберите mdadm RAID1 на паре разделов 2 Гб.

Решение
```bash
root@vagrant:~# mdadm --create --verbose /dev/md0 -l 1 -n 2 /dev/sd{b1,c1}
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 2094080K
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```
#### 7. Соберите mdadm RAID0 на второй паре маленьких разделов.

Решение
```bash
root@vagrant:~# mdadm --create --verbose /dev/md1 -l 0 -n 2 /dev/sd{b2,c2}
mdadm: chunk size defaults to 512K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
```
Проверка
```bash
root@vagrant:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 55.5M  1 loop  /snap/core18/2409
loop1                       7:1    0 55.4M  1 loop  /snap/core18/2128
loop2                       7:2    0 61.9M  1 loop  /snap/core20/1494
loop3                       7:3    0 61.9M  1 loop  /snap/core20/1434
loop4                       7:4    0 67.8M  1 loop  /snap/lxd/22753
loop5                       7:5    0 44.7M  1 loop  /snap/snapd/15904
loop6                       7:6    0 70.3M  1 loop  /snap/lxd/21029
sda                         8:0    0   64G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part  /boot
└─sda3                      8:3    0   63G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk
├─sdb1                      8:17   0    2G  0 part
│ └─md0                     9:0    0    2G  0 raid1
└─sdb2                      8:18   0  511M  0 part
  └─md1                     9:1    0 1018M  0 raid0
sdc                         8:32   0  2.5G  0 disk
├─sdc1                      8:33   0    2G  0 part
│ └─md0                     9:0    0    2G  0 raid1
└─sdc2                      8:34   0  511M  0 part
  └─md1                     9:1    0 1018M  0 raid0
```
#### 8. Создайте 2 независимых PV на получившихся md-устройствах.

Решение
```bash
root@vagrant:~# pvcreate /dev/md0 /dev/md1
  Physical volume "/dev/md0" successfully created.
  Physical volume "/dev/md1" successfully created.
```
#### 9. Создайте общую volume-group на этих двух PV.

Решение
```bash
root@vagrant:~# vgcreate vol_group1 /dev/md0 /dev/md1
  Volume group "vol_group1" successfully created
```
Проверка
```bash
root@vagrant:~# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               ubuntu-vg
  PV Size               <63.00 GiB / not usable 0
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              16127
  Free PE               8063
  Allocated PE          8064
  PV UUID               sDUvKe-EtCc-gKuY-ZXTD-1B1d-eh9Q-XldxLf

  --- Physical volume ---
  PV Name               /dev/md0
  VG Name               vol_group1
  PV Size               <2.00 GiB / not usable 0
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              511
  Free PE               511
  Allocated PE          0
  PV UUID               xuggTE-mqPx-u58q-g3sb-hvII-9pww-7xClhH

  --- Physical volume ---
  PV Name               /dev/md1
  VG Name               vol_group1
  PV Size               1018.00 MiB / not usable 2.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              254
  Free PE               254
  Allocated PE          0
  PV UUID               LTyKKO-9kxc-55vX-6QLG-4XE2-T12X-umMQZ5
```
#### 10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.

Решение
```bash
root@vagrant:~# lvcreate -L 100M -n lv1 vol_group1 /dev/md1
  Logical volume "lv1" created.
root@vagrant:~# lvdisplay
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/ubuntu-lv
  LV Name                ubuntu-lv
  VG Name                ubuntu-vg
  LV UUID                ftN15m-3lML-YH5x-R5P2-kLCd-kzW3-32dlqO
  LV Write Access        read/write
  LV Creation host, time ubuntu-server, 2021-12-19 19:37:44 +0000
  LV Status              available
  # open                 1
  LV Size                31.50 GiB
  Current LE             8064
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0

  --- Logical volume ---
  LV Path                /dev/vol_group1/lv1
  LV Name                lv1
  VG Name                vol_group1
  LV UUID                e5TB91-vVsA-R4ZN-EApN-bJ6z-nI6h-xXiivP
  LV Write Access        read/write
  LV Creation host, time vagrant, 2022-05-29 16:38:45 +0000
  LV Status              available
  # open                 0
  LV Size                100.00 MiB
  Current LE             25
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     4096
  Block device           253:1
```
#### 11. Создайте mkfs.ext4 ФС на получившемся LV.

Решение
```bash
root@vagrant:~# mkfs.ext4 /dev/vol_group1/lv1
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done

root@vagrant:~# lsblk -o NAME,PATH,SIZE,FSTYPE
NAME                      PATH                               SIZE FSTYPE
loop0                     /dev/loop0                        55.5M squashfs
loop1                     /dev/loop1                        55.4M squashfs
loop2                     /dev/loop2                        61.9M squashfs
loop3                     /dev/loop3                        61.9M squashfs
loop4                     /dev/loop4                        67.8M squashfs
loop5                     /dev/loop5                        44.7M squashfs
loop6                     /dev/loop6                        70.3M squashfs
sda                       /dev/sda                            64G
├─sda1                    /dev/sda1                            1M
├─sda2                    /dev/sda2                            1G ext4
└─sda3                    /dev/sda3                           63G LVM2_member
  └─ubuntu--vg-ubuntu--lv /dev/mapper/ubuntu--vg-ubuntu--lv 31.5G ext4
sdb                       /dev/sdb                           2.5G
├─sdb1                    /dev/sdb1                            2G linux_raid_member
│ └─md0                   /dev/md0                             2G LVM2_member
└─sdb2                    /dev/sdb2                          511M linux_raid_member
  └─md1                   /dev/md1                          1018M LVM2_member
    └─vol_group1-lv1      /dev/mapper/vol_group1-lv1         100M ext4
sdc                       /dev/sdc                           2.5G
├─sdc1                    /dev/sdc1                            2G linux_raid_member
│ └─md0                   /dev/md0                             2G LVM2_member
└─sdc2                    /dev/sdc2                          511M linux_raid_member
  └─md1                   /dev/md1                          1018M LVM2_member
    └─vol_group1-lv1      /dev/mapper/vol_group1-lv1         100M ext4
```
#### 12. Смонтируйте этот раздел в любую директорию, например, /tmp/new.

Решение
```bash
root@vagrant:~# mkdir /tmp/lv1
root@vagrant:~# mount /dev/vol_group1/lv1 /tmp/lv1/
```
#### 13. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.
Решение
```bash
root@vagrant:~# wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/lv1/test.gz
--2022-05-29 16:54:19--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 23318896 (22M) [application/octet-stream]
Saving to: ‘/tmp/lv1/test.gz’

/tmp/lv1/test.gz             100%[==============================================>]  22.24M  9.99MB/s    in 2.2s

2022-05-29 16:54:22 (9.99 MB/s) - ‘/tmp/lv1/test.gz’ saved [23318896/23318896]

root@vagrant:~# ls -l /tmp/lv1/
total 22792
drwx------ 2 root root    16384 May 29 16:42 lost+found
-rw-r--r-- 1 root root 23318896 May 29 14:42 test.gz
```
#### 14. Прикрепите вывод lsblk.

Решение 
```bash
root@vagrant:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 55.5M  1 loop  /snap/core18/2409
loop1                       7:1    0 55.4M  1 loop  /snap/core18/2128
loop2                       7:2    0 61.9M  1 loop  /snap/core20/1494
loop3                       7:3    0 61.9M  1 loop  /snap/core20/1434
loop4                       7:4    0 67.8M  1 loop  /snap/lxd/22753
loop5                       7:5    0 44.7M  1 loop  /snap/snapd/15904
loop6                       7:6    0 70.3M  1 loop  /snap/lxd/21029
sda                         8:0    0   64G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part  /boot
└─sda3                      8:3    0   63G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk
├─sdb1                      8:17   0    2G  0 part
│ └─md0                     9:0    0    2G  0 raid1
└─sdb2                      8:18   0  511M  0 part
  └─md1                     9:1    0 1018M  0 raid0
    └─vol_group1-lv1      253:1    0  100M  0 lvm   /tmp/lv1
sdc                         8:32   0  2.5G  0 disk
├─sdc1                      8:33   0    2G  0 part
│ └─md0                     9:0    0    2G  0 raid1
└─sdc2                      8:34   0  511M  0 part
  └─md1                     9:1    0 1018M  0 raid0
    └─vol_group1-lv1      253:1    0  100M  0 lvm   /tmp/lv1
```
#### 15. Протестируйте целостность файла:
```bash
root@vagrant:~# gzip -t /tmp/new/test.gz
root@vagrant:~# echo $?
0
```
Решение
```bash
root@vagrant:~# gzip -t /tmp/lv1/test.gz
root@vagrant:~# echo $?
0
```
#### 16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.

Решение
```bash
root@vagrant:~# pvmove -n /dev/vol_group1/lv1 /dev/md1 /dev/md0
  /dev/md1: Moved: 12.00%
  /dev/md1: Moved: 100.00%
root@vagrant:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 55.5M  1 loop  /snap/core18/2409
loop1                       7:1    0 55.4M  1 loop  /snap/core18/2128
loop2                       7:2    0 61.9M  1 loop  /snap/core20/1494
loop3                       7:3    0 61.9M  1 loop  /snap/core20/1434
loop4                       7:4    0 67.8M  1 loop  /snap/lxd/22753
loop5                       7:5    0 44.7M  1 loop  /snap/snapd/15904
loop6                       7:6    0 70.3M  1 loop  /snap/lxd/21029
sda                         8:0    0   64G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part  /boot
└─sda3                      8:3    0   63G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk
├─sdb1                      8:17   0    2G  0 part
│ └─md0                     9:0    0    2G  0 raid1
│   └─vol_group1-lv1      253:1    0  100M  0 lvm   /tmp/lv1
└─sdb2                      8:18   0  511M  0 part
  └─md1                     9:1    0 1018M  0 raid0
sdc                         8:32   0  2.5G  0 disk
├─sdc1                      8:33   0    2G  0 part
│ └─md0                     9:0    0    2G  0 raid1
│   └─vol_group1-lv1      253:1    0  100M  0 lvm   /tmp/lv1
└─sdc2                      8:34   0  511M  0 part
  └─md1                     9:1    0 1018M  0 raid0
```
#### 17. Сделайте `--fail` на устройство в вашем RAID1 md.

Решение 
```bash
mdadm /dev/md0 -f /dev/sdc1
mdadm: set /dev/sdc1 faulty in /dev/md0
root@vagrant:~# mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sun May 29 16:15:03 2022
        Raid Level : raid1
        Array Size : 2094080 (2045.00 MiB 2144.34 MB)
     Used Dev Size : 2094080 (2045.00 MiB 2144.34 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Sun May 29 17:10:10 2022
             State : clean, degraded
    Active Devices : 1
   Working Devices : 1
    Failed Devices : 1
     Spare Devices : 0

Consistency Policy : resync

              Name : vagrant:0  (local to host vagrant)
              UUID : 20c1cd34:183fb399:bdc5b8ed:f58eb597
            Events : 19

    Number   Major   Minor   RaidDevice State
       0       8       17        0      active sync   /dev/sdb1
       -       0        0        1      removed

       1       8       33        -      faulty   /dev/sdc1
```
#### 18. Подтвердите выводом dmesg, что RAID1 работает в деградированном состоянии.
Решение
```bash
root@vagrant:~# dmesg | grep 'md/raid1:md0'
[ 5629.415974] md/raid1:md0: not clean -- starting background reconstruction
[ 5629.415975] md/raid1:md0: active with 2 out of 2 mirrors
[ 8934.061329] md/raid1:md0: Disk failure on sdc1, disabling device.
               md/raid1:md0: Operation continuing on 1 devices.
```
#### 19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:
```bash
root@vagrant:~# gzip -t /tmp/new/test.gz
root@vagrant:~# echo $?
0
```
Решение
```bash
root@vagrant:~# gzip -t /tmp/lv1/test.gz
root@vagrant:~# echo $?
0
```
#### 20. Погасите тестовый хост, `vagrant destroy`.

Решение
```bash
D:\virtual machine>vagrant destroy
   default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Destroying VM and associated drives...
```

