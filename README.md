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
14. 
