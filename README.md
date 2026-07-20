
README-corrected.md


# Домашнее задание к занятию «ELK»

**Выполнил:** Чернобровкин Иван

Для выполнения работы использован Docker Compose. В состав стенда входят Elasticsearch, Kibana, Logstash, Filebeat и Nginx версии, указанные в конфигурационных файлах репозитория.

---

## Задание 1. Elasticsearch

### Условие

Установите и запустите Elasticsearch, после чего измените параметр `cluster_name` на случайное значение.

В качестве результата приложите скриншот выполнения команды:

```bash
curl -X GET 'localhost:9200/_cluster/health?pretty'
```

На скриншоте должно быть видно нестандартное имя кластера.

### Решение

В файле `docker-compose.yml` для Elasticsearch задано нестандартное имя кластера:

```yaml
environment:
  - cluster.name=riffshadow-elk-7421
  - discovery.type=single-node
  - xpack.security.enabled=false
  - ES_JAVA_OPTS=-Xms512m -Xmx512m
```

Elasticsearch запущен командой:

```bash
docker compose up -d elasticsearch
```

Состояние контейнера проверено командой:

```bash
docker compose ps
```

Состояние кластера проверено командой:

```bash
curl -X GET 'localhost:9200/_cluster/health?pretty'
```

В результате Elasticsearch вернул нестандартное имя кластера `riffshadow-elk-7421` и состояние `green`.

### Скриншот

![Состояние кластера Elasticsearch](screenshots/task1-elasticsearch.png)

---

## Задание 2. Kibana

### Условие

Установите и запустите Kibana.

В качестве результата приложите скриншот интерфейса Kibana на странице:

```text
http://<IP-адрес-сервера>:5601/app/dev_tools#/console
```

На странице должен быть выполнен запрос:

```http
GET /_cluster/health?pretty
```

### Решение

Kibana запущена командой:

```bash
docker compose up -d kibana
```

Готовность Kibana проверена по журналу контейнера:

```bash
docker compose logs --tail=20 kibana
```

После появления сообщения `Kibana is now available` в браузере открыта страница:

```text
http://localhost:5601/app/dev_tools#/console
```

В консоли Dev Tools выполнен запрос:

```http
GET /_cluster/health?pretty
```

Получен ответ `200 OK`, в котором отображаются имя кластера `riffshadow-elk-7421` и состояние `green`.

### Скриншот

![Запрос в Kibana Dev Tools](screenshots/task2-kibana.png)

---

## Задание 3. Logstash

### Условие

Установите и запустите Logstash и Nginx. С помощью Logstash отправьте журнал доступа Nginx в Elasticsearch.

В качестве результата приложите скриншот интерфейса Kibana, на котором отображаются журналы Nginx.

### Решение

Для Nginx настроено сохранение журнала доступа в файл `/var/log/nginx/access.log`. Этот файл подключён к контейнеру Logstash через общий Docker volume.

Конвейер Logstash описан в файле:

```text
logstash/pipeline/nginx.conf
```

Logstash читает `access.log`, разбирает записи фильтром `grok` и отправляет их в индекс:

```text
nginx-logstash-%{+YYYY.MM.dd}
```

Nginx и Logstash запущены командами:

```bash
docker compose --profile logstash up -d nginx logstash
docker compose --profile logstash ps
```

Поскольку порт `8080` на виртуальной машине занят HAProxy, для Nginx используется порт `8081`.

Тестовое обращение к Nginx выполнено командой:

```bash
curl http://localhost:8081/
```

Создание индекса проверено командой:

```bash
curl 'localhost:9200/_cat/indices/nginx-logstash-*?v'
```

В Kibana создан шаблон индекса `nginx-logstash-*` с полем времени `@timestamp`. В разделе **Discover** отображается журнал обращения к Nginx с кодом ответа `200`.

### Скриншот

![Логи Nginx, отправленные через Logstash](screenshots/task3-logstash-nginx.png)

---

## Задание 4. Filebeat

### Условие

Установите и запустите Filebeat. Переключите доставку журналов Nginx с Logstash на Filebeat.

В качестве результата приложите скриншот интерфейса Kibana, на котором отображаются журналы Nginx, отправленные через Filebeat.

### Решение

Доставка журналов через Logstash остановлена командой:

```bash
docker compose --profile logstash stop logstash
```

Filebeat настроен в файле:

```text
filebeat/filebeat.yml
```

Он читает файл `/var/log/nginx/access.log` и отправляет записи непосредственно в Elasticsearch в индекс:

```text
nginx-filebeat-%{+yyyy.MM.dd}
```

Filebeat запущен командой:

```bash
docker compose --profile filebeat up -d filebeat
```

Состояние контейнеров проверено командой:

```bash
docker compose --profile filebeat ps
```

Для создания новой записи в журнале выполнен запрос:

```bash
curl http://localhost:8081/filebeat-test
```

Ответ `404 Not Found` является ожидаемым: обращение к несуществующей странице было записано в журнал Nginx.

Создание индекса Filebeat проверено командой:

```bash
curl 'localhost:9200/_cat/indices/nginx-filebeat-*?v'
```

В Kibana создан шаблон индекса `nginx-filebeat-*` с полем времени `@timestamp`. В разделе **Discover** отображается запись со следующими признаками:

- `delivery_method: filebeat`;
- запрос `GET /filebeat-test`;
- индекс `nginx-filebeat-2026.07.20`.

### Скриншот

![Логи Nginx, отправленные через Filebeat](screenshots/task4-filebeat-nginx.png)

---

## Задание 5*. Данные о доставке

### Условие

Настройте доставку журнала в Elasticsearch через Logstash и Filebeat для любого другого сервиса, кроме Nginx. Журнал должен записываться в файл, а Logstash должен корректно разобрать его и распределить данные по полям.

В качестве результата приложите скриншот интерфейса Kibana, на котором отображается журнал выбранного приложения.

### Решение

Дополнительное задание не выполнялось.
