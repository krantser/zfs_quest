
## **Работа с ZFS**

Для начала подготовим имеющийся у нас Vagrant скрипт. В секции disks, меняем
параметры size на 1024 - это размер (Мб) будущий дисков в виртуальной машине.
Далее переходим в секцию SHELL и добавляем к устанавливаемым автоматически 
пакетам следующие программы: yum-utils и wget.

Посмотрим версию нашей операционной системы:
```
cat /etc/centos-release
```

Вывод команды:
```
    CentOS Linux release 7.8.2003 (Core)
```

Теперь начнём процесс установки компонентов для работы с ZFS:
```
sudo yum install http://download.zfsonlinux.org/epel/zfs-release.el7_8.noarch.rpm
```

```
sudo gpg --quiet --with-fingerprint /etc/pki/rpm-gpg/RPM-GPG-KEY-zfsonlinux
```

Вывод команды:
```
    gpg: new configuration file `/home/vagrant/.gnupg/gpg.conf' created
    gpg: WARNING: options in `/home/vagrant/.gnupg/gpg.conf' are not yet active during this run
    pub  2048R/F14AB620 2013-03-21 ZFS on Linux <zfs@zfsonlinux.org>
      Key fingerprint = C93A FFFD 9F3F 7B03 C310  CEB6 A9D5 A1C0 F14A B620
    sub  2048R/99685629 2013-03-21
```

```
sudo yum-config-manager --enable zfs-kmod
sudo yum-config-manager --disable zfs
```

```
sudo yum repolist enabled
```

Вывод команды:
```
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.sale-dedic.com
 * extras: mirror.awanti.com
 * updates: centos-mirror.rbc.ru
repo id                                                                               repo name                                                                                         status
base/7/x86_64                                                                         CentOS-7 - Base                                                                                   10,070
extras/7/x86_64                                                                       CentOS-7 - Extras                                                                                    397
updates/7/x86_64                                                                      CentOS-7 - Updates                                                                                   722
zfs-kmod/x86_64                                                                       ZFS on Linux for EL7 - kmod                                                                           13
```

Теперь установим сам пакет zfs:
```
sudo yum install -y zfs
```

На имеющихся четырёх дисках создадим разделы (ниже показано как это сделать 
только для диска /dev/sdc):
```
sudo fdisk /dev/sdc
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x07929152.

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): 
Using default response p
Partition number (1-4, default 1): 
First sector (2048-2097151, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-2097151, default 2097151): 
Using default value 2097151
Partition 1 of type Linux and of size 1023 MiB is set

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

Загрузим модуль ядра для работы с ZFS:
```
sudo modprobe zfs
```

Создадим zfs пулы для каждой файловой системы:
```
sudo zpool create vol_gzip /dev/sdb
sudo zpool create vol_lzjb /dev/sdc
sudo zpool create vol_zle /dev/sdd
sudo zpool create vol_lz4 /dev/sde
```

Просмотрим информацию о наличии пулов:
```
zpool list
```

Вывод команды:
```
NAME       SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
vol_gzip   960M  94.5K   960M        -         -     0%     0%  1.00x    ONLINE  -
vol_lz4    960M  94.5K   960M        -         -     0%     0%  1.00x    ONLINE  -
vol_lzjb   960M  94.5K   960M        -         -     0%     0%  1.00x    ONLINE  -
vol_zle    960M  94.5K   960M        -         -     0%     0%  1.00x    ONLINE  -
```

Просмотрим структуру файловой системы:
```
lsblk
```

Вывод команды:
``` 
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0    1G  0 disk 
├─sdb1                    8:17   0 1014M  0 part 
└─sdb9                    8:25   0    8M  0 part 
sdc                       8:32   0    1G  0 disk 
├─sdc1                    8:33   0 1014M  0 part 
└─sdc9                    8:41   0    8M  0 part 
sdd                       8:48   0    1G  0 disk 
├─sdd1                    8:49   0 1014M  0 part 
└─sdd9                    8:57   0    8M  0 part 
sde                       8:64   0    1G  0 disk 
├─sde1                    8:65   0 1014M  0 part 
└─sde9                    8:73   0    8M  0 part 
```

Теперь настроим компрессию для каждой файловой системы:
```
sudo zfs set compression=gzip vol_gzip
sudo zfs set compression=lz4 vol_lz4
sudo zfs set compression=lzjb vol_lzjb
sudo zfs set compression=zle vol_zle
```

Выведем информацию по параметру компрессии для имеющихся файловых систем ZFS:
```
sudo zfs get compression
```

Вывод команды:
```
NAME                PROPERTY     VALUE     SOURCE
vol_gzip            compression  gzip      local
vol_lz4             compression  lz4       local
vol_lzjb            compression  lzjb      local
vol_zle             compression  zle       local
```

Создадим директории для монтирования файловых систем:
```
sudo mkdir /mnt/gzip_folder
sudo mkdir /mnt/lz4_folder
sudo mkdir /mnt/lzjb_folder
sudo mkdir /mnt/zle_folder
```

Установим владельца и группу на созданные нами директории:
```
sudo chown -R vagrant:vagrant /mnt/gzip_folder/
sudo chown -R vagrant:vagrant /mnt/lz4_folder/
sudo chown -R vagrant:vagrant /mnt/lzjb_folder/
sudo chown -R vagrant:vagrant /mnt/zle_folder/
```

Выведем информацию по каталогам в /mnt:
```
ls -l /mnt/
```
Вывод команды:
```
total 2
drwxr-xr-x. 2 vagrant vagrant 2 Jun 10 13:50 gzip_folder
drwxr-xr-x. 2 vagrant vagrant 2 Jun 10 14:52 lz4_folder
drwxr-xr-x. 2 vagrant vagrant 2 Jun 10 14:24 lzjb_folder
drwxr-xr-x. 2 vagrant vagrant 2 Jun 10 14:33 zle_folder
```

Установим точку монтирования для файловых систем в нужные нам директории:
```
sudo zfs set mountpoint=/mnt/gzip_folder vol_gzip
sudo zfs set mountpoint=/mnt/lz4_folder vol_lz4
sudo zfs set mountpoint=/mnt/lzjb_folder vol_lzjb
sudo zfs set mountpoint=/mnt/zle_folder vol_zle
```

Выведем информацию о точках монтирования zfs:
```
sudo zfs get mountpoint
```

Вывод команды:
```
NAME      PROPERTY    VALUE             SOURCE
vol_gzip  mountpoint  /mnt/gzip_folder  local
vol_lz4   mountpoint  /mnt/lz4_folder   local
vol_lzjb  mountpoint  /mnt/lzjb_folder  local
vol_zle   mountpoint  /mnt/zle_folder   local
```

Выведем информацию по файловым системам:
```
df -h
```

Вывод команды:
```
Filesystem                       Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup00-LogVol00   38G  1.8G   36G   5% /
devtmpfs                         236M     0  236M   0% /dev
tmpfs                            244M     0  244M   0% /dev/shm
tmpfs                            244M  8.6M  236M   4% /run
tmpfs                            244M     0  244M   0% /sys/fs/cgroup
/dev/sda2                       1014M   61M  954M   7% /boot
tmpfs                             49M     0   49M   0% /run/user/1000
vol_gzip                         832M  128K  832M   1% /mnt/gzip_folder
vol_lz4                          832M  128K  832M   1% /mnt/lz4_folder
vol_lzjb                         832M  128K  832M   1% /mnt/lzjb_folder
vol_zle                          832M  128K  832M   1% /mnt/zle_folder
```

Теперь загрузим файлы для теста различных видов компрессии на наших файловых
системах:
```
curl https://docs.oracle.com/cd/E19253-01/816-5166/zfs-1m/index.html --output /mnt/gzip_folder/oracle_zfs.html
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  115k  100  115k    0     0   127k      0 --:--:-- --:--:-- --:--:--  127k


curl https://docs.oracle.com/cd/E19253-01/816-5166/zfs-1m/index.html --output /mnt/lz4_folder/oracle_zfs.html
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  115k  100  115k    0     0   195k      0 --:--:-- --:--:-- --:--:--  195k


curl https://docs.oracle.com/cd/E19253-01/816-5166/zfs-1m/index.html --output /mnt/lzjb_folder/oracle_zfs.html
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  115k  100  115k    0     0   170k      0 --:--:-- --:--:-- --:--:--  170k


curl https://docs.oracle.com/cd/E19253-01/816-5166/zfs-1m/index.html --output /mnt/zle_folder/oracle_zfs.html
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  115k  100  115k    0     0  62244      0  0:00:01  0:00:01 --:--:-- 62222
```

Попробуем использовать команду `ls` для вывода размера файла:
```
ls -lh /mnt/gzip_folder/
```

Вывод команды:
```
total 26K
-rw-rw-r--. 1 vagrant vagrant 116K Jun 10 20:27 oracle_zfs.html
```

```
ls -lh /mnt/lz4_folder/
```

Вывод команды:
```
total 44K
-rw-rw-r--. 1 vagrant vagrant 116K Jun 10 20:28 oracle_zfs.html
```

```
ls -lh /mnt/lzjb_folder/
```

Вывод команды:
```
total 59K
-rw-rw-r--. 1 vagrant vagrant 116K Jun 10 20:28 oracle_zfs.html
```

```
ls -lh /mnt/zle_folder/
```

Вывод команды:
```
total 117K
-rw-rw-r--. 1 vagrant vagrant 116K Jun 10 20:28 oracle_zfs.html
```

Как видим, размер файла на всех четырёх дисках одинаков, поэтому, что бы увидеть
его с учётом сжатия, воспользуемся командой `du`:
```
du -h /mnt/gzip_folder/oracle_zfs.html 
```

Вывод команды:
```
26K	/mnt/gzip_folder/oracle_zfs.html
```

```
du -h /mnt/lz4_folder/oracle_zfs.html 
```

Вывод команды:
```
44K	/mnt/lz4_folder/oracle_zfs.html
```

```
du -h /mnt/lzjb_folder/oracle_zfs.html 
```

Вывод команды:
```
59K	/mnt/lzjb_folder/oracle_zfs.html
```

```
du -h /mnt/zle_folder/oracle_zfs.html 
```

Вывод команды:
```
117K	/mnt/zle_folder/oracle_zfs.html
```

Выведем уровень компрессии для наших файловых систем:
```
sudo zfs get compressratio
```

Вывод команды:
```
NAME      PROPERTY       VALUE  SOURCE
vol_gzip  compressratio  2.35x  -
vol_lz4   compressratio  1.85x  -
vol_lzjb  compressratio  1.58x  -
vol_zle   compressratio  1.00x  -
```
Как видим из вывода команды, лучшие результаты в сжатии показывает алгоритм gzip.

Теперь перейдём к следущей части задания и импортируем пул zfs. Для этого 
загрузим на сервер файлы дисковых устройств. Воспользуемся утилитой wget. Что бы
загрузить файл по следующей [ссылке](https://drive.google.com/open?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg "Файл на GoogleDrive")

Воспользуемся запросом, приведённым ниже:
```
wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=FILEID' -O FILENAME
```

Где FILENAME будет zfs_task1.tar.gz, а FILEID будет 1KRBNW33QWqbvbVHa3hLJivOAt60yukkg

Выполняем команду:
```
wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg' -O zfs_task1.tar.gz
```

Просмотрим нашу директорию:
```
ls -lh
```

Вывод команды:
```
total 7.0M
-rw-rw-r--. 1 vagrant vagrant 7.0M Jun 11 17:15 zfs_task1.tar.gz
```

Распакуем загруженный архив командами:
```
gunzip zfs_task1.tar.gz
tar -xf zfs_task1.tar
```
Как видим, появилась папка с двумя файлами:
```
ls -lh
```

Вывод команды:
```
total 1001M
-rw-rw-r--. 1 vagrant vagrant 1001M Jun 11 17:15 zfs_task1.tar
drwxr-xr-x. 2 vagrant vagrant    32 May 15 04:35 zpoolexport
```

```
ls -lh zpoolexport/
```

Вывод команды:
```
total 1000M
-rw-r--r--. 1 vagrant vagrant 500M May 15 05:00 filea
-rw-r--r--. 1 vagrant vagrant 500M May 15 05:00 fileb
```

Выведем информацию по имеющимся файлам. Где видно, что это два диска из пула zfs
собранные в тип mirror.

```
zdb -l zpoolexport/filea
```

Вывод команды:
```
------------------------------------
LABEL 0
------------------------------------
    version: 5000
    name: 'otus'
    state: 1
    txg: 484
    pool_guid: 6554193320433390805
    errata: 0
    hostname: 'localhost.localdomain'
    top_guid: 9948927108404753686
    guid: 6859274022954884342
    vdev_children: 1
    vdev_tree:
        type: 'mirror'
        id: 0
        guid: 9948927108404753686
        metaslab_array: 68
        metaslab_shift: 25
        ashift: 9
        asize: 519569408
        is_log: 0
        create_txg: 4
        children[0]:
            type: 'file'
            id: 0
            guid: 6859274022954884342
            path: '/root/zpoolexport/filea'
            create_txg: 4
        children[1]:
            type: 'file'
            id: 1
            guid: 12715741920600713412
            path: '/root/zpoolexport/fileb'
            create_txg: 4
    features_for_read:
        com.delphix:hole_birth
        com.delphix:embedded_data
    labels = 0 1 2 3 
```

```
zdb -l zpoolexport/fileb
```

Вывод команды:
```
------------------------------------
LABEL 0
------------------------------------
    version: 5000
    name: 'otus'
    state: 1
    txg: 484
    pool_guid: 6554193320433390805
    errata: 0
    hostname: 'localhost.localdomain'
    top_guid: 9948927108404753686
    guid: 12715741920600713412
    vdev_children: 1
    vdev_tree:
        type: 'mirror'
        id: 0
        guid: 9948927108404753686
        metaslab_array: 68
        metaslab_shift: 25
        ashift: 9
        asize: 519569408
        is_log: 0
        create_txg: 4
        children[0]:
            type: 'file'
            id: 0
            guid: 6859274022954884342
            path: '/root/zpoolexport/filea'
            create_txg: 4
        children[1]:
            type: 'file'
            id: 1
            guid: 12715741920600713412
            path: '/root/zpoolexport/fileb'
            create_txg: 4
    features_for_read:
        com.delphix:hole_birth
        com.delphix:embedded_data
    labels = 0 1 2 3 
```

Импортируем имеющиеся диски, указав при этом директорию, где искать файлы,
иначе по умолчанию zpool будет искать их в /dev/:
```
sudo zpool import -d zpoolexport/
```

Вывод команды:
```
   pool: otus
     id: 6554193320433390805
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

	otus                                 ONLINE
	  mirror-0                           ONLINE
	    /home/vagrant/zpoolexport/filea  ONLINE
	    /home/vagrant/zpoolexport/fileb  ONLINE
```

Теперь импортируем сам пул, указав его имя и директорию с дисками:
```
sudo zpool import otus -d zpoolexport/
```

Выведем информацию по имеющимся пулам zfs:
```
sudo zpool status
```

Вывод команды:
```
  pool: otus
 state: ONLINE
  scan: none requested
config:

	NAME                                 STATE     READ WRITE CKSUM
	otus                                 ONLINE       0     0     0
	  mirror-0                           ONLINE       0     0     0
	    /home/vagrant/zpoolexport/filea  ONLINE       0     0     0
	    /home/vagrant/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors

  pool: vol_gzip
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	vol_gzip    ONLINE       0     0     0
	  sdb       ONLINE       0     0     0

errors: No known data errors

  pool: vol_lz4
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	vol_lz4     ONLINE       0     0     0
	  sde       ONLINE       0     0     0

errors: No known data errors

  pool: vol_lzjb
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	vol_lzjb    ONLINE       0     0     0
	  sdc       ONLINE       0     0     0

errors: No known data errors

  pool: vol_zle
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	vol_zle     ONLINE       0     0     0
	  sdd       ONLINE       0     0     0

errors: No known data errors
```

Выведем список пулов с небольшим количеством информации по ним:
```
sudo zpool list
```

Вывод команды:
```
NAME       SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus       480M  2.09M   478M        -         -     0%     0%  1.00x    ONLINE  -
vol_gzip   960M   157K   960M        -         -     0%     0%  1.00x    ONLINE  -
vol_lz4    960M   175K   960M        -         -     0%     0%  1.00x    ONLINE  -
vol_lzjb   960M   190K   960M        -         -     0%     0%  1.00x    ONLINE  -
vol_zle    960M   248K   960M        -         -     0%     0%  1.00x    ONLINE  -
```

Выведем данные по имеющимся файловым система zfs:
```
sudo zfs list
```

Вывод команды:
```
NAME             USED  AVAIL     REFER  MOUNTPOINT
otus            2.04M   350M       24K  /otus
otus/hometask2  1.88M   350M     1.88M  /otus/hometask2
vol_gzip         138K   832M     50.5K  /mnt/gzip_folder
vol_lz4          156K   832M     68.5K  /mnt/lz4_folder
vol_lzjb         170K   832M     83.5K  /mnt/lzjb_folder
vol_zle          228K   832M      141K  /mnt/zle_folder
```

Выведем информацию о размерах блока в файловых системах:
```
sudo zfs get recordsize
```

Вывод команды:
```
NAME            PROPERTY    VALUE    SOURCE
otus            recordsize  128K     local
otus/hometask2  recordsize  128K     inherited from otus
vol_gzip        recordsize  128K     default
vol_lz4         recordsize  128K     default
vol_lzjb        recordsize  128K     default
vol_zle         recordsize  128K     default
```

Выведем значение типа компрессии на файловых системах:
```
sudo zfs get compression
```

Вывод команды:
```
NAME            PROPERTY     VALUE     SOURCE
otus            compression  zle       local
otus/hometask2  compression  zle       inherited from otus
vol_gzip        compression  gzip      local
vol_lz4         compression  lz4       local
vol_lzjb        compression  lzjb      local
vol_zle         compression  zle       local
```

Выведем значение алгоритма для просчёта чек-суммы файлов:
```
sudo zfs get checksum
```

Вывод команды:
```
NAME            PROPERTY  VALUE      SOURCE
otus            checksum  sha256     local
otus/hometask2  checksum  sha256     inherited from otus
vol_gzip        checksum  on         default
vol_lz4         checksum  on         default
vol_lzjb        checksum  on         default
vol_zle         checksum  on         default
```

Теперь перейдём к третьему этапу, где используем функционал снэпшотов zfs.
Загрузим на сервер необходимый файл снэпшота:
```
wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG' -O otus_task2.file
```

Выведем иформацию по файлам в домашней директории:
```
ls -lh
```

Вывод команды:
```
total 5.2M
-rw-rw-r--. 1 vagrant vagrant 5.2M Jun 12 20:36 otus_task2.file
drwxr-xr-x. 2 vagrant vagrant   32 May 15 04:35 zpoolexport
```

Восстановим снэпшот следующей командой, используя имеющийся файл otus_task2.file:
```
sudo zfs receive otus/storage@task2 < otus_task2.file
```

Т.к. раздел содержит большое количество файлов, то воспользуемся командой `find`
для поиска того, что нам нужно:
```
find /otus/storage/ -type f -name secret_message
```

Вывод команды:
```
/otus/storage/task1/file_mess/secret_message
```

Мы нашли необходимый файл, теперь выведем его содержимое с посланием:
```
cat /otus/storage/task1/file_mess/secret_message 
```

Вывод команды:
```
https://github.com/sindresorhus/awesome
```

На этом всё, этап работы с zfs  окончен.
