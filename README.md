# Домашнее задание к занятию «ELK»

**Выполнил:** Чернобровкин Иван

Решение выполнено в Docker Compose на базе Elasticsearch, Kibana, Logstash и Filebeat версии 7.17.9. Для создания access-логов используется Nginx.

## Подготовка

Для запуска желательно выделить виртуальной машине не менее 4 ГБ оперативной памяти и 10 ГБ свободного места.

```bash
git clone https://github.com/Riffshadow/-ELK.git
cd ./-ELK
sudo sysctl -w vm.max_map_count=262144
docker compose version
```

Чтобы параметр `vm.max_map_count` сохранялся после перезагрузки:

```bash
echo 'vm.max_map_count=262144' | sudo tee /etc/sysctl.d/99-elasticsearch.conf
sudo sysctl --system
```

## Задание 1. Elasticsearch

Запуск Elasticsearch:

```bash
docker compose up -d elasticsearch
docker compose ps
```

Проверка состояния кластера:

```bash
curl -X GET 'localhost:9200/_cluster/health?pretty'
```

В результате отображается нестандартное имя кластера:

```text
"cluster_name" : "riffshadow-elk-7421"
```

Скриншот результата:

![Состояние кластера Elasticsearch](screenshots/task-1-elasticsearch.png)

## Задание 2. Kibana

Запуск Kibana:

```bash
docker compose up -d kibana
docker compose ps
```

После запуска интерфейс доступен по адресу:

```text
http://<IP-адрес-сервера>:5601/app/dev_tools#/console
```

IP-адрес виртуальной машины можно посмотреть командой:

```bash
hostname -I
```

В Kibana Dev Tools выполнен запрос:

```http
GET /_cluster/health?pretty
```

Скриншот результата:

![Запрос в Kibana Dev Tools](screenshots/task-2-kibana.png)

## Задание 3. Logstash

Запуск Nginx и Logstash:

```bash
docker compose up -d nginx
docker compose --profile logstash up -d logstash
docker compose ps
```

Создание тестовых обращений к Nginx:

```bash
for i in $(seq 1 20); do curl -s "http://localhost:8080/?request=$i" > /dev/null; done
```

Проверка Logstash и созданного индекса:

```bash
docker compose logs --tail=50 logstash
curl -s 'localhost:9200/_cat/indices/nginx-logstash-*?v'
```

В Kibana создан шаблон индекса `nginx-logstash-*` с полем времени `@timestamp`. В разделе **Discover** выбран этот шаблон. Поле `delivery` имеет значение `logstash`.

Скриншот логов Nginx, доставленных через Logstash:

![Логи Nginx из Logstash](screenshots/task-3-logstash.png)

## Задание 4. Filebeat

Поставка логов через Logstash остановлена, после чего запущен Filebeat:

```bash
docker compose stop logstash
docker compose --profile filebeat up -d filebeat
```

После переключения созданы новые обращения к Nginx:

```bash
for i in $(seq 21 40); do curl -s "http://localhost:8080/?request=$i" > /dev/null; done
```

Проверка Filebeat и созданного индекса:

```bash
docker compose logs --tail=50 filebeat
curl -s 'localhost:9200/_cat/indices/nginx-filebeat-*?v'
```

В Kibana создан шаблон индекса `nginx-filebeat-*` с полем времени `@timestamp`. В разделе **Discover** выбран этот шаблон. Поле `delivery` имеет значение `filebeat`.

Скриншот логов Nginx, доставленных через Filebeat:

![Логи Nginx из Filebeat](screenshots/task-4-filebeat.png)

## Проверка и диагностика

Состояние контейнеров:

```bash
docker compose ps
```

Последние сообщения нужного контейнера:

```bash
docker compose logs --tail=100 elasticsearch
docker compose logs --tail=100 kibana
docker compose logs --tail=100 logstash
docker compose logs --tail=100 filebeat
```

Полная остановка стенда без удаления данных:

```bash
docker compose --profile logstash --profile filebeat down
```

> Команда `docker compose down -v` дополнительно удалит все тестовые индексы и служебные данные из Docker volumes.

## Отправка решения

После добавления скриншотов в каталог `screenshots`:

```bash
git add .
git commit -m "Выполнено домашнее задание ELK"
git push origin main
```

Ссылка на решение:

<https://github.com/Riffshadow/-ELK/blob/main/README.md>
