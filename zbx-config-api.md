# ZabbixAPI

```sh
# Настройка параметров веб-интерфейса
/usr/share/zabbix/include/defines.inc.php
```

Файл `zabbix_server.conf`
```
# Настройки производительности
StartPollers=45
StartPollersUnreachable=10
StartPingers=15
CacheSize=4096M
Timeout=30
LogSlowQueries=3000


ExternalScripts=/opt/scripts
```

Файл `zabbix.json`
```json
{
    "protocol": "HTTP",
    "ip": "192.168.0.1",
    "login": "Admin",
    "password": "zabbix"
}
```

```python
import json
import requests


def zbx_load_auth_from_file(auth_file):
    """Загрузка учетных данных Zabbix-сервера из JSON-файла, используя которые будет получен токен."""
    with open(auth_file, 'r', encoding='utf-8') as af:
        auth_data = json.load(af)
        auth_data["url"] = auth_data["protocol"].lower() + "://" + auth_data["ip"] + "/api_jsonrpc.php"
        return auth_data


def zbx_apiinfo_version(url, id=1):
    """Получении версии Zabbix и его API"""
    request_json = f"""
        {{
            "jsonrpc":"2.0",
            "method":"apiinfo.version",
            "id":{id},
            "auth":null,
            "params":{{}}
        }}
    """
    r = requests.post(url, headers = {'content-type': 'application/json-rpc'}, data=request_json)
    return json.loads(r.content)


def zbx_get_token(url, login, password, id=1):
    """Аутентификация (получение токена) на Zabbix-сервере."""

    request_json = f"""
        {{
            "jsonrpc": "2.0",
            "method": "user.login",
            "params": {{
                "user": "{login}",
                "password": "{password}"
            }},
            "id": {id},
            "auth": null
        }}
    """
    r = requests.post(url, headers={'content-type': 'application/json-rpc'}, data=request_json)
    return json.loads(r.content)


def zbx_hostgroup_create(url, token, group_name, id=1):
    """Создание группы узлов сети"""
    request_json = f"""
        {{
            "jsonrpc": "2.0",
            "method": "hostgroup.create",
            "params": {{
                "name": "{group_name}"
            }},
            "auth": "{token}",
            "id": {id}
        }}
    """
    r = requests.post(url, headers={'content-type': 'application/json-rpc'}, data=request_json)
    return json.loads(r.content)


def main():
    zbx = zbx_load_auth_from_file('zabbix.json')
    zbx["version"] = zbx_apiinfo_version(zbx["url"])["result"]
    zbx["token"] =  zbx_get_token(zbx["url"], zbx["login"], zbx["password"])["result"]
    print(zbx)

    for i in range(1, 11):
        zbx_hostgroup_create(zbx["url"], zbx["token"], f"FIT Python group {i}")


if __name__ == "__main__":
    main()

```
