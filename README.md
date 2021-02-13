# lvm

1. По ссылке https://gitlab.com/otus_linux/stands-03-lvm скачал Vagrant-файл и выполнил vagrant up для запуска ВМ для данного задания
2. Проверил какие блочные устройства есть у ВМ
```
[vagrant@lvm ~]$ lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk
sdc                       8:32   0    2G  0 disk
sdd                       8:48   0    1G  0 disk
sde                       8:64   0    1G  0 disk
```
или 
```
[root@lvm ~]# lvmdiskscan
  /dev/VolGroup00/LogVol00 [     <37.47 GiB]
  /dev/VolGroup00/LogVol01 [       1.50 GiB]
  /dev/sda2                [       1.00 GiB]
  /dev/sda3                [     <39.00 GiB] LVM physical volume
  /dev/sdb                 [      10.00 GiB]
  /dev/sdc                 [       2.00 GiB]
  /dev/sdd                 [       1.00 GiB]
  /dev/sde                 [       1.00 GiB]
  4 disks
  3 partitions
  0 LVM physical volume whole disks
  1 LVM physical volume
  ```
3. Иерархия LVM представляет собой - Physical Volume (PV) -> Volume Group (VG) -> Logical Volume (LG)
4. Создать PV на блочном устройстве sdb
  ```
  [root@lvm ~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
  ```
5. Создать VG с именем otus на созданном PV
  ```
  [root@lvm ~]# vgcreate otus /dev/sdb
  Volume group "otus" successfully created
  ```
6. Создать LV с именем test на PV otus на 80% всего объема
  ```
  [root@lvm ~]# lvcreate -l+80%FREE -n test otus
  Logical volume "test" created.
  ```
7. Посмотреть информацию о созданном VG otus
  [root@lvm ~]# vgdisplay otus
  ```
  --- Volume group ---
  VG Name               otus
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <10.00 GiB
  PE Size               4.00 MiB
  Total PE              2559
  Alloc PE / Size       2047 / <8.00 GiB
  Free  PE / Size       512 / 2.00 GiB
  VG UUID               AyWcy1-Xug4-sklD-PIEG-hRzW-0eeG-tHp3qJ
  ```
8. Посмотреть информацию о том, какие диски входят в VG
[root@lvm ~]# vgdisplay -v otus | grep 'PV Name'
  PV Name               /dev/sdb
  
В методичке дана команда 
```
vgdisplay otus | grep 'PV NAME'
```
Должно быть так (с соблюдением регистра):
```
vgdisplay otus | grep 'PV Name'
```
9. Команда vgs выводит информацию о всех VG в системе
```
[root@lvm ~]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g    0
  otus         1   1   0 wz--n- <10.00g 2.00g
```
10. Команда lvs выводит информацию о всех LV в системе
```
[root@lvm ~]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao---- <37.47g
  LogVol01 VolGroup00 -wi-ao----   1.50g
  test     otus       -wi-a-----  <8.00g
```  
11. Создать еще один LV с именем small на оставшемся свободном месте в VG otus размером 100 Мб
```
[root@lvm ~]# lvcreate -L100M -n small otus
  Logical volume "small" created.
```
12. Проверить создание
```
[root@lvm ~]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao---- <37.47g
  LogVol01 VolGroup00 -wi-ao----   1.50g
  small    otus       -wi-a----- 100.00m
  test     otus       -wi-a-----  <8.00g
```
13. Создать файловую систему на LV test
```
[root@lvm ~]# mkfs.ext4 /dev/otus/test
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
524288 inodes, 2096128 blocks
104806 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2147483648
64 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```
14. Смотировать LV в созданную папку /data/ и проверить монтирование
```
[root@lvm ~]# mkdir /data
[root@lvm ~]# mount /dev/otus/test /data/
[root@lvm ~]# mount | grep data
/dev/mapper/otus-test on /data type ext4 (rw,relatime,seclabel,data=ordered)
```
---
15. Далее расширить VG otus, примонтированный в папку /data/ за счет еще одного блочного устройства /dev/sdc
```
[root@lvm ~]# pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
[root@lvm ~]# vgextend otus /dev/sdc
  Volume group "otus" successfully extended
```
16. Проверить, что в свойствах VG присутсвтует второй диск
```
[root@lvm ~]# vgdisplay -v otus | grep 'PV Name'
  PV Name               /dev/sdb
  PV Name               /dev/sdc
```
17. Проверить, что место действительно прибавилось
Было:
```
[root@lvm ~]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g    0
  otus         1   1   0 wz--n- <10.00g 2.00g
```
Стало:
```
[root@lvm ~]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g     0
  otus         2   2   0 wz--n-  11.99g <3.90g
```
18. Создать файл, который займет 100% места (2 Гб уже занято, нужно еще 8 Гб) на LV test
```
[root@lvm ~]# dd if=/dev/zero of=/data/test.log bs=1M count=8000 status=progress
8028946432 bytes (8.0 GB) copied, 25.132634 s, 319 MB/s
dd: error writing ‘/data/test.log’: No space left on device
7880+0 records in
7879+0 records out
8262189056 bytes (8.3 GB) copied, 25.9668 s, 318 MB/s
```
И проверить свободное место
```
[root@lvm ~]# df -h /data/
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/otus-test  7.8G  7.8G     0 100% /data
```
19. Расширить LV на часть свободного места (80%)
```
[root@lvm ~]# lvextend -l+80%FREE /dev/otus/test
  Size of logical volume otus/test changed from <8.00 GiB (2047 extents) to <11.12 GiB (2846 extents).
  Logical volume otus/test successfully resized.
```
20. Проверить факт расширения
Было:
```
[root@lvm ~]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao---- <37.47g
  LogVol01 VolGroup00 -wi-ao----   1.50g
  small    otus       -wi-a----- 100.00m
  test     otus       -wi-ao----  <8.00g
```
Стало:
```
[root@lvm ~]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao---- <37.47g
  LogVol01 VolGroup00 -wi-ao----   1.50g
  small    otus       -wi-a----- 100.00m
  test     otus       -wi-ao---- <11.12g
```
21. Проверить размер файловой системы на LV - размер остался прежний
```
[root@lvm ~]# df -h /data/
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/otus-test  7.8G  7.8G     0 100% /data
```
22. Провести ресайз файловой системы LV
```
[root@lvm ~]# resize2fs /dev/otus/test
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/otus/test is mounted on /data; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 2
The filesystem on /dev/otus/test is now 2914304 blocks long.
```
23. Проверить
Было:
```
[root@lvm ~]# df -h /data/
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/otus-test  7.8G  7.8G     0 100% /data
```
Стало:
```
[root@lvm ~]# df -h /data/
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/otus-test   11G  7.8G  2.6G  76% /data
```
---
24. Для уменьшения LV необходимо выполнить следующий порядок действий:
- Отмонтировать ФС
```
[root@lvm ~]# umount /data/
```
- Выполнить проверку на ошибки LV
```
[root@lvm ~]# e2fsck -f /dev/otus/test
e2fsck 1.42.9 (28-Dec-2013)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/otus/test: 12/729088 files (0.0% non-contiguous), 2105907/2914304 blocks
```
- Выполнить ресайз ФС
```
[root@lvm ~]# resize2fs /dev/otus/test 10G
resize2fs 1.42.9 (28-Dec-2013)
Resizing the filesystem on /dev/otus/test to 2621440 (4k) blocks.
The filesystem on /dev/otus/test is now 2621440 blocks long.
```
- Уменьшить LV
```
[root@lvm ~]# lvreduce /dev/otus/test -L 10G
  WARNING: Reducing active logical volume to 10.00 GiB.
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce otus/test? [y/n]: y
  Size of logical volume otus/test changed from <11.12 GiB (2846 extents) to 10.00 GiB (2560 extents).
  Logical volume otus/test successfully resized.
  ```
- Смонтировать ФС и проверить фактический размер
```
[root@lvm ~]# mount /dev/otus/test /data/
[root@lvm ~]# df -h
Filesystem                       Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup00-LogVol00   38G  753M   37G   2% /
devtmpfs                         109M     0  109M   0% /dev
tmpfs                            118M     0  118M   0% /dev/shm
tmpfs                            118M  4.5M  114M   4% /run
tmpfs                            118M     0  118M   0% /sys/fs/cgroup
/dev/sda2                       1014M   63M  952M   7% /boot
tmpfs                             24M     0   24M   0% /run/user/1000
/dev/mapper/otus-test            9.8G  7.8G  1.6G  84% /data
```
и
```
[root@lvm ~]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao---- <37.47g
  LogVol01 VolGroup00 -wi-ao----   1.50g
  small    otus       -wi-a----- 100.00m
  test     otus       -wi-ao----  10.00g
```
---
25.  Создать снапшот
```
[root@lvm ~]# lvcreate -L 500M -s -n test-snap /dev/otus/test
  Logical volume "test-snap" created.
```
26. Проверить наличие снапшота
```
[root@lvm ~]# vgs -o +lv_size,lv_name | grep test
  otus         2   3   1 wz--n-  11.99g <1.41g  10.00g test
  otus         2   3   1 wz--n-  11.99g <1.41g 500.00m test-snap
```
и
```
[root@lvm ~]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk
├─otus-small            253:3    0  100M  0 lvm
└─otus-test-real        253:4    0   10G  0 lvm
  ├─otus-test           253:2    0   10G  0 lvm  /data
  └─otus-test--snap     253:6    0   10G  0 lvm
sdc                       8:32   0    2G  0 disk
├─otus-test-real        253:4    0   10G  0 lvm
│ ├─otus-test           253:2    0   10G  0 lvm  /data
│ └─otus-test--snap     253:6    0   10G  0 lvm
└─otus-test--snap-cow   253:5    0  500M  0 lvm
  └─otus-test--snap     253:6    0   10G  0 lvm
sdd                       8:48   0    1G  0 disk
sde                       8:64   0    1G  0 disk
```
27. Смонтировать снапшот
```
[root@lvm ~]# mkdir /data-snap
[root@lvm ~]# mount /dev/otus/test-snap /data
data/      data-snap/
[root@lvm ~]# mount /dev/otus/test-snap /data-snap/
[root@lvm ~]# cd /data-snap/
[root@lvm data-snap]# ls -l
total 8068564
drwx------. 2 root root      16384 Feb 13 17:36 lost+found
-rw-r--r--. 1 root root 8262189056 Feb 13 18:50 test.log
```
28. Удалить файл test.log на оригинальном LV в папке /data/ и откатиться на снапшот
```
[root@lvm ~]# rm /data/test.log
rm: remove regular file ‘/data/test.log’? y
[root@lvm ~]# ls -l /data/
total 16
drwx------. 2 root root 16384 Feb 13 17:36 lost+found
[root@lvm ~]# umount /data
[root@lvm ~]# lvconvert --merge /dev/otus/test-snap
  Merging of volume otus/test-snap started.
  otus/test: Merged: 100.00%
[root@lvm ~]# mount /dev/otus/test /data
[root@lvm ~]# ls -l /data
total 8068564
drwx------. 2 root root      16384 Feb 13 17:36 lost+found
-rw-r--r--. 1 root root 8262189056 Feb 13 18:50 test.log
```
---
29. LVM Mirroring (зеркало). Создаем 2 PV
```
[root@lvm ~]# pvcreate /dev/sd{d,e}
  Physical volume "/dev/sdd" successfully created.
  Physical volume "/dev/sde" successfully created.
```
30. Создать VG vg0 из 2-ух PV
```
[root@lvm ~]# vgcreate vg0 /dev/sd{d,e}
  Volume group "vg0" successfully created
```
31. Создать LG с именем miror на 80% свободного места на VG vg0
```
[root@lvm ~]# lvcreate -l+80%FREE -m1 -n mirror vg0
  Logical volume "mirror" created.
```
32. Проверить создание LG
```
[root@lvm ~]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao---- <37.47g
  LogVol01 VolGroup00 -wi-ao----   1.50g
  small    otus       -wi-a----- 100.00m
  test     otus       -wi-ao----  10.00g
  mirror   vg0        rwi-a-r--- 816.00m                                    100.00
```
---
Домашнее задание
33. Установить пакет xfsdump для снятии копии тома /
```
[root@lvm ~]# yum install xfsdump
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.reconn.ru
 * extras: mirror.reconn.ru
 * updates: mirror.reconn.ru
Resolving Dependencies
--> Running transaction check
---> Package xfsdump.x86_64 0:3.1.7-1.el7 will be installed
--> Processing Dependency: attr >= 2.0.0 for package: xfsdump-3.1.7-1.el7.x86_64
--> Running transaction check
---> Package attr.x86_64 0:2.4.46-13.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

====================================================================================================================================
 Package                       Arch                         Version                                Repository                  Size
====================================================================================================================================
Installing:
 xfsdump                       x86_64                       3.1.7-1.el7                            base                       308 k
Installing for dependencies:
 attr                          x86_64                       2.4.46-13.el7                          base                        66 k

Transaction Summary
====================================================================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 374 k
Installed size: 1.1 M
Is this ok [y/d/N]: y
Downloading packages:
(1/2): attr-2.4.46-13.el7.x86_64.rpm                                                                         |  66 kB  00:00:00
(2/2): xfsdump-3.1.7-1.el7.x86_64.rpm                                                                        | 308 kB  00:00:00
------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                               809 kB/s | 374 kB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : attr-2.4.46-13.el7.x86_64                                                                                        1/2
  Installing : xfsdump-3.1.7-1.el7.x86_64                                                                                       2/2
  Verifying  : attr-2.4.46-13.el7.x86_64                                                                                        1/2
  Verifying  : xfsdump-3.1.7-1.el7.x86_64                                                                                       2/2

Installed:
  xfsdump.x86_64 0:3.1.7-1.el7

Dependency Installed:
  attr.x86_64 0:2.4.46-13.el7

Complete!
```
34. Создать временный том для раздела /
```
[root@lvm ~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
[root@lvm ~]# pvs
  PV         VG         Fmt  Attr PSize    PFree
  /dev/sda3  VolGroup00 lvm2 a--   <38.97g      0
  /dev/sdb              lvm2 ---    10.00g  10.00g
  /dev/sdc   otus       lvm2 a--    <2.00g  <2.00g
  /dev/sdd   vg0        lvm2 a--  1020.00m 200.00m
  /dev/sde   vg0        lvm2 a--  1020.00m 200.00m
[root@lvm ~]# vgcreate vg_root /dev/sdb
  Volume group "vg_root" successfully created
[root@lvm ~]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g      0
  otus         1   0   0 wz--n-  <2.00g  <2.00g
  vg0          2   1   0 wz--n-   1.99g 400.00m
  vg_root      1   0   0 wz--n- <10.00g <10.00g
[root@lvm ~]# lvcreate -n lv_root -l +100%FREE /dev/vg_root
WARNING: ext4 signature detected on /dev/vg_root/lv_root at offset 1080. Wipe it? [y/n]: y
  Wiping ext4 signature on /dev/vg_root/lv_root.
  Logical volume "lv_root" created.
```
35. Создать ФС и смонтировать том
```
[root@lvm ~]# mkfs.xfs /dev/vg_root/lv_root
meta-data=/dev/vg_root/lv_root   isize=512    agcount=4, agsize=655104 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2620416, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@lvm ~]# mount /dev/vg_root/lv_root /mnt
[root@lvm ~]# mount | grep lv_root
/dev/mapper/vg_root-lv_root on /mnt type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
```
36. Скопировать все данные из / в новый раздел
```
[root@lvm ~]# xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt
xfsrestore: using file dump (drive_simple) strategy
xfsrestore: version 3.1.7 (dump format 3.0)
xfsdump: using file dump (drive_simple) strategy
xfsdump: version 3.1.7 (dump format 3.0)
xfsrestore: searching media for dump
xfsdump: level 0 dump of lvm:/
xfsdump: dump date: Sat Feb 13 20:14:55 2021
xfsdump: session id: 3ea390f9-97ad-4ad2-b50c-f180cc5ae85e
xfsdump: session label: ""
xfsdump: ino map phase 1: constructing initial dump list
xfsdump: ino map phase 2: skipping (no pruning necessary)
xfsdump: ino map phase 3: skipping (only one dump stream)
xfsdump: ino map construction complete
xfsdump: estimated dump size: 751897536 bytes
xfsdump: creating dump session media file 0 (media 0, file 0)
xfsdump: dumping ino map
xfsdump: dumping directories
xfsrestore: examining media file 0
xfsrestore: dump description:
xfsrestore: hostname: lvm
xfsrestore: mount point: /
xfsrestore: volume: /dev/mapper/VolGroup00-LogVol00
xfsrestore: session time: Sat Feb 13 20:14:55 2021
xfsrestore: level: 0
xfsrestore: session label: ""
xfsrestore: media label: ""
xfsrestore: file system id: b60e9498-0baa-4d9f-90aa-069048217fee
xfsrestore: session id: 3ea390f9-97ad-4ad2-b50c-f180cc5ae85e
xfsrestore: media id: fc3c1f01-8aa2-4874-9d45-d78bc0b5b99d
xfsrestore: searching media for directory dump
xfsrestore: reading directories
xfsdump: dumping non-directory files
xfsrestore: 2720 directories and 23657 entries processed
xfsrestore: directory post-processing
xfsrestore: restoring non-directory files
xfsdump: ending media file
xfsdump: media file size 728909120 bytes
xfsdump: dump size (non-dir files) : 715718872 bytes
xfsdump: dump complete: 10 seconds elapsed
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 11 seconds elapsed
xfsrestore: Restore Status: SUCCESS
```
37. Проверить, что все скопировалось
```
[root@lvm ~]# ls /mnt
bin  boot  data  data-snap  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  vagrant  var
```
38. Gереконфигурировать grub, чтобы при загрузке автоматически переходить в новый / (/mnt)
```
[root@lvm ~]# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
[root@lvm ~]# chroot /mnt/
[root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done
```
39. 
