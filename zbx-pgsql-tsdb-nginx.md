# zbx-pgsql-tsdb-nginx
Установка и настройка связки Zabbix 5.4 + PostgreSQL 13 + TimescaleDB + NGiNX на Debian 11 (в контейнерах LXC на Proxmox VE).


## Трехкомпонентная установка

- Установка репозитория Zabbix:  
```sh
wget https://repo.zabbix.com/zabbix/5.4/debian/pool/main/z/zabbix-release/zabbix-release_5.4-1+debian11_all.deb
dpkg -i zabbix-release_5.4-1+debian11_all.deb
apt update -y
```

### Установка и настройка узла zbx-db    
```shell
apt install -y \
    gnupg \
    gnupg1 \
    gnupg2 \
    postgresql-13 \
    zabbix-agent

# Создаем пользователя с паролем zabbix_password1:   
su - postgres -c 'createuser --pwprompt zabbix_user1'
su - postgres -c 'createdb -O zabbix_user1 zabbix_db1'
```

#### Установка и настройка TimescaleDB 
```shell
# Установка и настройка TimescaleDB:
sh /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
#
sh -c "echo 'deb [signed-by=/usr/share/keyrings/timescale.keyring] https://packagecloud.io/timescale/timescaledb/debian/ $(lsb_release -c -s) main' > /etc/apt/sources.list.d/timescaledb.list"

wget --quiet -O - https://packagecloud.io/timescale/timescaledb/gpgkey | gpg --dearmor -o /usr/share/keyrings/timescale.keyring

apt-get update -y && apt-get upgrade -y
apt install -y timescaledb-2-postgresql-13
timescaledb-tune --quiet --yes
systemctl restart postgresql.service
```

В файле `/etc/postgresql/13/main/postgresql.conf` раскомментируем строку и изменяем её значение:  
`listen_addresses = '*' `  

```shell
sed -i "s/#listen_addresses = 'localhost'/listen_addresses = '*'/" /etc/postgresql/13/main/postgresql.conf
echo 'host zbx-db-1 zabbix_user1 <zbx_server_ip>/32 md5' >> /etc/postgresql/13/main/pg_hba.conf
systemctl restart postgresql.service
```

### Установка и настройка узла zbx-srv (Zabbix 5.4.6)
```shell
apt install -y \
    nmap \
    postgresql-client-13 \
    snmp \
    zabbix-agent \
    zabbix-server-pgsql \
    zabbix-sql-scripts
```  


Импорт начальной схемы и данных:  
```shell
zcat /usr/share/doc/zabbix-sql-scripts/postgresql/create.sql.gz | psql -h 192.168.0.1 -U zabbix_user1 -d zbx-db-1

psql -c 'CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;' -h 192.168.0.1 -U zabbix_user1 -d zbx-db-1

psql -f /usr/share/doc/zabbix-sql-scripts/postgresql/timescaledb.sql -h 192.168.0.1 -U zabbix_user1 -d zbx-db-1
```
 
Признаком успешного выполнения является сообщение:
```
ПРЕДУПРЕЖДЕНИЕ:  
WELCOME TO
 _____ _                               _     ____________  
|_   _(_)                             | |    |  _  \ ___ \ 
  | |  _ _ __ ___   ___  ___  ___ __ _| | ___| | | | |_/ / 
  | | | |  _ ` _ \ / _ \/ __|/ __/ _` | |/ _ \ | | | ___ \ 
  | | | | | | | | |  __/\__ \ (_| (_| | |  __/ |/ /| |_/ /
  |_| |_|_| |_| |_|\___||___/\___\__,_|_|\___|___/ \____/
               Running version 2.4.2
For more information on TimescaleDB, please visit the following links:

 1. Getting started: https://docs.timescale.com/timescaledb/latest/getting-started
 2. API reference documentation: https://docs.timescale.com/api/latest
 3. How TimescaleDB is designed: https://docs.timescale.com/timescaledb/latest/overview/core-concepts

Note: TimescaleDB collects anonymous reports to better understand and assist our users.
For more information and how to disable, please see our docs https://docs.timescale.com/timescaledb/latest/how-to-guides/configuration/telemetry.

CREATE EXTENSION
```

В итоге получаем: 
```
ЗАМЕЧАНИЕ:  PostgreSQL version 13.4 (Debian 13.4-4.pgdg110+1) is valid
ЗАМЕЧАНИЕ:  TimescaleDB extension is detected
ЗАМЕЧАНИЕ:  TimescaleDB version 2.4.2 is valid
ЗАМЕЧАНИЕ:  TimescaleDB is configured successfully
DO
```

Оптимизированные настройки Zabbix-сервера:  
```shell
sed -i 's/# StartPollers=5/StartPollers=200/' /etc/zabbix/zabbix_server.conf
sed -i 's/# StartPollersUnreachable=1/StartPollersUnreachable=100/' /etc/zabbix/zabbix_server.conf
sed -i 's/# StartPingers=1/StartPingers=50/' /etc/zabbix/zabbix_server.conf
sed -i 's/# StartTrappers=5/StartTrappers=10/' /etc/zabbix/zabbix_server.conf
sed -i 's/# StartDiscoverers=1/StartDiscoverers=15/' /etc/zabbix/zabbix_server.conf
sed -i 's/# StartPreprocessors=3/StartPreprocessors=15/' /etc/zabbix/zabbix_server.conf
sed -i 's/# StartHTTPPollers=1/StartHTTPPollers=5/' /etc/zabbix/zabbix_server.conf
sed -i 's/# StartAlerters=3/StartAlerters=5/' /etc/zabbix/zabbix_server.conf
sed -i 's/# StartTimers=1/StartTimers=2/' /etc/zabbix/zabbix_server.conf
sed -i 's/# StartEscalators=1/StartEscalators=2/' /etc/zabbix/zabbix_server.conf
sed -i 's/# CacheSize=8M/CacheSize=4096M/' /etc/zabbix/zabbix_server.conf
sed -i 's/# HistoryCacheSize=16M/HistoryCacheSize=64M/' /etc/zabbix/zabbix_server.conf
sed -i 's/# HistoryIndexCacheSize=4M/HistoryIndexCacheSize=32M/' /etc/zabbix/zabbix_server.conf
sed -i 's/# TrendCacheSize=4M/TrendCacheSize=32M/' /etc/zabbix/zabbix_server.conf
sed -i 's/# ValueCacheSize=8M/ValueCacheSize=256M/' /etc/zabbix/zabbix_server.conf
sed -i 's/Timeout=4/Timeout=30/' /etc/zabbix/zabbix_server.conf

systemctl enable zabbix-server.service
systemctl start zabbix-server.service
```

### Установка и настройка узла zbx-web (NGiNX)
```shell
apt install -y \
    nginx \
    php7.4-pgsql \
    zabbix-frontend-php \
    zabbix-nginx-conf \
    zabbix-agent 
```
В файле `/etc/php/7.4/fpm/php.ini` необходимо изменить некоторые параметры:  
```shell
sed -i 's/post_max_size = 8M/post_max_size = 16M/' /etc/php/7.4/fpm/php.ini
sed -i 's/max_execution_time = 30/max_execution_time = 300/' /etc/php/7.4/fpm/php.ini
sed -i 's/max_input_time = 60/max_input_time = 300/' /etc/php/7.4/fpm/php.ini
sed -i 's/;date.timezone =/date.timezone = Europe\/Moscow/' /etc/php/7.4/fpm/php.ini
``` 

В файле `/etc/nginx/sites-available/default` изменяем следующие строки:  
```shell
sed -i 's/#root \/var\/www\/html;/root \/usr\/share\/zabbix;/' /etc/nginx/sites-available/default
sed -i 's/index index.html index.htm index.nginx-debian.html;/index index.php index.html index.htm index.nginx-debian.html;/' /etc/nginx/sites-available/default
```

Добавляем раздел:  
```shell
location ~ .php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
}
```
В файле `/etc/zabbix/nginx.conf`:  
```shell
listen          80;
server_name     zabbix;
```


```shell
systemctl restart zabbix-agent nginx php7.4-fpm
systemctl enable zabbix-agent nginx php7.4-fpm 
```
В конце веб-установщик генерирует конфигурационный файл  
```shell
/usr/share/zabbix/conf/zabbix.conf.php
```

## Возможные проблемы
-- Не работает ping из графического интерфейса
Решение: на сервере выполнить команду `setcap CAP_NET_RAW+p /bin/ping`
