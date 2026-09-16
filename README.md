# Web Infrastructure Lab

Небольшой стенд веб-инфраструктуры, собранный с нуля на Docker Compose.

В основе — WordPress с MariaDB, перед ними Nginx в роли reverse proxy. Отдельно поднят стек мониторинга: Prometheus собирает метрики с Node Exporter, а Grafana используется для их просмотра.

Nginx принимает внешний HTTP/HTTPS-трафик и направляет запросы к нужным сервисам. Для WordPress настроено ограничение доступа к административным страницам по IP.

Конфигурации и данные сервисов вынесены из контейнеров, а Grafana и Prometheus настраиваются автоматически при запуске.

## Стек

- Docker / Docker Compose
- Nginx
- WordPress
- MariaDB
- Prometheus
- Node Exporter
- Grafana
- Fail2Ban


После запуска:

https://site.local — WordPress
http://metrics.local — Grafana
В планах
автоматизация развёртывания с Ansible;
CI/CD на GitHub Actions;
автоматический deployment на сервер.
