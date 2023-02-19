# systemd

Пример systemd-модуля по запуску Docker-контейнера.
```sh
[Unit]
Description=Docker Compose Multiping App for Project
Requires=docker.service
After=docker.service
 
[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/opt/pinger/project
ExecStart=/usr/bin/docker-compose up -d multiping
ExecStop=/usr/bin/docker-compose down multiping
TimeoutStartSec=0
 
[Install]
WantedBy=multi-user.target
```

```sh
systemctl daemon-reload
systemctl enable project.service
systemctl start project.service
```
