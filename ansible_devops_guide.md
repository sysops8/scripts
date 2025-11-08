# 20 Самых Используемых Ansible Features для DevOps - Пошаговая Инструкция

## Подготовка окружения

### 1. Установка Ansible
```bash
# Для Ubuntu/Debian
sudo apt update
sudo apt install software-properties-common -y
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y

# Для RHEL/CentOS
sudo yum install epel-release -y
sudo yum install ansible -y

# Через pip (рекомендуется для последней версии)
pip3 install ansible
```

### 2. Проверка установки
```bash
ansible --version
ansible-playbook --version
```

### 3. Создание рабочей директории
```bash
mkdir -p ~/ansible-projects
cd ~/ansible-projects
mkdir inventory playbooks roles group_vars host_vars
```

### 4. Базовая структура проекта
```
ansible-projects/
├── ansible.cfg
├── inventory/
│   ├── production
│   └── staging
├── playbooks/
├── roles/
├── group_vars/
└── host_vars/
```

---

## Feature 1: Inventory Management (Управление инвентарем)

### Статический инвентарь (INI формат)
```ini
# inventory/production
[webservers]
web1.example.com ansible_host=192.168.1.10
web2.example.com ansible_host=192.168.1.11

[databases]
db1.example.com ansible_host=192.168.1.20
db2.example.com ansible_host=192.168.1.21

[loadbalancers]
lb1.example.com ansible_host=192.168.1.30

[production:children]
webservers
databases
loadbalancers

[production:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
```

### Динамический инвентарь (YAML формат)
```yaml
# inventory/staging.yml
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.2.10
          http_port: 8080
        web2:
          ansible_host: 192.168.2.11
          http_port: 8080
    databases:
      hosts:
        db1:
          ansible_host: 192.168.2.20
          mysql_port: 3306
      vars:
        db_engine: mysql
```

### Проверка инвентаря
```bash
# Список всех хостов
ansible all -i inventory/production --list-hosts

# Проверка доступности
ansible all -i inventory/production -m ping
```

---

## Feature 2: Ad-Hoc Commands (Быстрые команды)

### Базовые команды
```bash
# Выполнить команду на всех хостах
ansible all -i inventory/production -a "uptime"

# Установить пакет
ansible webservers -i inventory/production -m apt -a "name=nginx state=present" --become

# Перезапустить сервис
ansible webservers -i inventory/production -m service -a "name=nginx state=restarted" --become

# Копировать файл
ansible all -i inventory/production -m copy -a "src=/local/file dest=/remote/file mode=0644"

# Собрать факты о системе
ansible all -i inventory/production -m setup

# Проверить дисковое пространство
ansible all -i inventory/production -a "df -h"

# Создать пользователя
ansible all -i inventory/production -m user -a "name=deploy state=present" --become
```

---

## Feature 3: Playbooks (Плейбуки)

### Базовый playbook
```yaml
# playbooks/webserver_setup.yml
---
- name: Setup Web Servers
  hosts: webservers
  become: yes
  
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
    
    - name: Install Nginx
      apt:
        name: nginx
        state: present
    
    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Copy index.html
      copy:
        src: files/index.html
        dest: /var/www/html/index.html
        mode: '0644'
      notify: Restart Nginx
  
  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

### Запуск playbook
```bash
ansible-playbook -i inventory/production playbooks/webserver_setup.yml

# С проверкой синтаксиса
ansible-playbook playbooks/webserver_setup.yml --syntax-check

# Dry-run (проверка без изменений)
ansible-playbook -i inventory/production playbooks/webserver_setup.yml --check

# С выводом изменений
ansible-playbook -i inventory/production playbooks/webserver_setup.yml --diff
```

---

## Feature 4: Variables (Переменные)

### Group variables
```yaml
# group_vars/webservers.yml
---
nginx_port: 80
nginx_worker_processes: 4
app_version: "1.2.3"
deployment_user: deploy

custom_packages:
  - git
  - curl
  - vim
```

### Host variables
```yaml
# host_vars/web1.example.com.yml
---
server_id: 1
max_connections: 1000
backup_enabled: true
```

### Использование переменных в playbook
```yaml
---
- name: Configure Nginx
  hosts: webservers
  become: yes
  
  vars:
    nginx_root: /var/www/html
    log_level: info
  
  tasks:
    - name: Deploy Nginx config
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Reload Nginx
    
    - name: Print variables
      debug:
        msg: "Configuring {{ inventory_hostname }} on port {{ nginx_port }}"
```

### Template с переменными
```jinja2
# templates/nginx.conf.j2
worker_processes {{ nginx_worker_processes }};

events {
    worker_connections {{ max_connections | default(768) }};
}

http {
    listen {{ nginx_port }};
    server_name {{ inventory_hostname }};
    root {{ nginx_root }};
    
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log {{ log_level }};
}
```

---

## Feature 5: Roles (Роли)

### Создание структуры роли
```bash
ansible-galaxy init roles/nginx
```

### Структура роли
```
roles/nginx/
├── defaults/
│   └── main.yml          # Переменные по умолчанию
├── files/                # Статические файлы
├── handlers/
│   └── main.yml          # Handlers
├── meta/
│   └── main.yml          # Зависимости роли
├── tasks/
│   └── main.yml          # Основные задачи
├── templates/            # Jinja2 шаблоны
├── tests/
│   └── test.yml          # Тесты роли
└── vars/
    └── main.yml          # Переменные роли
```

### Пример роли Nginx
```yaml
# roles/nginx/tasks/main.yml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Deploy Nginx configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    validate: 'nginx -t -c %s'
  notify: Restart Nginx

- name: Ensure Nginx is running
  service:
    name: nginx
    state: started
    enabled: yes
```

```yaml
# roles/nginx/handlers/main.yml
---
- name: Restart Nginx
  service:
    name: nginx
    state: restarted

- name: Reload Nginx
  service:
    name: nginx
    state: reloaded
```

```yaml
# roles/nginx/defaults/main.yml
---
nginx_port: 80
nginx_worker_processes: auto
nginx_user: www-data
```

### Использование роли в playbook
```yaml
# playbooks/deploy_webservers.yml
---
- name: Deploy Web Servers
  hosts: webservers
  become: yes
  
  roles:
    - role: nginx
      nginx_port: 8080
    - role: php-fpm
    - role: monitoring
```

---

## Feature 6: Handlers (Обработчики событий)

### Определение handlers
```yaml
---
- name: Configure services
  hosts: all
  become: yes
  
  tasks:
    - name: Update Nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify:
        - Validate Nginx config
        - Reload Nginx
    
    - name: Update PHP-FPM config
      template:
        src: php-fpm.conf.j2
        dest: /etc/php/7.4/fpm/php-fpm.conf
      notify: Restart PHP-FPM
  
  handlers:
    - name: Validate Nginx config
      command: nginx -t
    
    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
    
    - name: Restart PHP-FPM
      service:
        name: php7.4-fpm
        state: restarted
```

### Принудительный запуск handlers
```yaml
- name: Force handler execution
  meta: flush_handlers
```

---

## Feature 7: Templates (Шаблоны Jinja2)

### Базовый шаблон
```jinja2
# templates/app_config.j2
[application]
name = {{ app_name }}
version = {{ app_version }}
debug = {{ debug_mode | default('false') }}

[database]
host = {{ db_host }}
port = {{ db_port }}
name = {{ db_name }}
user = {{ db_user }}

[server]
{% if environment == 'production' %}
workers = {{ ansible_processor_vcpus * 2 }}
{% else %}
workers = 2
{% endif %}

listen = {{ ansible_default_ipv4.address }}:{{ app_port }}
```

### Условия и циклы в шаблонах
```jinja2
# templates/nginx_vhosts.j2
{% for vhost in virtual_hosts %}
server {
    listen {{ vhost.port }};
    server_name {{ vhost.name }};
    root {{ vhost.root }};
    
    {% if vhost.ssl_enabled %}
    ssl_certificate {{ vhost.ssl_cert }};
    ssl_certificate_key {{ vhost.ssl_key }};
    {% endif %}
    
    location / {
        {% if vhost.proxy_pass %}
        proxy_pass {{ vhost.proxy_pass }};
        {% else %}
        try_files $uri $uri/ =404;
        {% endif %}
    }
}
{% endfor %}
```

### Использование фильтров
```jinja2
# Преобразование в upper case
{{ app_name | upper }}

# Значение по умолчанию
{{ custom_var | default('default_value') }}

# Форматирование JSON
{{ complex_data | to_json }}

# Форматирование YAML
{{ complex_data | to_nice_yaml }}

# Объединение списков
{{ list1 | union(list2) }}
```

---

## Feature 8: Conditionals (Условия)

### When условия
```yaml
---
- name: Conditional tasks
  hosts: all
  become: yes
  
  tasks:
    - name: Install Apache on Debian/Ubuntu
      apt:
        name: apache2
        state: present
      when: ansible_os_family == "Debian"
    
    - name: Install Apache on RedHat/CentOS
      yum:
        name: httpd
        state: present
      when: ansible_os_family == "RedHat"
    
    - name: Configure firewall for production
      ufw:
        rule: allow
        port: 443
      when: 
        - environment == "production"
        - ansible_distribution == "Ubuntu"
    
    - name: Install development tools
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - build-essential
        - gdb
      when: deployment_type == "development"
```

### Сложные условия
```yaml
- name: Complex conditionals
  debug:
    msg: "This is a production web server"
  when: >
    (inventory_hostname in groups['webservers']) and
    (environment == 'production') and
    (ansible_memtotal_mb > 4096)
```

---

## Feature 9: Loops (Циклы)

### Базовые циклы
```yaml
---
- name: Loop examples
  hosts: all
  become: yes
  
  tasks:
    - name: Install multiple packages
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - git
        - curl
        - vim
    
    - name: Create multiple users
      user:
        name: "{{ item.name }}"
        state: present
        groups: "{{ item.groups }}"
      loop:
        - { name: 'alice', groups: 'developers' }
        - { name: 'bob', groups: 'admins' }
        - { name: 'charlie', groups: 'developers,sudo' }
    
    - name: Create directories
      file:
        path: "{{ item }}"
        state: directory
        mode: '0755'
      loop:
        - /opt/app/logs
        - /opt/app/data
        - /opt/app/config
```

### Loop с словарями
```yaml
- name: Deploy multiple applications
  template:
    src: "{{ item.key }}.conf.j2"
    dest: "/etc/{{ item.key }}.conf"
  loop: "{{ applications | dict2items }}"
  vars:
    applications:
      app1:
        port: 8080
        workers: 4
      app2:
        port: 8081
        workers: 2
```

### Loop с until
```yaml
- name: Wait for service to be ready
  uri:
    url: "http://{{ inventory_hostname }}:8080/health"
    status_code: 200
  register: result
  until: result.status == 200
  retries: 10
  delay: 5
```

---

## Feature 10: Ansible Vault (Шифрование данных)

### Создание зашифрованного файла
```bash
# Создать новый vault файл
ansible-vault create secrets.yml

# Редактировать vault файл
ansible-vault edit secrets.yml

# Изменить пароль
ansible-vault rekey secrets.yml

# Шифровать существующий файл
ansible-vault encrypt existing_file.yml

# Расшифровать файл
ansible-vault decrypt secrets.yml

# Просмотреть содержимое
ansible-vault view secrets.yml
```

### Содержимое secrets.yml
```yaml
---
db_password: "SuperSecretPassword123!"
api_key: "sk-1234567890abcdef"
ssl_private_key: |
  -----BEGIN PRIVATE KEY-----
  MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQC...
  -----END PRIVATE KEY-----
```

### Использование vault в playbook
```bash
# Запуск с запросом пароля
ansible-playbook playbook.yml --ask-vault-pass

# Использование файла с паролем
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass

# Несколько vault паролей
ansible-playbook playbook.yml --vault-id prod@~/.vault_pass_prod --vault-id dev@~/.vault_pass_dev
```

### Встроенные зашифрованные переменные
```yaml
---
- name: Use encrypted variables
  hosts: databases
  become: yes
  
  vars_files:
    - secrets.yml
  
  tasks:
    - name: Create database user
      mysql_user:
        name: admin
        password: "{{ db_password }}"
        priv: '*.*:ALL'
        state: present
```

---

## Feature 11: Tags (Теги)

### Определение тегов
```yaml
---
- name: Full application deployment
  hosts: webservers
  become: yes
  
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
      tags:
        - packages
        - setup
    
    - name: Install packages
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - php-fpm
      tags:
        - packages
    
    - name: Deploy application code
      git:
        repo: 'https://github.com/user/app.git'
        dest: /var/www/app
        version: main
      tags:
        - deploy
        - code
    
    - name: Configure Nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      tags:
        - config
        - nginx
      notify: Reload Nginx
    
    - name: Run database migrations
      command: /var/www/app/migrate.sh
      tags:
        - database
        - deploy
```

### Использование тегов
```bash
# Выполнить только задачи с тегом
ansible-playbook playbook.yml --tags "deploy"

# Выполнить несколько тегов
ansible-playbook playbook.yml --tags "config,nginx"

# Пропустить задачи с тегом
ansible-playbook playbook.yml --skip-tags "database"

# Список всех тегов
ansible-playbook playbook.yml --list-tags

# Выполнить все задачи с тегами (по умолчанию выполняются все)
ansible-playbook playbook.yml --tags "all"
```

---

## Feature 12: Blocks (Блоки задач)

### Группировка задач
```yaml
---
- name: Block examples
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install and configure web server
      block:
        - name: Install Nginx
          apt:
            name: nginx
            state: present
        
        - name: Copy configuration
          template:
            src: nginx.conf.j2
            dest: /etc/nginx/nginx.conf
        
        - name: Start Nginx
          service:
            name: nginx
            state: started
      when: ansible_os_family == "Debian"
      tags:
        - webserver
```

### Обработка ошибок с rescue
```yaml
- name: Error handling with blocks
  hosts: all
  
  tasks:
    - name: Attempt risky operation
      block:
        - name: Try to download file
          get_url:
            url: "https://example.com/file.tar.gz"
            dest: /tmp/file.tar.gz
        
        - name: Extract archive
          unarchive:
            src: /tmp/file.tar.gz
            dest: /opt/app
            remote_src: yes
      
      rescue:
        - name: Log error
          debug:
            msg: "Download or extraction failed, trying backup method"
        
        - name: Use backup source
          get_url:
            url: "https://backup.example.com/file.tar.gz"
            dest: /tmp/file.tar.gz
      
      always:
        - name: Cleanup temporary files
          file:
            path: /tmp/file.tar.gz
            state: absent
```

---

## Feature 13: Facts (Системная информация)

### Сбор фактов
```yaml
---
- name: Gather and use facts
  hosts: all
  
  tasks:
    - name: Display OS information
      debug:
        msg: >
          Host: {{ inventory_hostname }}
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          Kernel: {{ ansible_kernel }}
          Architecture: {{ ansible_architecture }}
          CPUs: {{ ansible_processor_vcpus }}
          Memory: {{ ansible_memtotal_mb }} MB
          IP: {{ ansible_default_ipv4.address }}
    
    - name: Install package based on OS
      package:
        name: "{{ 'httpd' if ansible_os_family == 'RedHat' else 'apache2' }}"
        state: present
      become: yes
```

### Кастомные факты
```bash
# Создать директорию для локальных фактов
sudo mkdir -p /etc/ansible/facts.d

# Создать файл с фактами
sudo cat > /etc/ansible/facts.d/custom.fact << 'EOF'
#!/bin/bash
echo "{\"environment\": \"production\", \"app_version\": \"2.1.0\"}"
EOF

sudo chmod +x /etc/ansible/facts.d/custom.fact
```

```yaml
# Использование кастомных фактов
- name: Use custom facts
  debug:
    msg: "Environment: {{ ansible_local.custom.environment }}"
```

### Отключение сбора фактов
```yaml
---
- name: Fast playbook without facts
  hosts: all
  gather_facts: no
  
  tasks:
    - name: Quick task
      command: echo "Hello"
```

---

## Feature 14: Register и Set_fact (Сохранение результатов)

### Register переменных
```yaml
---
- name: Register examples
  hosts: webservers
  
  tasks:
    - name: Check if file exists
      stat:
        path: /etc/myapp/config.yml
      register: config_file
    
    - name: Display result
      debug:
        msg: "Config file exists: {{ config_file.stat.exists }}"
    
    - name: Create config if missing
      template:
        src: config.yml.j2
        dest: /etc/myapp/config.yml
      when: not config_file.stat.exists
    
    - name: Get service status
      command: systemctl status nginx
      register: nginx_status
      failed_when: false
      changed_when: false
    
    - name: Show service status
      debug:
        var: nginx_status.stdout_lines
```

### Set_fact для вычисляемых переменных
```yaml
- name: Set fact examples
  hosts: all
  
  tasks:
    - name: Calculate deployment timestamp
      set_fact:
        deployment_time: "{{ ansible_date_time.iso8601 }}"
        deployment_id: "{{ ansible_date_time.epoch }}"
    
    - name: Set facts based on conditions
      set_fact:
        max_workers: "{{ ansible_processor_vcpus * 2 }}"
        memory_limit: "{{ (ansible_memtotal_mb * 0.7) | int }}m"
    
    - name: Complex fact calculation
      set_fact:
        server_tier: "{{ 'premium' if ansible_memtotal_mb > 16384 else 'standard' }}"
    
    - name: Use calculated facts
      debug:
        msg: "Server {{ inventory_hostname }} is {{ server_tier }} tier with {{ max_workers }} workers"
```

---

## Feature 15: Delegation и Local Actions

### Delegation (Делегирование задач)
```yaml
---
- name: Delegation examples
  hosts: webservers
  
  tasks:
    - name: Add host to load balancer
      command: /usr/local/bin/add_to_lb.sh {{ inventory_hostname }}
      delegate_to: lb1.example.com
    
    - name: Deploy application
      copy:
        src: /local/app.tar.gz
        dest: /opt/app/
    
    - name: Remove from load balancer
      command: /usr/local/bin/remove_from_lb.sh {{ inventory_hostname }}
      delegate_to: lb1.example.com
    
    - name: Query database
      postgresql_query:
        query: "SELECT version()"
      delegate_to: db1.example.com
      run_once: true
      register: db_version
```

### Local actions
```yaml
- name: Local action examples
  hosts: webservers
  
  tasks:
    - name: Create local backup directory
      local_action:
        module: file
        path: "/backups/{{ inventory_hostname }}"
        state: directory
    
    - name: Fetch remote log file
      fetch:
        src: /var/log/app.log
        dest: "/backups/{{ inventory_hostname }}/"
        flat: yes
    
    - name: Wait for server to respond
      local_action:
        module: wait_for
        host: "{{ inventory_hostname }}"
        port: 22
        timeout: 300
```

### Run_once
```yaml
- name: Database migration
  command: /opt/app/migrate.sh
  run_once: true
  delegate_to: "{{ groups['webservers'][0] }}"
```

---

## Feature 16: Docker и Containers

### Docker module примеры
```yaml
---
- name: Docker container management
  hosts: docker_hosts
  become: yes
  
  tasks:
    - name: Install Docker prerequisites
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
        state: present
    
    - name: Add Docker GPG key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present
    
    - name: Add Docker repository
      apt_repository:
        repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable
        state: present
    
    - name: Install Docker
      apt:
        name: docker-ce
        state: present
        update_cache: yes
    
    - name: Install Docker SDK for Python
      pip:
        name: docker
        state: present
    
    - name: Pull Docker image
      docker_image:
        name: nginx
        tag: latest
        source: pull
    
    - name: Create Docker network
      docker_network:
        name: app_network
        state: present
    
    - name: Run Nginx container
      docker_container:
        name: web_server
        image: nginx:latest
        state: started
        restart_policy: always
        ports:
          - "80:80"
          - "443:443"
        volumes:
          - /opt/nginx/conf:/etc/nginx/conf.d:ro
          - /opt/nginx/html:/usr/share/nginx/html:ro
        networks:
          - name: app_network
        env:
          NGINX_HOST: example.com
          NGINX_PORT: "80"
    
    - name: Run application container
      docker_container:
        name: app_backend
        image: myapp:latest
        state: started
        restart_policy: unless-stopped
        networks:
          - name: app_network
        env:
          DATABASE_URL: "postgresql://db:5432/myapp"
          REDIS_URL: "redis://redis:6379"
```

### Docker Compose
```yaml
- name: Deploy with Docker Compose
  hosts: docker_hosts
  become: yes
  
  tasks:
    - name: Install docker-compose
      pip:
        name: docker-compose
        state: present
    
    - name: Create app directory
      file:
        path: /opt/myapp
        state: directory
    
    - name: Copy docker-compose.yml
      copy:
        src: docker-compose.yml
        dest: /opt/myapp/docker-compose.yml
    
    - name: Deploy application
      docker_compose:
        project_src: /opt/myapp
        state: present
        pull: yes
```

---

## Feature 17: Kubernetes Integration

### Установка зависимостей
```bash
pip3 install kubernetes openshift
```

### Kubernetes playbook
```yaml
---
- name: Kubernetes deployment
  hosts: localhost
  connection: local
  
  vars:
    namespace: production
    app_name: myapp
    replicas: 3
  
  tasks:
    - name: Create namespace
      k8s:
        name: "{{ namespace }}"
        api_version: v1
        kind: Namespace
        state: present
    
    - name: Create ConfigMap
      k8s:
        state: present
        definition:
          apiVersion: v1
          kind: ConfigMap
          metadata:
            name: "{{ app_name }}-config"
            namespace: "{{ namespace }}"
          data:
            app.properties: |
              database.host=db.example.com
              database.port=5432
              log.level=INFO
    
    - name: Create Secret
      k8s:
        state: present
        definition:
          apiVersion: v1
          kind: Secret
          metadata:
            name: "{{ app_name }}-secret"
            namespace: "{{ namespace }}"
          type: Opaque
          stringData:
            db-password: "{{ vault_db_password }}"
            api-key: "{{ vault_api_key }}"
    
    - name: Deploy application
      k8s:
        state: present
        definition:
          apiVersion: apps/v1
          kind: Deployment
          metadata:
            name: "{{ app_name }}"
            namespace: "{{ namespace }}"
          spec:
            replicas: "{{ replicas }}"
            selector:
              matchLabels:
                app: "{{ app_name }}"
            template:
              metadata:
                labels:
                  app: "{{ app_name }}"
              spec:
                containers:
                - name: "{{ app_name }}"
                  image: "myregistry.com/{{ app_name }}:latest"
                  ports:
                  - containerPort: 8080
                  envFrom:
                  - configMapRef:
                      name: "{{ app_name }}-config"
                  - secretRef:
                      name: "{{ app_name }}-secret"
                  resources:
                    requests:
                      memory: "256Mi"
                      cpu: "250m"
                    limits:
                      memory: "512Mi"
                      cpu: "500m"
    
    - name: Create Service
      k8s:
        state: present
        definition:
          apiVersion: v1
          kind: Service
          metadata:
            name: "{{ app_name }}-service"
            namespace: "{{ namespace }}"
          spec:
            selector:
              app: "{{ app_name }}"
            ports:
            - protocol: TCP
              port: 80
              targetPort: 8080
            type: LoadBalancer
    
    - name: Wait for deployment to be ready
      k8s_info:
        kind: Deployment
        namespace: "{{ namespace }}"
        name: "{{ app_name }}"
      register: deployment
      until: deployment.resources[0].status.readyReplicas == replicas
      retries: 30
      delay: 10
```

---

## Feature 18: Rolling Updates и Zero Downtime Deployment

### Serial execution (постепенное обновление)
```yaml
---
- name: Rolling update of web servers
  hosts: webservers
  serial: 1  # Обновлять по одному хосту
  max_fail_percentage: 25
  
  tasks:
    - name: Remove from load balancer
      haproxy:
        state: disabled
        host: "{{ inventory_hostname }}"
        socket: /var/run/haproxy.sock
      delegate_to: "{{ item }}"
      loop: "{{ groups['loadbalancers'] }}"
    
    - name: Wait for connections to drain
      wait_for:
        timeout: 30
    
    - name: Stop application
      service:
        name: myapp
        state: stopped
    
    - name: Deploy new version
      copy:
        src: "/builds/myapp-{{ app_version }}.tar.gz"
        dest: /opt/myapp/
    
    - name: Extract application
      unarchive:
        src: "/opt/myapp/myapp-{{ app_version }}.tar.gz"
        dest: /opt/myapp/current
        remote_src: yes
    
    - name: Run database migrations
      command: /opt/myapp/current/migrate.sh
      run_once: true
    
    - name: Start application
      service:
        name: myapp
        state: started
    
    - name: Wait for application to be healthy
      uri:
        url: "http://{{ inventory_hostname }}:8080/health"
        status_code: 200
      register: result
      until: result.status == 200
      retries: 12
      delay: 5
    
    - name: Add back to load balancer
      haproxy:
        state: enabled
        host: "{{ inventory_hostname }}"
        socket: /var/run/haproxy.sock
      delegate_to: "{{ item }}"
      loop: "{{ groups['loadbalancers'] }}"
    
    - name: Wait before next server
      pause:
        seconds: 10
```

### Batch updates (пакетное обновление)
```yaml
---
- name: Batch update servers
  hosts: webservers
  serial: "30%"  # Обновлять 30% хостов одновременно
  
  tasks:
    - name: Update application
      # ... задачи обновления
```

### Canary deployment
```yaml
---
- name: Canary deployment
  hosts: webservers
  serial:
    - 1        # Первый сервер (canary)
    - 25%      # Затем 25%
    - 100%     # Затем все остальные
  
  tasks:
    - name: Deploy new version
      include_tasks: deploy_tasks.yml
    
    - name: Pause for canary validation
      pause:
        prompt: "Canary deployed. Check metrics and press enter to continue..."
      when: inventory_hostname == groups['webservers'][0]
```

---

## Feature 19: Ansible Tower/AWX Integration

### Настройка для Tower/AWX
```yaml
---
- name: Configure AWX inventory
  hosts: localhost
  connection: local
  
  tasks:
    - name: Create organization
      awx.awx.organization:
        name: "DevOps Team"
        description: "Main DevOps organization"
        state: present
        tower_host: "{{ tower_host }}"
        tower_username: "{{ tower_user }}"
        tower_password: "{{ tower_password }}"
    
    - name: Create inventory
      awx.awx.inventory:
        name: "Production Servers"
        organization: "DevOps Team"
        state: present
        tower_host: "{{ tower_host }}"
        tower_username: "{{ tower_user }}"
        tower_password: "{{ tower_password }}"
    
    - name: Add inventory source
      awx.awx.inventory_source:
        name: "AWS EC2"
        inventory: "Production Servers"
        source: ec2
        credential: "AWS Credentials"
        update_on_launch: yes
        state: present
        tower_host: "{{ tower_host }}"
        tower_username: "{{ tower_user }}"
        tower_password: "{{ tower_password }}"
    
    - name: Create job template
      awx.awx.job_template:
        name: "Deploy Web Application"
        job_type: run
        inventory: "Production Servers"
        project: "Web App Project"
        playbook: "deploy.yml"
        credentials:
          - "SSH Key"
        state: present
        tower_host: "{{ tower_host }}"
        tower_username: "{{ tower_user }}"
        tower_password: "{{ tower_password }}"
    
    - name: Launch job
      awx.awx.job_launch:
        job_template: "Deploy Web Application"
        extra_vars:
          app_version: "2.1.0"
          environment: "production"
        tower_host: "{{ tower_host }}"
        tower_username: "{{ tower_user }}"
        tower_password: "{{ tower_password }}"
      register: job
    
    - name: Wait for job completion
      awx.awx.job_wait:
        job_id: "{{ job.id }}"
        timeout: 600
        tower_host: "{{ tower_host }}"
        tower_username: "{{ tower_user }}"
        tower_password: "{{ tower_password }}"
```

---

## Feature 20: Testing и Validation (Molecule)

### Установка Molecule
```bash
pip3 install molecule molecule-docker ansible-lint yamllint
```

### Инициализация Molecule
```bash
cd roles/nginx
molecule init scenario -r nginx -d docker
```

### Структура тестов
```
roles/nginx/
└── molecule/
    └── default/
        ├── converge.yml
        ├── molecule.yml
        ├── prepare.yml
        └── verify.yml
```

### molecule.yml
```yaml
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: ubuntu-20
    image: ubuntu:20.04
    pre_build_image: true
    privileged: true
    command: /sbin/init
  - name: centos-8
    image: centos:8
    pre_build_image: true
    privileged: true
    command: /sbin/init
provisioner:
  name: ansible
  lint:
    name: ansible-lint
verifier:
  name: ansible
lint: |
  set -e
  yamllint .
  ansible-lint .
```

### converge.yml (применение роли)
```yaml
---
- name: Converge
  hosts: all
  become: yes
  
  roles:
    - role: nginx
      nginx_port: 8080
```

### verify.yml (проверка результата)
```yaml
---
- name: Verify
  hosts: all
  gather_facts: yes
  
  tasks:
    - name: Check if Nginx is installed
      package:
        name: nginx
        state: present
      check_mode: yes
      register: nginx_installed
      failed_when: nginx_installed is changed
    
    - name: Check if Nginx is running
      service:
        name: nginx
        state: started
      check_mode: yes
      register: nginx_running
      failed_when: nginx_running is changed
    
    - name: Check if Nginx responds
      uri:
        url: "http://localhost:8080"
        status_code: 200
      register: nginx_response
      failed_when: nginx_response.status != 200
    
    - name: Verify Nginx config syntax
      command: nginx -t
      changed_when: false
      register: nginx_syntax
      failed_when: nginx_syntax.rc != 0
```

### Запуск тестов
```bash
# Полный цикл тестирования
molecule test

# Создать контейнеры
molecule create

# Применить роль
molecule converge

# Запустить проверки
molecule verify

# Войти в контейнер для отладки
molecule login -h ubuntu-20

# Уничтожить контейнеры
molecule destroy

# Lint проверка
molecule lint
```

### Ansible-lint конфигурация
```yaml
# .ansible-lint
---
skip_list:
  - '204'  # Lines should be no longer than 160 chars
  - '306'  # Shells that use pipes should set the pipefail option

exclude_paths:
  - .cache/
  - .github/
  - molecule/

use_default_rules: true
```

---

## Бонус: Ansible Configuration File

### ansible.cfg
```ini
[defaults]
# Inventory
inventory = ./inventory/production
host_key_checking = False

# SSH settings
remote_user = ubuntu
private_key_file = ~/.ssh/id_rsa
timeout = 30
forks = 10

# Logging
log_path = ./ansible.log
display_skipped_hosts = False
display_ok_hosts = True

# Roles
roles_path = ./roles

# Callback plugins
stdout_callback = yaml
bin_ansible_callbacks = True

# Retry files
retry_files_enabled = False

# Performance
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 3600

# Privilege escalation
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[privilege_escalation]
become = True
become_method = sudo
become_user = root

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no
pipelining = True
control_path = /tmp/ansible-ssh-%%h-%%p-%%r

[colors]
highlight = white
verbose = blue
warn = bright purple
error = red
debug = dark gray
deprecate = purple
skip = cyan
unreachable = red
ok = green
changed = yellow
```

---

## Практические примеры: Полный CI/CD Pipeline

### 1. Playbook для полного деплоя
```yaml
# playbooks/full_deployment.yml
---
- name: Pre-deployment checks
  hosts: localhost
  connection: local
  
  tasks:
    - name: Verify variables are set
      assert:
        that:
          - app_version is defined
          - environment is defined
        fail_msg: "Required variables not set"
    
    - name: Run smoke tests
      command: "./scripts/smoke_test.sh {{ app_version }}"
      register: smoke_test
      failed_when: smoke_test.rc != 0

- name: Backup current version
  hosts: webservers
  serial: 1
  become: yes
  
  tasks:
    - name: Create backup
      archive:
        path: /opt/myapp/current
        dest: "/opt/backups/myapp-{{ ansible_date_time.epoch }}.tar.gz"
      when: environment == "production"

- name: Deploy application
  hosts: webservers
  serial: "{{ deployment_batch | default('100%') }}"
  become: yes
  
  pre_tasks:
    - name: Remove from load balancer
      include_tasks: tasks/lb_remove.yml
  
  roles:
    - role: app_deploy
      app_version: "{{ app_version }}"
  
  post_tasks:
    - name: Health check
      uri:
        url: "http://localhost:8080/health"
        status_code: 200
      register: health
      until: health.status == 200
      retries: 10
      delay: 5
    
    - name: Add to load balancer
      include_tasks: tasks/lb_add.yml

- name: Post-deployment tasks
  hosts: webservers
  run_once: true
  
  tasks:
    - name: Run database migrations
      command: /opt/myapp/current/migrate.sh
    
    - name: Clear cache
      command: /opt/myapp/current/cache_clear.sh
    
    - name: Send notification
      slack:
        token: "{{ slack_token }}"
        msg: "Deployment of {{ app_version }} to {{ environment }} completed successfully"
        channel: "#deployments"
```

### 2. Мониторинг и алертинг
```yaml
# playbooks/monitoring_setup.yml
---
- name: Setup monitoring stack
  hosts: monitoring
  become: yes
  
  roles:
    - role: prometheus
      prometheus_version: "2.40.0"
      prometheus_targets:
        - job_name: 'nodes'
          static_configs:
            - targets: "{{ groups['all'] | map('extract', hostvars, 'ansible_host') | map('regex_replace', '^(.*), '\\1:9100') | list }}"
        
        - job_name: 'applications'
          static_configs:
            - targets: "{{ groups['webservers'] | map('extract', hostvars, 'ansible_host') | map('regex_replace', '^(.*), '\\1:8080') | list }}"
    
    - role: grafana
      grafana_version: "9.3.0"
      grafana_datasources:
        - name: Prometheus
          type: prometheus
          url: http://localhost:9090
          isDefault: true
    
    - role: alertmanager
      alertmanager_receivers:
        - name: 'slack'
          slack_configs:
            - api_url: "{{ slack_webhook }}"
              channel: '#alerts'
              text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
```

### 3. Disaster Recovery
```yaml
# playbooks/disaster_recovery.yml
---
- name: Disaster recovery procedure
  hosts: webservers
  become: yes
  serial: 1
  
  tasks:
    - name: Stop all services
      service:
        name: "{{ item }}"
        state: stopped
      loop:
        - myapp
        - nginx
    
    - name: Restore from backup
      unarchive:
        src: "{{ backup_location }}/{{ backup_file }}"
        dest: /opt/myapp/
        remote_src: yes
    
    - name: Restore database
      mysql_db:
        name: myapp_db
        state: import
        target: "{{ backup_location }}/database.sql"
      run_once: true
      delegate_to: "{{ groups['databases'][0] }}"
    
    - name: Update configuration
      template:
        src: config.yml.j2
        dest: /opt/myapp/config.yml
    
    - name: Start services
      service:
        name: "{{ item }}"
        state: started
      loop:
        - myapp
        - nginx
    
    - name: Verify restoration
      uri:
        url: "http://localhost:8080/health"
        status_code: 200
      register: health_check
      until: health_check.status == 200
      retries: 5
      delay: 10
```

---

## Troubleshooting (Решение проблем)

### Отладка playbook
```bash
# Verbose режим (разные уровни)
ansible-playbook playbook.yml -v    # Базовый вывод
ansible-playbook playbook.yml -vv   # Больше деталей
ansible-playbook playbook.yml -vvv  # Максимум деталей
ansible-playbook playbook.yml -vvvv # Debug SSH

# Step режим (выполнение по шагам)
ansible-playbook playbook.yml --step

# Start-at-task (начать с определенной задачи)
ansible-playbook playbook.yml --start-at-task="Install Nginx"

# Limit (выполнить только на определенных хостах)
ansible-playbook playbook.yml --limit "web1,web2"
ansible-playbook playbook.yml --limit "webservers:!web3"
```

### Проверка синтаксиса
```bash
# Проверка playbook
ansible-playbook playbook.yml --syntax-check

# Проверка inventory
ansible-inventory -i inventory/production --list
ansible-inventory -i inventory/production --graph

# Lint проверка
ansible-lint playbook.yml
yamllint playbook.yml
```

### Типичные ошибки

#### SSH проблемы
```bash
# Проверка SSH подключения
ansible all -m ping -vvv

# Использование пароля вместо ключа
ansible-playbook playbook.yml --ask-pass

# Использование sudo пароля
ansible-playbook playbook.yml --ask-become-pass
```

#### Проблемы с правами
```yaml
# Убедитесь что become указан
- name: Task requiring root
  apt:
    name: nginx
    state: present
  become: yes
  become_user: root
```

#### Timeout проблемы
```yaml
# Увеличить timeout
- name: Long running task
  command: /usr/local/bin/long_script.sh
  async: 3600  # 1 час
  poll: 10     # Проверять каждые 10 секунд
```

---

## Best Practices (Лучшие практики)

### 1. Организация кода
```
ansible-project/
├── ansible.cfg
├── inventories/
│   ├── production/
│   │   ├── hosts
│   │   ├── group_vars/
│   │   └── host_vars/
│   └── staging/
│       ├── hosts
│       ├── group_vars/
│       └── host_vars/
├── playbooks/
│   ├── site.yml
│   ├── webservers.yml
│   └── databases.yml
├── roles/
│   ├── common/
│   ├── nginx/
│   └── mysql/
├── files/
├── templates/
├── library/         # Кастомные модули
├── filter_plugins/  # Кастомные фильтры
└── vars/
    └── secrets.yml  # Зашифрованные переменные
```

### 2. Идемпотентность
```yaml
# ПЛОХО - не идемпотентно
- name: Add line to file
  command: echo "server=192.168.1.1" >> /etc/config

# ХОРОШО - идемпотентно
- name: Add line to file
  lineinfile:
    path: /etc/config
    line: "server=192.168.1.1"
    state: present
```

### 3. Использование переменных
```yaml
# ПЛОХО - хардкод значений
- name: Install packages
  apt:
    name: nginx
    state: present

# ХОРОШО - использование переменных
- name: Install packages
  apt:
    name: "{{ web_server_package }}"
    state: present
```

### 4. Обработка ошибок
```yaml
- name: Task that might fail
  command: /usr/local/bin/risky_script.sh
  register: result
  failed_when: 
    - result.rc != 0
    - "'expected error' not in result.stderr"
  ignore_errors: yes
  
- name: Handle failure
  debug:
    msg: "Task failed but continuing"
  when: result is failed
```

### 5. Документация
```yaml
---
# playbooks/deploy.yml
# Description: Deploys web application to production servers
# Author: DevOps Team
# Usage: ansible-playbook playbooks/deploy.yml -e "app_version=1.2.3"
# Requirements:
#   - app_version variable must be set
#   - SSH keys configured for all hosts

- name: Deploy application
  hosts: webservers
  become: yes
  
  tasks:
    # Task descriptions should be clear
    - name: Ensure application directory exists with correct permissions
      file:
        path: /opt/myapp
        state: directory
        owner: appuser
        group: appuser
        mode: '0755'
```

---

## Полезные команды

### Inventory
```bash
# Список всех хостов
ansible all --list-hosts

# Список хостов в группе
ansible webservers --list-hosts

# Информация о хосте
ansible web1 -m setup

# Фильтрация фактов
ansible web1 -m setup -a "filter=ansible_distribution*"
```

### Playbook выполнение
```bash
# Dry run
ansible-playbook playbook.yml --check --diff

# Только определенные теги
ansible-playbook playbook.yml --tags "deploy,config"

# Пропустить теги
ansible-playbook playbook.yml --skip-tags "backup"

# Limit на определенные хосты
ansible-playbook playbook.yml --limit "web1,web2"

# Дополнительные переменные
ansible-playbook playbook.yml -e "version=2.0 environment=prod"
```

### Ansible Galaxy
```bash
# Поиск ролей
ansible-galaxy search nginx

# Установка роли
ansible-galaxy install geerlingguy.nginx

# Установка из requirements.yml
ansible-galaxy install -r requirements.yml

# Создание роли
ansible-galaxy init my_new_role

# Список установленных ролей
ansible-galaxy list
```

---

## Интеграция с CI/CD

### GitLab CI пример
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
    - pip install ansible ansible-lint yamllint
  script:
    - ansible-lint playbooks/
    - yamllint .

test:
  stage: test
  image: python:3.9
  before_script:
    - pip install ansible molecule molecule-docker
  script:
    - cd roles/nginx
    - molecule test

deploy_staging:
  stage: deploy
  image: python:3.9
  before_script:
    - pip install ansible
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | ssh-add -
  script:
    - ansible-playbook -i inventories/staging playbooks/deploy.yml -e "app_version=$CI_COMMIT_TAG"
  only:
    - develop

deploy_production:
  stage: deploy
  image: python:3.9
  before_script:
    - pip install ansible
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | ssh-add -
  script:
    - ansible-playbook -i inventories/production playbooks/deploy.yml -e "app_version=$CI_COMMIT_TAG"
  only:
    - tags
  when: manual
```

### Jenkins Pipeline
```groovy
pipeline {
    agent any
    
    stages {
        stage('Lint') {
            steps {
                sh 'ansible-lint playbooks/'
            }
        }
        
        stage('Test') {
            steps {
                sh 'molecule test'
            }
        }
        
        stage('Deploy to Staging') {
            steps {
                ansiblePlaybook(
                    playbook: 'playbooks/deploy.yml',
                    inventory: 'inventories/staging/hosts',
                    extras: '-e "app_version=${BUILD_NUMBER}"'
                )
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input 'Deploy to production?'
                ansiblePlaybook(
                    playbook: 'playbooks/deploy.yml',
                    inventory: 'inventories/production/hosts',
                    extras: '-e "app_version=${BUILD_NUMBER}"'
                )
            }
        }
    }
}
```

---

## Дополнительные ресурсы

- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [Molecule Documentation](https://molecule.readthedocs.io/)
- [AWX Project](https://github.com/ansible/awx)

---

## Заключение

Этот гайд покрывает 20 основных фич Ansible, которые DevOps инженер использует ежедневно:

1. ✅ Inventory Management
2. ✅ Ad-Hoc Commands
3. ✅ Playbooks
4. ✅ Variables
5. ✅ Roles
6. ✅ Handlers
7. ✅ Templates
8. ✅ Conditionals
9. ✅ Loops
10. ✅ Ansible Vault
11. ✅ Tags
12. ✅ Blocks
13. ✅ Facts
14. ✅ Register & Set_fact
15. ✅ Delegation
16. ✅ Docker Integration
17. ✅ Kubernetes Integration
18. ✅ Rolling Updates
19. ✅ Tower/AWX
20. ✅ Testing with Molecule

**Следующие шаги:**
1. Практикуйте каждую фичу на тестовом окружении
2. Создайте свои роли для типичных задач
3. Настройте CI/CD с Ansible
4. Изучите Ansible Tower/AWX для больших проектов