# Домашнее задание: Ansible. Разработка ролей

## Цель
Автоматизировать установку и настройку Apache, MySQL, PHP и WordPress с помощью ролей Ansible.

---

## 1. Структура проекта
```
homework_roles_wp/
├── group_vars/
│   └── lamp.yml
├── inventory/
│   └── hosts
├── logs/
├── roles/
│   ├── apache/
│   │   └── tasks/main.yml
│   ├── mysql/
│   │   └── tasks/main.yml
│   ├── php/
│   │   └── tasks/main.yml
│   └── wordpress/
│       ├── tasks/main.yml
│       └── templates/wp-config.php.j2
├── site.yml
└── tree.txt
```

---

## 2. Используемая среда
- macOS 12, VSCode Terminal  
- Виртуальная машина: **Ubuntu 22.04 (Parallels Desktop)**  
- Локальный проект: `/Volumes/Micron4Tb/Git/GitHub/PS_Projects_GitHub/ansible-playbooks/homework_roles_wp`  
- GitHub-репозиторий: [whoisoldman/ansible-playbooks](https://github.com/whoisoldman/ansible-playbooks.git)

---

## 3. Настройка окружения
### Инвентарь (`inventory/hosts`)
```ini
[lamp]
alexgrey ansible_host=10.211.55.4 ansible_user=alexgrey
```

### Групповые переменные (`group_vars/lamp.yml`)
```yaml
web_root: /var/www/wordpress
db_name: wpdb
db_user: wpuser
db_password: WpPass123!
```

---

## 4. Основные роли

### Apache
- Установка `apache2`
- Копирование шаблона виртуального хоста
- Перезапуск сервиса

### MySQL
- Установка `mysql-server`
- Создание базы данных и пользователя
- Выдача прав и `flush privileges`

### PHP
- Установка PHP 8.1 и модулей
- Связка Apache с PHP-FPM
- Перезапуск Apache

### WordPress
- Загрузка архива с официального сайта
- Распаковка в `{{ web_root }}`
- Разворачивание `wp-config.php` через шаблон
- Установка владельцев и прав
- Финальная проверка

---

## 5. Запуск сценария
```bash
cd homework_roles_wp

mkdir -p logs

ansible-playbook -i inventory/hosts -u alexgrey -b -K site.yml   | tee "logs/run_$(date +%Y%m%d_%H%M%S).log"
```

---

## 6. Проверка результата

1. После успешного выполнения:
   ```bash
   ansible -i inventory/hosts lamp -u alexgrey -b -m shell -a 'curl -I http://127.0.0.1/'
   ```
2. Открыть в браузере:  
   👉 http://10.211.55.4/

Ожидаемый результат: форма установки WordPress (выбор языка).

---

## 7. Возможные ошибки и решения

| Ошибка | Причина | Решение |
|--------|----------|----------|
| `Error establishing a database connection` | Пользователь MySQL создан только для `localhost` | Добавить `127.0.0.1` и обновить wp-config.php |
| `rsync: link_stat failed` | Папка `/tmp/wordpress` отсутствует | Использовать `unarchive` с `--strip-components=1` |
| `apache2.service failed` | Дубликаты в `sites-enabled` | Удалить `default.conf` перед созданием `vhost.conf` |

---

## 8. Финальный результат
✅ WordPress установлен и доступен по адресу  
👉 **http://10.211.55.4/**  
(отображается экран выбора языка).

---

## 9. Ссылки
- Репозиторий с кодом: [whoisoldman/ansible-playbooks](https://github.com/whoisoldman/ansible-playbooks/tree/feature/roles-wordpress)
- Лог выполнения сценария: `logs/run_YYYYMMDD_HHMMSS.log`

---
