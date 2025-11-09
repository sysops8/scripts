# Ansible Refresh: Ежегодный/Полугодовой курс для DevOps/SysAdmin

**Цель:** Освежить в памяти ключевые концепции Ansible за 2-3 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый раздел состоит из:
1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Playbook или роль, которую нужно написать с нуля
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

---

## Модуль 1: Основы Ansible и Ad-Hoc команды (15 минут)

### 🎯 Напоминалка

**Архитектура Ansible:**
- **Agentless** - работает по SSH, не требует агентов на целевых хостах
- **Inventory** - список хостов для управления
- **Modules** - единицы работы (apt, yum, copy, service и т.д.)
- **Playbooks** - YAML файлы с набором задач
- **Roles** - переиспользуемые наборы задач

**Установка:**
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ansible

# RHEL/CentOS
sudo yum install ansible

# macOS
brew install ansible

# Через pip
pip install ansible

# Проверка версии
ansible --version
```

**Inventory файл (hosts или inventory.ini):**
```ini
# Простой список хостов
web1.example.com
web2.example.com

# С группами
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
db2.example.com

# С переменными
[webservers]
web1.example.com ansible_host=192.168.1.10 ansible_user=ubuntu

# Диапазоны
[webservers]
web[1:5].example.com

# Вложенные группы
[production:children]
webservers
databases
```

**Ad-Hoc команды:**
```bash
# Ping всех хостов
ansible all -m ping

# Проверить uptime
ansible webservers -m command -a "uptime"

# Установить пакет
ansible webservers -m apt -a "name=nginx state=present" --become

# Копировать файл
ansible all -m copy -a "src=/local/file dest=/remote/file"

# Перезапустить сервис
ansible webservers -m service -a "name=nginx state=restarted" --become

# Получить факты о системе
ansible all -m setup

# Выполнить с повышенными привилегиями
ansible all -m apt -a "name=htop state=present" --become

# Указать конкретный inventory
ansible all -i inventory.ini -m ping

# Ограничить выполнение
ansible webservers --limit web1.example.com -m ping

# Check mode (dry-run)
ansible all -m apt -a "name=nginx state=present" --check
```

**Базовая конфигурация ansible.cfg:**
```ini
[defaults]
inventory = ./inventory.ini
remote_user = ubuntu
host_key_checking = False
retry_files_enabled = False
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 3600

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

### 💻 Задание

Настрой базовое окружение Ansible:
1. Создай директорию `ansible-lab/`
2. Создай `inventory.ini` с двумя группами: `webservers` и `databases` (используй `localhost` для тестирования)
3. Создай базовый `ansible.cfg`
4. Выполни ad-hoc команды:
   - Проверь связь со всеми хостами (`ping`)
   - Получи информацию о памяти (`ansible all -m setup -a "filter=ansible_memory_mb"`)
   - Создай директорию `/tmp/ansible-test` на всех хостах
   - Проверь версию ОС на всех хостах

### 🚀 Бонус (новое)

Используй **dynamic inventory** для работы с облаком. Создай простой Python скрипт, который возвращает JSON со списком хостов. Или настрой работу с AWS EC2 через плагин `amazon.aws.aws_ec2`.

---

## Модуль 2: Playbooks - Основы (30 минут)

### 🎯 Напоминалка

**Структура Playbook:**
```yaml
---
- name: Описание playbook
  hosts: webservers          # На каких хостах выполнять
  become: yes                # Использовать sudo
  gather_facts: yes          # Собирать факты о системе
  
  vars:                      # Переменные
    http_port: 80
    app_name: myapp
  
  tasks:                     # Список задач
    - name: Установить nginx
      apt:
        name: nginx
        state: present
        update_cache: yes
    
    - name: Копировать конфиг
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: restart nginx   # Триггерит handler
    
    - name: Убедиться что nginx запущен
      service:
        name: nginx
        state: started
        enabled: yes
  
  handlers:                  # Выполняются при notify
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

**Основные модули:**
```yaml
# Управление пакетами
- name: Установить пакеты (Debian/Ubuntu)
  apt:
    name: "{{ item }}"
    state: present
    update_cache: yes
  loop:
    - nginx
    - git
    - htop

- name: Установить пакеты (RHEL/CentOS)
  yum:
    name: "{{ packages }}"
    state: present
  vars:
    packages:
      - httpd
      - git

# Управление файлами
- name: Копировать файл
  copy:
    src: source.txt
    dest: /path/to/dest.txt
    owner: www-data
    group: www-data
    mode: '0644'

- name: Создать файл из шаблона
  template:
    src: config.j2
    dest: /etc/app/config.conf

- name: Создать директорию
  file:
    path: /opt/app
    state: directory
    mode: '0755'

- name: Создать symlink
  file:
    src: /opt/app/current
    dest: /usr/local/bin/app
    state: link

# Управление сервисами
- name: Запустить и включить сервис
  service:
    name: nginx
    state: started
    enabled: yes

# Выполнение команд
- name: Выполнить команду
  command: /usr/bin/make install
  args:
    chdir: /opt/app/source

- name: Shell команда (с пайпами)
  shell: cat /etc/passwd | grep root

# Пользователи и группы
- name: Создать пользователя
  user:
    name: appuser
    shell: /bin/bash
    groups: www-data
    append: yes

# Git
- name: Клонировать репозиторий
  git:
    repo: https://github.com/user/repo.git
    dest: /opt/app
    version: main

# Управление переменными окружения
- name: Добавить переменную окружения
  lineinfile:
    path: /etc/environment
    line: 'MYVAR=value'
```

**Переменные:**
```yaml
# В playbook
vars:
  http_port: 80
  app_name: myapp

# В отдельном файле
vars_files:
  - vars/common.yml
  - vars/{{ env }}.yml

# Из командной строки
ansible-playbook playbook.yml -e "version=1.2.3"

# Использование
- debug:
    msg: "Порт: {{ http_port }}"

# Факты системы
- debug:
    msg: "ОС: {{ ansible_distribution }} {{ ansible_distribution_version }}"
```

**Условия:**
```yaml
- name: Установить apache (только Ubuntu)
  apt:
    name: apache2
    state: present
  when: ansible_distribution == "Ubuntu"

- name: Выполнить только на production
  command: /opt/app/deploy.sh
  when: environment == "production"

- name: Множественные условия
  service:
    name: nginx
    state: started
  when:
    - ansible_os_family == "Debian"
    - nginx_enabled | default(false)
```

**Циклы:**
```yaml
# Loop с списком
- name: Создать пользователей
  user:
    name: "{{ item }}"
    state: present
  loop:
    - alice
    - bob
    - charlie

# Loop со словарями
- name: Установить и запустить сервисы
  service:
    name: "{{ item.name }}"
    state: "{{ item.state }}"
  loop:
    - { name: 'nginx', state: 'started' }
    - { name: 'postgresql', state: 'started' }

# With_items (старый синтаксис, но еще используется)
- name: Установить пакеты
  apt:
    name: "{{ item }}"
    state: present
  with_items:
    - vim
    - curl
    - wget
```

**Handlers:**
```yaml
tasks:
  - name: Изменить конфиг nginx
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify:
      - reload nginx
      - check nginx config

handlers:
  - name: check nginx config
    command: nginx -t
  
  - name: reload nginx
    service:
      name: nginx
      state: reloaded
```

### 💻 Задание

Напиши playbook для установки и настройки веб-сервера:
1. Создай `webserver.yml` playbook
2. Установи nginx (или apache2)
3. Создай простую HTML страницу из шаблона (используй `template` модуль)
4. Скопируй конфиг nginx
5. Убедись что nginx запущен и включен в автозагрузку
6. Добавь handler для перезапуска nginx при изменении конфига
7. Добавь task для открытия порта 80 в firewall (если есть ufw/firewalld)

**Ожидаемая структура:**
```
webserver.yml
templates/
  index.html.j2
  nginx.conf.j2
files/
  (если нужны статические файлы)
```

### 🚀 Бонус (новое)

Добавь **теги** к задачам и возможность выполнять playbook частично:
```yaml
- name: Установить nginx
  apt:
    name: nginx
  tags:
    - install
    - nginx

- name: Настроить nginx
  template:
    src: nginx.conf.j2
  tags:
    - config
    - nginx
```

Выполнение: `ansible-playbook webserver.yml --tags "install"` или `--skip-tags "config"`

---

## Модуль 3: Переменные и Jinja2 шаблоны (25 минут)

### 🎯 Напоминалка

**Приоритет переменных (от низкого к высокому):**
1. Role defaults (`roles/myrole/defaults/main.yml`)
2. Inventory file или script group vars
3. Inventory group_vars/all
4. Playbook group_vars/all
5. Inventory group_vars/*
6. Playbook group_vars/*
7. Inventory file или script host vars
8. Inventory host_vars/*
9. Playbook host_vars/*
10. Host facts
11. Play vars
12. Play vars_prompt
13. Play vars_files
14. Role vars (`roles/myrole/vars/main.yml`)
15. Block vars
16. Task vars
17. Extra vars (`-e` в CLI) - **высший приоритет**

**Структура переменных:**
```
inventory.ini
group_vars/
  all.yml          # Переменные для всех хостов
  webservers.yml   # Переменные для группы webservers
  production.yml   # Переменные для группы production
host_vars/
  web1.example.com.yml  # Переменные для конкретного хоста
```

**group_vars/all.yml:**
```yaml
# Общие переменные
domain: example.com
ntp_server: pool.ntp.org
timezone: UTC

# Переменные для разных окружений
environments:
  dev:
    db_host: dev-db.internal
  prod:
    db_host: prod-db.internal
```

**host_vars/web1.example.com.yml:**
```yaml
server_id: 1
backup_enabled: true
```

**Jinja2 шаблоны:**
```jinja2
{# Комментарий #}

{# Переменные #}
Server: {{ ansible_hostname }}
IP: {{ ansible_default_ipv4.address }}

{# Условия #}
{% if ansible_distribution == "Ubuntu" %}
Package manager: apt
{% elif ansible_distribution == "CentOS" %}
Package manager: yum
{% else %}
Package manager: unknown
{% endif %}

{# Циклы #}
{% for server in webservers %}
server {{ loop.index }} {{ server.name }} {{ server.ip }};
{% endfor %}

{# Фильтры #}
{{ my_string | upper }}
{{ my_list | join(', ') }}
{{ my_dict | to_json }}
{{ my_dict | to_yaml }}
{{ "2024-01-15" | to_datetime }}

{# Значение по умолчанию #}
{{ variable | default('default_value') }}

{# Математика #}
{{ (ansible_memtotal_mb * 0.8) | int }}

{# Доступ к вложенным значениям #}
{{ nginx.config.worker_processes }}
{{ users[0].name }}
```

**Факты (Facts):**
```yaml
# Собрать факты вручную
- name: Gather facts
  setup:

# Использовать факты
- debug:
    msg: |
      Hostname: {{ ansible_hostname }}
      OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
      CPU: {{ ansible_processor_cores }} cores
      RAM: {{ ansible_memtotal_mb }} MB
      IP: {{ ansible_default_ipv4.address }}

# Отключить сбор фактов (ускоряет playbook)
- hosts: all
  gather_facts: no

# Собрать только определенные факты
- setup:
    gather_subset:
      - network
      - hardware
```

**Зарегистрировать результат выполнения:**
```yaml
- name: Проверить версию nginx
  command: nginx -v
  register: nginx_version

- name: Вывести версию
  debug:
    msg: "{{ nginx_version.stdout }}"

- name: Условие на основе результата
  debug:
    msg: "Nginx установлен"
  when: nginx_version.rc == 0
```

**Vault (шифрование секретов):**
```bash
# Создать зашифрованный файл
ansible-vault create secrets.yml

# Редактировать
ansible-vault edit secrets.yml

# Зашифровать существующий
ansible-vault encrypt vars.yml

# Расшифровать
ansible-vault decrypt vars.yml

# Посмотреть содержимое
ansible-vault view secrets.yml

# Использовать в playbook
ansible-playbook playbook.yml --ask-vault-pass

# Использовать файл с паролем
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass
```

**secrets.yml:**
```yaml
db_password: super_secret_password
api_key: your_api_key_here
```

### 💻 Задание

Создай систему управления конфигурациями для разных окружений:
1. Создай структуру:
   ```
   group_vars/
     all.yml
     production.yml
     staging.yml
   templates/
     app_config.j2
   ```

2. В `all.yml` определи общие переменные:
   ```yaml
   app_name: myapp
   app_port: 8080
   log_level: info
   ```

3. В `production.yml` и `staging.yml` переопредели специфичные:
   ```yaml
   # production.yml
   db_host: prod-db.internal
   cache_enabled: true
   
   # staging.yml
   db_host: staging-db.internal
   cache_enabled: false
   ```

4. Создай шаблон `app_config.j2`:
   ```jinja2
   [application]
   name = {{ app_name }}
   port = {{ app_port }}
   environment = {{ env }}
   
   [database]
   host = {{ db_host }}
   port = {{ db_port | default(5432) }}
   
   [cache]
   enabled = {{ cache_enabled | lower }}
   ```

5. Напиши playbook, который разворачивает конфиг на основе окружения

### 🚀 Бонус (новое)

Используй **Ansible Vault** для хранения секретов. Создай `secrets.yml` с паролями БД, зашифруй его, и используй в playbook. Также попробуй **lookup plugins** для чтения паролей из внешних источников:
```yaml
- name: Прочитать пароль из файла
  debug:
    msg: "{{ lookup('file', '/path/to/password.txt') }}"

- name: Сгенерировать случайный пароль
  debug:
    msg: "{{ lookup('password', '/dev/null length=15 chars=ascii_letters,digits') }}"
```

---

## Модуль 4: Roles - Модульная архитектура (35 минут)

### 🎯 Напоминалка

**Что такое Role:**
Role - это способ организации playbook'ов в переиспользуемые компоненты. Это стандартная структура директорий с предопределенными именами.

**Структура Role:**
```
roles/
  common/
    tasks/         # Основные задачи
      main.yml
    handlers/      # Handlers
      main.yml
    templates/     # Jinja2 шаблоны
      config.j2
    files/         # Статические файлы
      script.sh
    vars/          # Переменные (высокий приоритет)
      main.yml
    defaults/      # Значения по умолчанию (низкий приоритет)
      main.yml
    meta/          # Метаданные роли и зависимости
      main.yml
    library/       # Кастомные модули (опционально)
    module_utils/  # Утилиты для модулей (опционально)
    lookup_plugins/ # Кастомные lookup плагины (опционально)
```

**Создание роли:**
```bash
# Автоматически создать структуру
ansible-galaxy init roles/webserver

# Или вручную
mkdir -p roles/webserver/{tasks,handlers,templates,files,vars,defaults,meta}
```

**roles/webserver/tasks/main.yml:**
```yaml
---
- name: Установить nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Копировать конфиг
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx

- name: Убедиться что nginx запущен
  service:
    name: nginx
    state: started
    enabled: yes
```

**roles/webserver/defaults/main.yml:**
```yaml
---
nginx_port: 80
nginx_worker_processes: auto
nginx_worker_connections: 1024
```

**roles/webserver/handlers/main.yml:**
```yaml
---
- name: restart nginx
  service:
    name: nginx
    state: restarted

- name: reload nginx
  service:
    name: nginx
    state: reloaded
```

**roles/webserver/meta/main.yml:**
```yaml
---
galaxy_info:
  author: Your Name
  description: Nginx web server role
  company: Your Company
  license: MIT
  min_ansible_version: 2.9
  
  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
    - name: Debian
      versions:
        - buster
        - bullseye

  galaxy_tags:
    - web
    - nginx

dependencies:
  - role: common
    vars:
      firewall_open_ports:
        - 80
        - 443
```

**Использование ролей в playbook:**
```yaml
---
- name: Настроить веб-серверы
  hosts: webservers
  become: yes
  
  roles:
    - common           # Простое включение
    - role: webserver  # С параметрами
      vars:
        nginx_port: 8080
    - role: monitoring
      tags: ['monitoring']

# Альтернативный синтаксис
- name: Настроить веб-серверы
  hosts: webservers
  become: yes
  
  tasks:
    - name: Применить роль webserver
      include_role:
        name: webserver
      vars:
        nginx_port: 8080

    - name: Применить роль условно
      include_role:
        name: monitoring
      when: enable_monitoring | default(false)
```

**Ansible Galaxy - репозиторий ролей:**
```bash
# Поиск ролей
ansible-galaxy search nginx

# Установить роль
ansible-galaxy install geerlingguy.nginx

# Установить с версией
ansible-galaxy install geerlingguy.nginx,3.1.4

# Установить из requirements.yml
ansible-galaxy install -r requirements.yml

# Список установленных
ansible-galaxy list

# Удалить роль
ansible-galaxy remove geerlingguy.nginx
```

**requirements.yml:**
```yaml
---
# Из Galaxy
- name: geerlingguy.nginx
  version: 3.1.4

- name: geerlingguy.postgresql
  version: 3.2.0

# Из Git
- name: custom-role
  src: https://github.com/user/ansible-role-custom.git
  version: main

# С указанием пути установки
- name: local-role
  src: https://github.com/user/role.git
  scm: git
  path: roles/
```

**Организация тасков внутри роли:**
```yaml
# roles/webserver/tasks/main.yml
---
- name: Include OS-specific variables
  include_vars: "{{ ansible_os_family }}.yml"

- name: Include installation tasks
  include_tasks: install.yml

- name: Include configuration tasks
  include_tasks: configure.yml

- name: Include SSL tasks
  include_tasks: ssl.yml
  when: nginx_ssl_enabled | default(false)
```

### 💻 Задание

Создай роль для установки и настройки MySQL/PostgreSQL:

1. Создай структуру роли `database`:
   ```bash
   ansible-galaxy init roles/database
   ```

2. В `defaults/main.yml` определи переменные:
   ```yaml
   db_type: mysql  # или postgresql
   db_version: "8.0"
   db_root_password: changeme
   db_port: 3306
   databases:
     - name: app_db
       encoding: utf8mb4
   db_users:
     - name: app_user
       password: secret
       priv: "app_db.*:ALL"
   ```

3. Создай таски:
   - Установка БД
   - Настройка конфига
   - Создание БД и пользователей
   - Обеспечение безопасности (удаление тестовых БД и пользователей)

4. Добавь поддержку разных ОС (Ubuntu/CentOS)

5. Создай playbook `database.yml`, который использует эту роль

### 🚀 Бонус (новое)

Добавь в роль **тесты с Molecule**:
```bash
# Установить Molecule
pip install molecule molecule-docker

# Инициализировать тесты
cd roles/database
molecule init scenario

# Запустить тесты
molecule test
```

Также создай **CI/CD pipeline** для автоматического тестирования роли при каждом коммите (GitHub Actions или GitLab CI).

---

## Модуль 5: Работа с динамическими данными (30 минут)

### 🎯 Напоминалка

**Include и Import:**
```yaml
# include_tasks - динамическое включение (во время выполнения)
- name: Include tasks dynamically
  include_tasks: "{{ ansible_os_family }}.yml"

# import_tasks - статическое включение (при парсинге)
- name: Import tasks
  import_tasks: common.yml

# include_role
- name: Include role
  include_role:
    name: myrole
  vars:
    role_var: value

# import_role
- name: Import role
  import_role:
    name: myrole
```

**Делегирование задач:**
```yaml
# Выполнить на другом хосте
- name: Добавить сервер в балансировщик
  command: /usr/local/bin/add_to_pool.sh {{ inventory_hostname }}
  delegate_to: loadbalancer.example.com

# Выполнить локально
- name: Ждать когда хост поднимется
  wait_for:
    host: "{{ inventory_hostname }}"
    port: 22
    delay: 10
  delegate_to: localhost

# Выполнить один раз для всех хостов
- name: Обновить DNS запись
  command: /usr/local/bin/update_dns.sh
  run_once: true
  delegate_to: dns-server.example.com
```

**Работа с группами хостов:**
```yaml
# Получить всех хостов из группы
- debug:
    msg: "{{ groups['webservers'] }}"

# Получить переменную другого хоста
- debug:
    msg: "{{ hostvars['web1.example.com']['ansible_eth0']['ipv4']['address'] }}"

# Пройтись по всем хостам группы
- name: Настроить бэкенды в HAProxy
  template:
    src: haproxy.cfg.j2
    dest: /etc/haproxy/haproxy.cfg
  vars:
    backend_servers: "{{ groups['webservers'] }}"
```

**Шаблон с группами:**
```jinja2
# haproxy.cfg.j2
backend web_backend
    balance roundrobin
{% for host in groups['webservers'] %}
    server {{ host }} {{ hostvars[host]['ansible_default_ipv4']['address'] }}:80 check
{% endfor %}
```

**Стратегии выполнения:**
```yaml
- name: Playbook с разными стратегиями
  hosts: webservers
  strategy: free  # free, linear, debug
  serial: 2       # По 2 хоста одновременно
  
  tasks:
    - name: Update
      apt:
        update_cache: yes

# Rolling update
- hosts: webservers
  serial:
    - 1          # Сначала 1 хост
    - 25%        # Потом 25% от оставшихся
    - 100%       # Потом все остальные
  
  tasks:
    - name: Update application
      # ...
```

**Blocks - группировка задач:**
```yaml
- name: Установка с обработкой ошибок
  block:
    - name: Установить пакет
      apt:
        name: nginx
        state: present
    
    - name: Запустить сервис
      service:
        name: nginx
        state: started
  
  rescue:
    - name: Откатиться при ошибке
      debug:
        msg: "Установка не удалась, откат изменений"
    
    - name: Удалить пакет
      apt:
        name: nginx
        state: absent
  
  always:
    - name: Очистить временные файлы
      file:
        path: /tmp/nginx-install
        state: absent
  
  when: ansible_os_family == "Debian"
```

**Асинхронное выполнение:**
```yaml
# Запустить задачу асинхронно
- name: Долгая операция
  command: /usr/local/bin/long_running_script.sh
  async: 3600        # Таймаут (секунды)
  poll: 0            # Не ждать завершения

# Запустить и проверить статус позже
- name: Долгая операция
  command: /usr/local/bin/backup.sh
  async: 3600
  poll: 0
  register: backup_task

# Другие задачи...

- name: Проверить статус backup
  async_status:
    jid: "{{ backup_task.ansible_job_id }}"
  register: job_result
  until: job_result.finished
  retries: 30
  delay: 60
```

**Retry механизм:**
```yaml
- name: Проверить доступность сервиса
  uri:
    url: http://{{ inventory_hostname }}/health
  register: result
  until: result.status == 200
  retries: 5
  delay: 10
```

### 💻 Задание

Создай playbook для деплоя приложения с zero-downtime:

1. Создай `deploy.yml` playbook со следующей логикой:
   - Выполняется по одному хосту за раз (`serial: 1`)
   - Убирает хост из балансировщика (делегация на localhost)
   - Обновляет приложение
   - Проверяет health check
   - Возвращает хост в балансировщик
   - Если ошибка - откатывается

2. Структура:
   ```yaml
   - name: Zero-downtime deployment
     hosts: webservers
     serial: 1
     
     tasks:
       - block:
           - name: Убрать из балансировщика
             # Имитируй командой
           
           - name: Обновить приложение
             # Git pull или копирование
           
           - name: Рестарт сервиса
             service:
               name: myapp
               state: restarted
           
           - name: Проверить health
             uri:
               url: "http://{{ inventory_hostname }}/health"
             register: health
             until: health.status == 200
             retries: 5
             delay: 3
           
           - name: Вернуть в балансировщик
             # Имитируй командой
         
         rescue:
           - name: Откат
             # Восстановление из backup
   ```

### 🚀 Бонус (новое)

Используй **Ansible Tower/AWX** или создай простой REST API wrapper для запуска playbook'ов. Также попробуй **Ansible Pull** режим, где целевые хосты сами забирают конфигурацию из Git репозитория:

```bash
# На целевом хосте
ansible-pull -U https://github.com/user/ansible-repo.git -i inventory.ini playbook.yml
```

---

## Модуль 6: Работа с облаками и контейнерами (30 минут)

### 🎯 Напоминалка

**Docker модули:**
```yaml
# Установить Docker
- name: Установить Docker
  apt:
    name:
      - docker.io
      - docker-compose
    state: present

# Управление контейнерами
- name: Запустить контейнер
  docker_container:
    name: nginx
    image: nginx:latest
    state: started
    ports:
      - "80:80"
    volumes:
      - /opt/data:/usr/share/nginx/html
    env:
      NGINX_HOST: example.com
    restart_policy: always

# Docker Compose
- name: Запустить стек через docker-compose
  docker_compose:
    project_src: /opt/myapp
    files:
      - docker-compose.yml
    state: present

# Управление образами
- name: Собрать образ
  docker_image:
    name: myapp
    build:
      path: /opt/myapp
      pull: yes
    source: build
    state: present

- name: Загрузить образ из registry
  docker_image:
    name: nginx:latest
    source: pull
```

**AWS модули (через boto3):**
```yaml
# EC2 инстансы
- name: Создать EC2 инстанс
  amazon.aws.ec2_instance:
    name: web-server-01
    key_name: mykey
    instance_type: t3.micro
    image_id: ami-0c55b159cbfafe1f0
    region: us-east-1
    vpc_subnet_id: subnet-12345
    security_group: web-sg
    tags:
      Environment: production
      Role: webserver
    state: running
  register: ec2

# S3
- name: Загрузить файл в S3
  amazon.aws.s3_object:
    bucket: mybucket
    object: /path/to/file.txt
    src: /local/file.txt
    mode: put

- name: Синхронизировать директорию с S3
  amazon.aws.s3_sync:
    bucket: mybucket
    file_root: /local/dir
    key_prefix: backup/

# RDS
- name: Создать RDS инстанс
  amazon.aws.rds_instance:
    id: mydb
    state: present
    engine: postgres
    size: 20
    instance_type: db.t3.micro
    username: admin
    password: "{{ db_password }}"
```

**Kubernetes/OpenShift:**
```yaml
# Установка модулей
# pip install kubernetes openshift

# Применить манифест
- name: Создать deployment
  kubernetes.core.k8s:
    state: present
    definition:
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: nginx
        namespace: default
      spec:
        replicas: 3
        selector:
          matchLabels:
            app: nginx
        template:
          metadata:
            labels:
              app: nginx
          spec:
            containers:
            - name: nginx
              image: nginx:latest
              ports:
              - containerPort: 80

# Применить из файла
- name: Применить манифесты
  kubernetes.core.k8s:
    state: present
    src: /path/to/manifest.yaml

# Получить информацию о ресурсах
- name: Получить список pods
  kubernetes.core.k8s_info:
    kind: Pod
    namespace: default
  register: pods

# Helm
- name: Установить chart
  kubernetes.core.helm:
    name: nginx
    chart_ref: stable/nginx
    release_namespace: default
    values:
      replicaCount: 3
```

**Terraform интеграция:**
```yaml
# Запустить Terraform из Ansible
- name: Инициализировать Terraform
  command: terraform init
  args:
    chdir: /path/to/terraform

- name: Применить конфигурацию
  command: terraform apply -auto-approve
  args:
    chdir: /path/to/terraform
  register: tf_output

# Или через модуль
- name: Управлять инфраструктурой через Terraform
  community.general.terraform:
    project_path: /path/to/terraform
    state: present
    variables:
      region: us-east-1
      instance_count: 3
```

**Dynamic Inventory для облаков:**
```yaml
# aws_ec2.yml (для AWS)
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1
filters:
  tag:Environment: production
keyed_groups:
  - key: tags.Role
    prefix: role
  - key: placement.availability_zone
    prefix: az
hostnames:
  - private-ip-address

# Использование
ansible-inventory -i aws_ec2.yml --graph
ansible-playbook -i aws_ec2.yml playbook.yml
```

**GCP:**
```yaml
# gcp_compute.yml
plugin: gcp_compute
projects:
  - my-gcp-project
auth_kind: serviceaccount
service_account_file: /path/to/credentials.json
filters:
  - status = RUNNING
keyed_groups:
  - key: labels.environment
    prefix: env
```

### 💻 Задание

Создай playbook для развертывания приложения в Docker:

1. Создай `docker-deploy.yml`:
   ```yaml
   - name: Deploy application in Docker
     hosts: docker_hosts
     become: yes
     
     vars:
       app_name: myapp
       app_version: "1.0.0"
       app_port: 8080
     
     tasks:
       - name: Установить Docker
         # ...
       
       - name: Создать сеть
         docker_network:
           name: "{{ app_name }}_network"
       
       - name: Запустить БД
         docker_container:
           name: "{{ app_name }}_db"
           image: postgres:13
           # ...
       
       - name: Запустить приложение
         docker_container:
           name: "{{ app_name }}"
           image: "mycompany/{{ app_name }}:{{ app_version }}"
           # ...
   ```

2. Добавь health check и rollback при ошибке

3. Создай tasks для очистки старых контейнеров и образов

### 🚀 Бонус (новое)

Создай **dynamic inventory script** для Docker, который автоматически обнаруживает запущенные контейнеры и позволяет управлять ими через Ansible. Или настрой интеграцию с **Kubernetes** и создай playbook для деплоя Helm chart.

---

## Модуль 7: Тестирование и CI/CD (25 минут)

### 🎯 Напоминалка

**Ansible-lint - линтер для playbook'ов:**
```bash
# Установка
pip install ansible-lint

# Проверка playbook
ansible-lint playbook.yml

# Проверка роли
ansible-lint roles/myrole

# С конфигурацией
cat > .ansible-lint <<EOF
skip_list:
  - '301'  # Commands should not change things
  - '303'  # Using command rather than module
warn_list:
  - experimental
EOF
```

**Molecule - фреймворк для тестирования ролей:**
```bash
# Установка
pip install molecule molecule-docker

# Инициализация в роли
cd roles/myrole
molecule init scenario

# Структура molecule
molecule/
  default/
    molecule.yml       # Конфигурация
    converge.yml       # Playbook для тестирования
    verify.yml         # Тесты
    prepare.yml        # Подготовка окружения

# Команды
molecule create        # Создать тестовое окружение
molecule converge      # Применить роль
molecule verify        # Запустить тесты
molecule destroy       # Удалить окружение
molecule test          # Полный цикл
```

**molecule/default/molecule.yml:**
```yaml
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: ubuntu-20
    image: geerlingguy/docker-ubuntu2004-ansible
    pre_build_image: true
  - name: ubuntu-22
    image: geerlingguy/docker-ubuntu2204-ansible
    pre_build_image: true
  - name: centos-8
    image: geerlingguy/docker-centos8-ansible
    pre_build_image: true
provisioner:
  name: ansible
  config_options:
    defaults:
      callbacks_enabled: profile_tasks
verifier:
  name: ansible
```

**molecule/default/verify.yml (тесты):**
```yaml
---
- name: Verify
  hosts: all
  gather_facts: false
  tasks:
    - name: Check nginx is installed
      package:
        name: nginx
        state: present
      check_mode: yes
      register: pkg
      failed_when: pkg.changed

    - name: Check nginx is running
      service:
        name: nginx
        state: started
      check_mode: yes
      register: svc
      failed_when: svc.changed

    - name: Test HTTP response
      uri:
        url: http://localhost
        return_content: yes
      register: response
      failed_when: "'Welcome to nginx' not in response.content"
```

**Check mode и Diff:**
```bash
# Dry-run (не применяет изменения)
ansible-playbook playbook.yml --check

# Показать diff изменений
ansible-playbook playbook.yml --check --diff

# Теги для выборочной проверки
ansible-playbook playbook.yml --check --tags config
```

**GitHub Actions для CI/CD:**
```yaml
# .github/workflows/ansible-ci.yml
name: Ansible CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'
      
      - name: Install dependencies
        run: |
          pip install ansible ansible-lint
      
      - name: Lint playbooks
        run: |
          ansible-lint playbook.yml
  
  molecule:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'
      
      - name: Install Molecule
        run: |
          pip install molecule molecule-docker docker
      
      - name: Run Molecule tests
        run: |
          cd roles/webserver
          molecule test
```

**GitLab CI:**
```yaml
# .gitlab-ci.yml
stages:
  - lint
  - test
  - deploy

lint:
  stage: lint
  image: python:3.9
  before_script:
    - pip install ansible ansible-lint
  script:
    - ansible-lint playbook.yml

molecule:
  stage: test
  image: python:3.9
  services:
    - docker:dind
  before_script:
    - pip install molecule molecule-docker docker
  script:
    - cd roles/webserver
    - molecule test

deploy-staging:
  stage: deploy
  only:
    - develop
  script:
    - ansible-playbook -i inventory/staging playbook.yml

deploy-production:
  stage: deploy
  only:
    - main
  when: manual
  script:
    - ansible-playbook -i inventory/production playbook.yml
```

**Ansible Tower/AWX (GUI для Ansible):**
```yaml
# Основные концепции:
# - Projects: Git репозитории с playbook'ами
# - Inventories: Списки хостов
# - Credentials: SSH ключи, пароли
# - Templates: Готовые к запуску playbook'и
# - Jobs: История выполнений
# - Workflows: Цепочки playbook'ов

# REST API для интеграции
curl -X POST https://tower.example.com/api/v2/job_templates/42/launch/ \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"extra_vars": {"version": "1.2.3"}}'
```

### 💻 Задание

Настрой тестирование для своей роли:

1. Инициализируй Molecule в роли:
   ```bash
   cd roles/webserver
   molecule init scenario
   ```

2. Настрой `molecule.yml` для тестирования на Ubuntu и CentOS

3. Создай `verify.yml` с тестами:
   - Проверка установки пакетов
   - Проверка запуска сервисов
   - Проверка HTTP ответа
   - Проверка конфигурационных файлов

4. Запусти полный цикл тестирования:
   ```bash
   molecule test
   ```

5. Создай GitHub Actions workflow для автоматического запуска тестов

### 🚀 Бонус (новое)

Настрой **Ansible Tower/AWX** (можно в Docker) и создай Job Template для автоматического деплоя. Или используй **Ansible Semaphore** - open-source альтернатива Tower. Также попробуй **Testinfra** для написания тестов на Python:

```python
# tests/test_nginx.py
def test_nginx_installed(host):
    nginx = host.package("nginx")
    assert nginx.is_installed

def test_nginx_running(host):
    nginx = host.service("nginx")
    assert nginx.is_running
    assert nginx.is_enabled

def test_nginx_listening(host):
    socket = host.socket("tcp://0.0.0.0:80")
    assert socket.is_listening
```

---

## Модуль 8: Best Practices и оптимизация (25 минут)

### 🎯 Напоминалка

**Структура проекта:**
```
ansible-project/
├── ansible.cfg
├── inventory/
│   ├── production/
│   │   ├── hosts
│   │   └── group_vars/
│   │       ├── all.yml
│   │       ├── webservers.yml
│   │       └── databases.yml
│   └── staging/
│       ├── hosts
│       └── group_vars/
├── group_vars/
│   └── all.yml
├── host_vars/
├── roles/
│   ├── common/
│   ├── webserver/
│   └── database/
├── playbooks/
│   ├── site.yml
│   ├── webservers.yml
│   └── databases.yml
├── files/
├── templates/
├── library/          # Кастомные модули
├── filter_plugins/   # Кастомные фильтры
├── vars/
│   ├── dev.yml
│   ├── staging.yml
│   └── prod.yml
├── secrets/          # Vault файлы
│   └── prod.yml
├── requirements.yml  # Зависимости от Galaxy
├── .ansible-lint
├── .gitignore
└── README.md
```

**Оптимизация производительности:**
```yaml
# ansible.cfg
[defaults]
# Отключить сбор фактов если не нужны
gathering = explicit

# Кэширование фактов
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 86400

# SSH настройки
[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
pipelining = True  # Уменьшает количество SSH соединений
control_path = /tmp/ansible-ssh-%%h-%%p-%%r

# Параллелизм
forks = 20

# Strategy
strategy = free  # Не ждать всех хостов на каждом шаге
```

**Ускорение playbook'ов:**
```yaml
# Отключить сбор фактов
- hosts: all
  gather_facts: no

# Собрать только нужные факты
- hosts: all
  gather_facts: yes
  gather_subset:
    - '!all'
    - '!min'
    - network

# Асинхронное выполнение
- name: Длительная задача
  command: /usr/bin/long_script
  async: 300
  poll: 0

# Использовать free strategy
- hosts: all
  strategy: free

# Кэш пакетов
- name: Установить пакеты
  apt:
    name: nginx
    state: present
    cache_valid_time: 3600  # Не обновлять кэш если свежий
```

**Best Practices:**
```yaml
# 1. Всегда используй name для задач
- name: Install nginx  # ✅ Хорошо
  apt: name=nginx

- apt: name=nginx      # ❌ Плохо

# 2. Используй state явно
- name: Ensure nginx is installed
  apt:
    name: nginx
    state: present  # ✅ Явно

# 3. Проверяй идемпотентность
- name: Добавить строку в файл (идемпотентно)
  lineinfile:
    path: /etc/hosts
    line: "192.168.1.10 myhost"
    state: present

# 4. Используй changed_when для команд
- name: Проверить статус
  command: systemctl is-active nginx
  register: result
  changed_when: false  # Эта команда не меняет систему
  failed_when: result.rc not in [0, 3]

# 5. Используй block для группировки
- name: Configure web server
  block:
    - name: Install packages
      apt: name=nginx
    
    - name: Copy config
      template: src=nginx.conf.j2
  
  when: ansible_os_family == "Debian"
  tags: webserver

# 6. Избегай command/shell когда есть модуль
- name: Create directory
  file:  # ✅ Хорошо
    path: /opt/app
    state: directory

- name: Create directory
  command: mkdir /opt/app  # ❌ Плохо (не идемпотентно)

# 7. Используй failed_when и ignore_errors осторожно
- name: Check if file exists
  stat:
    path: /opt/app/config
  register: config
  failed_when: false  # Не падать если файла нет

# 8. Документируй playbook'и
---
# Playbook: webserver.yml
# Purpose: Configure nginx web servers
# Requirements: Ubuntu 20.04+
# Author: Your Name
# Date: 2024-01-15

- name: Configure web servers
  hosts: webservers
  # ...

# 9. Используй tags разумно
- name: Install nginx
  apt: name=nginx
  tags:
    - packages
    - webserver
    - install

# 10. Валидируй конфиги перед применением
- name: Copy nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    validate: 'nginx -t -c %s'
```

**Безопасность:**
```yaml
# 1. Используй Vault для секретов
ansible-vault create secrets.yml
ansible-vault encrypt_string 'secret_password' --name 'db_password'

# 2. Не коммить пароли в Git
# .gitignore
secrets.yml
*.vault
.vault_pass

# 3. Ограничь права на файлы
- name: Copy sensitive file
  copy:
    src: secret.key
    dest: /etc/app/secret.key
    mode: '0600'
    owner: appuser
    group: appuser

# 4. Используй become только когда нужно
- name: Task requiring root
  apt: name=nginx
  become: yes
  become_user: root

# 5. Проверяй SSL сертификаты
- name: Download file
  get_url:
    url: https://example.com/file
    dest: /tmp/file
    validate_certs: yes

# 6. Аудит изменений
- name: Log changes
  lineinfile:
    path: /var/log/ansible-changes.log
    line: "{{ ansible_date_time.iso8601 }} - {{ ansible_user_id }} - {{ inventory_hostname }}"
    create: yes
  delegate_to: localhost
```

**Отладка:**
```yaml
# Debug модуль
- name: Print variable
  debug:
    msg: "Value is {{ my_var }}"
    verbosity: 1  # Показывать только с -v

- name: Print all variables
  debug:
    var: hostvars[inventory_hostname]

# Assert для проверок
- name: Validate configuration
  assert:
    that:
      - ansible_distribution == "Ubuntu"
      - ansible_distribution_major_version >= "20"
    fail_msg: "Ubuntu 20.04+ required"
    success_msg: "OS version OK"

# Debugger
- name: Debug this task
  command: /usr/bin/failing_command
  debugger: on_failed

# Запуск с отладкой
ansible-playbook playbook.yml --start-at-task="Task name"
ansible-playbook playbook.yml --step  # Пошаговое выполнение
ansible-playbook playbook.yml -vvv    # Максимальная детализация
```

**Мониторинг выполнения:**
```yaml
# Callback plugins для метрик
# ansible.cfg
[defaults]
callbacks_enabled = profile_tasks, timer

# Логирование
log_path = /var/log/ansible.log

# Или в playbook
- name: Start timer
  set_fact:
    start_time: "{{ ansible_date_time.epoch }}"

- name: Do work
  # ...

- name: Calculate duration
  debug:
    msg: "Duration: {{ (ansible_date_time.epoch | int) - (start_time | int) }} seconds"
```

### 💻 Задание

Оптимизируй существующий playbook:

1. Возьми один из ранее созданных playbook'ов

2. Примени best practices:
   - Добавь проверки идемпотентности
   - Добавь валидацию перед применением
   - Используй blocks для группировки
   - Добавь changed_when где нужно
   - Добавь tags
   - Документируй каждую секцию

3. Оптимизируй производительность:
   - Настрой ansible.cfg
   - Отключи лишний сбор фактов
   - Используй асинхронность где возможно

4. Добавь безопасность:
   - Перенеси секреты в Vault
   - Проверь права на файлы
   - Добавь валидацию входных данных

5. Запусти с профилированием:
   ```bash
   ANSIBLE_CALLBACK_WHITELIST=profile_tasks ansible-playbook playbook.yml
   ```

### 🚀 Бонус (новое)

Создай **кастомный модуль** на Python для специфичной задачи или **кастомный фильтр** для Jinja2. Также изучи **Ansible Collections** - новый способ распространения контента:

```bash
# Создать collection
ansible-galaxy collection init myorg.mycollection

# Установить collection
ansible-galaxy collection install myorg.mycollection

# Использовать
- name: Use module from collection
  myorg.mycollection.my_module:
    option: value
```

---

## Финальный проект (60 минут)

### Задача: Полный стек инфраструктуры с Ansible

Создай комплексное решение для автоматизации инфраструктуры веб-приложения.

**Требования:**

**1. Структура проекта:**
```
infrastructure-automation/
├── ansible.cfg
├── inventory/
│   ├── production/
│   │   ├── hosts
│   │   └── group_vars/
│   └── staging/
│       ├── hosts
│       └── group_vars/
├── roles/
│   ├── common/           # Базовая настройка всех серверов
│   ├── webserver/        # Nginx/Apache
│   ├── appserver/        # Application server (Gunicorn/uWSGI)
│   ├── database/         # PostgreSQL/MySQL
│   ├── cache/            # Redis/Memcached
│   ├── monitoring/       # Prometheus/Grafana
│   └── backup/           # Backup solution
├── playbooks/
│   ├── site.yml          # Главный playbook
│   ├── deploy.yml        # Деплой приложения
│   ├── backup.yml        # Backup задачи
│   └── monitoring.yml    # Настройка мониторинга
├── group_vars/
│   ├── all.yml
│   └── all/
│       ├── vars.yml
│       └── vault.yml     # Зашифрованные секреты
├── host_vars/
├── templates/
├── files/
├── library/              # Кастомные модули
├── filter_plugins/
├── requirements.yml
├── molecule/             # Тесты
├── .ansible-lint
├── .github/
│   └── workflows/
│       └── ansible-ci.yml
└── README.md
```

**2. Функциональность:**

**Role: common**
- Настройка hostname
- Настройка timezone и NTP
- Установка базовых пакетов (vim, htop, curl, git)
- Настройка SSH (отключить root, ключи, fail2ban)
- Настройка firewall (ufw/firewalld)
- Настройка users и sudo
- Базовый мониторинг (node_exporter)

**Role: webserver**
- Установка и настройка Nginx
- SSL сертификаты (Let's Encrypt)
- Настройка виртуальных хостов
- Load balancing конфигурация
- Логирование и ротация логов

**Role: appserver**
- Установка Python/Node.js/etc
- Настройка application server
- Управление systemd сервисами
- Конфигурация через environment variables
- Health checks

**Role: database**
- Установка PostgreSQL/MySQL
- Настройка производительности
- Создание БД и пользователей
- Настройка репликации (опционально)
- Автоматический backup

**Role: monitoring**
- Prometheus для метрик
- Grafana для визуализации
- Alertmanager для алертов
- Exporters (node, nginx, postgres)

**3. Playbooks:**

**site.yml (главный):**
```yaml
---
- name: Configure all servers
  hosts: all
  roles:
    - common

- name: Configure web servers
  hosts: webservers
  roles:
    - webserver

- name: Configure application servers
  hosts: appservers
  roles:
    - appserver

- name: Configure databases
  hosts: databases
  roles:
    - database

- name: Configure cache servers
  hosts: cache
  roles:
    - cache

- name: Setup monitoring
  hosts: monitoring
  roles:
    - monitoring

- name: Configure backups
  hosts: all
  roles:
    - backup
  tags: backup
```

**deploy.yml (деплой приложения):**
```yaml
---
- name: Deploy application with zero downtime
  hosts: appservers
  serial: 1
  
  pre_tasks:
    - name: Validate deployment variables
      assert:
        that:
          - app_version is defined
          - app_version | length > 0
        fail_msg: "app_version must be specified"
  
  roles:
    - role: appserver
      tags: deploy
  
  tasks:
    - block:
        - name: Remove from load balancer
          # Implementation
        
        - name: Stop application
          service:
            name: myapp
            state: stopped
        
        - name: Backup current version
          # Implementation
        
        - name: Deploy new version
          git:
            repo: "{{ app_repo }}"
            dest: "{{ app_path }