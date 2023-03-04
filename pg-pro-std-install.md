# Установка и настройка Postgres Pro

## Содержание
- [Введение](#intro)  
- [Установка](#setup)  
  - [Подготовительные действия](#prepare)
  - [Установка Postgres Pro Standard 14](#setup-pg-pro-std-14)
  - [Установка Postgres Pro Standard 15](#setup-pg-pro-std-15)
    - [Подключение репозитория]()
    - [Установка]()
- [Настройка](#config)
- [Администрирование](#admin)
- [Ссылки на полезные ресурсы](#links)

## Введение <a name="intro"></a>
В этом документе рассматривается типовое развёртывание `Postgres Pro Standard` и её отдельных компонентов на примере [тестового стенда](https://github.com/che-a/test-stands/tree/master/virtualbox).

| № | Имя узла   | Роль                          | Соответствие узлу тестового стенда |
| - | ---------- |-------------------------------|------------------------------------|
| 1 | `pgpro`    | Сервер Postgres Pro Standart  | `srv-01`                           |
| 2 | `psql`     | Консольный клиент psql        | `srv-02`                           |
| 3 | `pgadmin4` | Веб-клиент pgadmin4           | `srv-03`                           |


## Установка <a name="setup"></a>
### Подготовительные действия <a name="prepare"></a>
На каждом узле необходимо выполнить следующий набор команд:
```shell
apt update && \
apt upgrade -y && \
apt install -y gnupg gnupg1 gnupg2
```

### Установка Postgres Pro Standard 14 <a name="setup-pg-pro-std-14"></a>
#### Настройка репозитория
Для узлов `pgpro` и `psql`:
```shell
wget http://repo.postgrespro.ru/pgpro-14/keys/GPG-KEY-POSTGRESPRO && \
    apt-key add GPG-KEY-POSTGRESPRO

echo deb http://repo.postgrespro.ru/pgpro-14/debian/ bullseye main > /etc/apt/sources.list.d/postgrespro.list
apt update
```

Для узла `pgpro`:
```shell
apt install -y postgrespro-std-14
```
Для узла `psql`:
```shell
apt install -y postgrespro-std-14-client

# Для того, чтобы вызывать бинарные файлы PostgreSQL без указания пути создадим необходимые символические ссылки:
/opt/pgpro/std-14/bin/pg-wrapper links update
```

### Установка Postgres Pro Standard 15 <a name="setup-pg-pro-std-15"></a>
#### Подключение репозитория
```shell
wget https://repo.postgrespro.ru/std-15/keys/pgpro-repo-add.sh
sh pgpro-repo-add.sh
```
#### Установка
Если наш продукт единственный Postgres на вашей машине и вы хотите сразу получить готовую к употреблению базу:
```shell
apt-get install postgrespro-std-15
```
Если у вас уже установлен другой Postgres и вы хотите чтобы он продолжал работать параллельно (в том числе и для апгрейда с более старой major-версии):
```shell
apt-get install postgrespro-std-15-contrib
/opt/pgpro/std-15/bin/pg-setup initdb
/opt/pgpro/std-15/bin/pg-setup service enable
/opt/pgpro/std-15/bin/pg-setup service start
```
Если вы хотите создать базу пригодную для использования с продуктами `1С`:
```shell
apt-get install postgrespro-std-15-contrib
/opt/pgpro/std-15/bin/pg-setup initdb --tune=1c
/opt/pgpro/std-15/bin/pg-setup service enable
/opt/pgpro/std-15/bin/pg-setup service start
```
В состав `Postgres Pro Standard` входят многочисленные дополнительные компоненты, которые могут быть установлены с помощью apt-get после установки собственно Postgres:
```shell
orafce-std-15 3.24.4
pg-filedump-std-15 14.4
pg-portal-modify-std-15 0.3.3
pg-probackup-std-15 2.6.0
pg-repack-std-15 1.4.8
pgbouncer 1.18.0
pgpro-controldata 15.1.0
pgpro-stats-std-15 1.5
pldebugger-std-15 1.1.4
postgrespro-std-15 15.2.1
postgrespro-std-15-client 15.2.1
postgrespro-std-15-contrib 15.2.1
postgrespro-std-15-devel 15.2.1
postgrespro-std-15-libs 15.2.1
postgrespro-std-15-plperl 15.2.1
postgrespro-std-15-plpython3 15.2.1
postgrespro-std-15-pltcl 15.2.1
postgrespro-std-15-server 15.2.1
tds-fdw-std-15 2.0.2
postgrespro-std-15-jit 15.2.1
plv8-std-15 3.1.5
mamonsu 3.5.2
pgpro-pgbadger 11.6
pgpro-pwr-std-15 4.1.1
postgrespro-std-15-docs 15.2.1
postgrespro-std-15-docs-ru 15.2.1
libsybdb5 1.1.36
oracle-fdw-std-15 2.5.0
postgrespro-std-15-test 15.2.1
freetds 1.3.3
freetds-libs 1.3.3
liblz4-1_7 131
liblz4-devel 131
lz4-devel 1.8.3
```


## Настройка <a name="config"></a>
### Каталог с файлами настроек
```shell
# Так можно узнать расположение главного конфигурационного файла
su - postgres -c 'psql -c "SHOW config_file;"'
```
```
                config_file                 
--------------------------------------------
 /var/lib/pgpro/std-14/data/postgresql.conf
(1 строка)
```

### Настройка подключения
В файле `/var/lib/pgpro/std-14/data/postgresql.conf` раскомментировать директиву `listen_addresses` и указать IP-адрес сервера, с которого будет доступно внешнее подключение.
```shell
listen_addresses = '192.168.0.11'    # what IP address(es) to listen on; 
# или
# listen_addresses = '*'
```
В файле `/var/lib/pgpro/std-14/data/pg_hba.conf` настраивается подлкючение к СУБД.
```sh
# Нешифрованный доступ к wiki_db
host    wiki_db        wiki_user        192.168.0.12/32      md5
host    wiki_db        wiki_user        192.168.0.13/32      md5

# Шифрованный (TLS) доступ к cloud_db
hostssl cloud_db       cloud_user       192.168.0.12/32      cert
hostssl cloud_db       cloud_user       192.168.0.13/32      cert
```
Если в файле `/var/lib/pgpro/std-14/data/pg_hba.conf` настроено подключение пользователя `postgres`, то необходимо для него установить пароль:
```sh
su - postgres -c 'psql -c "\password postgres"'
```

#### Создание сертификатов
Для организации шифрованного TLS-соединения необходмы сертификаты.
```sh
# Корневой сертификат ЦС
openssl req -new -nodes -text -out root.csr \
  -keyout root.key -subj "/CN=root.lan"
chmod og-rwx root.key
openssl x509 -req -in root.csr -text -days 3650 \
  -extfile /etc/ssl/openssl.cnf -extensions v3_ca \
  -signkey root.key -out root.crt

# Сертификат сервера pgpro
openssl req -new -nodes -text -out server.csr \
  -keyout server.key -subj "/CN=pgpro.lan"
chmod og-rwx server.key
openssl x509 -req -in server.csr -text -days 365 \
  -CA root.crt -CAkey root.key -CAcreateserial \
  -out server.crt
  
# Сертификат клиента (пользователь cloud_user)
openssl req -new -nodes -text -out client.csr \
  -keyout client.key -subj "/CN=cloud_user"
chmod og-rwx client.key
openssl x509 -req -in client.csr -text -days 365 \
  -CA root.crt -CAkey root.key -CAcreateserial \
  -out client.crt  
```
Выходные файлы:  
|  № | Файл             | Назначение                | Место хранения        |
|----|------------------|---------------------------|-----------------------|
|  1 | root.csr         | Запрос на получение серт. |                       |
|  2 | root.key         | Закрытый ключ             | В изолированном месте |
|  3 | root.crt         | Сертификат корневого ЦС   | На клиенте            |
|  4 | root.srl         |                           |                       |
|  5 | server.csr       |                           |                       |
|  6 | server.key       |                           | На сервере            |
|  7 | server.crt       |                           | На сервере            |
|  8 | client.csr       |                           |                       |
|  9 | client.key       |                           |                       |
| 10 | client.crt       |                           |                       |


##### Установка сертификатов на сервер
Файлы сертификатов необходимо поместить в каталог: `/var/lib/pgpro/std-14/data`.  
```sh
scp root.crt server.{key,crt} pgpro:/var/lib/pgpro/std-14/data/
ssh pgpro "chown postgres: /var/lib/pgpro/std-14/data/server.{key,crt} /var/lib/pgpro/std-14/data/root.crt"
ssh pgpro "chmod 600 /var/lib/pgpro/std-14/data/server.crt"
```
Настройка `TLS` в файле `/var/lib/pgpro/std-14/data/postgresql.conf`
```sh
# - SSL -

ssl = on
ssl_ca_file = 'root.crt'
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
ssl_ciphers = 'HIGH:MEDIUM:+3DES:!aNULL' # allowed SSL ciphers
```
```shell
systemctl restart postgrespro-std-14.service
```
##### Установка сертификатов на клиент
```sh
ssh psql "mkdir /root/.postgresql/"

scp root.crt psql:/root/.postgresql/root.crt
scp client.key psql:/root/.postgresql/postgresql.key
scp client.crt psql:/root/.postgresql/postgresql.crt

ssh psql "chmod 600 /root/.postgresql/postgresql.key"
```
```sh
ssh pgadmin "mkdir -p /var/www/.postgresql/"

scp root.crt pgadmin:/var/www/.postgresql/root.crt
scp client.key pgadmin:/var/www/.postgresql/postgresql.key
scp client.crt pgadmin:/var/www/.postgresql/postgresql.crt

ssh pgadmin "chmod 600 /var/www/.postgresql/postgresql.key"
```

#### Подключение к серверу
Удаленное подключение консольного клиента к серверу:
```sh
psql -U user2 -d db1 -h pgpro-1.lan
```

## Настройка клиента
### Установка и настройка pgAdmin4
```sh
wget https://www.pgadmin.org/static/packages_pgadmin_org.pub
apt-key add packages_pgadmin_org.pub

echo "deb https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/$(lsb_release -cs) pgadmin4 main" > /etc/apt/sources.list.d/pgadmin4.list

apt update
apt install pgadmin4-web
```
Далее необходимо запустить скрипт начальной настройки сервера:
```sh
/usr/pgadmin4/bin/setup-web.sh
```

#### Настройка HTTPS на веб-сервере
Создание самоподписанного сертификата:
```sh
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /etc/ssl/private/pgadm4-selfsigned.key -out /etc/ssl/certs/pgadm4-selfsigned.crt
```
В файле `/etc/apache2/sites-available/default-ssl.conf` изменим строки:
```sh
SSLCertificateFile      /etc/ssl/certs/pgadm4-selfsigned.crt
SSLCertificateKeyFile /etc/ssl/private/pgadm4-selfsigned.key
```
Далее необходимо создать файл `/etc/apache2/conf-available/ssl.conf` и внести в него следующие строки:
```sh
SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
SSLCipherSuite ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
SSLHonorCipherOrder off
SSLSessionTickets off

SSLUseStapling On
SSLStaplingCache "shmcb:logs/ssl_stapling(32768)"
```
```sh
# Сохраняем изменения и включаем нужные модули, конфигурации и виртуальные хосты:
a2enmod ssl
a2enconf ssl
a2ensite default-ssl

# Проверяем конфигурацию Apache на ошибки:
apachectl -t

# И перезапускаем веб-сервер:
systemctl reload apache2
```

#### Настройка автоматической переадресации с HTTP на HTTPS
Откроем файл `/etc/apache2/sites-available/000-default.conf` и в пределах секции `VirtualHost` внесем следующие строки:
```sh
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI}
```
```sh
# Подключим необходимые модули:
a2enmod rewrite

# И перезапустим веб-сервер:
systemctl reload apache2
```

## Администрирование <a name="admin"></a>
### Создание пользователей и БД
```shell
su - postgres -c psql
```
```sql
CREATE USER wiki_user WITH NOCREATEDB NOCREATEROLE NOSUPERUSER ENCRYPTED PASSWORD 'password';
CREATE DATABASE wiki_db WITH OWNER wiki_user;

CREATE USER cloud_user WITH NOCREATEDB NOCREATEROLE NOSUPERUSER ENCRYPTED PASSWORD 'password';
CREATE DATABASE cloud_db WITH OWNER cloud_user;

-- DROP DATABASE db;
-- DROP USER user;
```


## Ссылки на полезные ресурсы <a name="links"></a>
- [Защита соединений TCP/IP с применением SSL](https://postgrespro.ru/docs/postgrespro/14/ssl-tcp)
- [Поддержка SSL](https://postgrespro.ru/docs/postgrespro/14/libpq-ssl)
- [Файл pg_hba.conf](https://postgrespro.ru/docs/postgrespro/14/auth-pg-hba-conf)
- [Установка и настройка pgAdmin 4 в режиме сервера](https://interface31.ru/tech_it/2021/01/ustanovka-i-nastroyka-pgadmin-4-v-rezhime-servera.html)
- [Установка Postgres Pro 10 для 1С:Предприятие на Debian / Ubuntu](https://interface31.ru/tech_it/2018/10/ustanovka-postgresql-10-dlya-1spredpriyatie-na-debian-ubuntu.html)
- [ Настройка PostgreSQL для работы с клиентами через SSL](http://www.zaweel.ru/2016/08/postgresql-ssl.html)
