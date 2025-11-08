# Руководство по 20 популярным модулям Ansible для DevOps инженеров

## Введение

Ansible - это инструмент автоматизации IT-инфраструктуры, использующий декларативный подход для управления конфигурациями. Модули Ansible - это переиспользуемые единицы кода, выполняющие конкретные задачи на управляемых узлах.

---

## 1. `ansible.builtin.copy`

### Назначение
Копирование файлов с управляющей машины на удаленные хосты.

### Основные параметры
- `src` - источник файла на локальной машине
- `dest` - путь назначения на удаленном хосте
- `owner` - владелец файла
- `group` - группа файла
- `mode` - права доступа (например, '0644')
- `backup` - создать резервную копию перед заменой

### Примеры использования

```yaml
# Простое копирование файла
- name: Копирование конфигурационного файла
  ansible.builtin.copy:
    src: /local/path/config.conf
    dest: /etc/myapp/config.conf
    owner: root
    group: root
    mode: '0644'

# Копирование с созданием резервной копии
- name: Обновление конфигурации с бэкапом
  ansible.builtin.copy:
    src: nginx.conf
    dest: /etc/nginx/nginx.conf
    backup: yes
    
# Создание файла с содержимым напрямую
- name: Создание файла с текстом
  ansible.builtin.copy:
    content: |
      server {
        listen 80;
        server_name example.com;
      }
    dest: /etc/nginx/sites-available/example.conf
```

---

## 2. `ansible.builtin.template`

### Назначение
Обработка Jinja2 шаблонов и развертывание результата на удаленных хостах.

### Основные параметры
- `src` - путь к шаблону Jinja2
- `dest` - путь назначения
- `validate` - команда для валидации файла перед заменой

### Примеры использования

```yaml
# Развертывание шаблона конфигурации
- name: Генерация конфигурации из шаблона
  ansible.builtin.template:
    src: templates/app_config.j2
    dest: /etc/myapp/config.yml
    owner: appuser
    group: appuser
    mode: '0640'

# С валидацией синтаксиса
- name: Конфигурация Nginx с проверкой
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    validate: 'nginx -t -c %s'
```

**Пример шаблона (nginx.conf.j2):**
```jinja2
server {
    listen {{ nginx_port }};
    server_name {{ server_name }};
    root {{ document_root }};
    
    {% for location in locations %}
    location {{ location.path }} {
        proxy_pass {{ location.backend }};
    }
    {% endfor %}
}
```

---

## 3. `ansible.builtin.file`

### Назначение
Управление файлами, директориями, симлинками и их атрибутами.

### Основные параметры
- `path` - путь к файлу/директории
- `state` - желаемое состояние (file, directory, link, absent, touch)
- `recurse` - рекурсивное применение (для директорий)

### Примеры использования

```yaml
# Создание директории
- name: Создание директории приложения
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
    owner: appuser
    group: appuser
    mode: '0755'

# Создание символической ссылки
- name: Создание симлинка
  ansible.builtin.file:
    src: /opt/myapp/current
    dest: /opt/myapp/release-1.2.3
    state: link

# Удаление файла
- name: Удаление старого конфига
  ansible.builtin.file:
    path: /etc/old_config.conf
    state: absent

# Изменение прав рекурсивно
- name: Установка прав на директорию
  ansible.builtin.file:
    path: /var/www/html
    state: directory
    recurse: yes
    owner: www-data
    group: www-data
```

---

## 4. `ansible.builtin.apt`

### Назначение
Управление пакетами в Debian/Ubuntu системах.

### Основные параметры
- `name` - имя пакета или список пакетов
- `state` - состояние (present, absent, latest)
- `update_cache` - обновить кэш пакетов
- `cache_valid_time` - время актуальности кэша в секундах

### Примеры использования

```yaml
# Установка одного пакета
- name: Установка Nginx
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: yes

# Установка нескольких пакетов
- name: Установка необходимых пакетов
  ansible.builtin.apt:
    name:
      - nginx
      - postgresql
      - redis-server
      - git
    state: latest
    update_cache: yes
    cache_valid_time: 3600

# Удаление пакета
- name: Удаление Apache
  ansible.builtin.apt:
    name: apache2
    state: absent
    purge: yes

# Обновление всех пакетов
- name: Обновление системы
  ansible.builtin.apt:
    upgrade: dist
    update_cache: yes
```

---

## 5. `ansible.builtin.yum`

### Назначение
Управление пакетами в RHEL/CentOS/Fedora системах.

### Основные параметры
- `name` - имя пакета
- `state` - состояние (present, absent, latest)
- `enablerepo` - включить репозиторий для операции
- `disablerepo` - отключить репозиторий

### Примеры использования

```yaml
# Установка пакета
- name: Установка httpd
  ansible.builtin.yum:
    name: httpd
    state: present

# Установка из определенного репозитория
- name: Установка из EPEL
  ansible.builtin.yum:
    name: htop
    state: present
    enablerepo: epel

# Установка нескольких пакетов
- name: Установка dev tools
  ansible.builtin.yum:
    name:
      - gcc
      - make
      - git
      - vim
    state: present

# Обновление всех пакетов
- name: Обновление системы
  ansible.builtin.yum:
    name: '*'
    state: latest
```

---

## 6. `ansible.builtin.service`

### Назначение
Управление системными сервисами (systemd, init.d и др.).

### Основные параметры
- `name` - имя сервиса
- `state` - желаемое состояние (started, stopped, restarted, reloaded)
- `enabled` - включить автозапуск

### Примеры использования

```yaml
# Запуск и включение сервиса
- name: Запуск Nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: yes

# Перезапуск сервиса
- name: Перезапуск после изменения конфигурации
  ansible.builtin.service:
    name: postgresql
    state: restarted

# Остановка и отключение сервиса
- name: Остановка Apache
  ansible.builtin.service:
    name: apache2
    state: stopped
    enabled: no

# Reload конфигурации без перезапуска
- name: Перечитывание конфигурации Nginx
  ansible.builtin.service:
    name: nginx
    state: reloaded
```

---

## 7. `ansible.builtin.systemd`

### Назначение
Управление systemd юнитами с расширенными возможностями.

### Основные параметры
- `name` - имя юнита
- `daemon_reload` - перечитать конфигурацию systemd
- `masked` - маскировать/размаскировать юнит

### Примеры использования

```yaml
# Перезагрузка демона и запуск сервиса
- name: Применение нового юнит-файла
  ansible.builtin.systemd:
    name: myapp
    state: restarted
    daemon_reload: yes
    enabled: yes

# Маскирование сервиса
- name: Отключение ненужного сервиса
  ansible.builtin.systemd:
    name: bluetooth.service
    masked: yes

# Проверка состояния сервиса
- name: Получение статуса
  ansible.builtin.systemd:
    name: nginx
  register: nginx_status

- name: Вывод статуса
  ansible.builtin.debug:
    var: nginx_status.status.ActiveState
```

---

## 8. `ansible.builtin.user`

### Назначение
Управление пользователями системы.

### Основные параметры
- `name` - имя пользователя
- `state` - состояние (present, absent)
- `groups` - список групп
- `shell` - оболочка пользователя
- `create_home` - создать домашнюю директорию

### Примеры использования

```yaml
# Создание пользователя
- name: Создание пользователя приложения
  ansible.builtin.user:
    name: appuser
    comment: "Application User"
    shell: /bin/bash
    create_home: yes
    groups: www-data,docker
    append: yes

# Создание системного пользователя
- name: Системный пользователь для сервиса
  ansible.builtin.user:
    name: serviceuser
    system: yes
    shell: /usr/sbin/nologin
    create_home: no

# Удаление пользователя
- name: Удаление старого пользователя
  ansible.builtin.user:
    name: olduser
    state: absent
    remove: yes

# Установка SSH ключа
- name: Добавление SSH ключа
  ansible.builtin.user:
    name: deployuser
    state: present
    generate_ssh_key: yes
    ssh_key_bits: 4096
```

---

## 9. `ansible.builtin.group`

### Назначение
Управление группами пользователей.

### Основные параметры
- `name` - имя группы
- `state` - состояние (present, absent)
- `gid` - ID группы
- `system` - системная группа

### Примеры использования

```yaml
# Создание группы
- name: Создание группы разработчиков
  ansible.builtin.group:
    name: developers
    state: present
    gid: 1500

# Создание системной группы
- name: Группа для приложения
  ansible.builtin.group:
    name: appgroup
    system: yes
    state: present

# Удаление группы
- name: Удаление старой группы
  ansible.builtin.group:
    name: oldgroup
    state: absent
```

---

## 10. `ansible.builtin.command`

### Назначение
Выполнение команд на удаленных хостах (без использования shell).

### Основные параметры
- `cmd` - команда для выполнения
- `chdir` - изменить директорию перед выполнением
- `creates` - путь к файлу, если существует - команда не выполняется
- `removes` - путь к файлу, если не существует - команда не выполняется

### Примеры использования

```yaml
# Простое выполнение команды
- name: Проверка версии Python
  ansible.builtin.command: python3 --version
  register: python_version

# С изменением директории
- name: Сборка проекта
  ansible.builtin.command: make build
  args:
    chdir: /opt/myapp

# С условием существования файла
- name: Инициализация базы данных
  ansible.builtin.command: /opt/app/init_db.sh
  args:
    creates: /var/lib/app/db_initialized

# Использование register для сохранения результата
- name: Получение списка контейнеров
  ansible.builtin.command: docker ps -a
  register: docker_containers

- name: Вывод результата
  ansible.builtin.debug:
    msg: "{{ docker_containers.stdout_lines }}"
```

---

## 11. `ansible.builtin.shell`

### Назначение
Выполнение команд через shell (поддержка пайпов, редиректов и переменных окружения).

### Основные параметры
- `cmd` - команда для выполнения
- `executable` - shell для использования (по умолчанию /bin/sh)

### Примеры использования

```yaml
# Команда с пайпом
- name: Поиск процессов Java
  ansible.builtin.shell: ps aux | grep java | grep -v grep
  register: java_processes
  failed_when: false

# С редиректом
- name: Сохранение логов
  ansible.builtin.shell: journalctl -u myapp > /tmp/app.log

# Использование переменных окружения
- name: Выполнение с переменными окружения
  ansible.builtin.shell: |
    export APP_ENV=production
    export DB_HOST={{ db_host }}
    /opt/app/deploy.sh
  args:
    executable: /bin/bash

# Многострочный скрипт
- name: Комплексная операция
  ansible.builtin.shell: |
    cd /opt/myapp
    git pull origin main
    npm install
    npm run build
  args:
    executable: /bin/bash
```

---

## 12. `ansible.builtin.git`

### Назначение
Управление Git репозиториями.

### Основные параметры
- `repo` - URL репозитория
- `dest` - путь назначения
- `version` - ветка, тег или commit
- `force` - сбросить локальные изменения
- `update` - обновить при повторном запуске

### Примеры использования

```yaml
# Клонирование репозитория
- name: Клонирование приложения
  ansible.builtin.git:
    repo: https://github.com/example/myapp.git
    dest: /opt/myapp
    version: main

# Клонирование определенной ветки
- name: Checkout production ветки
  ansible.builtin.git:
    repo: git@github.com:company/app.git
    dest: /var/www/app
    version: production
    key_file: /home/deploy/.ssh/id_rsa
    accept_hostkey: yes

# Обновление до определенного тега
- name: Деплой релиза
  ansible.builtin.git:
    repo: https://github.com/example/myapp.git
    dest: /opt/myapp
    version: v1.2.3
    force: yes

# С рекурсивным клонированием submodules
- name: Клонирование с подмодулями
  ansible.builtin.git:
    repo: https://github.com/example/project.git
    dest: /opt/project
    recursive: yes
    update: yes
```

---

## 13. `ansible.builtin.lineinfile`

### Назначение
Управление строками в текстовых файлах (добавление, изменение, удаление).

### Основные параметры
- `path` - путь к файлу
- `line` - строка для вставки
- `regexp` - регулярное выражение для поиска строки
- `state` - состояние (present, absent)
- `insertafter`/`insertbefore` - позиция вставки

### Примеры использования

```yaml
# Добавление строки если её нет
- name: Добавление записи в hosts
  ansible.builtin.lineinfile:
    path: /etc/hosts
    line: "192.168.1.100 app.example.com"
    state: present

# Замена строки по регулярному выражению
- name: Изменение порта SSH
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^#?Port '
    line: 'Port 2222'
    backup: yes

# Добавление после определенной строки
- name: Добавление конфигурации
  ansible.builtin.lineinfile:
    path: /etc/nginx/nginx.conf
    insertafter: '^http {'
    line: '    client_max_body_size 100M;'

# Удаление строки
- name: Удаление старой конфигурации
  ansible.builtin.lineinfile:
    path: /etc/app/config.ini
    regexp: '^old_setting='
    state: absent

# Добавление в начало файла
- name: Добавление shebang
  ansible.builtin.lineinfile:
    path: /opt/script.sh
    line: '#!/bin/bash'
    insertbefore: BOF
```

---

## 14. `ansible.builtin.replace`

### Назначение
Замена текста в файлах по регулярному выражению.

### Основные параметры
- `path` - путь к файлу
- `regexp` - регулярное выражение для поиска
- `replace` - строка замены

### Примеры использования

```yaml
# Замена значения
- name: Изменение параметра в конфиге
  ansible.builtin.replace:
    path: /etc/myapp/config.yml
    regexp: 'max_connections: \d+'
    replace: 'max_connections: 1000'
    backup: yes

# Замена с использованием групп
- name: Обновление версии в файле
  ansible.builtin.replace:
    path: /opt/app/version.txt
    regexp: '^version: (\d+\.\d+)\.\d+$'
    replace: 'version: \g<1>.{{ new_patch_version }}'

# Множественная замена
- name: Замена переменных окружения
  ansible.builtin.replace:
    path: /etc/environment
    regexp: 'OLD_VAR_NAME'
    replace: 'NEW_VAR_NAME'
```

---

## 15. `ansible.builtin.get_url`

### Назначение
Скачивание файлов из интернета.

### Основные параметры
- `url` - URL источника
- `dest` - путь назначения
- `checksum` - контрольная сумма для проверки
- `timeout` - таймаут подключения

### Примеры использования

```yaml
# Простое скачивание файла
- name: Скачивание бинарника
  ansible.builtin.get_url:
    url: https://releases.example.com/app/v1.2.3/app-linux-amd64
    dest: /usr/local/bin/app
    mode: '0755'

# С проверкой контрольной суммы
- name: Скачивание с верификацией
  ansible.builtin.get_url:
    url: https://example.com/package.tar.gz
    dest: /tmp/package.tar.gz
    checksum: sha256:abc123...

# С авторизацией
- name: Скачивание из защищенного источника
  ansible.builtin.get_url:
    url: https://private.repo.com/artifact.jar
    dest: /opt/app/lib/artifact.jar
    url_username: "{{ repo_user }}"
    url_password: "{{ repo_password }}"
    force_basic_auth: yes

# С таймаутом и retry
- name: Скачивание большого файла
  ansible.builtin.get_url:
    url: https://downloads.example.com/bigfile.iso
    dest: /tmp/bigfile.iso
    timeout: 300
  retries: 3
  delay: 10
```

---

## 16. `ansible.builtin.unarchive`

### Назначение
Распаковка архивов (tar, gz, bz2, zip) на удаленных хостах.

### Основные параметры
- `src` - источник архива
- `dest` - путь для распаковки
- `remote_src` - архив находится на удаленном хосте
- `creates` - путь, если существует - распаковка не выполняется

### Примеры использования

```yaml
# Распаковка локального архива на удаленный хост
- name: Распаковка приложения
  ansible.builtin.unarchive:
    src: /local/path/app-1.2.3.tar.gz
    dest: /opt/app
    owner: appuser
    group: appuser

# Распаковка архива, уже находящегося на удаленном хосте
- name: Распаковка скачанного архива
  ansible.builtin.unarchive:
    src: /tmp/package.tar.gz
    dest: /opt/
    remote_src: yes

# Распаковка с извлечением определенных файлов
- name: Извлечение конфигурации
  ansible.builtin.unarchive:
    src: backup.tar.gz
    dest: /etc/
    remote_src: yes
    include:
      - app/config.yml
      - app/secrets.env

# Распаковка с условием
- name: Распаковка если не распаковано
  ansible.builtin.unarchive:
    src: https://example.com/release.tar.gz
    dest: /opt/app
    remote_src: yes
    creates: /opt/app/bin/app
```

---

## 17. `ansible.builtin.cron`

### Назначение
Управление cron заданиями.

### Основные параметры
- `name` - имя задания (комментарий в crontab)
- `job` - команда для выполнения
- `minute`, `hour`, `day`, `month`, `weekday` - расписание
- `user` - пользователь для crontab
- `state` - состояние (present, absent)

### Примеры использования

```yaml
# Создание ежедневного задания
- name: Ежедневный бэкап базы данных
  ansible.builtin.cron:
    name: "Daily DB backup"
    minute: "0"
    hour: "2"
    job: "/opt/scripts/backup_db.sh"
    user: root

# Задание выполняемое каждые 15 минут
- name: Проверка состояния сервисов
  ansible.builtin.cron:
    name: "Health check"
    minute: "*/15"
    job: "/usr/local/bin/health_check.sh"

# Еженедельное задание
- name: Еженедельная очистка логов
  ansible.builtin.cron:
    name: "Weekly log cleanup"
    minute: "0"
    hour: "3"
    weekday: "0"
    job: "find /var/log/app -name '*.log' -mtime +30 -delete"

# Удаление cron задания
- name: Удаление старого задания
  ansible.builtin.cron:
    name: "Old backup job"
    state: absent

# С переменными окружения
- name: Задание с переменными
  ansible.builtin.cron:
    name: "Application task"
    minute: "30"
    hour: "1"
    job: "cd /opt/app && /usr/bin/python3 task.py"
    env: yes
    cron_file: app_tasks
```

---

## 18. `ansible.builtin.debug`

### Назначение
Вывод отладочной информации во время выполнения playbook.

### Основные параметры
- `msg` - сообщение для вывода
- `var` - переменная для вывода
- `verbosity` - уровень детализации (0-3)

### Примеры использования

```yaml
# Вывод простого сообщения
- name: Информационное сообщение
  ansible.builtin.debug:
    msg: "Начинается деплой версии {{ app_version }}"

# Вывод переменной
- name: Показать значение переменной
  ansible.builtin.debug:
    var: ansible_default_ipv4.address

# Вывод сложной структуры
- name: Показать результат команды
  ansible.builtin.debug:
    var: command_result.stdout_lines

# Условный вывод
- name: Предупреждение при низкой памяти
  ansible.builtin.debug:
    msg: "ВНИМАНИЕ: Мало свободной памяти - {{ ansible_memfree_mb }}MB"
  when: ansible_memfree_mb < 1000

# Вывод только при высоком уровне verbosity
- name: Детальная отладочная информация
  ansible.builtin.debug:
    msg: "Детали конфигурации: {{ config }}"
    verbosity: 2

# Форматированный вывод
- name: Красивый вывод JSON
  ansible.builtin.debug:
    msg: "{{ api_response | to_nice_json }}"
```

---

## 19. `ansible.builtin.wait_for`

### Назначение
Ожидание выполнения условий (доступность порта, наличие файла, строки в файле).

### Основные параметры
- `port` - номер порта для проверки
- `host` - хост для проверки
- `path` - путь к файлу
- `timeout` - максимальное время ожидания
- `delay` - задержка перед началом проверки
- `state` - состояние (started, stopped, present, absent)

### Примеры использования

```yaml
# Ожидание доступности порта
- name: Ожидание запуска базы данных
  ansible.builtin.wait_for:
    port: 5432
    host: "{{ db_host }}"
    delay: 5
    timeout: 300

# Ожидание создания файла
- name: Ожидание завершения инициализации
  ansible.builtin.wait_for:
    path: /var/lib/app/initialized
    state: present
    timeout: 600

# Ожидание остановки сервиса
- name: Ожидание остановки старого процесса
  ansible.builtin.wait_for:
    port: 8080
    state: stopped
    timeout: 60

# Ожидание строки в файле
- name: Ожидание сообщения в логе
  ansible.builtin.wait_for:
    path: /var/log/app/startup.log
    search_regex: "Application started successfully"
    timeout: 120

# Ожидание с проверкой по HTTP
- name: Проверка здоровья приложения
  ansible.builtin.wait_for:
    host: localhost
    port: 8080
    delay: 10
    timeout: 300
  
- name: HTTP проверка
  ansible.builtin.uri:
    url: http://localhost:8080/health
    status_code: 200
  retries: 30
  delay: 2
```

---

## 20. `ansible.builtin.uri`

### Назначение
Взаимодействие с HTTP и HTTPS сервисами (REST API).

### Основные параметры
- `url` - URL для запроса
- `method` - HTTP метод (GET, POST, PUT, DELETE и т.д.)
- `body` - тело запроса
- `headers` - HTTP заголовки
- `status_code` - ожидаемые коды ответа
- `return_content` - вернуть содержимое ответа

### Примеры использования

```yaml
# GET запрос
- name: Проверка API
  ansible.builtin.uri:
    url: https://api.example.com/v1/status
    method: GET
    return_content: yes
  register: api_status

# POST запрос с JSON
- name: Создание ресурса через API
  ansible.builtin.uri:
    url: https://api.example.com/v1/resources
    method: POST
    body_format: json
    body:
      name: "New Resource"
      type: "server"
      region: "us-east-1"
    headers:
      Authorization: "Bearer {{ api_token }}"
      Content-Type: "application/json"
    status_code: [200, 201]
  register: create_result

# PUT запрос для обновления
- name: Обновление конфигурации
  ansible.builtin.uri:
    url: "https://api.example.com/v1/config/{{ config_id }}"
    method: PUT
    body_format: json
    body:
      enabled: true
      max_connections: 100
    headers:
      Authorization: "Bearer {{ api_token }}"

# DELETE запрос
- name: Удаление ресурса
  ansible.builtin.uri:
    url: "https://api.example.com/v1/resources/{{ resource_id }}"
    method: DELETE
    headers:
      Authorization: "Bearer {{ api_token }}"
    status_code: [200, 204]

#