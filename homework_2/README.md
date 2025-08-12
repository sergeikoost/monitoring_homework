### Задание повышенной сложности
При решении задания 1 не используйте директорию help для сборки проекта. Самостоятельно разверните grafana, где в роли источника данных будет выступать prometheus, а сборщиком данных будет node-exporter.

**Ответ.**

Решил сам поднять Grafana server, в качестве цели мониторинга создал отдельную вм на которой будет подниматься prometheus и node_exporter.

### Задание 1

1. Используя директорию [help](./help) внутри этого домашнего задания, запустите связку prometheus-grafana. (ДЕЛАЕМ ПО ПОВЫШЕННОЙ СЛОЖНОСТИ)
1. Зайдите в веб-интерфейс grafana, используя авторизационные данные, указанные в манифесте docker-compose.
1. Подключите поднятый вами prometheus, как источник данных.
1. Решение домашнего задания — скриншот веб-интерфейса grafana со списком подключенных Datasource.


**Ответ.**


Конфиг prometheus.yml: 

```
# /etc/prometheus/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']

rule_files:
  - "alerts.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter-remote'
    static_configs:
      - targets: ['84.252.136.35:9100'] 
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']

```

Конфиг alerts.yml:

```

groups:
  - name: instance-down
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 30s
        labels:
          severity: critical
        annotations:
          summary: "VM {{ $labels.instance }} unreachable"

  - name: high-cpu-load
    rules:
      - alert: HighCpuLoad
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High CPU utilization on {{ $labels.instance }}"
```


<img width="1916" height="451" alt="monitor_homework1 3" src="https://github.com/user-attachments/assets/721fbe91-218c-4fe3-8c6c-057c32ef31f7" />


prometheus.service:

```

# /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus/ \
    --web.listen-address=0.0.0.0:9090 \
    --web.enable-lifecycle \
    --web.external-url=http://130.193.41.161:9090
Restart=always

[Install]
WantedBy=multi-user.target
```

node-exporter.service на удаленном хосте:

```

# /etc/systemd/system/node-exporter.service
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node-exporter
Group=node-exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
```

При помощи alertmanager сделал нотификации через телеграм бота, alertmanager.yml:

```
route:
  receiver: 'telegram'
  group_wait: 10s
  group_interval: 1m
  repeat_interval: 1h

receivers:
  - name: 'telegram'
    telegram_configs:
      - api_url: 'https://api.telegram.org'
        bot_token: 'xxxxxx'
        chat_id: xxxxxx
        send_resolved: true
        message: |
          {{ template "telegram.default.message" . }}
```


В конце импортировал официальный дашборд https://grafana.com/grafana/dashboards/1860-node-exporter-full/?spm=a2ty_o01.29997173.0.0.1e90c921jj4qR7

#### Получаем такое:

<img width="1683" height="951" alt="monitor_homework1 2" src="https://github.com/user-attachments/assets/6f23962f-5305-4537-a6ef-2128d12481cd" />


#### Уведомления в чат-бота тоже приходят корректно:

<img width="584" height="211" alt="monitor_homework1 4" src="https://github.com/user-attachments/assets/3139fce4-8ff3-4a2b-8e53-faa4c6c63401" />

<img width="1907" height="535" alt="monitor_homework1 5" src="https://github.com/user-attachments/assets/2b6cbf84-89ad-4ab5-9e69-231be01e60ef" />


### Задание 2

Создайте Dashboard и в ней создайте Panels:

- утилизация CPU для nodeexporter (в процентах, 100-idle);
- CPULA 1/5/15;
- количество свободной оперативной памяти;
- количество места на файловой системе.

Для решения этого задания приведите promql-запросы для выдачи этих метрик, а также скриншот получившейся Dashboard.


**Ответ.**

- Утилизация CPU для nodeexporter в %:
```
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```
- CPU Load Average 1/5/15:
```
node_load1{instance="84.252.136.35:9100"}
node_load5{instance="84.252.136.35:9100"}
node_load15{instance="84.252.136.35:9100"}
```
- Количество свободной ОЗУ:
```
node_memory_MemAvailable_bytes{instance="84.252.136.35:9100"} / 1024 / 1024
```
- Количество места на файловой системе в %:
```
100 - ((node_filesystem_avail_bytes{instance="ваш_хост:9100", fstype!~"tmpfs|squashfs|autofs"} * 100) / node_filesystem_size_bytes{instance="ваш_хост:9100", fstype!~"tmpfs|squashfs|autofs"})```
```

<img width="1678" height="802" alt="monitor_homework1 6" src="https://github.com/user-attachments/assets/fa845992-88a4-4673-8f12-9348e9a4795a" />


## Задание 3

1. Создайте для каждой Dashboard подходящее правило alert — можно обратиться к первой лекции в блоке «Мониторинг».
2. В качестве решения задания приведите скриншот вашей итоговой Dashboard.

**Ответ.**
<img width="1605" height="888" alt="monitor_homework1 7" src="https://github.com/user-attachments/assets/41e33409-de88-4955-96d3-da192912fbd6" />


## Задание 4

1. Сохраните ваш Dashboard.Для этого перейдите в настройки Dashboard, выберите в боковом меню «JSON MODEL». Далее скопируйте отображаемое json-содержимое в отдельный файл и сохраните его.
1. В качестве решения задания приведите листинг этого файла.

<img width="955" height="868" alt="monitor_homework1 8" src="https://github.com/user-attachments/assets/7b6cda38-0bb2-4ed9-a4fe-832b65c4e0d6" />
