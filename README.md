# otus_zfs
1) Определение алгоритма с лучшим сжатием<br>
Создаем 4 пула, в каждом пуле два диска собраных в RAID 1
<pre>
[root@lvm vagrant]# zpool create otus1 mirror /dev/sdb /dev/sdc
[root@lvm vagrant]# zpool create otus2 mirror /dev/sdd /dev/sde
[root@lvm vagrant]# zpool create otus3 mirror /dev/sdf /dev/sdg
[root@lvm vagrant]# zpool create otus4 mirror /dev/sdh /dev/sdi
[root@lvm vagrant]# zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   240M   106K   240M        -         -     3%     0%  1.00x    ONLINE  -
otus2   240M   106K   240M        -         -     3%     0%  1.00x    ONLINE  -
otus3   240M  91.5K   240M        -         -     3%     0%  1.00x    ONLINE  -
otus4   240M  91.5K   240M        -         -     3%     0%  1.00x    ONLINE  - 
</pre>

Создадим в пулах файловую систему 
<pre>
[root@lvm vagrant]# zfs create otus1/z01
[root@lvm vagrant]# zfs create otus2/z02
[root@lvm vagrant]# zfs create otus3/z03
[root@lvm vagrant]# zfs create otus4/z04
[root@lvm vagrant]# zfs list
NAME        USED  AVAIL     REFER  MOUNTPOINT
otus1       123K   120M     25.5K  /otus1
otus1/z01    24K   120M       24K  /otus1/z01
otus2       123K   120M     25.5K  /otus2
otus2/z02    24K   120M       24K  /otus2/z02
otus3       123K   120M     25.5K  /otus3
otus3/z03    24K   120M       24K  /otus3/z03
otus4       123K   120M     25.5K  /otus4
otus4/z04    24K   120M       24K  /otus4/z04
</pre>
Добавляем разное сжатие на пулы
<pre>
[root@lvm vagrant]# zfs set compression=lzjb otus1
[root@lvm vagrant]# zfs set compression=lz4 otus2
[root@lvm vagrant]# zfs set compression=gzip-9 otus3
[root@lvm vagrant]# zfs set compression=zle otus4
[root@lvm vagrant]# zfs get all | grep compression
otus1      compression           lzjb                   local
otus1/z01  compression           lzjb                   inherited from otus1
otus2      compression           lz4                    local
otus2/z02  compression           lz4                    inherited from otus2
otus3      compression           gzip-9                 local
otus3/z03  compression           gzip-9                 inherited from otus3
otus4      compression           zle                    local
otus4/z04  compression           zle                    inherited from otus4
</pre>
Скачиваем файл в созданные пулы для проверки сжатия
<pre>[root@lvm vagrant]# for i in {1..4}; do wget -P /otus$i/z0$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done</pre>
Проверяем размеры пула
<pre>
[root@lvm vagrant]# zfs list
NAME        USED  AVAIL     REFER  MOUNTPOINT
otus1      21.8M  98.1M     25.5K  /otus1
otus1/z01  21.6M  98.1M     21.6M  /otus1/z01
otus2      17.7M   102M     25.5K  /otus2
otus2/z02  17.6M   102M     17.6M  /otus2/z02
otus3      10.8M   109M     25.5K  /otus3
otus3/z03  10.7M   109M     10.7M  /otus3/z03
otus4      38.4M  81.5M     25.5K  /otus4
otus4/z04  38.3M  81.5M     38.3M  /otus4/z04
</pre>
Можно сделать вывод что лучшее сжатие в otus3/z03, а именно сжатое в gzip-9

2) Определение настроек пула

Скачиваем архив
<pre>wget -O archive.tar.gz --no-check-certificate 'https://drive.google.com/u/0/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg&export=download'</pre>

Разархивируем его 
<pre>
[root@lvm vagrant]# tar -xzvf archive.tar.gz
zpoolexport/
zpoolexport/filea
zpoolexport/fileb  
</pre>
Проверяем возможность импортирования каталога в пул
<pre>[root@lvm vagrant]# zpool import -d zpoolexport/
   pool: otus
     id: 6554193320433390805
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

        otus                                 ONLINE
          mirror-0                           ONLINE
            /home/vagrant/zpoolexport/filea  ONLINE
            /home/vagrant/zpoolexport/fileb  ONLINE
</pre>
Делаем импорт данного каталога в пул 
<pre>
[root@lvm vagrant]# zpool import -d zpoolexport/ otus
[root@lvm vagrant]# zpool status
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

  pool: otus1
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        otus1       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdb     ONLINE       0     0     0
            sdc     ONLINE       0     0     0

errors: No known data errors

  pool: otus2
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        otus2       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdd     ONLINE       0     0     0
            sde     ONLINE       0     0     0

errors: No known data errors

  pool: otus3
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        otus3       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdf     ONLINE       0     0     0
            sdg     ONLINE       0     0     0

errors: No known data errors

  pool: otus4
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        otus4       ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            sdh     ONLINE       0     0     0
            sdi     ONLINE       0     0     0

errors: No known data errors
 
</pre>
Просмотр всех параметров файловой системы zfs 
<pre>
[root@lvm vagrant]# zpool get all otus
NAME  PROPERTY                       VALUE                          SOURCE
otus  size                           480M                           -
otus  capacity                       0%                             -
otus  altroot                        -                              default
otus  health                         ONLINE                         -
otus  guid                           6554193320433390805            -
otus  version                        -                              default
otus  bootfs                         -                              default
otus  delegation                     on                             default
otus  autoreplace                    off                            default
otus  cachefile                      -                              default
otus  failmode                       wait                           default
otus  listsnapshots                  off                            default
otus  autoexpand                     off                            default
otus  dedupditto                     0                              default
otus  dedupratio                     1.00x                          -
otus  free                           478M                           -
otus  allocated                      2.11M                          -
otus  readonly                       off                            -
otus  ashift                         0                              default
otus  comment                        -                              default
otus  expandsize                     -                              -
otus  freeing                        0                              -
otus  fragmentation                  0%                             -
otus  leaked                         0                              -
otus  multihost                      off                            default
otus  checkpoint                     -                              -
otus  load_guid                      5746304083361541856            -
otus  autotrim                       off                            default
otus  feature@async_destroy          enabled                        local
otus  feature@empty_bpobj            active                         local
otus  feature@lz4_compress           active                         local
otus  feature@multi_vdev_crash_dump  enabled                        local
otus  feature@spacemap_histogram     active                         local
otus  feature@enabled_txg            active                         local
otus  feature@hole_birth             active                         local
otus  feature@extensible_dataset     active                         local
otus  feature@embedded_data          active                         local
otus  feature@bookmarks              enabled                        local
otus  feature@filesystem_limits      enabled                        local
otus  feature@large_blocks           enabled                        local
otus  feature@large_dnode            enabled                        local
otus  feature@sha512                 enabled                        local
otus  feature@skein                  enabled                        local
otus  feature@edonr                  enabled                        local
otus  feature@userobj_accounting     active                         local
otus  feature@encryption             enabled                        local
otus  feature@project_quota          active                         local
otus  feature@device_removal         enabled                        local
otus  feature@obsolete_counts        enabled                        local
otus  feature@zpool_checkpoint       enabled                        local
otus  feature@spacemap_v2            active                         local
otus  feature@allocation_classes     enabled                        local
otus  feature@resilver_defer         enabled                        local
otus  feature@bookmark_v2            enabled                        local
</pre>
Можно вытащить определенный параметр с момощью grep, а можно указав название параметра в команде 
<pre>
[root@lvm vagrant]# zfs get mountpoint otus
NAME  PROPERTY    VALUE       SOURCE
otus  mountpoint  /otus       default
</pre>
Определяем размер хранилища
<pre>
[root@lvm vagrant]# zfs get available otus
NAME  PROPERTY   VALUE  SOURCE
otus  available  350M   -
</pre>

Определяем тип пула 
<pre>
[root@lvm vagrant]# zfs get readonly otus
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     default
</pre>
Определяем тип сжатия
<pre>
[root@lvm vagrant]# zfs get compression otus
NAME  PROPERTY     VALUE     SOURCE
otus  compression  zle       local
</pre>

Определяем значение recordsize
<pre>
[root@lvm vagrant]# zfs get recordsize otus
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local
</pre>
Определяем контрольную сумму
<pre>
[root@lvm vagrant]# zfs get checksum otus
NAME  PROPERTY  VALUE      SOURCE
otus  checksum  sha256     local  
</pre>

3) Работа со снапшотами
Скачиваем файл
<pre>
[root@lvm vagrant]# wget -O otus_task2.file --no-check-certificate "https://drive.google.com/u/0/uc?id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG&export=download"

</pre>
Создаем файловую систему в otus 
<pre>
[root@lvm vagrant]# zfs create otus/restore
</pre>
Восстанавливаем туда файловую систему 
<pre>
[root@lvm vagrant]# zfs receive otus/restore < otus_task2.file -F
</pre>
Находи в каталоге файл с именем “secret_message”
<pre>
[root@lvm vagrant]# find /otus/restore -name "secret_message"
/otus/restore/task1/file_mess/secret_message
</pre>
Смотрим содержимое данного файла 
<pre>
[root@lvm vagrant]# cat /otus/restore/task1/file_mess/secret_message
https://github.com/sindresorhus/awesome
</pre>
