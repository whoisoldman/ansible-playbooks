# Домашнее задание: Ansible. Роли и переменные

## Цель работы
Развернуть CMS **Drupal** на удалённом хосте Ubuntu 22.04 с использованием ролей Ansible.
Стек установки: **NGINX + PHP-FPM (8.1) + PostgreSQL + Drupal**.
Все компоненты устанавливаются автоматически при выполнении одного сценария (playbook).

---

## 1. Рабочая среда

**Управляющий хост:**
- macOS 12
- Python 3.13
- Ansible [core 2.19.3]

**Целевой хост (управляемый):**
- Ubuntu 22.04 (Parallels Desktop VM)
- IP: `10.211.55.4`
- Пользователь: `alexgrey`
- Доступ по SSH

---

## 2. Подготовка проекта

### 2.1. Форк и клон репозитория
Создан форк репозитория преподавателя:
[https://github.com/Bumeranghc/ansible-playbooks](https://github.com/Bumeranghc/ansible-playbooks))

Затем выполнено:
```bash
git clone https://github.com/whoisoldman/ansible-playbooks.git
cd ansible-playbooks
```

---

### 2.2. Inventory

Файл `inventory/hosts`:
```ini
[drupal]
alexgrey ansible_host=10.211.55.4 ansible_user=alexgrey
```

Проверка соединения:
```bash
ansible -i inventory/hosts drupal -m ping -u alexgrey
```

Результат:
```
alexgrey | SUCCESS => {
    "ping": "pong"
}
```

---

## 3. Установка ролей Ansible Galaxy

Создан файл `requirements.yml`:

```yaml
roles:
  - name: geerlingguy.nginx
    version: 3.2.0
  - name: geerlingguy.postgresql
    version: 4.0.2
  - name: geerlingguy.php
    version: 6.0.0
```

Установка ролей:
```bash
ansible-galaxy install -r requirements.yml -p roles
```

---

## 4. Настройка переменных

Файл `group_vars/drupal.yml`:

```yaml
# --- PostgreSQL users & DB ---
postgresql_users:
  - name: drupal
    password: "drupal_pass_123"
    priv: "ALL"

postgresql_databases:
  - name: drupal
    owner: drupal
    encoding: "UTF8"
    lc_collate: "en_US.UTF-8"
    lc_ctype: "en_US.UTF-8"

# --- PHP packages ---
php_packages_extra:
  - php-fpm
  - php-cli
  - php-common
  - php-curl
  - php-gd
  - php-mbstring
  - php-xml
  - php-zip
  - php-opcache
  - php-pgsql

php_fpm_listen: "unix:/run/php/php8.1-fpm.sock"

# --- Drupal settings ---
drupal_core_path: /var/www/drupal
drupal_version: "10"
drupal_domain: "10.211.55.4"
drupal_site_name: "Drupal Demo"
drupal_account_name: "admin"
drupal_account_pass: "AdminPass123"
drupal_site_install: true
drupal_db_user: drupal
drupal_db_password: "drupal_pass_123"
drupal_db_name: drupal
drupal_db_driver: pgsql
drupal_db_port: 5432
drupal_db_host: "127.0.0.1"

# --- NGINX ---
nginx_remove_default_vhost: true
nginx_vhosts:
  - listen: "80 default_server"
    server_name: "_"
    root: "/var/www/drupal/web"
    index: "index.php index.html"
    state: "present"
    filename: "drupal.conf"
    extra_parameters: |
      location / {
        try_files $uri /index.php?$query_string;
      }

      location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
      }

      location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        try_files $uri /index.php?$query_string;
        expires max;
        log_not_found off;
      }
```

---

## 5. Сценарий (playbook)

Файл `playbook.yml`:

```yaml
- name: Provision Drupal stack (NGINX + PostgreSQL + PHP + Drupal)
  hosts: drupal
  become: true
  vars:
    php_packages: "{{ php_packages_extra | default([]) }}"
  roles:
    - geerlingguy.nginx
    - geerlingguy.postgresql
    - geerlingguy.php
  tasks:
    - import_tasks: tasks/drupal.yml
```

---

## 6. Установка Drupal

Файл `tasks/drupal.yml`:

```yaml
---
- name: Ensure base packages for composer/drush
  apt:
    name:
      - git
      - unzip
      - curl
      - composer
    state: present
    update_cache: true

- name: Ensure web root
  file:
    path: "{{ drupal_core_path }}"
    state: directory
    owner: www-data
    group: www-data
    mode: '0755'

- name: Create Drupal project
  command: >
    composer create-project drupal/recommended-project:^{{ drupal_version | default('10') }} {{ drupal_core_path }}
  args:
    creates: "{{ drupal_core_path }}/composer.json"
  environment:
    COMPOSER_ALLOW_SUPERUSER: "1"

- name: Install Drush 12
  command: >
    composer require drush/drush:^12 --no-interaction
  args:
    chdir: "{{ drupal_core_path }}"
  environment:
    COMPOSER_ALLOW_SUPERUSER: "1"

- name: Site install via Drush
  command: >
    {{ drupal_core_path }}/vendor/bin/drush site:install minimal -y
    --site-name="{{ drupal_site_name }}"
    --account-name="{{ drupal_account_name }}"
    --account-pass="{{ drupal_account_pass }}"
    --db-url="pgsql://{{ drupal_db_user }}:{{ drupal_db_password }}@{{ drupal_db_host }}:{{ drupal_db_port }}/{{ drupal_db_name }}"
  args:
    chdir: "{{ drupal_core_path }}"

- name: Fix ownership
  file:
    path: "{{ drupal_core_path }}"
    state: directory
    recurse: true
    owner: www-data
    group: www-data

- name: Restart NGINX
  service:
    name: nginx
    state: restarted
```

---

## 7. Запуск сценария

```bash
ansible-playbook -i inventory/hosts -u alexgrey -b -K playbook.yml | tee logs/run_$(date +%Y%m%d_%H%M%S).log
```

- `-i` — путь к inventory
- `-u` — имя пользователя на удалённом хосте
- `-b` — выполнить с правами root (sudo)
- `-K` — запросить sudo-пароль

---

## 8. Результат выполнения

- Все роли установлены успешно.
- Drupal развёрнут и доступен по адресу:

  **http://10.211.55.4/**

- Учётные данные администратора:
  ```
  admin / AdminPass123
  ```

---

## 9. Логи выполнения

Логи сохраняются в директорию:
```
logs/run_YYYYMMDD_HHMMSS.log
```

---

## 10. Заключение

- Роли `geerlingguy.*` обеспечили установку NGINX, PHP и PostgreSQL.
- Drupal установлен из Composer-пакета `drupal/recommended-project`.
- Для управления сайтом установлен Drush версии 12 (совместим с PHP 8.1).
- Настройка воспроизводима на любой машине с Ansible 2.19+ и SSH-доступом к Ubuntu 22.04.
