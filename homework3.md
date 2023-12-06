```
root@otuspgs001:~# pg_lsclusters
Ver Cluster Port Status Owner    Data directory               Log file
14  main    5434 online postgres /var/lib/postgresql/14/main  /var/log/postgresql/postgresql-14-main.log
14  main1   5432 online postgres /var/lib/postgresql/14/main1 /var/log/postgresql/postgresql-14-main1.log

```
Создал таблицу, выключил базу, подмонтировал диск, создал каталог, выдал права

```
root@otuspgs001:~# pg_lsclusters
Ver Cluster Port Status Owner    Data directory               Log file
14  main    5434 online postgres /var/lib/postgresql/14/main  /var/log/postgresql/postgresql-14-main.log
14  main1   5432 down   postgres /var/lib/postgresql/14/main1 /var/log/postgresql/postgresql-14-main1.log
root@otuspgs001:~# lsblk
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0                        2:0    1    4K  0 disk
sda                        8:0    0   16G  0 disk
├─sda1                     8:1    0  487M  0 part /boot
├─sda2                     8:2    0    1K  0 part
└─sda5                     8:5    0 15.5G  0 part
  ├─localhost--vg-root   253:0    0 14.6G  0 lvm  /
  └─localhost--vg-swap_1 253:1    0  980M  0 lvm  [SWAP]
sdb                        8:16   0   10G  0 disk
└─sdb1                     8:17   0   10G  0 part /mnt/data1
sr0                       11:0    1 1024M  0 rom

root@otuspgs001:~# ls -lah /mnt/data1
total 24K
drwxr-xr-x 3 postgres postgres 4.0K Nov 24 12:52 .
drwxr-xr-x 4 root     root     4.0K Nov 24 12:51 ..
drwx------ 2 postgres postgres  16K Nov 14 18:01 lost+found
-rw-r--r-- 1 root     root        0 Nov 24 12:52 test

mv /var/lib/postgresql/14/main1/ /mnt/data1/

```
т.к. физически мы переместили файлы, а не указали в параметрах конфига новое месторастположение - получаем ошибку
```
root@otuspgs001:~# pg_ctlcluster 14 main1 start
Error: /var/lib/postgresql/14/main1 is not accessible or does not exist

```


```

#------------------------------------------------------------------------------
# FILE LOCATIONS
#------------------------------------------------------------------------------

# The default values of these variables are driven from the -D command-line
# option or PGDATA environment variable, represented here as ConfigDir.

data_directory = '/var/lib/postgresql/14/main1'         # use data in another directory
                                        # (change requires restart)
hba_file = '/etc/postgresql/14/main1/pg_hba.conf'       # host-based authentication file
                                        # (change requires restart)
ident_file = '/etc/postgresql/14/main1/pg_ident.conf'   # ident configuration file
                                        # (change requires restart)

# If external_pid_file is not explicitly set, no extra PID file is written.
external_pid_file = '/var/run/postgresql/14-main1.pid'                  # write an extra PID file
                                        # (change requires restart)


```
Меняем на 
```
# The default values of these variables are driven from the -D command-line
# option or PGDATA environment variable, represented here as ConfigDir.

data_directory = '/mnt/data1/main1/'            # use data in another directory
                                        # (change requires restart)

```
запускаем инстанс, проверяем наличие значений
```
Ver Cluster Port Status Owner    Data directory               Log file
14  main    5434 online postgres /var/lib/postgresql/14/main  /var/log/postgresql/postgresql-14-main.log
14  main1   5432 online postgres /mnt/data1/main1/            /var/log/postgresql/postgresql-14-main1.log
14  main2   5433 online postgres /var/lib/postgresql/14/main2 /var/log/postgresql/postgresql-14-main2.log


postgres=# select * from test;
 c1
----
 1
(1 row)

```
задание со звездочкой *: не удаляя существующий инстанс ВМ сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось

```
т.к. нет возможности в той виртуализации где я делаю домашки провернуть подобное, не знаю эквивалентно ли:
сделал следующее - создал волюм в докере
[root@centos88-template ~]# docker volume inspect pgsql_vol
[
    {
        "CreatedAt": "2023-12-04T21:36:45+03:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/pgsql_vol/_data",
        "Name": "pgsql_vol",
        "Options": null,
        "Scope": "local"
    }
]
Технически +- тоже самое что и диск, ну и перемапливал его новому контейнеру =)
В принципе будь то физический диск - мы бы так же мапили его в нужный каталог, и получили бы тот же результат.
```
