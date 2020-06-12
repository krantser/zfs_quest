sudo yum install -y yum-utils

cat /etc/centos-release
    CentOS Linux release 7.8.2003 (Core)

sudo yum install http://download.zfsonlinux.org/epel/zfs-release.el7_8.noarch.rpm

sudo gpg --quiet --with-fingerprint /etc/pki/rpm-gpg/RPM-GPG-KEY-zfsonlinux
    gpg: new configuration file `/home/vagrant/.gnupg/gpg.conf' created
    gpg: WARNING: options in `/home/vagrant/.gnupg/gpg.conf' are not yet active during this run
    pub  2048R/F14AB620 2013-03-21 ZFS on Linux <zfs@zfsonlinux.org>
      Key fingerprint = C93A FFFD 9F3F 7B03 C310  CEB6 A9D5 A1C0 F14A B620
    sub  2048R/99685629 2013-03-21

sudo yum-config-manager --enable zfs-kmod
sudo yum-config-manager --disable zfs

sudo yum repolist enabled
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

sudo yum install -y zfs

$ sudo yum install epel-release
$ sudo yum install "kernel-devel-uname-r == $(uname -r)" zfs

[vagrant@zfsquest ~]$ sudo fdisk /dev/sdc
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

sudo modprobe zfs

sudo zpool create vol_gzip /dev/sdb
[vagrant@zfsquest ~]$ sudo zpool create vol_lzjb /dev/sdc
[vagrant@zfsquest ~]$ sudo zpool create vol_zle /dev/sdd
sudo zpool create vol_lz4 /dev/sde
[vagrant@zfsquest ~]$ zpool list
NAME       SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
vol_gzip   960M  94.5K   960M        -         -     0%     0%  1.00x    ONLINE  -
vol_lz4    960M  94.5K   960M        -         -     0%     0%  1.00x    ONLINE  -
vol_lzjb   960M  94.5K   960M        -         -     0%     0%  1.00x    ONLINE  -
vol_zle    960M  94.5K   960M        -         -     0%     0%  1.00x    ONLINE  -

[vagrant@zfsquest ~]$ lsblk 
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

[vagrant@zfsquest ~]$ sudo zfs set compression=gzip vol_gzip
[vagrant@zfsquest ~]$ sudo zfs set compression=lz4 vol_lz4
[vagrant@zfsquest ~]$ sudo zfs set compression=lzjb vol_lzjb
[vagrant@zfsquest ~]$ sudo zfs set compression=zle vol_zle

sudo zfs get compression

[vagrant@zfsquest ~]$ sudo zfs set mountpoint=/mnt/gzip_folder vol_gzip
[vagrant@zfsquest ~]$ sudo zfs set mountpoint=/mnt/lz4_folder vol_lz4
[vagrant@zfsquest ~]$ sudo zfs set mountpoint=/mnt/lzjb_folder vol_lzjb
[vagrant@zfsquest ~]$ sudo zfs set mountpoint=/mnt/zle_folder vol_zle

[vagrant@zfsquest ~]$ sudo zfs get mountpoint
NAME      PROPERTY    VALUE             SOURCE
vol_gzip  mountpoint  /mnt/gzip_folder  local
vol_lz4   mountpoint  /mnt/lz4_folder   local
vol_lzjb  mountpoint  /mnt/lzjb_folder  local
vol_zle   mountpoint  /mnt/zle_folder   local
[vagrant@zfsquest ~]$ df -h
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

[vagrant@zfsquest ~]$ sudo chown -R vagrant:vagrant /mnt/gzip_folder/
[vagrant@zfsquest ~]$ sudo chown -R vagrant:vagrant /mnt/lz4_folder/
[vagrant@zfsquest ~]$ sudo chown -R vagrant:vagrant /mnt/lzjb_folder/
[vagrant@zfsquest ~]$ sudo chown -R vagrant:vagrant /mnt/zle_folder/
[vagrant@zfsquest ~]$ ls -l /mnt/
total 2
drwxr-xr-x. 2 vagrant vagrant 2 Jun 10 13:50 gzip_folder
drwxr-xr-x. 2 vagrant vagrant 2 Jun 10 14:52 lz4_folder
drwxr-xr-x. 2 vagrant vagrant 2 Jun 10 14:24 lzjb_folder
drwxr-xr-x. 2 vagrant vagrant 2 Jun 10 14:33 zle_folder
[vagrant@zfsquest ~]$ curl https://docs.oracle.com/cd/E19253-01/816-5166/zfs-1m/index.html --output /mnt/gzip_folder/oracle_zfs.html
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  115k  100  115k    0     0   127k      0 --:--:-- --:--:-- --:--:--  127k
[vagrant@zfsquest ~]$ curl https://docs.oracle.com/cd/E19253-01/816-5166/zfs-1m/index.html --output /mnt/lz4_folder/oracle_zfs.html
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  115k  100  115k    0     0   195k      0 --:--:-- --:--:-- --:--:--  195k
[vagrant@zfsquest ~]$ curl https://docs.oracle.com/cd/E19253-01/816-5166/zfs-1m/index.html --output /mnt/lzjb_folder/oracle_zfs.html
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  115k  100  115k    0     0   170k      0 --:--:-- --:--:-- --:--:--  170k
[vagrant@zfsquest ~]$ curl https://docs.oracle.com/cd/E19253-01/816-5166/zfs-1m/index.html --output /mnt/zle_folder/oracle_zfs.html
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  115k  100  115k    0     0  62244      0  0:00:01  0:00:01 --:--:-- 62222
[vagrant@zfsquest ~]$ ls -lh /mnt/gzip_folder/
total 26K
-rw-rw-r--. 1 vagrant vagrant 116K Jun 10 20:27 oracle_zfs.html
[vagrant@zfsquest ~]$ ls -lh /mnt/lz4_folder/
total 44K
-rw-rw-r--. 1 vagrant vagrant 116K Jun 10 20:28 oracle_zfs.html
[vagrant@zfsquest ~]$ ls -lh /mnt/lzjb_folder/
total 59K
-rw-rw-r--. 1 vagrant vagrant 116K Jun 10 20:28 oracle_zfs.html
[vagrant@zfsquest ~]$ ls -lh /mnt/zle_folder/
total 117K
-rw-rw-r--. 1 vagrant vagrant 116K Jun 10 20:28 oracle_zfs.html
[vagrant@zfsquest ~]$ du -h /mnt/gzip_folder/oracle_zfs.html 
26K	/mnt/gzip_folder/oracle_zfs.html
[vagrant@zfsquest ~]$ du -h /mnt/lz4_folder/oracle_zfs.html 
44K	/mnt/lz4_folder/oracle_zfs.html
[vagrant@zfsquest ~]$ du -h /mnt/lzjb_folder/oracle_zfs.html 
59K	/mnt/lzjb_folder/oracle_zfs.html
[vagrant@zfsquest ~]$ du -h /mnt/zle_folder/oracle_zfs.html 

[vagrant@zfsquest ~]$ sudo zfs get compressratio
NAME      PROPERTY       VALUE  SOURCE
vol_gzip  compressratio  2.35x  -
vol_lz4   compressratio  1.85x  -
vol_lzjb  compressratio  1.58x  -
vol_zle   compressratio  1.00x  -


wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=FILEID' -O FILENAME

zfs_task1.tar.gz
1KRBNW33QWqbvbVHa3hLJivOAt60yukkg
wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg' -O zfs_task1.tar.gz

[vagrant@zfsquest ~]$ ls -lh
total 7.0M
-rw-rw-r--. 1 vagrant vagrant 7.0M Jun 11 17:15 zfs_task1.tar.gz

gunzip zfs_task1.tar.gz
tar -xf zfs_task1.tar

[vagrant@zfsquest ~]$ ls -lh
total 1001M
-rw-rw-r--. 1 vagrant vagrant 1001M Jun 11 17:15 zfs_task1.tar
drwxr-xr-x. 2 vagrant vagrant    32 May 15 04:35 zpoolexport

[vagrant@zfsquest ~]$ ls -lh zpoolexport/
total 1000M
-rw-r--r--. 1 vagrant vagrant 500M May 15 05:00 filea
-rw-r--r--. 1 vagrant vagrant 500M May 15 05:00 fileb

[vagrant@zfsquest ~]$ zdb -l zpoolexport/filea
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
[vagrant@zfsquest ~]$ zdb -l zpoolexport/fileb
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


[vagrant@zfsquest ~]$ sudo zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

	otus                                 ONLINE
	  mirror-0                           ONLINE
	    /home/vagrant/zpoolexport/filea  ONLINE
	    /home/vagrant/zpoolexport/fileb  ONLINE

[vagrant@zfsquest ~]$ sudo zpool import otus -d zpoolexport/
[vagrant@zfsquest ~]$ sudo zpool status
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


[vagrant@zfsquest ~]$ sudo zpool list
NAME       SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus       480M  2.09M   478M        -         -     0%     0%  1.00x    ONLINE  -
vol_gzip   960M   157K   960M        -         -     0%     0%  1.00x    ONLINE  -
vol_lz4    960M   175K   960M        -         -     0%     0%  1.00x    ONLINE  -
vol_lzjb   960M   190K   960M        -         -     0%     0%  1.00x    ONLINE  -
vol_zle    960M   248K   960M        -         -     0%     0%  1.00x    ONLINE  -

[vagrant@zfsquest ~]$ sudo zfs list
NAME             USED  AVAIL     REFER  MOUNTPOINT
otus            2.04M   350M       24K  /otus
otus/hometask2  1.88M   350M     1.88M  /otus/hometask2
vol_gzip         138K   832M     50.5K  /mnt/gzip_folder
vol_lz4          156K   832M     68.5K  /mnt/lz4_folder
vol_lzjb         170K   832M     83.5K  /mnt/lzjb_folder
vol_zle          228K   832M      141K  /mnt/zle_folder

[vagrant@zfsquest ~]$ sudo zfs get recordsize
NAME            PROPERTY    VALUE    SOURCE
otus            recordsize  128K     local
otus/hometask2  recordsize  128K     inherited from otus
vol_gzip        recordsize  128K     default
vol_lz4         recordsize  128K     default
vol_lzjb        recordsize  128K     default
vol_zle         recordsize  128K     default

[vagrant@zfsquest ~]$ sudo zfs get compression
NAME            PROPERTY     VALUE     SOURCE
otus            compression  zle       local
otus/hometask2  compression  zle       inherited from otus
vol_gzip        compression  gzip      local
vol_lz4         compression  lz4       local
vol_lzjb        compression  lzjb      local
vol_zle         compression  zle       local

[vagrant@zfsquest ~]$ sudo zfs get checksum
NAME            PROPERTY  VALUE      SOURCE
otus            checksum  sha256     local
otus/hometask2  checksum  sha256     inherited from otus
vol_gzip        checksum  on         default
vol_lz4         checksum  on         default
vol_lzjb        checksum  on         default
vol_zle         checksum  on         default


[vagrant@zfsquest ~]$ wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG' -O otus_task2.file
[vagrant@zfsquest ~]$ ls -lh
total 5.2M
-rw-rw-r--. 1 vagrant vagrant 5.2M Jun 12 20:36 otus_task2.file
drwxr-xr-x. 2 vagrant vagrant   32 May 15 04:35 zpoolexport

sudo zfs receive otus/storage@task2 < otus_task2.file


[vagrant@zfsquest ~]$ find /otus/storage/ -type f -name secret_message
/otus/storage/task1/file_mess/secret_message

[vagrant@zfsquest ~]$ cat /otus/storage/task1/file_mess/secret_message 
https://github.com/sindresorhus/awesome

