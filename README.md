# vagrant-raid-script
Домашнее задание: работа с mdadm

Задание:

• добавитþ в Vagrantfile еûе дисков

• собратþ R0/R5/R10 на вýбор

• прописатþ собраннýй рейд в конф, ùтобý рейд собиралсā при загрузке

• сломатþ/поùинитþ raid

• создатþ GPT раздел и 5 партиøий и смонтироватþ их на диск.

В каùестве проверки принимаĀтсā - измененнýй Vagrantfile, скрипт длā
созданиā рейда, конф длā автосборки рейда при загрузке.

* Доп. задание - Vagrantfile, которýй сразу собирает систему с подклĀùеннýм
рейдом

Установка Vagrant

● Установите VirtualBox на локалþнуĀ маúину.
● Установите сам Vagrant, скаùав подходāûий под ваúу операøионнуĀ
систему пакет. Проверитþ установку можно командой:

[root@mdadm ~]$ vagrant -v

● Про Vagrant можно посмотретþ в наúем открýтом уроке.

Наùалþнýй стенд можно взāтþ отсĀда: https://github.com/erlong15/otus-linux
В принøипе на нем уже можно собратþ лĀбой RAID.

Длā каждого следуĀûего диска нужно добавитþ следуĀûий блок в Vagrantfile
:sata5 => {
:dfile => './sata5.vdi', # Путь, по которому будет создан файл диска
:size => 250, # Размер диска в мегабайтах

:port => 5 # Номер порта на который будет зацеплен диск
},

Обāзателþно увелиùив номер порта и изменив имā файла диска, ùтобý
исклĀùитþ дублирование.

Далее подразумеваем, ùто мý добавили в Vagrantfile 5-ýй диск.
Добавить в Vagrantfile еще дисков

Далее нужно определитþсā какого уровнā RAID будем собиратþ. Длā ÿто
посмотрим какие блоùнýе устройства у нас естþ и исходā из их кол-во,
размера и поставленной задаùи определимсā.

Сделатþ ÿто можно несколþкими способами:

● fdisk -l
● lsblk
● lshw
● lsscsi
Собрать RAID0/1/5/10 - на выбор

[root@mdadm ~]$ sudo lshw -short | grep disk
/0/100/d/0 /dev/sdb disk 1048MB VBOX HARDDISK
/0/100/d/1 /dev/sdc disk 262MB VBOX HARDDISK
/0/100/d/2 /dev/sdd disk 262MB VBOX HARDDISK
/0/100/d/3 /dev/sde disk 262MB VBOX HARDDISK
/0/100/d/0.0.0 /dev/sdf disk 262MB VBOX HARDDISK

[root@mdadm ~]$ sudo fdisk -l

Disk /dev/sda: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000b47f7

Device Boot Start End Blocks Id System
/dev/sda1 2048 4095 1024 83 Linux
/dev/sda2 * 4096 2101247 1048576 83 Linux
/dev/sda3 2101248 8388607940892416 8e Linux LVM
Собрать RAID0/1/5/10 - на выбор

Занулим на всāкий слуùай суперблоки:
[root@mdadm ~]$ mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}

И можно создаватþ рейд следуĀûей командой:
[root@mdadm ~]$ mdadm --create --verbose /dev/md0 -l 6 -n 5 /dev/sd{b,c,d,e,f}
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 253952K
mdadm: largest drive (/dev/sdb) exceeds size (253952K) by more than 1%
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

● Мý вýбрали RAID 6. Опøиā -l какого уровнā RAID создаватþ
● Опøиā - n указýвает на кол-во устройств в RAID
Собрать RAID0/1/5/10 - на выбор

Проверим ùто RAID собралсā нормалþно:
[root@mdadm ~]$ cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid6 sdf[4] sde[3] sdd[2] sdc[1] sdb[0]
761856 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/5] [UUUUU]

[root@mdadm ~]$ mdadm -D /dev/md0
/dev/md0:
Raid Level : raid6
Number Major Minor RaidDevice State
0 8 16 0 active sync /dev/sdb
1 8 32 1 active sync /dev/sdc
2 8 48 2 active sync /dev/sdd
3 8 64 3 active sync /dev/sde
4 8 80 4 active sync /dev/sdf

Полнýй вýвод можно посмотретþ тут:
https://gist.github.com/lalbrekht/05a750161f63a2f892b5c314a58ff28b
Собрать RAID0/1/5/10 - на выбор

Кол-во Āнитов в RAID
Размер одного ùанка

Длā того, ùтобý бýтþ увереннýм ùто ОС запомнила какой RAID массив
требуетсā создатþ и какие компонентý в него входāт создадим файл
mdadm.conf

Снаùала убедимсā, ùто информаøиā верна:

[root@mdadm ~]$ mdadm --detail --scan --verbose
ARRAY /dev/md0 level=raid6 num-devices=5 metadata=1.2 name=mdadm:0
UUID=11fc7859:98d4e7d3:48b30582:b2630265
devices=/dev/sdb,/dev/sdc,/dev/sdd,/dev/sde,/dev/sdf

А затем в две командý создадим файл mdadm.conf
[vagrant@mdadm ~]$ echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
[vagrant@mdadm ~]$ mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >>
/etc/mdadm/mdadm.conf
Создание конфигурационного файла mdadm.conf

Сделатþ ÿто можно, например, искусственно “зафейлив” одно из блоùнýх
устройств командной:

[root@mdadm ~]$ mdadm /dev/md0 --fail /dev/sde
mdadm: set /dev/sde faulty in /dev/md0

Посмотрим как ÿто отразилосþ на RAID:

[root@mdadm ~]$ cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid6 sdf[4] sde[3](F) sdd[2] sdc[1] sdb[0]
761856 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/4] [UUU_U]

[root@mdadm ~]$ mdadm -D /dev/md0
Number Major Minor RaidDevice State
0 8 16 0 active sync /dev/sdb
1 8 32 1 active sync /dev/sdc
2 8 48 2 active sync /dev/sdd
- 0 0 3 removed
4 8 80 4 active sync /dev/sdf
3 8 64 - faulty /dev/sde
Сломать/починить RAID

“зафейливúийсā” диск

Удалим “сломаннýй” диск из массива:

[root@mdadm ~]$ mdadm /dev/md0 --remove /dev/sde
mdadm: hot removed /dev/sde from /dev/md0

Представим, ùто мý вставили новýй диск в сервер и теперþ нам нужно
добавитþ его в RAID. Делаетсā ÿто так:

[root@mdadm ~]$ mdadm /dev/md0 --add /dev/sde
mdadm: added /dev/sde

Диск должен пройти стадиĀ rebuilding. Например, если ÿто бýл RAID 1
(зеркало), то даннýе должнý скопироватþсā на новýй диск.
Сломать/починить RAID

Проøесс rebuild-а можно увидетþ в вýводе следуĀûих команд:

[root@mdadm ~]$ cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid6 sde[5] sdf[4] sdd[2] sdc[1] sdb[0]
761856 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/4] [UUU_U]
[========>............] recovery = 44.6% (113664/253952) finish=0.0min
speed=113664K/sec

[root@mdadm ~]$ mdadm -D /dev/md0
Number Major Minor RaidDevice State
0 8 16 0 active sync /dev/sdb
1 8 32 1 active sync /dev/sdc
2 8 48 2 active sync /dev/sdd
5 8 64 3 spare rebuilding /dev/sde

На маленþких обüемах занāтого пространства можно и пропуститþ момент
перестроениā RAID-а - так бýстро он проходит.
Сломать/починить RAID

Создаем раздел GPT на RAID
[root@mdadm ~]$ parted -s /dev/md0 mklabel gpt

Создаем партиøии
[root@mdadm ~]$ parted /dev/md0 mkpart primary ext4 0% 20%
[root@mdadm ~]$ parted /dev/md0 mkpart primary ext4 20% 40%
[root@mdadm ~]$ parted /dev/md0 mkpart primary ext4 40% 60%
[root@mdadm ~]$ parted /dev/md0 mkpart primary ext4 60% 80%
[root@mdadm ~]$ parted /dev/md0 mkpart primary ext4 80% 100%

Далее можно создатþ на ÿтих партиøиāх ФС
[root@mdadm ~]$ for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done

И смонтироватþ их по каталогам
[root@mdadm ~]$ mkdir -p /raid/part{1,2,3,4,5}
[root@mdadm ~]$ for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done
