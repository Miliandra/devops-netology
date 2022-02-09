1. Прочитали о sparse (разряженных) файлах.
2. Файлы, являющиеся жесткой ссылкой на один объект, не могут иметь разные права доступа и владельца, так как имеют один и тот же inode.
3. Сделали `vagrant destroy` на имеющийся ВМ Ubuntu. Замените содержимое Vagrantfile и запустили ВМ, проверили наличие 2 дисков по 2.5GB:
```
vagrant@vagrant:~$ vagrant@vagrant:~$ lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
sdb                    8:16   0  2.5G  0 disk
sdc                    8:32   0  2.5G  0 disk
```
4. Для создания разделов использовали команду `fdisk /dev/sdb`. И разбили диск на 2 раздела
```
Device     Boot   Start     End Sectors  Size Id Type
/dev/sdb1          2048 4196351 4194304    2G 83 Linux
/dev/sdb2       4196352 5242879 1046528  511M 83 Linux
```
Проверим командой `lsblk`:
```
root@vagrant:~# lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
sdb                    8:16   0  2.5G  0 disk
├─sdb1                 8:17   0    2G  0 part
└─sdb2                 8:18   0  511M  0 part
sdc                    8:32   0  2.5G  0 disk

```
5. Перенос осуществляем командой `sfdisk -d /dev/sdb | sfdisk /dev/sdc`:
```
Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1          2048 4196351 4194304    2G 83 Linux
/dev/sdc2       4196352 5242879 1046528  511M 83 Linux
```
Вывод `lsblk`:
```
root@vagrant:~# lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
sdb                    8:16   0  2.5G  0 disk
├─sdb1                 8:17   0    2G  0 part
└─sdb2                 8:18   0  511M  0 part
sdc                    8:32   0  2.5G  0 disk
├─sdc1                 8:33   0    2G  0 part
└─sdc2                 8:34   0  511M  0 part
```
6. Создание `RAID1` командой `mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1`
```
root@vagrant:~# cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid1 sdc1[1] sdb1[0]
      2094080 blocks super 1.2 [2/2] [UU]
```
7. Создание `RAID0` командой `mdadm --create --verbose /dev/md1 --level=0 --raid-devices=2 /dev/sdb2 /dev/sdc2`
```
root@vagrant:~# cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md1 : active raid0 sdc2[1] sdb2[0]
      1042432 blocks super 1.2 512k chunks

md0 : active raid1 sdc1[1] sdb1[0]
      2094080 blocks super 1.2 [2/2] [UU]
```
После выполнения 6 и 7 пункта вывод команды `lsblk`:
```
root@vagrant:~# lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part  /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
sdb                    8:16   0  2.5G  0 disk
├─sdb1                 8:17   0    2G  0 part
│ └─md0                9:0    0    2G  0 raid1
└─sdb2                 8:18   0  511M  0 part
  └─md1                9:1    0 1018M  0 raid0
sdc                    8:32   0  2.5G  0 disk
├─sdc1                 8:33   0    2G  0 part
│ └─md0                9:0    0    2G  0 raid1
└─sdc2                 8:34   0  511M  0 part
  └─md1                9:1    0 1018M  0 raid0
```
8. Создали 2 независимых PV на получившихся md-устройствах:
```
root@vagrant:~# pvcreate /dev/md0
  Physical volume "/dev/md0" successfully created.
root@vagrant:~# pvcreate /dev/md1
  Physical volume "/dev/md1" successfully created.
```
9. Создали общую `volume_group` командой `vgcreate vg_new /dev/md0 /dev/md1`:
```
root@vagrant:~# vgcreate vg_new /dev/md0 /dev/md1
  Volume group "vg_new" successfully created
root@vagrant:~# vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  vg_new      2   0   0 wz--n-  <2.99g <2.99g
  vgvagrant   1   2   0 wz--n- <63.50g     0
```
10. Создали LV `lvcreate -n lv0 -L100 vg_new /dev/md1`:
```
root@vagrant:~# lvs
  LV     VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv0    vg_new    -wi-a----- 100.00m
  root   vgvagrant -wi-ao---- <62.54g
  swap_1 vgvagrant -wi-ao---- 980.00m
  
  root@vagrant:~# lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part  /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
sdb                    8:16   0  2.5G  0 disk
├─sdb1                 8:17   0    2G  0 part
│ └─md0                9:0    0    2G  0 raid1
└─sdb2                 8:18   0  511M  0 part
  └─md1                9:1    0 1018M  0 raid0
    └─vg_new-lv0     253:2    0  100M  0 lvm
sdc                    8:32   0  2.5G  0 disk
├─sdc1                 8:33   0    2G  0 part
│ └─md0                9:0    0    2G  0 raid1
└─sdc2                 8:34   0  511M  0 part
  └─md1                9:1    0 1018M  0 raid0
    └─vg_new-lv0     253:2    0  100M  0 lvm
```
11. Создали `mkfs.ext4` ФС на получившемся LV:
```
root@vagrant:~# mkfs.ext4 /dev/vg_new/lv0
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done

root@vagrant:/tmp# df -Th
Filesystem                 Type      Size  Used Avail Use% Mounted on
udev                       devtmpfs  447M     0  447M   0% /dev
tmpfs                      tmpfs      99M  708K   98M   1% /run
/dev/mapper/vgvagrant-root ext4       62G  1.6G   57G   3% /
tmpfs                      tmpfs     491M     0  491M   0% /dev/shm
tmpfs                      tmpfs     5.0M     0  5.0M   0% /run/lock
tmpfs                      tmpfs     491M     0  491M   0% /sys/fs/cgroup
/dev/sda1                  vfat      511M  4.0K  511M   1% /boot/efi
tmpfs                      tmpfs      99M     0   99M   0% /run/user/0
/dev/mapper/vg_new-lv0     ext4       93M   72K   86M   1% /tmp/new
```
12. Смонтировали наш раздел в директорию `mount /dev/vg_new/lv0 /tmp/new`:
13. Поместили тестовый файл `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`:
```
--2021-11-27 15:58:07--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 22616192 (22M) [application/octet-stream]
Saving to: ‘/tmp/new/test.gz’

/tmp/new/test.gz          100%[===================================>]  21.57M  41.7MB/s    in 0.5s

2021-11-27 15:58:07 (41.7 MB/s) - ‘/tmp/new/test.gz’ saved [22616192/22616192]
```
14. Вывод команды `lsblk`:
```
NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part  /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
sdb                    8:16   0  2.5G  0 disk
├─sdb1                 8:17   0    2G  0 part
│ └─md0                9:0    0    2G  0 raid1
└─sdb2                 8:18   0  511M  0 part
  └─md1                9:1    0 1018M  0 raid0
    └─vg_new-lv0     253:2    0  100M  0 lvm   /tmp/new
sdc                    8:32   0  2.5G  0 disk
├─sdc1                 8:33   0    2G  0 part
│ └─md0                9:0    0    2G  0 raid1
└─sdc2                 8:34   0  511M  0 part
  └─md1                9:1    0 1018M  0 raid0
    └─vg_new-lv0     253:2    0  100M  0 lvm   /tmp/new
```
15. Протестировали наш файл:
```
root@vagrant:/tmp/new# gzip -t /tmp/new/test.gz
root@vagrant:/tmp/new# echo $?
0
```
16. Переместили содержимое PV с RAID0 на RAID1 командой `pvmove -v /dev/md1 /dev/md0`:
```
  Executing: /sbin/modprobe dm-mirror
  Archiving volume group "vg_new" metadata (seqno 4).
  Creating logical volume pvmove0
  activation/volume_list configuration setting not defined: Checking only host tags for vg_new/lv0.
  Moving 25 extents of logical volume vg_new/lv0.
  activation/volume_list configuration setting not defined: Checking only host tags for vg_new/lv0.
  Creating vg_new-pvmove0
  Loading table for vg_new-pvmove0 (253:3).
  Loading table for vg_new-lv0 (253:2).
  Suspending vg_new-lv0 (253:2) with device flush
  Resuming vg_new-pvmove0 (253:3).
  Resuming vg_new-lv0 (253:2).
  Creating volume group backup "/etc/lvm/backup/vg_new" (seqno 5).
  activation/volume_list configuration setting not defined: Checking only host tags for vg_new/pvmove0        .
  Checking progress before waiting every 15 seconds.
  /dev/md1: Moved: 12.00%
  /dev/md1: Moved: 100.00%
  Polling finished successfully.
  
  root@vagrant:/tmp/new# lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part  /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
sdb                    8:16   0  2.5G  0 disk
├─sdb1                 8:17   0    2G  0 part
│ └─md0                9:0    0    2G  0 raid1
│   └─vg_new-lv0     253:2    0  100M  0 lvm   /tmp/new
└─sdb2                 8:18   0  511M  0 part
  └─md1                9:1    0 1018M  0 raid0
sdc                    8:32   0  2.5G  0 disk
├─sdc1                 8:33   0    2G  0 part
│ └─md0                9:0    0    2G  0 raid1
│   └─vg_new-lv0     253:2    0  100M  0 lvm   /tmp/new
└─sdc2                 8:34   0  511M  0 part
  └─md1                9:1    0 1018M  0 raid0
```
17. Пометили диск как сбойный с помощью ключа `--fail (-f)`:
```
root@vagrant:/tmp/new# mdadm /dev/md0 -f /dev/sdc1
mdadm: set /dev/sdc1 faulty in /dev/md0
root@vagrant:/tmp/new# cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md1 : active raid0 sdc2[1] sdb2[0]
      1042432 blocks super 1.2 512k chunks

md0 : active raid1 sdc1[1](F) sdb1[0]
      2094080 blocks super 1.2 [2/1] [U_]
```
18. Подтвердили выводом команды `dmseg`, что RAID1 работает в деградированном состоянии:
```
[ 5301.852386] md/raid1:md0: Disk failure on sdc1, disabling device.
               md/raid1:md0: Operation continuing on 1 devices.
```
19. Протестировали целостность файла, несмотря на "сбойный" диск и он продолжать быть доступен:
```
root@vagrant:~# gzip -t /tmp/new/test.gz
root@vagrant:~# echo $?
0
```
20. ВМ погасили.
