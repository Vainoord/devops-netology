## 3.5. Файловые системы
---
1. >Узнайте о sparse (разряженных) файлах.

Sparse file - вид файлов, который вместо хранения последовательности нулевых сегментов хранит информацию об этой последовательности. Проще говоря, по сравнению с обычным файлом, разреженный будет занимать в файловой системе весь выделенный ему объем, а на жесткий диск будет записана только та часть этого файла, которая отлична от нулей (массива нулей).

Примеры использования:
1) Образы виртуальных ОС - на хост машине такой файл будет занимать место столько, сколько фактически используется виртуальной ОС. (Выделяем для ОС 100Gb, VM заняла 6Gb из 100Gb выделенных, а размер файла в свойствах хост ОС будет равен 6Gb)
2) При скачивании файла с интернета ОС создает разреженный временный файл равный общему размеру скачанных данных, и затем по мере скачивания, размер файла не увеличивается, но увеличивается количество записанных секторов на физическом носителе хоста.

Создать разреженный файл можно утилитой  `dd`: `dd if=/dev/zero of=file-sparse bs=1 count=0 seek=1G`.
В linux конвертировать файлы в разряженные можно через `cp`: `cp --sparse=always file_source file_result`.

---
2. >Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?

Hardlink и файл для которого эта ссылка создаются имеют одинаковый индексный дескриптор (inode). Следовательно, жесткая ссылка имеет те же права доступа, владельца и время последней модификации, что и целевой файл. Различаются только имена файлов. Фактически жесткая ссылка это еще одно имя для файла.

Докажем. Есть файл `testfile.a` и две жесткие ссылки на него:
```
vagrant@dev-vm:~/hlink_check$ ls -lh
total 0
-rw-rw-r-- 3 vagrant vagrant 0 Jul  9 15:06 root_hlink_testfile
-rw-rw-r-- 3 vagrant vagrant 0 Jul  9 15:06 testfile.a
-rw-rw-r-- 3 vagrant vagrant 0 Jul  9 15:06 vagrant_hlink_testfile
```
Поменяем права доступа у одной из них:
```
vagrant@dev-vm:~/hlink_check$ chmod 777 root_hlink_testfile 
vagrant@dev-vm:~/hlink_check$ ls -lh
total 0
-rwxrwxrwx 3 vagrant vagrant 0 Jul  9 15:06 root_hlink_testfile
-rwxrwxrwx 3 vagrant vagrant 0 Jul  9 15:06 testfile.a
-rwxrwxrwx 3 vagrant vagrant 0 Jul  9 15:06 vagrant_hlink_testfile
```
Затем поменяем владельца ссылки:
```
vagrant@dev-vm:~/hlink_check$ sudo chown root:root root_hlink_testfile 
vagrant@dev-vm:~/hlink_check$ ls -lh
total 0
-rwxrwxrwx 3 root root 0 Jul  9 15:06 root_hlink_testfile
-rwxrwxrwx 3 root root 0 Jul  9 15:06 testfile.a
-rwxrwxrwx 3 root root 0 Jul  9 15:06 vagrant_hlink_testfile
```
Жесткая ссылка, по сути, это еще одно имя файла.

--- 
3. >Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile...

Выполнено:
```
vagrant@vagrant:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 61.9M  1 loop /snap/core20/1328
loop1                       7:1    0 43.6M  1 loop /snap/snapd/14978
loop2                       7:2    0 67.2M  1 loop /snap/lxd/21835
sda                         8:0    0   64G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0  1.5G  0 part /boot
└─sda3                      8:3    0 62.5G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm  /
sdb                         8:16   0  2.5G  0 disk 
sdc                         8:32   0  2.5G  0 disk 

```

---
4. >Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.

Выполнено:
```
Device     Boot   Start     End Sectors  Size Id Type
/dev/sdb1          2048 4196351 4194304    2G 83 Linux
/dev/sdb2       4196352 5242879 1046528  511M 83 Linux

```

---
5. >Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.

Выполнено:
```
vagrant@vagrant:~$ sudo sfdisk -d /dev/sdb | sudo sfdisk /dev/sdc
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
>>> Created a new DOS disklabel with disk identifier 0x44a2f1db.
/dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.
/dev/sdc2: Created a new partition 2 of type 'Linux' and of size 511 MiB.
/dev/sdc3: Done.

New situation:
Disklabel type: dos
Disk identifier: 0x44a2f1db

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1          2048 4196351 4194304    2G 83 Linux
/dev/sdc2       4196352 5242879 1046528  511M 83 Linux

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

```
Результат:
```
vagrant@vagrant:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT

...

sdb                         8:16   0  2.5G  0 disk 
├─sdb1                      8:17   0    2G  0 part 
└─sdb2                      8:18   0  511M  0 part 
sdc                         8:32   0  2.5G  0 disk 
├─sdc1                      8:33   0    2G  0 part 
└─sdc2                      8:34   0  511M  0 part 

```
---
6. >Соберите `mdadm` RAID1 на паре разделов 2 Гб.

Используем команду `sudo mdadm --create --verbose /dev/md1 -l raid1 -n 2 /dev/sd{b2,c2}`. `-l` - уровень рейд массива, `-n` - количество разделов.
```
vagrant@vagrant:~$ cat /proc/mdstat 
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
md1 : active raid1 sdc1[1] sdb1[0]
      2094080 blocks super 1.2 [2/2] [UU]
```

---
7. >Соберите `mdadm` RAID0 на второй паре маленьких разделов.

```
vagrant@vagrant:~$ sudo mdadm --create --verbose /dev/md0 -l raid0 -n 2 /dev/sd{b2,c2}
mdadm: chunk size defaults to 512K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
vagrant@vagrant:~$ cat /proc/mdstat 
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
md0 : active raid0 sdc2[1] sdb2[0]
      1042432 blocks super 1.2 512k chunks
      
md1 : active raid1 sdc1[1] sdb1[0]
      2094080 blocks super 1.2 [2/2] [UU]
```


---
8. >Создайте 2 независимых PV на получившихся md-устройствах.

Сделано.
```
vagrant@vagrant:~$ sudo pvcreate /dev/md0 /dev/md1
  Physical volume "/dev/md0" successfully created.
  Physical volume "/dev/md1" successfully created.
```

---
9. >Создайте общую volume-group на этих двух PV.

Готово. Теперь в VM две группы томов.
```
vagrant@vagrant:~$ sudo vgcreate vg_mutual /dev/md1 /dev/md0
  Volume group "vg_mutual" successfully created

vagrant@vagrant:~$ sudo vgdisplay
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <62.50 GiB
  PE Size               4.00 MiB
  Total PE              15999
  Alloc PE / Size       7999 / <31.25 GiB
  Free  PE / Size       8000 / 31.25 GiB
  VG UUID               4HbbNB-kISH-fXeQ-qzbV-XeNd-At34-cCUUuJ
   
  --- Volume group ---
  VG Name               vg_mutual
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               <2.99 GiB
  PE Size               4.00 MiB
  Total PE              765
  Alloc PE / Size       0 / 0   
  Free  PE / Size       765 / <2.99 GiB
  VG UUID               zBGJU1-ESML-zyPc-k4oG-YIbz-xokL-E2vGru
```

---
10. >Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.
```
vagrant@vagrant:~$ sudo lvcreate -L 100M -n lv_100Mb vg_mutual /dev/md0
  Logical volume "lv_100Mb" created.

vagrant@vagrant:~$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/ubuntu-lv
  LV Name                ubuntu-lv
  VG Name                ubuntu-vg
  LV UUID                mJ8K7e-F4uw-o8Sx-iwt0-JfLQ-Dpoh-E7lSU1
  LV Write Access        read/write
  LV Creation host, time ubuntu-server, 2022-06-07 11:41:15 +0000
  LV Status              available
  # open                 1
  LV Size                <31.25 GiB
  Current LE             7999
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
   
  --- Logical volume ---
  LV Path                /dev/vg_mutual/lv_100Mb
  LV Name                lv_100Mb
  VG Name                vg_mutual
  LV UUID                xZ8BWi-qrJ8-Gfif-9ykT-WDFZ-4z6y-OB0jFZ
  LV Write Access        read/write
  LV Creation host, time vagrant, 2022-07-09 14:51:00 +0000
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
---
11. >Создайте `mkfs.ext4` ФС на получившемся LV.

```
vagrant@vagrant:~$ sudo mkfs.ext4 /dev/vg_mutual/lv_100Mb
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
```

---
12. >Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.

LVM будет смонтирована в директорию `/tmp/curious_dir`:
```
vagrant@vagrant:~$ mkdir /tmp/curious_dir

vagrant@vagrant:~$ sudo mount /dev/vg_mutual/lv_100Mb /tmp/curious_dir
```
---
13. >Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.
```
vagrant@vagrant:~$ sudo wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/curious_dir/test.gz
--2022-07-09 19:35:17--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 23792149 (23M) [application/octet-stream]
Saving to: ‘/tmp/curious_dir/test.gz’

/tmp/curious_dir/test.gz                               100%[============================================================================================================================>]  22.69M  3.25MB/s    in 8.7s    

2022-07-09 19:35:26 (2.60 MB/s) - ‘/tmp/curious_dir/test.gz’ saved [23792149/23792149]
```

---
14. >Прикрепите вывод `lsblk`.	
```
vagrant@vagrant:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 61.9M  1 loop  /snap/core20/1328
loop1                       7:1    0 43.6M  1 loop  /snap/snapd/14978
loop2                       7:2    0 67.2M  1 loop  /snap/lxd/21835
loop3                       7:3    0 61.9M  1 loop  /snap/core20/1518
loop4                       7:4    0 67.8M  1 loop  /snap/lxd/22753
sda                         8:0    0   64G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0  1.5G  0 part  /boot
└─sda3                      8:3    0 62.5G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
└─sdb2                      8:18   0  511M  0 part  
  └─md0                     9:0    0 1018M  0 raid0 
    └─vg_mutual-lv_100Mb  253:1    0  100M  0 lvm   /tmp/curious_dir
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
└─sdc2                      8:34   0  511M  0 part  
  └─md0                     9:0    0 1018M  0 raid0 
    └─vg_mutual-lv_100Mb  253:1    0  100M  0 lvm   /tmp/curious_dir

```
---
15. >Протестируйте целостность файла:
```
vagrant@vagrant:/tmp/curious_dir$ sudo gzip -t /tmp/curious_dir/test.gz
 
vagrant@vagrant:/tmp/curious_dir$ echo $?
0
```

---
16. >Используя pvmove, переместите содержимое PV с RAID0 на RAID1.
```
vagrant@vagrant:/tmp/curious_dir$ sudo pvmove /dev/md0
  /dev/md0: Moved: 4.00%
  /dev/md0: Moved: 100.00%

vagrant@vagrant:/tmp/curious_dir$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 61.9M  1 loop  /snap/core20/1328
loop1                       7:1    0 43.6M  1 loop  /snap/snapd/14978
loop2                       7:2    0 67.2M  1 loop  /snap/lxd/21835
loop3                       7:3    0 61.9M  1 loop  /snap/core20/1518
loop4                       7:4    0 67.8M  1 loop  /snap/lxd/22753
sda                         8:0    0   64G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0  1.5G  0 part  /boot
└─sda3                      8:3    0 62.5G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.3G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
│   └─vg_mutual-lv_100Mb  253:1    0  100M  0 lvm   /tmp/curious_dir
└─sdb2                      8:18   0  511M  0 part  
  └─md0                     9:0    0 1018M  0 raid0 
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
│   └─vg_mutual-lv_100Mb  253:1    0  100M  0 lvm   /tmp/curious_dir
└─sdc2                      8:34   0  511M  0 part  
  └─md0                     9:0    0 1018M  0 raid0 
```

---
17. >Сделайте `--fail` на устройство в вашем RAID1 md.
```
vagrant@vagrant:/tmp/curious_dir$ sudo mdadm /dev/md1 --fail /dev/sdc1
mdadm: set /dev/sdc1 faulty in /dev/md1

vagrant@vagrant:/tmp/curious_dir$ cat /proc/mdstat 
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
md0 : active raid0 sdc2[1] sdb2[0]
      1042432 blocks super 1.2 512k chunks
      
md1 : active raid1 sdc1[1](F) sdb1[0]
      2094080 blocks super 1.2 [2/1] [U_]
```
---
18. > Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.
```
[12943.901280] md/raid1:md1: Disk failure on sdc1, disabling device.
               md/raid1:md1: Operation continuing on 1 devices.
```
---
19. >Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:
```
vagrant@vagrant:/tmp/curious_dir$ sudo gzip -t /tmp/curious_dir/test.gz
 
vagrant@vagrant:/tmp/curious_dir$ echo $?
0

```
---
20. >Погасите тестовый хост, `vagrant destroy`.
```
[vainoord@vnrd-mypc:vagrant_conf]$ vagrant halt
==> default: Attempting graceful shutdown of VM...

[vainoord@vnrd-mypc:vagrant_conf]$ vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Destroying VM and associated drives...
```

---
