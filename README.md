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
- Выполнить проверку на ошибки LV
- Выполнить ресайз ФС
- Уменьшить LV
- Смонтировать ФС
