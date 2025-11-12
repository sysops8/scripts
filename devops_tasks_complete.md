# Сборник практических задач DevOps
## Для отработки на Proxmox/Linux (Fedora/Ubuntu)

---

## 🟢 JUNIOR DEVOPS ENGINEER

### Задача 1: Автоматический бэкап базы данных PostgreSQL
**Цель:** Создать скрипт для резервного копирования БД с ротацией

**Требования:**
- Создать дамп PostgreSQL базы
- Исключить таблицу `logs` из бэкапа
- Добавить timestamp к имени файла
- Сжать в .tar.gz
- Хранить только последние 7 дней бэкапов
- Настроить через cron на запуск в 2:00

**Решение:**

```bash
#!/bin/bash
# backup_postgres.sh

# Конфигурация
DB_NAME="myapp"
DB_USER="postgres"
BACKUP_DIR="/var/backups/postgresql"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${DB_NAME}_${TIMESTAMP}.sql"
RETENTION_DAYS=7

# Создаём директорию если не существует
mkdir -p $BACKUP_DIR

# Создаём дамп исключая таблицу logs
pg_dump -U $DB_USER \
  --exclude-table=logs \
  --format=plain \
  $DB_NAME > $BACKUP_DIR/$BACKUP_FILE

# Сжимаем
tar -czf $BACKUP_DIR/$BACKUP_FILE.tar.gz -C $BACKUP_DIR $BACKUP_FILE
rm $BACKUP_DIR/$BACKUP_FILE

# Удаляем старые бэкапы
find $BACKUP_DIR -name "backup_*.tar.gz" -mtime +$RETENTION_DAYS -delete

# Логирование результата
if [ $? -eq 0 ]; then
    echo "[$(date)] Backup successful: $BACKUP_FILE.tar.gz" >> /var/log/db_backup.log
else
    echo "[$(date)] Backup FAILED" >> /var/log/db_backup.log
    exit 1
fi
```

**Установка PostgreSQL на Ubuntu/Fedora:**
```bash
# Ubuntu
sudo apt update && sudo apt install postgresql postgresql-contrib -y

# Fedora
sudo dnf install postgresql postgresql-server -y
sudo postgresql-setup --initdb
sudo systemctl enable --now postgresql

# Создание тестовой базы
sudo -u postgres psql -c "CREATE DATABASE myapp;"
sudo -u postgres psql -d myapp -c "CREATE TABLE logs (id serial, message text);"
sudo -u postgres psql -d myapp -c "CREATE TABLE users (id serial, name text);"
```

**Настройка cron:**
```bash
# Добавить права на выполнение
chmod +x /path/to/backup_postgres.sh

# Добавить в crontab
crontab -e
# Добавить строку:
0 2 * * * /path/to/backup_postgres.sh
```

---

### Задача 2: Мониторинг CPU и автоматический перезапуск сервиса
**Цель:** Мониторить использование CPU процессом и перезапускать при превышении порога

**Требования:**
- Проверять использование CPU каждую минуту
- Если процесс использует >80% CPU последние 5 минут
- Автоматически перезапустить сервис
- Отправить уведомление в лог

**Решение:**

```bash
#!/bin/bash
# monitor_cpu.sh

SERVICE_NAME="nginx"  # Замените на ваш сервис
CPU_THRESHOLD=80
CHECK_COUNT=5
COUNTER_FILE="/tmp/cpu_high_counter_${SERVICE_NAME}"
LOG_FILE="/var/log/cpu_monitor.log"

# Получаем использование CPU для процесса
CPU_USAGE=$(ps aux | grep -v grep | grep $SERVICE_NAME | awk '{sum+=$3} END {print int(sum)}')

# Если процесс не найден
if [ -z "$CPU_USAGE" ]; then
    echo "[$(date)] Service $SERVICE_NAME not running" >> $LOG_FILE
    exit 1
fi

# Проверяем превышение порога
if [ $CPU_USAGE -gt $CPU_THRESHOLD ]; then
    # Увеличиваем счётчик
    if [ -f $COUNTER_FILE ]; then
        COUNT=$(cat $COUNTER_FILE)
        COUNT=$((COUNT + 1))
    else
        COUNT=1
    fi
    echo $COUNT > $COUNTER_FILE
    
    echo "[$(date)] High CPU: ${CPU_USAGE}% (count: ${COUNT}/${CHECK_COUNT})" >> $LOG_FILE
    
    # Если превышение 5 раз подряд - перезапускаем
    if [ $COUNT -ge $CHECK_COUNT ]; then
        echo "[$(date)] RESTARTING $SERVICE_NAME due to high CPU" >> $LOG_FILE
        systemctl restart $SERVICE_NAME
        rm -f $COUNTER_FILE
    fi
else
    # Сбрасываем счётчик если CPU в норме
    rm -f $COUNTER_FILE
fi
```

**Настройка:**
```bash
chmod +x /path/to/monitor_cpu.sh

# Добавить в crontab для проверки каждую минуту
* * * * * /path/to/monitor_cpu.sh
```

---

### Задача 3: Настройка Nginx reverse proxy с SSL
**Цель:** Настроить Nginx как обратный прокси для приложения с HTTPS

**Решение:**

```bash
# Установка Nginx
# Ubuntu
sudo apt install nginx certbot python3-certbot-nginx -y

# Fedora
sudo dnf install nginx certbot python3-certbot-nginx -y

# Конфигурация Nginx
sudo cat > /etc/nginx/sites-available/myapp << 'EOF'
upstream backend {
    server 127.0.0.1:3000;
}

server {
    listen 80;
    server_name myapp.local;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Websocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
    
    # Health check
    location /nginx-health {
        access_log off;
        return 200 "healthy\n";
    }
}
EOF

# Активируем конфиг
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

# Для самоподписанного сертификата (тестирование)
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/nginx-selfsigned.key \
    -out /etc/ssl/certs/nginx-selfsigned.crt \
    -subj "/C=US/ST=State/L=City/O=Org/CN=myapp.local"
```

---

### Задача 4: Docker Compose для локальной разработки
**Цель:** Создать окружение для разработки с несколькими сервисами

**Решение:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://postgres:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    volumes:
      - ./src:/app/src
      - ./node_modules:/app/node_modules
    depends_on:
      - db
      - redis
    command: npm run dev

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  adminer:
    image: adminer
    ports:
      - "8080:8080"
    depends_on:
      - db

volumes:
  postgres_data:
  redis_data:
```

**Использование:**
```bash
# Запуск всех сервисов
docker-compose up -d

# Просмотр логов
docker-compose logs -f app

# Остановка
docker-compose down

# Пересборка после изменений
docker-compose up -d --build
```

---

### Задача 5: Базовый CI/CD с Git hooks
**Цель:** Автоматический деплой при push в репозиторий

**Решение:**

```bash
# На сервере создаём bare репозиторий
sudo mkdir -p /opt/git/myapp.git
cd /opt/git/myapp.git
sudo git init --bare

# Создаём post-receive hook
cat > /opt/git/myapp.git/hooks/post-receive << 'EOF'
#!/bin/bash

DEPLOY_DIR="/var/www/myapp"
BACKUP_DIR="/var/www/myapp_backup"
LOG_FILE="/var/log/deploy.log"

echo "[$(date)] Starting deployment..." >> $LOG_FILE

# Создаём backup текущей версии
if [ -d $DEPLOY_DIR ]; then
    rm -rf $BACKUP_DIR
    cp -r $DEPLOY_DIR $BACKUP_DIR
fi

# Клонируем новую версию
rm -rf $DEPLOY_DIR
git clone /opt/git/myapp.git $DEPLOY_DIR

cd $DEPLOY_DIR

# Устанавливаем зависимости
npm install >> $LOG_FILE 2>&1

# Запускаем приложение
pm2 restart myapp || pm2 start app.js --name myapp

echo "[$(date)] Deployment completed" >> $LOG_FILE
EOF

chmod +x /opt/git/myapp.git/hooks/post-receive

# На локальной машине
git remote add production ssh://user@server:/opt/git/myapp.git
git push production main
```

---

## 🟡 MIDDLE DEVOPS ENGINEER

### Задача 6: Kubernetes кластер на Proxmox VM (K3s)
**Цель:** Развернуть легковесный Kubernetes кластер

**Решение:**

```bash
#!/bin/bash
# install_k3s_cluster.sh

# На мастер-ноде
curl -sfL https://get.k3s.io | sh -s - server \
  --write-kubeconfig-mode 644 \
  --disable traefik

# Получить токен для worker нод
sudo cat /var/lib/rancher/k3s/server/node-token

# На worker нодах
K3S_URL=https://master-ip:6443
K3S_TOKEN=your-node-token

curl -sfL https://get.k3s.io | K3S_URL=$K3S_URL K3S_TOKEN=$K3S_TOKEN sh -

# Проверка кластера (на мастере)
kubectl get nodes
kubectl get pods -A
```

**Деплой тестового приложения:**
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-demo
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
        image: nginx:alpine
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

---

### Задача 7: Централизованный мониторинг (Prometheus + Grafana)
**Цель:** Настроить стек мониторинга для инфраструктуры

**Решение через Docker Compose:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    ports:
      - "9090:9090"
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    ports:
      - "3000:3000"
    restart: unless-stopped
    depends_on:
      - prometheus

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    restart: unless-stopped

  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    ports:
      - "9093:9093"
    restart: unless-stopped

volumes:
  prometheus_data:
  grafana_data:
```

**prometheus.yml:**
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - "alerts.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```

---

### Задача 8: Ansible для управления конфигурацией
**Цель:** Автоматизировать настройку множества серверов

**Структура проекта:**
```
ansible/
├── ansible.cfg
├── inventory/
│   └── hosts.yml
├── roles/
│   ├── common/
│   ├── webserver/
│   └── database/
└── playbooks/
    └── site.yml
```

**inventory/hosts.yml:**
```yaml
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.10
        web2:
          ansible_host: 192.168.1.11
    
    databases:
      hosts:
        db1:
          ansible_host: 192.168.1.20
```

**playbooks/site.yml:**
```yaml
---
- name: Configure all servers
  hosts: all
  become: yes
  roles:
    - common

- name: Configure web servers
  hosts: webservers
  become: yes
  roles:
    - webserver

- name: Configure database servers
  hosts: databases
  become: yes
  roles:
    - database
```

---

### Задача 9: Terraform для Proxmox
**Цель:** Автоматизировать создание VM через Infrastructure as Code

**main.tf:**
```hcl
terraform {
  required_providers {
    proxmox = {
      source  = "telmate/proxmox"
      version = "2.9.14"
    }
  }
}

provider "proxmox" {
  pm_api_url      = var.proxmox_api_url
  pm_api_token_id = var.proxmox_api_token_id
  pm_api_token_secret = var.proxmox_api_token_secret
  pm_tls_insecure = true
}

resource "proxmox_vm_qemu" "devops_vm" {
  count       = var.vm_count
  name        = "${var.vm_name}-${count.index + 1}"
  target_node = var.proxmox_node
  clone       = var.template_name
  
  cores   = var.cpu_cores
  sockets = 1
  memory  = var.memory
  
  disk {
    size    = var.disk_size
    type    = "scsi"
    storage = var.storage
  }
  
  network {
    model  = "virtio"
    bridge = "vmbr0"
  }
  
  ipconfig0 = "ip=${var.ip_address_base}.${count.index + 10}/24,gw=${var.gateway}"
  sshkeys = file(var.ssh_public_key_file)
}
```

---

### Задача 10: Логирование с ELK Stack
**Цель:** Централизованный сбор и анализ логов

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5044:5044"
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

  filebeat:
    image: docker.elastic.co/beats/filebeat:8.11.0
    user: root
    volumes:
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml
      - /var/log:/var/log:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
    depends_on:
      - elasticsearch
      - logstash

volumes:
  elasticsearch_data:
```

---

## 🔴 SENIOR DEVOPS ENGINEER

### Задача 11: GitOps с ArgoCD
**Цель:** Внедрить GitOps практики для управления Kubernetes

**Установка ArgoCD:**
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Получить начальный пароль
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port-forward для доступа
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

**ArgoCD Application манифест:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
  namespace: argocd
spec:
  project: default
  
  source:
    repoURL: https://github.com/yourorg/gitops-repo.git
    targetRevision: main
    path: apps/production/myapp
  
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

---

### Задача 12: High Availability PostgreSQL с Patroni
**Цель:** Развернуть отказоустойчивый кластер БД

**Установка на каждую ноду:**
```bash
#!/bin/bash
# install_patroni.sh

# Установка зависимостей
sudo apt update
sudo apt install -y postgresql postgresql-contrib etcd python3-pip python3-psycopg2

# Установка Patroni
sudo pip3 install patroni[etcd]

# Конфигурация Patroni
sudo cat > /etc/patroni.yml << EOF
scope: postgres-cluster
namespace: /db/
name: $(hostname)

restapi:
  listen: 0.0.0.0:8008
  connect_address: $(hostname -I | awk '{print $1}'):8008

etcd:
  hosts: 192.168.1.10:2379,192.168.1.11:2379,192.168.1.12:2379

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
      parameters:
        max_connections: 100
        shared_buffers: 256MB

  initdb:
    - encoding: UTF8
    - data-checksums

  pg_hba:
    - host replication replicator 0.0.0.0/0 md5
    - host all all 0.0.0.0/0 md5

postgresql:
  listen: 0.0.0.0:5432
  connect_address: $(hostname -I | awk '{print $1}'):5432
  data_dir: /var/lib/postgresql/14/main
  bin_dir: /usr/lib/postgresql/14/bin
  authentication:
    replication:
      username: replicator
      password: rep-pass
    superuser:
      username: postgres
      password: postgres-pass
EOF

# Systemd service
sudo cat > /etc/systemd/system/patroni.service << EOF
[Unit]
Description=Patroni (PostgreSQL HA)
After=syslog.target network.target

[Service]
Type=simple
User=postgres
Group=postgres
ExecStart=/usr/local/bin/patroni /etc/patroni.yml
KillMode=process
TimeoutSec=30
Restart=no

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable patroni
sudo systemctl start patroni
```

---

### Задача 13: Disaster Recovery с автоматизацией
**Цель:** Создать систему резервного копирования и восстановления всей инфраструктуры

**disaster_recovery_backup.sh:**
```bash
#!/bin/bash
# disaster_recovery_backup.sh

set -euo pipefail

BACKUP_ROOT="/mnt/backup"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="$BACKUP_ROOT/$TIMESTAMP"
LOG_FILE="/var/log/dr_backup.log"
RETENTION_DAYS=30

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a $LOG_FILE
}

# Создание структуры директорий
mkdir -p $BACKUP_DIR/{databases,configs,containers,vms,apps}

log "=== Starting Disaster Recovery Backup ==="

# 1. Backup всех баз данных
backup_databases() {
    # PostgreSQL
    if systemctl is-active --quiet postgresql; then
        pg_dumpall -U postgres | gzip > $BACKUP_DIR/databases/postgresql_all.sql.gz
        log "PostgreSQL backup completed"
    fi
    
    # MySQL/MariaDB
    if systemctl is-active --quiet mysql; then
        mysqldump --all-databases --single-transaction | gzip > $BACKUP_DIR/databases/mysql_all.sql.gz
        log "MySQL backup completed"
    fi
}

# 2. Backup конфигураций
backup_configs() {
    tar -czf $BACKUP_DIR/configs/etc.tar.gz \
        /etc/nginx \
        /etc/systemd/system \
        /etc/ssh \
        2>/dev/null || true
    
    log "Configuration backup completed"
}

# 3. Backup Docker контейнеров и volumes
backup_docker() {
    if command -v docker &> /dev/null; then
        docker ps -a --format "{{.Names}}" > $BACKUP_DIR/containers/container_list.txt
        
        # Backup volumes
        docker volume ls -q | while read volume; do
            docker run --rm -v $volume:/data -v $BACKUP_DIR/containers:/backup \
                alpine tar czf /backup/volume_$volume.tar.gz -C /data .
        done
        
        log "Docker backup completed"
    fi
}

# Выполнение всех бэкапов
backup_databases
backup_configs
backup_docker

# Создание общего архива
cd $BACKUP_ROOT
tar -czf backup_$TIMESTAMP.tar.gz $TIMESTAMP

# Очистка старых бэкапов
find $BACKUP_ROOT -name "backup_*.tar.gz" -mtime +$RETENTION_DAYS -delete

# Расчет контрольной суммы
sha256sum backup_$TIMESTAMP.tar.gz > backup_$TIMESTAMP.tar.gz.sha256

log "=== Backup completed successfully ==="
log "Backup location: $BACKUP_ROOT/backup_$TIMESTAMP.tar.gz"
```

---

### Задача 14: Security Hardening и Compliance
**Цель:** Автоматизация проверки безопасности и соответствия стандартам

**security_audit.sh:**
```bash
#!/bin/bash
# security_audit.sh

set -euo pipefail

REPORT_DIR="/var/log/security_audit"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
REPORT_FILE="$REPORT_DIR/audit_$TIMESTAMP.txt"

mkdir -p $REPORT_DIR

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a $REPORT_FILE
}

log "=== Security Audit Report ==="
log "Hostname: $(hostname)"
log "OS: $(lsb_release -d | cut -f2)"

# 1. Проверка обновлений системы
log "\n--- System Updates ---"
if apt list --upgradable 2>/dev/null | grep -q upgradable; then
    UPDATES=$(apt list --upgradable 2>/dev/null | grep upgradable | wc -l)
    log "⚠️  $UPDATES security updates available"
else
    log "✅ System is up to date"
fi

# 2. Проверка открытых портов
log "\n--- Open Ports ---"
ss -tuln | grep LISTEN >> $REPORT_FILE

# 3. Проверка SSH конфигурации
log "\n--- SSH Configuration ---"
if grep -q "^PermitRootLogin yes" /etc/ssh/sshd_config 2>/dev/null; then
    log "❌ Root login is enabled"
else
    log "✅ Root login is disabled"
fi

if grep -q "^PasswordAuthentication yes" /etc/ssh/sshd_config 2>/dev/null; then
    log "⚠️  Password authentication is enabled"
else
    log "✅ Password authentication is disabled"
fi

# 4. Проверка firewall
log "\n--- Firewall ---"
if systemctl is-active --quiet ufw; then
    log "✅ Firewall is active"
    ufw status >> $REPORT_FILE
else
    log "❌ No firewall is active"
fi

# 5. Проверка fail2ban
log "\n--- Fail2ban ---"
if systemctl is-active --quiet fail2ban; then
    log "✅ Fail2ban is active"
else
    log "⚠️  Fail2ban is not running"
fi

# 6. Проверка прав на критичные файлы
log "\n--- File Permissions ---"
for file in /etc/passwd /etc/shadow; do
    PERMS=$(stat -c %a $file)
    log "$file: $PERMS"
done

# 7. Проверка SUID/SGID файлов
log "\n--- SUID/SGID Files ---"
find / -type f \( -perm -4000 -o -perm -2000 \) 2>/dev/null | head -20 >> $REPORT_FILE

# 8. Проверка неудачных попыток входа
log "\n--- Failed Login Attempts ---"
grep "Failed password" /var/log/auth.log 2>/dev/null | tail -10 >> $REPORT_FILE || log "No recent failed attempts"

# 9. Проверка Docker безопасности
if command -v docker &> /dev/null; then
    log "\n--- Docker Security ---"
    PRIV_CONTAINERS=$(docker ps --format "{{.Names}}" | xargs -I {} docker inspect {} --format '{{.Name}}: Privileged={{.HostConfig.Privileged}}' | grep "Privileged=true" || echo "None")
    log "Privileged containers: $PRIV_CONTAINERS"
fi

# 10. Проверка дискового пространства
log "\n--- Disk Space ---"
df -h | awk '$5+0 > 80 {print "⚠️  "$0}' >> $REPORT_FILE

log "\n=== Audit Completed ==="
log "Full report: $REPORT_FILE"
```

---

### Задача 15: Blue-Green Deployment
**Цель:** Реализовать стратегию деплоя без простоя сервиса

**blue_green_deploy.sh:**
```bash
#!/bin/bash
# blue_green_deploy.sh

set -euo pipefail

APP_NAME="myapp"
BLUE_PORT=3000
GREEN_PORT=3001
HEALTH_CHECK_URL="http://localhost"
NGINX_UPSTREAM="/etc/nginx/conf.d/${APP_NAME}_upstream.conf"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

# Определить текущую активную среду
get_active_env() {
    if grep -q "server 127.0.0.1:$BLUE_PORT" $NGINX_UPSTREAM && \
       ! grep -q "^[[:space:]]*#.*server 127.0.0.1:$BLUE_PORT" $NGINX_UPSTREAM; then
        echo "blue"
    else
        echo "green"
    fi
}

# Проверка здоровья приложения
health_check() {
    local port=$1
    local max_attempts=30
    local attempt=0
    
    log "Checking health on port $port..."
    
    while [ $attempt -lt $max_attempts ]; do
        if curl -sf "${HEALTH_CHECK_URL}:${port}/health" > /dev/null; then
            log "Health check passed on port $port"
            return 0
        fi
        
        attempt=$((attempt + 1))
        log "Health check attempt $attempt/$max_attempts failed, retrying..."
        sleep 2
    done
    
    log "ERROR: Health check failed on port $port"
    return 1
}

# Деплой приложения
deploy_app() {
    local env=$1
    local port=$2
    
    log "Deploying to $env environment (port $port)..."
    
    # Остановить старую версию
    pm2 delete ${APP_NAME}-${env} 2>/dev/null || true
    
    # Обновить код
    cd /opt/${APP_NAME}
    git pull origin main
    npm install --production
    
    # Запустить новую версию
    PORT=$port pm2 start app.js --name ${APP_NAME}-${env}
    
    sleep 5
    health_check $port
}

# Переключить трафик
switch_traffic() {
    local target_env=$1
    local target_port=$2
    
    log "Switching traffic to $target_env environment..."
    
    cat > $NGINX_UPSTREAM << EOF
upstream ${APP_NAME}_backend {
    server 127.0.0.1:${target_port} max_fails=3 fail_timeout=30s;
}
EOF
    
    nginx -t && nginx -s reload
    log "Traffic switched to $target_env"
}

# Откат
rollback() {
    local previous_env=$1
    local previous_port=$2
    
    log "Rolling back to $previous_env..."
    
    if health_check $previous_port; then
        switch_traffic $previous_env $previous_port
        log "Rollback completed"
    else
        log "ERROR: Cannot rollback, previous version unhealthy"
        exit 1
    fi
}

# Основная логика
main() {
    local active_env=$(get_active_env)
    local target_env target_port previous_port
    
    if [ "$active_env" = "blue" ]; then
        target_env="green"
        target_port=$GREEN_PORT
        previous_port=$BLUE_PORT
    else
        target_env="blue"
        target_port=$BLUE_PORT
        previous_port=$GREEN_PORT
    fi
    
    log "=== Blue-Green Deployment Started ==="
    log "Current active: $active_env"
    log "Deploying to: $target_env"
    
    # Деплой в неактивную среду
    if ! deploy_app $target_env $target_port; then
        log "Deployment failed"
        exit 1
    fi
    
    # Переключить трафик
    switch_traffic $target_env $target_port
    
    # Проверка после переключения
    sleep 10
    if ! health_check $target_port; then
        log "New version unhealthy, rolling back..."
        rollback $active_env $previous_port
        exit 1
    fi
    
    # Остановить старую версию
    pm2 delete ${APP_NAME}-${active_env} 2>/dev/null || true
    
    log "=== Deployment Completed Successfully ==="
}

main "$@"
```

---

### Задача 16: Canary Deployment в Kubernetes
**Цель:** Постепенное переключение трафика на новую версию

**canary-deployment.yml:**
```yaml
# Стабильная версия
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-stable
spec:
  replicas: 9
  selector:
    matchLabels:
      app: myapp
      version: stable
  template:
    metadata:
      labels:
        app: myapp
        version: stable
    spec:
      containers:
      - name: myapp
        image: myapp:v1.0
        ports:
        - containerPort: 8080
---
# Canary версия (10% трафика)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      version: canary
  template:
    metadata:
      labels:
        app: myapp
        version: canary
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0
        ports:
        - containerPort: 8080
---
# Общий Service
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

**canary_promote.sh:**
```bash
#!/bin/bash
# canary_promote.sh - Постепенное увеличение canary трафика

APP="myapp"
NAMESPACE="default"

# Этапы canary: 10% -> 25% -> 50% -> 100%
STAGES=(1 3 5 9)
STABLE_STAGES=(9 7 5 1)

for i in "${!STAGES[@]}"; do
    CANARY_REPLICAS=${STAGES[$i]}
    STABLE_REPLICAS=${STABLE_STAGES[$i]}
    
    echo "Stage $((i+1)): Canary $CANARY_REPLICAS replicas, Stable $STABLE_REPLICAS replicas"
    
    # Масштабировать canary
    kubectl scale deployment ${APP}-canary -n $NAMESPACE --replicas=$CANARY_REPLICAS
    
    # Масштабировать stable
    kubectl scale deployment ${APP}-stable -n $NAMESPACE --replicas=$STABLE_REPLICAS
    
    # Ждать готовности
    kubectl rollout status deployment/${APP}-canary -n $NAMESPACE
    
    # Проверить метрики
    echo "Checking metrics for 5 minutes..."
    sleep 300
    
    # Проверить error rate (пример с Prometheus)
    ERROR_RATE=$(curl -s "http://prometheus:9090/api/v1/query?query=rate(http_requests_total{job='myapp',status=~'5..'}[5m])" | jq -r '.data.result[0].value[1]')
    
    if (( $(echo "$ERROR_RATE > 0.01" | bc -l) )); then
        echo "Error rate too high ($ERROR_RATE), rolling back..."
        kubectl scale deployment ${APP}-canary -n $NAMESPACE --replicas=0
        kubectl scale deployment ${APP}-stable -n $NAMESPACE --replicas=10
        exit 1
    fi
done

# Полное переключение на новую версию
echo "Canary successful, promoting to stable..."
kubectl set image deployment/${APP}-stable myapp=myapp:v2.0
kubectl scale deployment ${APP}-canary -n $NAMESPACE --replicas=0
```

---

### Задача 17: Service Mesh с Istio
**Цель:** Управление микросервисами с продвинутыми возможностями

**Установка Istio:**
```bash
# Скачать Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH=$PWD/bin:$PATH

# Установить
istioctl install --set profile=demo -y

# Включить автоматический sidecar injection
kubectl label namespace default istio-injection=enabled
```

**Виртуальный сервис с traffic splitting:**
```yaml
# virtualservice.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp
  http:
  - match:
    - headers:
        user-type:
          exact: beta
    route:
    - destination:
        host: myapp
        subset: v2
  - route:
    - destination:
        host: myapp
        subset: v1
      weight: 90
    - destination:
        host: myapp
        subset: v2
      weight: 10
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp
spec:
  host: myapp
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

**Circuit Breaker конфигурация:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp-circuit-breaker
spec:
  host: myapp
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        http2MaxRequests: 100
        maxRequestsPerConnection: 2
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
      minHealthPercent: 40
```

---

### Задача 18: Secrets Management с Vault
**Цель:** Централизованное управление секретами

**Установка Vault:**
```bash
# Docker compose для Vault
cat > vault-compose.yml << 'EOF'
version: '3.8'

services:
  vault:
    image: vault:latest
    container_name: vault
    ports:
      - "8200:8200"
    environment:
      VAULT_DEV_ROOT_TOKEN_ID: myroot
      VAULT_DEV_LISTEN_ADDRESS: 0.0.0.0:8200
    cap_add:
      - IPC_LOCK
    volumes:
      - vault_data:/vault/data
      - vault_logs:/vault/logs

volumes:
  vault_data:
  vault_logs:
EOF

docker-compose -f vault-compose.yml up -d
```

**Интеграция с Kubernetes:**
```bash
# Включить Kubernetes auth
export VAULT_ADDR='http://localhost:8200'
export VAULT_TOKEN='myroot'

vault auth enable kubernetes

vault write auth/kubernetes/config \
    kubernetes_host="https://kubernetes.default.svc:443"

# Создать политику
vault policy write myapp - <<EOF
path "secret/data/myapp/*" {
  capabilities = ["read"]
}
EOF

# Связать с service account
vault write auth/kubernetes/role/myapp \
    bound_service_account_names=myapp \
    bound_service_account_namespaces=default \
    policies=myapp \
    ttl=24h
```

**Использование в Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  serviceAccountName: myapp
  containers:
  - name: myapp
    image: myapp:latest
    env:
    - name: VAULT_ADDR
      value: "http://vault:8200"
  initContainers:
  - name: vault-agent
    image: vault:latest
    volumeMounts:
    - name: shared-data
      mountPath: /vault/secrets
    command:
    - sh
    - -c
    - |
      vault kv get -format=json secret/myapp/config > /vault/secrets/config.json
  volumes:
  - name: shared-data
    emptyDir: {}
```

---

### Задача 19: Multi-Cloud Infrastructure
**Цель:** Управление ресурсами в нескольких облаках

**Terraform multi-provider:**
```hcl
# providers.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
    proxmox = {
      source  = "telmate/proxmox"
      version = "~> 2.9"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

provider "azurerm" {
  features {}
}

provider "google" {
  project = var.gcp_project
  region  = var.gcp_region
}

provider "proxmox" {
  pm_api_url = var.proxmox_api_url
}
```

**Модуль для создания VM в разных облаках:**
```hcl
# modules/compute/main.tf
variable "provider" {
  type = string
}

variable "instance_config" {
  type = object({
    name         = string
    instance_type = string
    disk_size    = number
  })
}

# AWS EC2
resource "aws_instance" "vm" {
  count         = var.provider == "aws" ? 1 : 0
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_config.instance_type
  
  tags = {
    Name = var.instance_config.name
  }
}

# Azure VM
resource "azurerm_linux_virtual_machine" "vm" {
  count               = var.provider == "azure" ? 1 : 0
  name                = var.instance_config.name
  resource_group_name = azurerm_resource_group.rg[0].name
  location            = azurerm_resource_group.rg[0].location
  size                = var.instance_config.instance_type
  
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }
}

# Proxmox VM
resource "proxmox_vm_qemu" "vm" {
  count       = var.provider == "proxmox" ? 1 : 0
  name        = var.instance_config.name
  target_node = var.proxmox_node
  
  cores  = 2
  memory = 2048
}

output "ip_address" {
  value = (
    var.provider == "aws" ? aws_instance.vm[0].public_ip :
    var.provider == "azure" ? azurerm_linux_virtual_machine.vm[0].public_ip_address :
    var.provider == "proxmox" ? proxmox_vm_qemu.vm[0].default_ipv4_address :
    null
  )
}
```

---

### Задача 20: Chaos Engineering
**Цель:** Тестирование отказоустойчивости системы

**Установка Chaos Mesh:**
```bash
# Установить Chaos Mesh в Kubernetes
curl -sSL https://mirrors.chaos-mesh.org/latest/install.sh | bash

# Или через Helm
helm repo add chaos-mesh https://charts.chaos-mesh.org
helm install chaos-mesh chaos-mesh/chaos-mesh -n chaos-testing --create-namespace
```

**Эксперименты с хаосом:**
```yaml
# pod-kill-experiment.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: pod-kill-experiment
  namespace: chaos-testing
spec:
  action: pod-kill
  mode: one
  selector:
    namespaces:
      - default
    labelSelectors:
      app: myapp
  scheduler:
    cron: '@every 10m'
---
# network-delay-experiment.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: network-delay
  namespace: chaos-testing
spec:
  action: delay
  mode: all
  selector:
    namespaces:
      - default
    labelSelectors:
      app: myapp
  delay:
    latency: "100ms"
    correlation: "100"
    jitter: "50ms"
  duration: "5m"
---
# cpu-stress-experiment.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: StressChaos
metadata:
  name: cpu-stress
  namespace: chaos-testing
spec:
  mode: one
  selector:
    namespaces:
      - default
    labelSelectors:
      app: myapp
  stressors:
    cpu:
      workers: 2
      load: 80
  duration: "3m"
```

**Автоматизированный chaos test runner:**
```bash
#!/bin/bash
# chaos_test_runner.sh

set -euo pipefail

CHAOS_NAMESPACE="chaos-testing"
APP_NAMESPACE="default"
METRICS_URL="http://prometheus:9090"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

# Проверить базовые метрики перед тестом
check_baseline() {
    log "Collecting baseline metrics..."
    
    BASELINE_ERROR_RATE=$(curl -s "$METRICS_URL/api/v1/query?query=rate(http_requests_total{status=~'5..'}[5m])" | jq -r '.data.result[0].value[1] // 0')
    BASELINE_LATENCY=$(curl -s "$METRICS_URL/api/v1/query?query=histogram_quantile(0.95,rate(http_request_duration_seconds_bucket[5m]))" | jq -r '.data.result[0].value[1] // 0')
    
    log "Baseline error rate: $BASELINE_ERROR_RATE"
    log "Baseline p95 latency: $BASELINE_LATENCY"
}

# Запустить chaos эксперимент
run_chaos() {
    local experiment=$1
    
    log "Starting chaos experiment: $experiment"
    kubectl apply -f experiments/$experiment.yaml
    
    # Дождаться завершения
    sleep 360
    
    log "Chaos experiment completed"
}

# Проверить метрики после теста
check_impact() {
    log "Collecting post-chaos metrics..."
    
    ERROR_RATE=$(curl -s "$METRICS_URL/api/v1/query?query=rate(http_requests_total{status=~'5..'}[5m])" | jq -r '.data.result[0].value[1] // 0')
    LATENCY=$(curl -s "$METRICS_URL/api/v1/query?query=histogram_quantile(0.95,rate(http_request_duration_seconds_bucket[5m]))" | jq -r '.data.result[0].value[1] // 0')
    
    log "Post-chaos error rate: $ERROR_RATE"
    log "Post-chaos p95 latency: $LATENCY"
    
    # Проверить SLO
    ERROR_THRESHOLD=0.01
    LATENCY_THRESHOLD=1.0
    
    if (( $(echo "$ERROR_RATE > $ERROR_THRESHOLD" | bc -l) )); then
        log "❌ ERROR: Error rate exceeded threshold"
        return 1
    fi
    
    if (( $(echo "$LATENCY > $LATENCY_THRESHOLD" | bc -l) )); then
        log "❌ ERROR: Latency exceeded threshold"
        return 1
    fi
    
    log "✅ System passed chaos test"
    return 0
}

# Основной цикл тестирования
EXPERIMENTS=("pod-kill" "network-delay" "cpu-stress")

for exp in "${EXPERIMENTS[@]}"; do
    log "=== Testing: $exp ==="
    
    check_baseline
    run_chaos $exp
    
    if ! check_impact; then
        log "System failed chaos test: $exp"
        kubectl delete -f experiments/$exp.yaml
        exit 1
    fi
    
    kubectl delete -f experiments/$exp.yaml
    
    # Пауза между тестами
    log "Cooling down for 5 minutes..."
    sleep 300
done

log "=== All chaos tests passed ==="
```

---

## 📚 ДОПОЛНИТЕЛЬНЫЕ ПРАКТИКИ

### Задача 21: Observability Stack (Полный стек)
**Цель:** Комплексный мониторинг, логирование и трейсинг

**docker-compose.yml для полного стека:**
```yaml
version: '3.8'

services:
  # Метрики
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:latest
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources
    ports:
      - "3000:3000"

  # Логирование
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5601:5601"

  # Трейсинг
  jaeger:
    image: jaegertracing/all-in-one:latest
    environment:
      - COLLECTOR_ZIPKIN_HOST_PORT=:9411
    ports:
      - "5775:5775/udp"
      - "6831:6831/udp"
      - "6832:6832/udp"
      - "5778:5778"
      - "16686:16686"
      - "14268:14268"
      - "14250:14250"
      - "9411:9411"

  # Алерты
  alertmanager:
    image: prom/alertmanager:latest
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    ports:
      - "9093:9093"

volumes:
  prometheus_data:
  grafana_data:
  elasticsearch_data:
```

---

### Задача 22: GitLab CI/CD Pipeline
**Цель:** Полноценный CI/CD с тестами, сканированием и деплоем

**.gitlab-ci.yml:**
```yaml
stages:
  - build
  - test
  - security
  - deploy

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
  KUBECONFIG: /root/.kube/config

# Сборка
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE
  only:
    - main
    - develop

# Юнит-тесты
unit-tests:
  stage: test
  image: node:18
  script:
    - npm install
    - npm run test:unit
  coverage: '/Statements\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

# Интеграционные тесты
integration-tests:
  stage: test
  image: docker:latest
  services:
    - docker:dind
    - postgres:14
  variables:
    DATABASE_URL: postgresql://postgres:password@postgres:5432/test
  script:
    - docker-compose -f docker-compose.test.yml up -d
    - npm run test:integration
    - docker-compose -f docker-compose.test.yml down

# Security сканирование
security-scan:
  stage: security
  image: aquasec/trivy:latest
  script:
    - trivy image --exit-code 1 --severity HIGH,CRITICAL $DOCKER_IMAGE
  allow_failure: true

# SAST сканирование
sast:
  stage: security
  image: returntocorp/semgrep:latest
  script:
    - semgrep --config=auto --json --output=sast-report.json
  artifacts:
    reports:
      sast: sast-report.json

# Деплой в staging
deploy-staging:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config set-cluster k8s --server="$KUBE_URL" --insecure-skip-tls-verify=true
    - kubectl config set-credentials admin --token="$KUBE_TOKEN"
    - kubectl config set-context default --cluster=k8s --user=admin
    - kubectl config use-context default
    - kubectl set image deployment/myapp myapp=$DOCKER_IMAGE -n staging
    - kubectl rollout status deployment/myapp -n staging --timeout=5m
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

# Деплой в production
deploy-production:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config set-cluster k8s --server="$KUBE_URL" --insecure-skip-tls-verify=true
    - kubectl config set-credentials admin --token="$KUBE_TOKEN"
    - kubectl config set-context default --cluster=k8s --user=admin
    - kubectl config use-context default
    - kubectl set image deployment/myapp myapp=$DOCKER_IMAGE -n production
    - kubectl rollout status deployment/myapp -n production --timeout=5m
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
```

---

### Задача 23: Cost Optimization
**Цель:** Мониторинг и оптимизация затрат на инфраструктуру

**cost_optimizer.sh:**
```bash
#!/bin/bash
# cost_optimizer.sh

set -euo pipefail

REPORT_FILE="/var/log/cost_report_$(date +%Y%m%d).txt"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a $REPORT_FILE
}

log "=== Infrastructure Cost Analysis ==="

# 1. Kubernetes unused resources
log "\n--- Kubernetes Resource Analysis ---"

# Найти pods с низким CPU usage
kubectl top pods -A | awk '$3 ~ /[0-9]+m$/ {if ($3+0 < 50) print $0}' | tee -a $REPORT_FILE

# Найти unused PVCs
log "\nUnused Persistent Volume Claims:"
kubectl get pvc -A -o json | jq -r '.items[] | select(.status.phase=="Bound") | select(.spec.volumeName as $v | [kubectl get pods -A -o json | .items[] | select(.spec.volumes[]?.persistentVolumeClaim.claimName==$v)] | length == 0) | "\(.metadata.namespace)/\(.metadata.name)"' | tee -a $REPORT_FILE

# 2. Docker образы
log "\n--- Docker Images Analysis ---"
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}" | sort -k3 -h -r | head -20 | tee -a $REPORT_FILE

# Найти dangling images
DANGLING=$(docker images -f "dangling=true" -q | wc -l)
log "Dangling images: $DANGLING"

# 3. Старые backups
log "\n--- Old Backups ---"
find /var/backups -type f -mtime +30 -exec ls -lh {} \; | awk '{sum+=$5}END{print "Total size of old backups: "sum/1024/1024" MB"}' | tee -a $REPORT_FILE

# 4. Рекомендации
log "\n--- Cost Optimization Recommendations ---"
log "1. Scale down underutilized pods"
log "2. Remove unused PVCs"
log "3. Cleanup old Docker images: docker image prune -a"
log "4. Review backup retention policies"
log "5. Consider spot/preemptible instances for non-critical workloads"

log "\n=== Report saved to $REPORT_FILE ==="
```

---

## 🎯 ИТОГОВЫЙ ПРОЕКТ

### Задача 24: Полная DevOps платформа
**Цель:** Создать комплексную платформу с всеми best practices

**Компоненты:**
1. Git (GitLab/Gitea)
2. CI/CD (GitLab CI / Jenkins)
3. Container Registry
4. Kubernetes Cluster
5. Monitoring (Prometheus + Grafana)
6. Logging (ELK Stack)
7. Secrets Management (Vault)
8. Service Mesh (Istio)
9. GitOps (ArgoCD)
10. Backup & DR

**deploy_platform.sh:**
```bash
#!/bin/bash
# deploy_platform.sh - Развертывание полной DevOps платформы

set -euo pipefail

PLATFORM_NAMESPACE="devops-platform"
DOMAIN="devops.local"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

# 1. Подготовка Kubernetes
setup_kubernetes() {
    log "Setting up Kubernetes namespaces..."
    
    kubectl create namespace $PLATFORM_NAMESPACE --dry-run=client -o yaml | kubectl apply -f -
    kubectl create namespace monitoring --dry-run=client -o yaml | kubectl apply -f -
    kubectl create namespace logging --dry-run=client -o yaml | kubectl apply -f -
    kubectl create namespace argocd --dry-run=client -o yaml | kubectl apply -f -
}

# 2. Установка Istio
install_istio() {
    log "Installing Istio..."
    istioctl install --set profile=demo -y
    kubectl label namespace default istio-injection=enabled
}

# 3. Установка мониторинга
install_monitoring() {
    log "Installing Prometheus and Grafana..."
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring
}

# 4. Установка логирования
install_logging() {
    log "Installing ELK Stack..."
    helm repo add elastic https://helm.elastic.co
    helm install elasticsearch elastic/elasticsearch -n logging
    helm install kibana elastic/kibana -n logging
    helm install filebeat elastic/filebeat -n logging
}

# 5. Установка ArgoCD
install_argocd() {
    log "Installing ArgoCD..."
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
}

# 6. Установка Vault
install_vault() {
    log "Installing Vault..."
    helm repo add hashicorp https://helm.releases.hashicorp.com
    helm install vault hashicorp/vault -n $PLATFORM_NAMESPACE
}

# 7. Установка GitLab
install_gitlab() {
    log "Installing GitLab..."
    helm repo add gitlab https://charts.gitlab.io/
    helm install gitlab gitlab/gitlab \
        --set global.hosts.domain=$DOMAIN \
        --set certmanager-issuer.email=admin@$DOMAIN \
        -n $PLATFORM_NAMESPACE
}

# Основная функция
main() {
    log "=== Starting DevOps Platform Deployment ==="
    
    setup_kubernetes
    install_istio
    install_monitoring
    install_logging
    install_argocd
    install_vault
    install_gitlab
    
    log "=== Platform Deployment Completed ==="
    log "Access URLs:"
    log "- GitLab: https://gitlab.$DOMAIN"
    log "- ArgoCD: https://argocd.$DOMAIN"
    log "- Grafana: https://grafana.$DOMAIN"
    log "- Kibana: https://kibana.$DOMAIN"
}

main "$@"
```

---

## 📖 Полезные команды и шпаргалки

### Docker
```bash
# Очистка
docker system prune -a
docker volume prune

# Логи
docker logs -f <container>
docker logs --tail 100 <container>

# Exec
docker exec -it <container> bash
docker exec <container> cat /etc/hosts

# Статистика
docker stats
docker top <container>
```

### Kubernetes
```bash
# Pods
kubectl get pods -A
kubectl describe pod <pod-name>
kubectl logs -f <pod-name>
kubectl exec -it <pod-name> -- bash

# Deployments
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>

# Debug
kubectl get events --sort-by='.lastTimestamp'
kubectl top nodes
kubectl top pods
```

### Git
```bash
# Полезные алиасы
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
```

---

## 🎓 Рекомендации по изучению

### Junior уровень
- Основы Linux
- Bash scripting
- Git basics
- Docker basics
- CI/CD концепции

### Middle уровень
- Kubernetes
- Terraform
- Ansible
- Monitoring & Logging
- Cloud providers

### Senior уровень
- Service Mesh
- Security & Compliance
- Cost Optimization
- Disaster Recovery
- Team Leadership

---

## ✅ Чеклист готовности

### Junior DevOps Engineer
- [ ] Могу написать bash скрипт для автоматизации
- [ ] Понимаю основы Docker
- [ ] Настроил базовый CI/CD pipeline
- [ ] Умею работать с Git
- [ ] Настроил мониторинг сервиса

### Middle DevOps