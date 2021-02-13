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
  7. 
