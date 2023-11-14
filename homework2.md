создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом
  ```
  Создал вм Centos, установил docker
  поставить на нем Docker Engine
  ```
сделать каталог /var/lib/postgres
развернуть контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgresql
  ```
  [root@centos88-template ~]# ls /var/lib/postgresql
base    pg_commit_ts  pg_hba.conf    pg_logical    pg_notify    pg_serial     pg_stat      pg_subtrans  pg_twophase  pg_wal   postgresql.auto.conf  postmaster.opts
global  pg_dynshmem   pg_ident.conf  pg_multixact  pg_replslot  pg_snapshots  pg_stat_tmp  pg_tblspc    PG_VERSION   pg_xact  postgresql.conf
```
Создал клиент с pg админом и постгресом с помощью композ файла, так же создал еще один контейнер 
```
[root@centos88-template ~]# cat /docker/docker-compose.yml
version: '3.1'

services:

  postgres:
    container_name: postgres_db
    image: postgres:latest
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - postgres:/var/lib/postgresql/data/
    ports:
      - 5555:5432
    networks:
      - postgres

  pgadmin:
    container_name: pgadmin_container
    image: dpage/pgadmin4:7.2
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.ru"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
      PGADMIN_CONFIG_SERVER_MODE: "False"
    volumes:
     - pgadmin:/var/lib/pgadmin
    ports:
      - 5050:80
    restart: unless-stopped
    networks:
      - postgres

volumes:
 postgres:
 pgadmin:

networks:
  postgres:
  ```
```
docker run --name my_psql -e POSTGRES_PASSWORD=postgres -d postgres
[root@centos88-template ~]# docker ps -a
CONTAINER ID   IMAGE                COMMAND                  CREATED          STATUS                  PORTS                                            NAMES
9e5f1dfa886f   postgres             "docker-entrypoint.s…"   12 minutes ago   Up 12 minutes           5432/tcp                                         my_psql
d4970702d3bd   postgres:latest      "docker-entrypoint.s…"   15 minutes ago   Up 15 minutes           0.0.0.0:5555->5432/tcp, :::5555->5432/tcp        postgres_db
71a7dee6f25a   dpage/pgadmin4:7.2   "/entrypoint.sh"         15 minutes ago   Up 15 minutes           443/tcp, 0.0.0.0:5050->80/tcp, :::5050->80/tcp   pgadmin_container
```

развернуть контейнер с клиентом postgres
подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк
подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера
```
Подключился с пгадмина

CREATE TYPE sex AS ENUM (
	'male',
	'female'
);
CREATE TABLE homeworktest(
	user_id serial PRIMARY KEY;,
	user_lastname varchar(30),
	user_firstname varchar(20),
	user_age integer,
	user_sex sex
	);
INSERT INTO homeworktest(user_lastname,user_firstname,user_age,user_sex) VALUES 
('petrov','petr',20,'male'),
('ivanov','ivan',40,'male'),
('petrova','anna',15,'female');

```
удалить контейнер с сервером
```

CONTAINER ID   IMAGE                COMMAND                  CREATED       STATUS                  PORTS                                            NAMES
9e5f1dfa886f   postgres             "docker-entrypoint.s…"   2 hours ago   Up 2 hours              5432/tcp                                         my_psql
d4970702d3bd   postgres:latest      "docker-entrypoint.s…"   2 hours ago   Up 2 hours              0.0.0.0:5555->5432/tcp, :::5555->5432/tcp        postgres_db
71a7dee6f25a   dpage/pgadmin4:7.2   "/entrypoint.sh"         2 hours ago   Up 2 hours              443/tcp, 0.0.0.0:5050->80/tcp, :::5050->80/tcp   pgadmin_container
2707e7bbf9f4   alpine               "bash sh -c 'apk --n…"   4 days ago    Created                                                                  postgres_client
39484429ac87   hello-world          "/hello"                 4 days ago    Exited (0) 4 days ago                                                    nifty_yalow
[root@centos88-template docker]# docker compose down
[+] Running 3/3
 ✔ Container postgres_db        Removed                                                                                                                                                                      0.4s
 ✔ Container pgadmin_container  Removed                                                                                                                                                                      1.5s
 ✔ Network docker_postgres      Removed                                                                                                                                                                      0.3s
[root@centos88-template docker]# docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED       STATUS                  PORTS      NAMES
9e5f1dfa886f   postgres      "docker-entrypoint.s…"   2 hours ago   Up 2 hours              5432/tcp   my_psql
2707e7bbf9f4   alpine        "bash sh -c 'apk --n…"   4 days ago    Created                            postgres_client
39484429ac87   hello-world   "/hello"                 4 days ago    Exited (0) 4 days ago              nifty_yalow

```
создать его заново
подключится снова из контейнера с клиентом к контейнеру с сервером
```
Подключаемся созданому контейнеру
 docker exec -it my_psql bash
Подключаемся к контейнеру удаленному\пересозданному
 psql -U postgres -h 172.27.65.49 -p 5555
Password for user postgres:
psql (16.0 (Debian 16.0-1.pgdg120+1))
Type "help" for help.

postgres=# \l
                                                         List of databases
    Name    |    Owner     | Encoding | Locale Provider |  Collate   |   Ctype    | ICU Locale | ICU Rules |   Access privileges
------------+--------------+----------+-----------------+------------+------------+------------+-----------+-----------------------
 homework_2 | homeworkuser | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           |
 postgres   | postgres     | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           |
 template0  | postgres     | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           | =c/postgres          +
            |              |          |                 |            |            |            |           | postgres=CTc/postgres
 template1  | postgres     | UTF8     | libc            | en_US.utf8 | en_US.utf8 |            |           | =c/postgres          +
            |              |          |                 |            |            |            |           | postgres=CTc/postgres
(4 rows)

postgres=# \c homework_2

homework_2=# \dt
            List of relations
 Schema |     Name     | Type  |  Owner
--------+--------------+-------+----------
 public | homeworktest | table | postgres

homework_2=# select * from homeworktest
homework_2-# ;
 user_lastname | user_firstname | user_age | user_sex | user_id
---------------+----------------+----------+----------+---------
 petrov        | petr           |       20 | male     |       1
 ivanov        | ivan           |       40 | male     |       2
 petrova       | anna           |       15 | female   |       3
(3 rows)

```
проверить, что данные остались на месте
оставляйте в ЛК ДЗ комментарии что и как вы делали и как боролись с проблемами
