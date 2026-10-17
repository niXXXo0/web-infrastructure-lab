# Web Infrastructure Lab

Учебный проект веб-инфраструктуры на Docker Compose с автоматизированным развёртыванием через Ansible.

## Архитектура

Проект включает:

- **Nginx** — reverse proxy для WordPress и Grafana.
- **WordPress** — веб-приложение.
- **MariaDB** — база данных WordPress.
- **Prometheus** — сбор метрик.
- **Node Exporter** — экспорт системных метрик хоста.
- **Grafana** — визуализация метрик.
- **Fail2Ban** — защита SSH от повторяющихся неудачных попыток входа.
- **Docker Compose** — запуск и управление контейнерами.
- **Ansible** — автоматическая установка Docker, настройка конфигураций и запуск сервисов.
- **GitHub Actions** — автоматическая проверка конфигураций при push в `main`.

## Требования

Для развёртывания необходимы:

- управляющая машина с Linux и установленным Ansible;
- целевой сервер с Debian и SSH-доступом;
- пользователь SSH с правами `sudo`;
- сетевой доступ к репозиториям пакетов и Docker Hub.

## Развёртывание через Ansible

### 1. Клонирование репозитория

На управляющей машине:

```bash
git clone https://github.com/niXXXo0/web-infrastructure-lab.git
cd web-infrastructure-lab
```

### 2. Установка зависимостей Ansible

Установите Ansible, если он ещё не установлен. Например, на Debian:

```bash
sudo apt update
sudo apt install ansible git
```

Установите необходимую коллекцию:

```bash
ansible-galaxy collection install -r ansible/requirements.yml
```

### 3. Настройка целевого сервера

Создайте локальный inventory из примера:

```bash
cp ansible/inventory.example.yml ansible/inventory.yml
```

Откройте `ansible/inventory.yml` и укажите адрес своего Debian-сервера и пользователя SSH.

Пример структуры:

```yaml
all:
  hosts:
    target-server:
      ansible_host: 192.168.1.100
      ansible_user: deploy
```

Здесь `192.168.1.100` и `deploy` — пример адреса и имени пользователя. Замените их своими значениями.

У пользователя SSH должны быть права на выполнение команд через `sudo`.

### 4. Настройка параметров проекта

Откройте `ansible/group_vars/all.yml`.

Укажите IP-адрес, с которого разрешён доступ к административным страницам WordPress:

```yaml
admin_allowed_ip: "YOUR_ADMIN_IP"
```

Замените `YOUR_ADMIN_IP` своим адресом. Это не IP целевого сервера, а адрес клиента, с которого будет выполняться вход в административную часть WordPress.

Остальные параметры можно оставить без изменений, если вам подходят значения по умолчанию.

### 5. Настройка секретов через Ansible Vault

Создайте локальный файл с секретами на основе шаблона:

```bash
cp ansible/group_vars/vault.example.yml ansible/group_vars/vault.yml
```

Откройте `ansible/group_vars/vault.yml` и замените шаблонные значения на надёжные пароли для MariaDB:

- `db_password` — пароль пользователя базы данных WordPress;
- `db_root_password` — пароль root-пользователя MariaDB.

Зашифруйте файл:

```bash
ansible-vault encrypt ansible/group_vars/vault.yml
```

Запомните пароль Ansible Vault: он понадобится при запуске playbook.

**Не публикуйте незашифрованный файл с секретами.** `ansible/group_vars/vault.yml` исключён из Git.

### 6. Проверка SSH-подключения

Проверьте доступность сервера:

```bash
ansible all -i ansible/inventory.yml -m ping
```

Если для подключения требуется SSH-пароль, добавьте `-k`. Если SSH-пользователю нужен пароль для `sudo`, при запуске playbook используйте `-K`.

### 7. Проверка playbook

Перед развёртыванием выполните проверку синтаксиса:

```bash
ansible-playbook --syntax-check ansible/deploy.yml
```

### 8. Запуск развёртывания

```bash
ansible-playbook \
  -i ansible/inventory.yml \
  ansible/deploy.yml \
  --ask-vault-pass \
  -K
```

Ansible запросит пароль Vault и пароль `sudo`, если он требуется.

Playbook установит Docker Engine и Docker Compose plugin, подготовит конфигурации, настроит Fail2Ban, создаст самоподписанный TLS-сертификат при необходимости и запустит сервисы.

## Доступ к сервисам

После успешного развёртывания:

- `https://site.local` — WordPress через HTTPS;
- `http://site.local` — WordPress через HTTP;
- `http://metrics.local` — Grafana.

Чтобы домены открывались на клиентском компьютере, настройте локальное разрешение имён (например, через файл `hosts`), направив `site.local` и `metrics.local` на IP целевого сервера.

TLS-сертификат самоподписанный, поэтому браузер может предупреждать о недоверенном сертификате.

## Проверка сервисов

На целевом сервере:

```bash
cd /opt/web-infrastructure-lab
sudo docker compose ps
```

Просмотр логов:

```bash
sudo docker compose logs -f
```

Проверка состояния Fail2Ban:

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

## Данные и постоянное хранение

Docker Compose использует именованные тома для хранения данных MariaDB, WordPress, Prometheus и Grafana.

Обычная остановка контейнеров не удаляет эти данные. Перед удалением томов или переинициализацией проекта рекомендуется сделать резервную копию.

## CI

GitHub Actions проверяет конфигурации Docker Compose, Nginx и Prometheus при отправке изменений в `main`.

Эти проверки не заменяют полноценное тестовое развёртывание: фактический запуск всех сервисов следует проверять отдельно.
