## Задание 1


Вам необходимо поднять в докере и связать между собой:

- elasticsearch (hot и warm ноды);
- logstash;
- kibana;
- filebeat.

Logstash следует сконфигурировать для приёма по tcp json-сообщений.

Filebeat следует сконфигурировать для отправки логов docker вашей системы в logstash.

В директории [help](./help) находится манифест docker-compose и конфигурации filebeat/logstash для быстрого 
выполнения этого задания.

Результатом выполнения задания должны быть:

- скриншот `docker ps` через 5 минут после старта всех контейнеров (их должно быть 5);
- скриншот интерфейса kibana;
- docker-compose манифест (если вы не использовали директорию help);
- ваши yml-конфигурации для стека (если вы не использовали директорию help).

**Ответ.**

- <img width="1580" height="156" alt="monitor_homework2 0" src="https://github.com/user-attachments/assets/673095b1-7b6f-4fb9-b29b-f5d504ba1c20" />

- <img width="1789" height="969" alt="monitor_homework2 1" src="https://github.com/user-attachments/assets/ee8b0956-6169-4903-a02d-22a81d5218b9" />


---
