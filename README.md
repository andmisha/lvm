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
38. Переконфигурировать grub, чтобы при загрузке автоматически переходить в новый / (/mnt)
```
[root@lvm ~]# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
[root@lvm ~]# chroot /mnt/
[root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done
```
39. Обновить образ initrd
```
[root@lvm /]# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g;
> s/.img//g"` --force; done
Executing: /sbin/dracut -v initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64 --force
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** No early-microcode cpio image needed ***
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
```
40. В файле /boot/grub2/grub.cfg заменить строку rd.lvm.lv=VolGroup00/LogVol00 на rd.lvm.lv=vg_root/lv_root чтобы был смонтирован нужный / (/mnt)
41. Перезагрузить ВМ и проверить загрузку с нового / (/mnt)
```
[vagrant@lvm ~]$ lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  ├─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol00 253:7    0 37.5G  0 lvm
sdb                       8:16   0   10G  0 disk
└─vg_root-lv_root       253:0    0   10G  0 lvm  /
sdc                       8:32   0    2G  0 disk
sdd                       8:48   0    1G  0 disk
├─vg0-mirror_rmeta_0    253:2    0    4M  0 lvm
│ └─vg0-mirror          253:6    0  816M  0 lvm
└─vg0-mirror_rimage_0   253:3    0  816M  0 lvm
  └─vg0-mirror          253:6    0  816M  0 lvm
sde                       8:64   0    1G  0 disk
├─vg0-mirror_rmeta_1    253:4    0    4M  0 lvm
│ └─vg0-mirror          253:6    0  816M  0 lvm
└─vg0-mirror_rimage_1   253:5    0  816M  0 lvm
  └─vg0-mirror          253:6    0  816M  0 lvm
```
42. Удалить старый раздел LV, создать новый
```
[root@lvm ~]# lvremove /dev/VolGroup00/LogVol00
Do you really want to remove active logical volume VolGroup00/LogVol00? [y/n]: y
  Logical volume "LogVol00" successfully removed
[root@lvm ~]#  lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00
WARNING: xfs signature detected on /dev/VolGroup00/LogVol00 at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/VolGroup00/LogVol00.
  Logical volume "LogVol00" created.
```
43. Создать файловую систему
```
[root@lvm ~]#  mkfs.xfs /dev/VolGroup00/LogVol00
meta-data=/dev/VolGroup00/LogVol00 isize=512    agcount=4, agsize=524288 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2097152, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```
44. Смонтировать
```
mount /dev/VolGroup00/LogVol00 /mnt
```
45. И перенести содержимое
```
[root@lvm /]# xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt
xfsdump: using file dump (drive_simple) strategy
xfsdump: version 3.1.7 (dump format 3.0)
xfsdump: level 0 dump of lvm:/
xfsdump: dump date: Sat Feb 13 20:49:12 2021
xfsdump: session id: fbf6d88f-433f-4c15-850c-1e24a75247c6
xfsdump: session label: ""
xfsrestore: using file dump (drive_simple) strategy
xfsrestore: version 3.1.7 (dump format 3.0)
xfsrestore: searching media for dump
xfsdump: ino map phase 1: constructing initial dump list
xfsdump: ino map phase 2: skipping (no pruning necessary)
xfsdump: ino map phase 3: skipping (only one dump stream)
xfsdump: ino map construction complete
xfsdump: estimated dump size: 803508608 bytes
xfsdump: creating dump session media file 0 (media 0, file 0)
xfsdump: dumping ino map
xfsdump: dumping directories
xfsrestore: examining media file 0
xfsrestore: dump description:
xfsrestore: hostname: lvm
xfsrestore: mount point: /
xfsrestore: volume: /dev/mapper/vg_root-lv_root
xfsrestore: session time: Sat Feb 13 20:49:12 2021
xfsrestore: level: 0
xfsrestore: session label: ""
xfsrestore: media label: ""
xfsrestore: file system id: f4176c06-454a-4719-ad64-2694c2d3964e
xfsrestore: session id: fbf6d88f-433f-4c15-850c-1e24a75247c6
xfsrestore: media id: 5f24838e-5f4d-4fba-ab29-6eda13a83d71
xfsrestore: searching media for directory dump
xfsrestore: reading directories
xfsdump: dumping non-directory files
xfsrestore: 3144 directories and 26935 entries processed
xfsrestore: directory post-processing
xfsrestore: restoring non-directory files
xfsdump: ending media file
xfsdump: media file size 775747608 bytes
xfsdump: dump size (non-dir files) : 760570880 bytes
xfsdump: dump complete: 13 seconds elapsed
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 13 seconds elapsed
xfsrestore: Restore Status: SUCCESS
```
45. Переконфигурируем Grub и initramfs
```
[root@lvm /]# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
[root@lvm /]# chroot /mnt/
[root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done
[root@lvm /]# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g;
> s/.img//g"` --force; done
Executing: /sbin/dracut -v initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64 --force
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** No early-microcode cpio image needed ***
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
```
46. Создать mirror для /var (сперва удалить ранее созданный LV mirror)
```
[root@lvm boot]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao----   8.00g
  LogVol01 VolGroup00 -wi-ao----   1.50g
  mirror   vg0        rwi-a-r--- 816.00m                                    100.00
  lv_root  vg_root    -wi-ao---- <10.00g
[root@lvm boot]# lvremove /dev/vg0/mirror
Do you really want to remove active logical volume vg0/mirror? [y/n]: y
  Logical volume "mirror" successfully removed
[root@lvm boot]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao----   8.00g
  LogVol01 VolGroup00 -wi-ao----   1.50g
  lv_root  vg_root    -wi-ao---- <10.00g
[root@lvm boot]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk
├─sda1                    8:1    0    1M  0 part
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part
  ├─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol00 253:7    0    8G  0 lvm  /
sdb                       8:16   0   10G  0 disk
└─vg_root-lv_root       253:0    0   10G  0 lvm
sdc                       8:32   0    2G  0 disk
sdd                       8:48   0    1G  0 disk
sde                       8:64   0    1G  0 disk
```
47. При создании нового PV стала появляться ошибка "Can't initialize physical volume "/dev/sdc" of volume group "otus" without -ff" потому что я удалил LV, но не удалил VG и PV.
Исправился - удалил некоректно созданный VG vg_var, удалил не вернно удаленный VG vg0, удалил PV /dev/sdc, /dev/sdd, /dev/sde.
```
[root@lvm boot]# pvcreate /dev/sdc /dev/sdd
  Can't initialize physical volume "/dev/sdc" of volume group "otus" without -ff
  /dev/sdc: physical volume not initialized.
  Can't initialize physical volume "/dev/sdd" of volume group "vg0" without -ff
  /dev/sdd: physical volume not initialized.
[root@lvm boot]# pvcreate /dev/sdc /dev/sdd -ff
Really INITIALIZE physical volume "/dev/sdc" of volume group "otus" [y/n]? y
  WARNING: Forcing physical volume creation on /dev/sdc of volume group "otus"
Really INITIALIZE physical volume "/dev/sdd" of volume group "vg0" [y/n]? y
  WARNING: Forcing physical volume creation on /dev/sdd of volume group "vg0"
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.
[root@lvm boot]# pvs
  WARNING: Device for PV JtVJeF-1Lbr-8Zdt-pDtS-b03M-x9fw-odky2X not found or rejected by a filter.
  PV         VG         Fmt  Attr PSize    PFree
  /dev/sda3  VolGroup00 lvm2 a--   <38.97g  <29.47g
  /dev/sdb   vg_root    lvm2 a--   <10.00g       0
  /dev/sdc              lvm2 ---     2.00g    2.00g
  /dev/sdd              lvm2 ---     1.00g    1.00g
  /dev/sde   vg0        lvm2 a--  1020.00m 1020.00m
  [unknown]  vg0        lvm2 a-m  1020.00m 1020.00m
[root@lvm boot]# vgcreate vg_var /dev/sdc /dev/sdd
  WARNING: Device for PV JtVJeF-1Lbr-8Zdt-pDtS-b03M-x9fw-odky2X not found or rejected by a filter.
  Volume group "vg_var" successfully created
[root@lvm boot]# vgs
  WARNING: Device for PV JtVJeF-1Lbr-8Zdt-pDtS-b03M-x9fw-odky2X not found or rejected by a filter.
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g <29.47g
  vg0          2   0   0 wz-pn-   1.99g   1.99g
  vg_root      1   1   0 wz--n- <10.00g      0
  vg_var       2   0   0 wz--n-   2.99g   2.99g
[root@lvm boot]# pvremove /dev/sdc
  WARNING: Device for PV JtVJeF-1Lbr-8Zdt-pDtS-b03M-x9fw-odky2X not found or rejected by a filter.
  PV /dev/sdc is used by VG vg_var so please use vgreduce first.
  (If you are certain you need pvremove, then confirm by using --force twice.)
  /dev/sdc: physical volume label not removed.
[root@lvm boot]# pvremove /dev/sdc --force
  WARNING: Device for PV JtVJeF-1Lbr-8Zdt-pDtS-b03M-x9fw-odky2X not found or rejected by a filter.
  PV /dev/sdc is used by VG vg_var so please use vgreduce first.
  (If you are certain you need pvremove, then confirm by using --force twice.)
  /dev/sdc: physical volume label not removed.
[root@lvm boot]# pvs
  WARNING: Device for PV JtVJeF-1Lbr-8Zdt-pDtS-b03M-x9fw-odky2X not found or rejected by a filter.
  PV         VG         Fmt  Attr PSize    PFree
  /dev/sda3  VolGroup00 lvm2 a--   <38.97g  <29.47g
  /dev/sdb   vg_root    lvm2 a--   <10.00g       0
  /dev/sdc   vg_var     lvm2 a--    <2.00g   <2.00g
  /dev/sdd   vg_var     lvm2 a--  1020.00m 1020.00m
  /dev/sde   vg0        lvm2 a--  1020.00m 1020.00m
  [unknown]  vg0        lvm2 a-m  1020.00m 1020.00m
[root@lvm boot]# # lvcreate -L 950M -m1 -n lv_var vg_var
[root@lvm boot]# lvs
  WARNING: Device for PV JtVJeF-1Lbr-8Zdt-pDtS-b03M-x9fw-odky2X not found or rejected by a filter.
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao----   8.00g
  LogVol01 VolGroup00 -wi-ao----   1.50g
  lv_root  vg_root    -wi-ao---- <10.00g
[root@lvm boot]# vgremove vg_var
  Volume group "vg_var" successfully removed
[root@lvm boot]# pvremove /dev/sdc
  WARNING: Device for PV JtVJeF-1Lbr-8Zdt-pDtS-b03M-x9fw-odky2X not found or rejected by a filter.
  Labels on physical volume "/dev/sdc" successfully wiped.
[root@lvm boot]# pvremove /dev/sdd
  WARNING: Device for PV JtVJeF-1Lbr-8Zdt-pDtS-b03M-x9fw-odky2X not found or rejected by a filter.
  Labels on physical volume "/dev/sdd" successfully wiped.
[root@lvm boot]# vgremove vg0
  WARNING: Device for PV JtVJeF-1Lbr-8Zdt-pDtS-b03M-x9fw-odky2X not found or rejected by a filter.
  Volume group "vg0" not found, is inconsistent or has PVs missing.
  Consider vgreduce --removemissing if metadata is inconsistent.
[root@lvm boot]# vgremove vg0 --removemissing
vgremove: unrecognized option '--removemissing'
  Error during parsing of command line.
[root@lvm boot]# vgremove --removemissing vg0
vgremove: unrecognized option '--removemissing'
  Error during parsing of command line.
[root@lvm boot]# vgremove vg0
  WARNING: Device for PV JtVJeF-1Lbr-8Zdt-pDtS-b03M-x9fw-odky2X not found or rejected by a filter.
  Volume group "vg0" not found, is inconsistent or has PVs missing.
  Consider vgreduce --removemissing if metadata is inconsistent.
[root@lvm boot]# vgreduce --removemissing vg0
  WARNING: Device for PV JtVJeF-1Lbr-8Zdt-pDtS-b03M-x9fw-odky2X not found or rejected by a filter.
  Wrote out consistent volume group vg0.
[root@lvm boot]# vgs
  VG         #PV #LV #SN Attr   VSize    VFree
  VolGroup00   1   2   0 wz--n-  <38.97g  <29.47g
  vg0          1   0   0 wz--n- 1020.00m 1020.00m
  vg_root      1   1   0 wz--n-  <10.00g       0
[root@lvm boot]# vgreduce --removemissing vg0
  Volume group "vg0" is already consistent.
[root@lvm boot]# vgremove vg0
  Volume group "vg0" successfully removed
[root@lvm boot]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g <29.47g
  vg_root      1   1   0 wz--n- <10.00g      0
[root@lvm boot]# pvs
  PV         VG         Fmt  Attr PSize   PFree
  /dev/sda3  VolGroup00 lvm2 a--  <38.97g <29.47g
  /dev/sdb   vg_root    lvm2 a--  <10.00g      0
  /dev/sde              lvm2 ---    1.00g   1.00g
[root@lvm boot]# pvremove /dev/sde
  Labels on physical volume "/dev/sde" successfully wiped.
 ```
 48. Команды стали выполняться без ошибок
 ```
[root@lvm boot]# pvcreate /dev/sdc /dev/sdd
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.
[root@lvm boot]# vgcreate vg_var /dev/sdc /dev/sdd
  Volume group "vg_var" successfully created
[root@lvm boot]# lvcreate -L 950M -m1 -n lv_var vg_var
  Rounding up size to full physical extent 952.00 MiB
  Logical volume "lv_var" created.
```
49. Создать ФС и скопировать в нее /var:
```
[root@lvm boot]#  mkfs.ext4 /dev/vg_var/lv_var
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
60928 inodes, 243712 blocks
12185 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=249561088
8 block groups
32768 blocks per group, 32768 fragments per group
7616 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

[root@lvm boot]# mount /dev/vg_var/lv_var /mnt
[root@lvm boot]# cp -aR /var/* /mnt/ # rsync -avHPSAX /var/ /mnt/
```
50. Смонтировать /var в новый каталог
```
[root@lvm /]# umount /mnt
[root@lvm /]# mount /dev/vg_var/lv_var /var
```
51. Внести изменения в fstab для автоматического монтирования /var
```
[root@lvm /]#  echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab
```
52. Перезагрузить ВМ и проверить
```
[vagrant@lvm ~]$ lsblk
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                        8:0    0   40G  0 disk
├─sda1                     8:1    0    1M  0 part
├─sda2                     8:2    0    1G  0 part /boot
└─sda3                     8:3    0   39G  0 part
  ├─VolGroup00-LogVol00  253:0    0    8G  0 lvm  /
  └─VolGroup00-LogVol01  253:1    0  1.5G  0 lvm  [SWAP]
sdb                        8:16   0   10G  0 disk
└─vg_root-lv_root        253:2    0   10G  0 lvm
sdc                        8:32   0    2G  0 disk
├─vg_var-lv_var_rmeta_0  253:3    0    4M  0 lvm
│ └─vg_var-lv_var        253:7    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_0 253:4    0  952M  0 lvm
  └─vg_var-lv_var        253:7    0  952M  0 lvm  /var
sdd                        8:48   0    1G  0 disk
├─vg_var-lv_var_rmeta_1  253:5    0    4M  0 lvm
│ └─vg_var-lv_var        253:7    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_1 253:6    0  952M  0 lvm
  └─vg_var-lv_var        253:7    0  952M  0 lvm  /var
sde                        8:64   0    1G  0 disk
```
53. Выделить том под /home
```
[root@lvm ~]#  lvcreate -n LogVol_Home -L 2G /dev/VolGroup00
  Logical volume "LogVol_Home" created.
[root@lvm ~]# mkfs.xfs /dev/VolGroup00/LogVol_Home
meta-data=/dev/VolGroup00/LogVol_Home isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@lvm ~]#  mount /dev/VolGroup00/LogVol_Home /mnt/
[root@lvm ~]# cp -aR /home/* /mnt/
[root@lvm ~]# rm -rf /home/*
[root@lvm ~]# umount /mnt
[root@lvm ~]# mount /dev/VolGroup00/LogVol_Home /home/
```
54. Проверить
```
[root@lvm ~]# lsblk
NAME                       MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                          8:0    0   40G  0 disk
├─sda1                       8:1    0    1M  0 part
├─sda2                       8:2    0    1G  0 part /boot
└─sda3                       8:3    0   39G  0 part
  ├─VolGroup00-LogVol00    253:0    0    8G  0 lvm  /
  ├─VolGroup00-LogVol01    253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol_Home 253:8    0    2G  0 lvm  /home
sdb                          8:16   0   10G  0 disk
└─vg_root-lv_root          253:2    0   10G  0 lvm
sdc                          8:32   0    2G  0 disk
├─vg_var-lv_var_rmeta_0    253:3    0    4M  0 lvm
│ └─vg_var-lv_var          253:7    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_0   253:4    0  952M  0 lvm
  └─vg_var-lv_var          253:7    0  952M  0 lvm  /var
sdd                          8:48   0    1G  0 disk
├─vg_var-lv_var_rmeta_1    253:5    0    4M  0 lvm
│ └─vg_var-lv_var          253:7    0  952M  0 lvm  /var
└─vg_var-lv_var_rimage_1   253:6    0  952M  0 lvm
  └─vg_var-lv_var          253:7    0  952M  0 lvm  /var
sde                          8:64   0    1G  0 disk
```
55. Снапшоты. Сгенерировать 20 файлов в папке /home
```
[root@lvm ~]# touch /home/file{1..20}
[root@lvm ~]# ls -l /home
total 0
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file1
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file10
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file11
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file12
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file13
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file14
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file15
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file16
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file17
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file18
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file19
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file2
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file20
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file3
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file4
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file5
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file6
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file7
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file8
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file9
drwx------. 3 vagrant vagrant 74 May 12  2018 vagrant
```
56. Создать снапшот
```
[root@lvm ~]# lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LogVol_Home
  Rounding up size to full physical extent 128.00 MiB
  Logical volume "home_snap" created.
```
57. Удалить часть файлов
```
[root@lvm ~]# rm -f /home/file{5..12}
[root@lvm ~]# ls -l /home
total 0
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file1
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file13
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file14
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file15
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file16
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file17
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file18
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file19
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file2
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file20
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file3
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file4
drwx------. 3 vagrant vagrant 74 May 12  2018 vagrant
```
58. Накатить снапшот и проверить, что файлы появились (помнить, что файловая система должна быть размонтирована)
```
[root@lvm ~]# umount /home
[root@lvm ~]#  lvconvert --merge /dev/VolGroup00/home_snap
  Merging of volume VolGroup00/home_snap started.
  VolGroup00/LogVol_Home: Merged: 100.00%
[root@lvm ~]# mount /home
[root@lvm ~]# ls -l /home
total 0
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file1
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file10
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file11
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file12
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file13
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file14
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file15
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file16
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file17
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file18
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file19
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file2
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file20
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file3
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file4
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file5
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file6
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file7
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file8
-rw-r--r--. 1 root    root     0 Feb 13 21:30 file9
drwx------. 3 vagrant vagrant 74 May 12  2018 vagrant
```
