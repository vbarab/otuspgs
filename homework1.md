
создать новый проект в Google Cloud Platform, Яндекс облако или на любых ВМ, докере  
Cоздано на рабочей виртуализации  
создать инстанс виртуальной машины с дефолтными параметрами  
```
root@otuspgs001:~# hostnamectl
   Static hostname: otuspgs001
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 6e27552f4691f8bd45a9f4075afbee5b
           Boot ID: f8640d00230d474796a6a06432b14c3c
    Virtualization: vmware
  Operating System: Ubuntu 18.04.6 LTS
            Kernel: Linux 4.15.0-169-generic
      Architecture: x86-64
```
добавить свой ssh ключ в metadata ВМ  
```
root@otuspgs001:~# tree ./.ssh/
./.ssh/
├── authorized_keys
├── id_rsa
└── id_rsa.pub

0 directories, 3 files
```
зайти удаленным ssh (первая сессия), не забывайте про ssh-add  
поставить PostgreSQL  
```
root@otuspgs001:~# apt list --installed | grep postgres

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

postgresql/bionic-pgdg,bionic-pgdg,now 15+250.pgdg18.04+1 all [installed]
postgresql-14/bionic-pgdg,now 14.8-1.pgdg18.04+1 amd64 [installed]
postgresql-15/bionic-pgdg,now 15.3-1.pgdg18.04+1 amd64 [installed,automatic]
postgresql-9.5/xenial-updates,now 9.5.25-0ubuntu0.16.04.1 amd64 [installed,upgradable to: 9.5.25-5.pgdg18.04+1]
postgresql-client-14/bionic-pgdg,now 14.8-1.pgdg18.04+1 amd64 [installed,automatic]
postgresql-client-15/bionic-pgdg,now 15.3-1.pgdg18.04+1 amd64 [installed,automatic]
postgresql-client-9.5/xenial-updates,now 9.5.25-0ubuntu0.16.04.1 amd64 [installed,upgradable to: 9.5.25-5.pgdg18.04+1]
postgresql-client-common/bionic-pgdg,bionic-pgdg,now 250.pgdg18.04+1 all [installed,automatic]
postgresql-common/bionic-pgdg,bionic-pgdg,now 250.pgdg18.04+1 all [installed,automatic]
postgresql-contrib/xenial-updates,xenial-updates,xenial-security,xenial-security,now 9.5+173ubuntu0.3 all [installed,upgradable to: 15+250.pgdg18.04+1]
postgresql-contrib-9.5/xenial-updates,now 9.5.25-0ubuntu0.16.04.1 amd64 [installed,upgradable to: 9.5.25-5.pgdg18.04+1]
```

зайти вторым ssh (вторая сессия)  
```
root@otuspgs001:~# who
root     tty1         2023-10-12 17:03
root     pts/0        2023-10-26 16:53 (172.26.17.133)
root     pts/1        2023-10-26 16:53 (172.26.17.133)
```
запустить везде psql из под пользователя postgres
выключить auto commit
сделатьв первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;

посмотреть текущий уровень изоляции: show transaction isolation level  
начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции  
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');  
сделать select * from persons во второй сессии  
видите ли вы новую запись и если да то почему?  
завершить первую транзакцию - commit;  
сделать select * from persons во второй сессии  
видите ли вы новую запись и если да то почему?  
```
В пока не сделаем комит(не подтвердим изменения)- не появится новой записи во второй сессии, т.к.

 transaction_isolation
-----------------------
 read committed

```
завершите транзакцию во второй сессии  
начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;  
в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');  

сделать select * from persons во второй сессии  
видите ли вы новую запись и если да то почему?  
завершить первую транзакцию - commit;  
сделать select * from persons во второй сессии  
видите ли вы новую запись и если да то почему?  
завершить вторую транзакцию  
сделать select * from persons во второй сессии  
видите ли вы новую запись и если да то почему? ДЗ сдаем в виде миниотчета в markdown в гите
```
в начале не будет никаких изменений, после завершения обеих транзакций - увидим изменения, связано с запретом на фантомное чтение. свой снимок у обеих транзакций.
снимок строится на момент транзакции, а не отдельного оператора. если транзакции много разных операторов и они должны видеть согласованные данные на один и тот же момент времени.
```


