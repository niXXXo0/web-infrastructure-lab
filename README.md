# RocketDev DevOps Test 

Тестовое задание на позицию Intern/Junior DevOps.

Проект разворачивает WordPress, MariaDB, Nginx, Prometheus, Grafana и Node Exporter с использованием Docker Compose.

## Запуск проекта

### 1. Клонировать репозиторий

git clone https://github.com/niXXXo0/rocket-dev_test.git
cd rocket-dev_test

### 2. Настроить переменные окружения

Создать .env из примера:

cp .env.example .env

При необходимости изменить значения переменных в .env:

DB_NAME=wordpress
DB_USER=wordpress
DB_PASSWORD=change_me
DB_ROOT_PASSWORD=change_me_root

### 3. Настроить локальные доменные имена

Добавить в /etc/hosts доменные имена site.local и metrics.local, указав IP-адрес хоста, на котором запускается Docker:

<IP-адрес> site.local
<IP-адрес> metrics.local

### 4. Настроить SSL

Для site.local используется самоподписанный SSL-сертификат.

Сертификат и приватный ключ необходимо разместить в каталоге:

nginx/ssl/

SSL-сертификаты и приватные ключи не хранятся в репозитории.

### 5. Запустить проект

docker compose up -d

Проверить состояние контейнеров:

docker compose ps

После запуска:

- https://site.local — WordPress;
- http://metrics.local — Grafana.

## Принятые решения

### Docker Compose

Docker Compose используется для запуска и управления всеми сервисами проекта из единого конфигурационного файла. Конфигурационные файлы сервисов хранятся на хосте и монтируются в контейнеры в режиме read-only.

### Nginx

Nginx используется как reverse proxy:

- site.local проксируется на WordPress;
- metrics.local проксируется на Grafana;
- для site.local настроен HTTPS с самоподписанным сертификатом.

Доступ к /wp-admin и /wp-login.php ограничен по IP средствами Nginx.

### WordPress и MariaDB

WordPress работает в отдельном контейнере и использует MariaDB в качестве базы данных.

Параметры подключения и пароли передаются через переменные окружения. Файл .env не хранится в Git, вместо него предоставлен .env.example.

### Prometheus и Node Exporter

Node Exporter собирает системные метрики хоста, а Prometheus используется для их сбора и хранения.

### Grafana

Grafana использует Prometheus как источник метрик.

Через provisioning добавляется dashboard OS General, содержащий требуемые метрики:

- отправленный и полученный сетевой трафик;
- свободное место на диске;
- количество файловых дескрипторов.

### Fail2Ban

Fail2Ban используется на уровне хоста для базовой защиты SSH от перебора паролей.

## Безопасность

Секреты, файл .env, SSL-сертификаты и приватные ключи не хранятся в репозитории.
