# nginx-conf
Настройка nginx


```sh
# Проверка синтаксиса конфигурации
nginx -t
```
## Файл конфигурации
### Директивы
Файлы конфигурации: `/etx/nginx/nginx.conf`
```sh
# Простые директивы
listen 80 default_server;
listen 443 ssl default_server;
root /var/www;

# Директивы блочные (или контексты)

# Директива для обработки соединений
events {}
# http -- это обработка HTTP-трафика (уровень L7 модели OSI)
http {
  server {
    location {
      ...
    }
  }
}
```
### Порядок обработки запроса
1. `listen` <хост>:<порт> .
2. `server_name` значение HTTP-заголовка Host.
3. Если server_name не найдено, то сервер по умолчанию.
```sh
http {
  server {
    listen 80 default_server;
    # _ означает, что все запросы пойдут на default_server
    server_name _;
  }
}
```
### Директива listen
```sh
# По умолчанию: 
listen *:80 | *:8000;

listen 127.0.0.1:8000;
listen 127.0.0.1;
listen 8000;
listen *:8000;
listen localhost:8000;

listen [::]:8000;
listen [::1];

listen unix:/var/run/nginx.sock;
```
