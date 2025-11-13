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
        
        # Отправка уведомления (опционально)
        # echo "Service $SERVICE_NAME restarted due to high CPU" | mail -s "Alert" admin@example.com
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

### Задача 3: CI/CD Pipeline для простого веб-приложения
**Цель:** Настроить автоматический деплой при push в Git

**Требования:**
- Использовать Git hooks или Gitea/Gitlab на Proxmox
- При push в master автоматически деплоить
- Запускать тесты перед деплоем
- Откатываться на предыдущую версию при ошибке

**Решение (с использованием Git hooks):**

**1. Создание тестового приложения:**
```bash
# Создаём простое Node.js приложение
mkdir ~/myapp && cd ~/myapp
npm init -y
npm install express

# app.js
cat > app.js << 'EOF'
const express = require('express');
const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
    res.send('Hello DevOps!');
});

app.get('/health', (req, res) => {
    res.status(200).json({ status: 'OK' });
});

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
EOF

# test.js
cat > test.js << 'EOF'
const http = require('http');

http.get('http://localhost:3000/health', (res) => {
    if (res.statusCode === 200) {
        console.log('Tests PASSED');
        process.exit(0);
    } else {
        console.log('Tests FAILED');
        process.exit(1);
    }
}).on('error', (e) => {
    console.error('Tests FAILED:', e.message);
    process.exit(1);
});
EOF
```

**2. Настройка Git репозитория:**
```bash
# На сервере создаём bare репозиторий
sudo mkdir -p /opt/git/myapp.git
cd /opt/git/myapp.git
sudo git init --bare
```

**3. Post-receive hook для автодеплоя:**
```bash
# /opt/git/myapp.git/hooks/post-receive
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
    echo "[$(date)] Backup created" >> $LOG_FILE
fi

# Клонируем новую версию
rm -rf $DEPLOY_DIR
git clone /opt/git/myapp.git $DEPLOY_DIR

cd $DEPLOY_DIR

# Устанавливаем зависимости
npm install >> $LOG_FILE 2>&1

# Запускаем приложение
pm2 stop myapp 2>/dev/null || true
pm2 start app.js --name myapp

# Ждём запуска
sleep 3

# Запускаем тесты
node test.js >> $LOG_FILE 2>&1

if [ $? -eq 0 ]; then
    echo "[$(date)] Deployment SUCCESSFUL" >> $LOG_FILE
    rm -rf $BACKUP_DIR
else
    echo "[$(date)] Tests FAILED. Rolling back..." >> $LOG_FILE
    pm2 stop myapp
    rm -rf $DEPLOY_DIR
    mv $BACKUP_DIR $DEPLOY_DIR
    cd $DEPLOY_DIR
    pm2 start app.js --name myapp
    echo "[$(date)] Rollback completed" >> $LOG_FILE
    exit 1
fi
EOF

chmod +x /opt/git/myapp.git/hooks/post-receive
```

**4. Использование:**
```bash
# На локальной машине
cd ~/myapp
git init
git add .
git commit -m "Initial commit"
git remote add origin ssh://user@your-server:/opt/git/myapp.git
git push origin master  # Автоматически запустится деплой
```

---

### Задача 4: Настройка Nginx reverse proxy с SSL
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

## 🟡 MIDDLE DEVOPS ENGINEER

### Задача 5: Инфраструктура как код - Terraform для Proxmox
**Цель:** Автоматизировать создание VM в Proxmox через Terraform

**Требования:**
- Создать модуль Terraform для VM
- Параметризовать CPU, RAM, диск
- Использовать cloud-init для начальной настройки
- Автоматически добавлять SSH ключи

**Решение:**

**1. Установка Terraform:**
```bash
# Fedora/Ubuntu
wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip
unzip terraform_1.6.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/
```

**2. Настройка Proxmox API:**
```bash
# На Proxmox сервере создать API токен
pveum user add terraform@pve
pveum aclmod / -user terraform@pve -role Administrator
pveum user token add terraform@pve terraform-token --privsep=0
```

**3. Структура Terraform проекта:**
```hcl
# main.tf
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

# VM Module
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
  
  # Cloud-init
  ipconfig0 = "ip=${var.ip_address_base}.${count.index + 10}/24,gw=${var.gateway}"
  
  sshkeys = file(var.ssh_public_key_file)
  
  lifecycle {
    ignore_changes = [
      network,
    ]
  }
}

# variables.tf
variable "proxmox_api_url" {
  description = "Proxmox API URL"
  type        = string
}

variable "proxmox_api_token_id" {
  description = "Proxmox API Token ID"
  type        = string
  sensitive   = true
}

variable "proxmox_api_token_secret" {
  description = "Proxmox API Token Secret"
  type        = string
  sensitive   = true
}

variable "proxmox_node" {
  description = "Proxmox node name"
  type        = string
  default     = "pve"
}

variable "vm_name" {
  description = "Base name for VMs"
  type        = string
  default     = "devops-vm"
}

variable "vm_count" {
  description = "Number of VMs to create"
  type        = number
  default     = 1
}

variable "cpu_cores" {
  description = "Number of CPU cores"
  type        = number
  default     = 2
}

variable "memory" {
  description = "RAM in MB"
  type        = number
  default     = 2048
}

variable "disk_size" {
  description = "Disk size"
  type        = string
  default     = "20G"
}

variable "storage" {
  description = "Storage location"
  type        = string
  default     = "local-lvm"
}

variable "template_name" {
  description = "Template name to clone"
  type        = string
  default     = "ubuntu-22-04-template"
}

variable "ip_address_base" {
  description = "Base IP address"
  type        = string
  default     = "192.168.1"
}

variable "gateway" {
  description = "Gateway IP"
  type        = string
  default     = "192.168.1.1"
}

variable "ssh_public_key_file" {
  description = "Path to SSH public key"
  type        = string
  default     = "~/.ssh/id_rsa.pub"
}

# outputs.tf
output "vm_ips" {
  value = proxmox_vm_qemu.devops_vm[*].default_ipv4_address
}

output "vm_names" {
  value = proxmox_vm_qemu.devops_vm[*].name
}
```

**4. Файл переменных terraform.tfvars:**
```hcl
proxmox_api_url          = "https://192.168.1.100:8006/api2/json"
proxmox_api_token_id     = "terraform@pve!terraform-token"
proxmox_api_token_secret = "your-secret-token"
vm_count                 = 3
cpu_cores                = 2
memory                   = 4096
```

**5. Использование:**
```bash
terraform init
terraform plan
terraform apply

# Уничтожение инфраструктуры
terraform destroy
```

---

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

```bash
kubectl apply -f deployment.yaml
kubectl get svc
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
      
  - job_name: 'nginx'
    static_configs:
      - targets: ['192.168.1.10:9113']  # nginx-exporter
```

**alerts.yml:**
```yaml
groups:
  - name: system_alerts
    interval: 30s
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage detected"
          description: "CPU usage is above 80% for 5 minutes"

      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Memory usage is above 85%"

      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 < 15
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Disk space is low"
          description: "Less than 15% disk space available"
```

**Запуск:**
```bash
docker-compose up -d

# Доступ:
# Grafana: http://localhost:3000 (admin/admin)
# Prometheus: http://localhost:9090
```

---

## 🔴 SENIOR DEVOPS ENGINEER

### Задача 8: GitOps с ArgoCD и автоматическая синхронизация
**Цель:** Внедрить GitOps практики для управления Kubernetes

**Решение:**

**1. Установка ArgoCD:**
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Получить начальный пароль
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port-forward для доступа
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

**2. Структура GitOps репозитория:**
```
gitops-repo/
├── apps/
│   ├── dev/
│   │   └── myapp/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       └── kustomization.yaml
│   ├── staging/
│   │   └── myapp/
│   └── production/
│       └── myapp/
└── argocd/
    ├── applications/
    │   ├── myapp-dev.yaml
    │   ├── myapp-staging.yaml
    │   └── myapp-prod.yaml
    └── projects/
        └── myapp-project.yaml
```

**3. ArgoCD Application манифест:**
```yaml
# argocd/applications/myapp-dev.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-dev
  namespace: argocd
spec:
  project: default
  
  source:
    repoURL: https://github.com/yourorg/gitops-repo.git
    targetRevision: main
    path: apps/dev/myapp
  
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  
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

**4. CI/CD Pipeline с автоматическим обновлением:**
```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build Docker image
        run: |
          docker build -t myapp:${{ github.sha }} .
          docker tag myapp:${{ github.sha }} myapp:latest
      
      - name: Push to registry
        run: |
          echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
          docker push myapp:${{ github.sha }}
          docker push myapp:latest
      
      - name: Update GitOps repo
        run: |
          git clone https://${{ secrets.GIT_TOKEN }}@github.com/yourorg/gitops-repo.git
          cd gitops-repo
          
          # Обновить image tag в Kubernetes манифесте
          sed -i "s|image: myapp:.*|image: myapp:${{ github.sha }}|g" apps/dev/myapp/deployment.yaml
          
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add .
          git commit -m "Update myapp to ${{ github.sha }}"
          git push
```

---

### Задача 9: High Availability PostgreSQL кластер с Patroni
**Цель:** Развернуть отказоустойчивый кластер БД

**Решение:**

```bash
#!/bin/bash
# Скрипт установки на каждую ноду (3 ноды минимум)

# 1. Установка зависимостей
sudo apt update
sudo apt install -y postgresql postgresql-contrib etcd python3-pip python3-psycopg2

# 2. Установка Patroni
sudo pip3 install patroni[etcd]

# 3. Конфигурация Patroni
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

tags:
    nofailover: false
    noloadbalance: false
    clonefrom: false
    nosync: false
EOF

# 4. Systemd service
sudo cat > /etc/systemd/system/patroni.service << EOF
[Unit]
Description=Patroni (PostgreSQL HA)
After=syslog.target network.target

[Service]
Type=simple
User=postgres
Group=postgres
ExecStart=/usr/local/bin/patroni /etc/patroni.yml
ExecReload=/bin/kill -HUP $MAINPID
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

**HAProxy для балансировки:**
```bash
# /etc/haproxy/haproxy.cfg
global
    maxconn 100

defaults
    log global
    mode tcp
    retries 2
    timeout client 30m
    timeout connect 4s
    timeout server 30m
    timeout check 5s

listen postgres
    bind *:5000
    option httpchk
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server pg1 192.168.1.10:5432 maxconn 100 check port 8008
    server pg2 192.168.1.11:5432 maxconn 100 check port 8008
    server pg3 192.168.1.12:5432 maxconn 100 check port 8008
```

**Проверка кластера:**
```bash
# Проверить статус кластера
patronictl -c /etc/patroni.yml list

# Ручной failover (для тестирования)
patronictl -c /etc/patroni.yml failover

# Подключение к мастеру
psql -h 127.0.0.1 -p 5000 -U postgres

# Подключение к репликам (read-only)
psql -h 127.0.0.1 -p 5001 -U postgres
```

---

### Задача 10: Полный цикл Disaster Recovery с автоматизацией
**Цель:** Создать систему резервного копирования и восстановления всей инфраструктуры

**Решение:**

**1. Скрипт полного бэкапа инфраструктуры:**
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

error_exit() {
    log "ERROR: $1"
    exit 1
}

# Создание структуры директорий
mkdir -p $BACKUP_DIR/{databases,configs,containers,vms,apps}

log "=== Starting Disaster Recovery Backup ==="

# 1. Backup всех баз данных
log "Backing up databases..."
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
    
    # MongoDB
    if systemctl is-active --quiet mongod; then
        mongodump --out=$BACKUP_DIR/databases/mongodb
        tar -czf $BACKUP_DIR/databases/mongodb.tar.gz -C $BACKUP_DIR/databases mongodb
        rm -rf $BACKUP_DIR/databases/mongodb
        log "MongoDB backup completed"
    fi
}

# 2. Backup конфигураций
log "Backing up system configurations..."
backup_configs() {
    tar -czf $BACKUP_DIR/configs/etc.tar.gz \
        /etc/nginx \
        /etc/apache2 \
        /etc/haproxy \
        /etc/systemd/system \
        /etc/ssh \
        /etc/hosts \
        /etc/fstab \
        2>/dev/null || true
    
    # Docker configs
    if command -v docker &> /dev/null; then
        cp -r /etc/docker $BACKUP_DIR/configs/ 2>/dev/null || true
    fi
    
    # Kubernetes configs
    if [ -d ~/.kube ]; then
        cp -r ~/.kube $BACKUP_DIR/configs/
    fi
    
    log "Configuration backup completed"
}

# 3. Backup Docker контейнеров и volumes
log "Backing up Docker containers..."
backup_docker() {
    if command -v docker &> /dev/null; then
        # Список всех контейнеров
        docker ps -a --format "{{.Names}}" > $BACKUP_DIR/containers/container_list.txt
        
        # Backup docker-compose файлов
        find /opt /home -name "docker-compose.yml" -exec cp --parents {} $BACKUP_DIR/containers/ \; 2>/dev/null || true
        
        # Backup volumes
        docker volume ls -q | while read volume; do
            docker run --rm -v $volume:/data -v $BACKUP_DIR/containers:/backup \
                alpine tar czf /backup/volume_$volume.tar.gz -C /data .
        done
        
        log "Docker backup completed"
    fi
}

# 4. Backup Proxmox VMs (если запущено на Proxmox хосте)
log "Backing up VM configurations..."
backup_vms() {
    if command -v pvesh &> /dev/null; then
        pvesh get /cluster/resources --type vm --output-format json > $BACKUP_DIR/vms/vm_list.json
        cp -r /etc/pve $BACKUP_DIR/vms/ 2>/dev/null || true
        log "VM configuration backup completed"
    fi
}

# 5. Backup приложений
log "Backing up applications..."
backup_apps() {
    # Web приложения
    if [ -d /var/www ]; then
        tar -czf $BACKUP_DIR/apps/var_www.tar.gz /var/www 2>/dev/null || true
    fi
    
    # Домашние директории
    if [ -d /opt ]; then
        tar -czf $BACKUP_DIR/apps/opt.tar.gz /opt 2>/dev/null || true
    fi
    
    log "Application backup completed"
}

# 6. Создание манифеста бэкапа
create_manifest() {
    cat > $BACKUP_DIR/manifest.json << EOF
{
    "timestamp": "$TIMESTAMP",
    "hostname": "$(hostname)",
    "os": "$(lsb_release -d | cut -f2)",
    "kernel": "$(uname -r)",
    "backup_size": "$(du -sh $BACKUP_DIR | cut -f1)",
    "databases": $(ls -1 $BACKUP_DIR/databases 2>/dev/null | wc -l),
    "containers": $(cat $BACKUP_DIR/containers/container_list.txt 2>/dev/null | wc -l),
    "completion_time": "$(date '+%Y-%m-%d %H:%M:%S')"
}
EOF
    log "Manifest created"
}

# 7. Создание recovery инструкций
create_recovery_guide() {
    cat > $BACKUP_DIR/RECOVERY_GUIDE.md << 'EOF'
# Disaster Recovery Guide

## Быстрое восстановление

### 1. Восстановление баз данных
```bash
# PostgreSQL
gunzip -c databases/postgresql_all.sql.gz | psql -U postgres

# MySQL
gunzip -c databases/mysql_all.sql.gz | mysql -u root -p

# MongoDB
tar -xzf databases/mongodb.tar.gz
mongorestore mongodb/
```

### 2. Восстановление конфигураций
```bash
cd configs
tar -xzf etc.tar.gz -C /
systemctl daemon-reload
```

### 3. Восстановление Docker
```bash
# Восстановить volumes
cd containers
for vol in volume_*.tar.gz; do
    name=$(echo $vol | sed 's/volume_//;s/.tar.gz//')
    docker volume create $name
    docker run --rm -v $name:/data -v $(pwd):/backup alpine \
        tar xzf /backup/$vol -C /data
done

# Запустить docker-compose проекты
find . -name "docker-compose.yml" -execdir docker-compose up -d \;
```

### 4. Проверка восстановления
```bash
# Проверить сервисы
systemctl status postgresql mysql nginx docker

# Проверить Docker
docker ps
docker volume ls

# Проверить подключения к БД
psql -U postgres -c "SELECT version();"
```

## Контакты для экстренной помощи
- DevOps Team: devops@company.com
- On-call: +1-555-0100
EOF
    log "Recovery guide created"
}

# Выполнение всех бэкапов
backup_databases
backup_configs
backup_docker
backup_vms
backup_apps
create_manifest
create_recovery_guide

# Создание общего архива
log "Creating final archive..."
cd $BACKUP_ROOT
tar -czf backup_$TIMESTAMP.tar.gz $TIMESTAMP

# Очистка старых бэкапов
log "Cleaning old backups..."
find $BACKUP_ROOT -name "backup_*.tar.gz" -mtime +$RETENTION_DAYS -delete
find $BACKUP_ROOT -maxdepth 1 -type d -mtime +$RETENTION_DAYS -exec rm -rf {} \;

# Расчет контрольной суммы
sha256sum backup_$TIMESTAMP.tar.gz > backup_$TIMESTAMP.tar.gz.sha256

log "=== Backup completed successfully ==="
log "Backup location: $BACKUP_ROOT/backup_$TIMESTAMP.tar.gz"
log "Backup size: $(du -h $BACKUP_ROOT/backup_$TIMESTAMP.tar.gz | cut -f1)"

# Отправка уведомления (опционально)
if command -v curl &> /dev/null; then
    curl -X POST https://hooks.slack.com/services/YOUR/WEBHOOK/URL \
        -H 'Content-Type: application/json' \
        -d "{\"text\":\"✅ DR Backup completed: $TIMESTAMP\"}" 2>/dev/null || true
fi
```

**2. Автоматическое восстановление:**
```bash
#!/bin/bash
# disaster_recovery_restore.sh

set -euo pipefail

if [ $# -lt 1 ]; then
    echo "Usage: $0 <backup_file.tar.gz> [--full|--databases|--configs]"
    exit 1
fi

BACKUP_FILE=$1
MODE=${2:-"--full"}
TEMP_DIR="/tmp/dr_restore_$"
LOG_FILE="/var/log/dr_restore.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a $LOG_FILE
}

# Проверка контрольной суммы
verify_backup() {
    if [ -f "$BACKUP_FILE.sha256" ]; then
        log "Verifying backup integrity..."
        sha256sum -c "$BACKUP_FILE.sha256" || {
            log "ERROR: Backup integrity check failed!"
            exit 1
        }
        log "Backup integrity verified"
    fi
}

# Извлечение бэкапа
extract_backup() {
    log "Extracting backup..."
    mkdir -p $TEMP_DIR
    tar -xzf $BACKUP_FILE -C $TEMP_DIR --strip-components=1
    log "Backup extracted to $TEMP_DIR"
}

# Восстановление баз данных
restore_databases() {
    log "Restoring databases..."
    
    if [ -f "$TEMP_DIR/databases/postgresql_all.sql.gz" ]; then
        log "Restoring PostgreSQL..."
        systemctl stop postgresql || true
        gunzip -c $TEMP_DIR/databases/postgresql_all.sql.gz | sudo -u postgres psql
        systemctl start postgresql
        log "PostgreSQL restored"
    fi
    
    if [ -f "$TEMP_DIR/databases/mysql_all.sql.gz" ]; then
        log "Restoring MySQL..."
        systemctl stop mysql || true
        gunzip -c $TEMP_DIR/databases/mysql_all.sql.gz | mysql -u root
        systemctl start mysql
        log "MySQL restored"
    fi
}

# Восстановление конфигураций
restore_configs() {
    log "Restoring configurations..."
    
    if [ -f "$TEMP_DIR/configs/etc.tar.gz" ]; then
        tar -xzf $TEMP_DIR/configs/etc.tar.gz -C /
        systemctl daemon-reload
        log "System configurations restored"
    fi
}

# Восстановление Docker
restore_docker() {
    log "Restoring Docker environment..."
    
    # Восстановление volumes
    for vol_backup in $TEMP_DIR/containers/volume_*.tar.gz; do
        if [ -f "$vol_backup" ]; then
            vol_name=$(basename $vol_backup | sed 's/volume_//;s/.tar.gz//')
            log "Restoring volume: $vol_name"
            docker volume create $vol_name 2>/dev/null || true
            docker run --rm -v $vol_name:/data -v $TEMP_DIR/containers:/backup \
                alpine tar xzf /backup/$(basename $vol_backup) -C /data
        fi
    done
    
    log "Docker volumes restored"
}

# Основная логика восстановления
main() {
    log "=== Starting Disaster Recovery Restore ==="
    log "Backup file: $BACKUP_FILE"
    log "Mode: $MODE"
    
    verify_backup
    extract_backup
    
    case $MODE in
        --full)
            restore_databases
            restore_configs
            restore_docker
            ;;
        --databases)
            restore_databases
            ;;
        --configs)
            restore_configs
            ;;
        *)
            log "Unknown mode: $MODE"
            exit 1
            ;;
    esac
    
    # Очистка
    rm -rf $TEMP_DIR
    
    log "=== Restore completed successfully ==="
    log "Please verify all services are running correctly"
    log "Run: systemctl status postgresql mysql nginx docker"
}

# Подтверждение перед восстановлением
read -p "This will restore from backup. Continue? (yes/no) " -n 3 -r
echo
if [[ $REPLY =~ ^yes$ ]]; then
    main
else
    echo "Restore cancelled"
    exit 0
fi
```

**3. Настройка автоматического бэкапа:**
```bash
# Добавить в crontab
sudo crontab -e

# Ежедневный бэкап в 3:00
0 3 * * * /usr/local/bin/disaster_recovery_backup.sh

# Еженедельная проверка восстановления (в тестовой среде)
0 4 * * 0 /usr/local/bin/test_dr_restore.sh
```

---

### Задача 11: Многорегиональный Terraform с remote state
**Цель:** Управление инфраструктурой в нескольких регионах/дата-центрах

**Решение:**

**Структура проекта:**
```
terraform-multi-region/
├── global/
│   ├── backend.tf
│   └── versions.tf
├── modules/
│   ├── network/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── compute/
│   └── database/
├── environments/
│   ├── production/
│   │   ├── region-eu/
│   │   │   ├── main.tf
│   │   │   ├── backend.tf
│   │   │   └── terraform.tfvars
│   │   └── region-us/
│   │       ├── main.tf
│   │       ├── backend.tf
│   │       └── terraform.tfvars
│   ├── staging/
│   └── development/
└── scripts/
    ├── deploy-all.sh
    └── validate-all.sh
```

**1. Backend конфигурация (S3 или MinIO на Proxmox):**
```hcl
# global/backend.tf
terraform {
  backend "s3" {
    bucket         = "terraform-state"
    key            = "global/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

**2. Модуль для создания VM:**
```hcl
# modules/compute/main.tf
variable "vm_name" {
  type = string
}

variable "cpu_cores" {
  type    = number
  default = 2
}

variable "memory" {
  type    = number
  default = 2048
}

variable "disk_size" {
  type    = string
  default = "20G"
}

variable "network_config" {
  type = object({
    ip      = string
    gateway = string
    netmask = string
  })
}

variable "tags" {
  type    = map(string)
  default = {}
}

resource "proxmox_vm_qemu" "vm" {
  name        = var.vm_name
  target_node = var.target_node
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
  
  ipconfig0 = "ip=${var.network_config.ip}/${var.network_config.netmask},gw=${var.network_config.gateway}"
  
  tags = join(",", [for k, v in var.tags : "${k}=${v}"])
  
  lifecycle {
    create_before_destroy = true
  }
}

output "vm_id" {
  value = proxmox_vm_qemu.vm.id
}

output "vm_ip" {
  value = var.network_config.ip
}
```

**3. Environment-specific конфигурация:**
```hcl
# environments/production/region-eu/main.tf
terraform {
  backend "s3" {
    bucket = "terraform-state"
    key    = "production/eu/terraform.tfstate"
    region = "us-east-1"
  }
}

locals {
  region      = "eu"
  environment = "production"
  
  common_tags = {
    Environment = local.environment
    Region      = local.region
    ManagedBy   = "Terraform"
    Team        = "DevOps"
  }
}

module "web_servers" {
  source = "../../../modules/compute"
  count  = var.web_server_count
  
  vm_name    = "web-${local.region}-${count.index + 1}"
  cpu_cores  = 4
  memory     = 8192
  disk_size  = "50G"
  
  network_config = {
    ip      = "10.0.1.${count.index + 10}"
    gateway = "10.0.1.1"
    netmask = "24"
  }
  
  tags = merge(local.common_tags, {
    Role = "WebServer"
    Name = "web-${local.region}-${count.index + 1}"
  })
}

module "database_servers" {
  source = "../../../modules/compute"
  count  = var.db_server_count
  
  vm_name    = "db-${local.region}-${count.index + 1}"
  cpu_cores  = 8
  memory     = 16384
  disk_size  = "200G"
  
  network_config = {
    ip      = "10.0.2.${count.index + 10}"
    gateway = "10.0.2.1"
    netmask = "24"
  }
  
  tags = merge(local.common_tags, {
    Role = "Database"
    Name = "db-${local.region}-${count.index + 1}"
  })
}

output "web_server_ips" {
  value = module.web_servers[*].vm_ip
}

output "database_server_ips" {
  value = module.database_servers[*].vm_ip
}
```

**4. Скрипт для деплоя всех регионов:**
```bash
#!/bin/bash
# scripts/deploy-all.sh

set -euo pipefail

ENVIRONMENTS=("production" "staging")
REGIONS=("region-eu" "region-us")

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

deploy_region() {
    local env=$1
    local region=$2
    local path="environments/$env/$region"
    
    log "Deploying $env/$region..."
    
    cd $path
    
    # Инициализация
    terraform init -upgrade
    
    # Валидация
    terraform validate
    
    # План
    terraform plan -out=tfplan
    
    # Применение (с подтверждением)
    if [ "${AUTO_APPROVE:-false}" = "true" ]; then
        terraform apply tfplan
    else
        terraform apply tfplan
    fi
    
    # Очистка
    rm -f tfplan
    
    cd - > /dev/null
    
    log "✓ Completed $env/$region"
}

# Деплой production с подтверждением
for region in "${REGIONS[@]}"; do
    deploy_region "production" "$region"
done

# Деплой staging автоматически
AUTO_APPROVE=true
for region in "${REGIONS[@]}"; do
    deploy_region "staging" "$region"
done

log "All deployments completed successfully"
```

---

### Задача 12: Автоматизация с Ansible - Configuration Management
**Цель:** Управление конфигурацией множества серверов

**Решение:**

**Структура Ansible проекта:**
```
ansible/
├── ansible.cfg
├── inventory/
│   ├── production/
│   │   ├── hosts.yml
│   │   └── group_vars/
│   │       ├── all.yml
│   │       ├── webservers.yml
│   │       └── databases.yml
│   └── staging/
├── roles/
│   ├── common/
│   │   ├── tasks/
│   │   ├── handlers/
│   │   ├── templates/
│   │   └── files/
│   ├── webserver/
│   ├── database/
│   └── monitoring/
├── playbooks/
│   ├── site.yml
│   ├── webservers.yml
│   └── databases.yml
└── scripts/
    └── deploy.sh
```

**1. Inventory файл:**
```yaml
# inventory/production/hosts.yml
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.10
          ansible_user: deploy
        web2:
          ansible_host: 192.168.1.11
          ansible_user: deploy
        web3:
          ansible_host: 192.168.1.12
          ansible_user: deploy
    
    databases:
      hosts:
        db1:
          ansible_host: 192.168.1.20
          ansible_user: deploy
          postgres_role: master
        db2:
          ansible_host: 192.168.1.21
          ansible_user: deploy
          postgres_role: replica
    
    loadbalancers:
      hosts:
        lb1:
          ansible_host: 192.168.1.30
          ansible_user: deploy
    
    monitoring:
      hosts:
        monitor1:
          ansible_host: 192.168.1.40
          ansible_user: deploy
```

**2. Group variables:**
```yaml
# inventory/production/group_vars/all.yml
---
ansible_python_interpreter: /usr/bin/python3
ansible_ssh_private_key_file: ~/.ssh/id_rsa

# Common packages
common_packages:
  - vim
  - htop
  - curl
  - wget
  - git
  - net-tools

# NTP Configuration
ntp_servers:
  - 0.pool.ntp.org
  - 1.pool.ntp.org

# Monitoring
node_exporter_version: "1.6.1"
prometheus_server: "192.168.1.40:9090"

# Backup
backup_destination: "/mnt/backup"
backup_retention_days: 30
```

```yaml
# inventory/production/group_vars/webservers.yml
---
nginx_version: latest
nginx_worker_processes: auto
nginx_worker_connections: 1024

app_name: "myapp"
app_port: 3000
app_user: "appuser"
app_directory: "/opt/{{ app_name }}"

ssl_certificate: "/etc/ssl/certs/{{ app_name }}.crt"
ssl_certificate_key: "/etc/ssl/private/{{ app_name }}.key"
```

**3. Common role:**
```yaml
# roles/common/tasks/main.yml
---
- name: Update package cache
  apt:
    update_cache: yes
    cache_valid_time: 3600
  when: ansible_os_family == "Debian"

- name: Install common packages
  package:
    name: "{{ common_packages }}"
    state: present

- name: Configure timezone
  timezone:
    name: UTC

- name: Configure NTP
  template:
    src: ntp.conf.j2
    dest: /etc/ntp.conf
  notify: restart ntp

- name: Create deploy user
  user:
    name: deploy
    groups: sudo
    shell: /bin/bash
    create_home: yes

- name: Add SSH key for deploy user
  authorized_key:
    user: deploy
    key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"

- name: Configure sudo without password
  lineinfile:
    path: /etc/sudoers.d/deploy
    line: 'deploy ALL=(ALL) NOPASSWD: ALL'
    create: yes
    validate: 'visudo -cf %s'

- name: Install node_exporter
  include_tasks: node_exporter.yml

- name: Configure firewall
  include_tasks: firewall.yml
```

**4. Webserver role:**
```yaml
# roles/webserver/tasks/main.yml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present

- name: Create application user
  user:
    name: "{{ app_user }}"
    system: yes
    create_home: no

- name: Create application directory
  file:
    path: "{{ app_directory }}"
    state: directory
    owner: "{{ app_user }}"
    group: "{{ app_user }}"
    mode: '0755'

- name: Deploy application configuration
  template:
    src: nginx-app.conf.j2
    dest: "/etc/nginx/sites-available/{{ app_name }}"
  notify: reload nginx

- name: Enable application site
  file:
    src: "/etc/nginx/sites-available/{{ app_name }}"
    dest: "/etc/nginx/sites-enabled/{{ app_name }}"
    state: link
  notify: reload nginx

- name: Remove default site
  file:
    path: /etc/nginx/sites-enabled/default
    state: absent
  notify: reload nginx

- name: Ensure Nginx is running
  systemd:
    name: nginx
    state: started
    enabled: yes

- name: Configure log rotation
  template:
    src: logrotate.conf.j2
    dest: "/etc/logrotate.d/{{ app_name }}"
```

**5. Template для Nginx:**
```jinja2
# roles/webserver/templates/nginx-app.conf.j2
upstream {{ app_name }}_backend {
    least_conn;
    {% for host in groups['webservers'] %}
    server {{ hostvars[host].ansible_host }}:{{ app_port }} max_fails=3 fail_timeout=30s;
    {% endfor %}
}

server {
    listen 80;
    server_name {{ app_domain | default('_') }};
    
    # Redirect to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name {{ app_domain | default('_') }};
    
    ssl_certificate {{ ssl_certificate }};
    ssl_certificate_key {{ ssl_certificate_key }};
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    access_log /var/log/nginx/{{ app_name }}_access.log;
    error_log /var/log/nginx/{{ app_name }}_error.log;
    
    location / {
        proxy_pass http://{{ app_name }}_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
    
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
```

**6. Main playbook:**
```yaml
# playbooks/site.yml
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
  tags: webservers

- name: Configure database servers
  hosts: databases
  become: yes
  roles:
    - database
  tags: databases

- name: Configure load balancers
  hosts: loadbalancers
  become: yes
  roles:
    - loadbalancer
  tags: loadbalancers

- name: Configure monitoring
  hosts: monitoring
  become: yes
  roles:
    - prometheus
    - grafana
  tags: monitoring
```

**7. Deployment script:**
```bash
#!/bin/bash
# scripts/deploy.sh

set -euo pipefail

INVENTORY=${1:-"production"}
TAGS=${2:-"all"}
CHECK_MODE=${CHECK_MODE:-false}

echo "Deploying to: $INVENTORY"
echo "Tags: $TAGS"

# Проверка синтаксиса
ansible-playbook playbooks/site.yml \
    -i inventory/$INVENTORY/hosts.yml \
    --syntax-check

# Dry-run
if [ "$CHECK_MODE" = "true" ]; then
    ansible-playbook playbooks/site.yml \
        -i inventory/$INVENTORY/hosts.yml \
        --tags "$TAGS" \
        --check \
        --diff
    exit 0
fi

# Реальный деплой
ansible-playbook playbooks/site.yml \
    -i inventory/$INVENTORY/hosts.yml \
    --tags "$TAGS" \
    -v

echo "Deployment completed!"
```

**Использование:**
```bash
# Проверка конфигурации
CHECK_MODE=true ./scripts/deploy.sh production

# Деплой только веб-серверов
./scripts/deploy.sh production webservers

# Деплой всей инфраструктуры
./scripts/deploy.sh production all

# Ad-hoc команды
ansible all -i inventory/production/hosts.yml -m ping
ansible webservers -i inventory/production/hosts.yml -a "systemctl status nginx"
```

---

### Задача 13: Security Hardening и Compliance
**Цель:** Автоматизация проверки безопасности и соответствия стандартам

**Решение:**

**1. Скрипт аудита безопасности:**
```bash
#!/bin/bash
# security_audit.sh

set -euo pipefail

REPORT_DIR="/var/log/security_audit"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
REPORT_FILE="$REPORT_DIR/audit_$TIMESTAMP.html"

mkdir -p $REPORT_DIR

# HTML Header
cat > $REPORT_FILE << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Security Audit Report</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .pass { color: green; font-weight: bold; }
        .fail { color: red; font-weight: bold; }
        .warn { color: orange; font-weight: bold; }
        table { border-collapse: collapse; width: 100%; margin: 20px 0; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #4CAF50; color: white; }
        .section { margin-top: 30px; }
    </style>
</head>
<body>
EOF

echo "<h1>Security Audit Report</h1>" >> $REPORT_FILE
echo "<p>Generated: $(date)</p>" >> $REPORT_FILE
echo "<p>Hostname: $(hostname)</p>" >> $REPORT_FILE
echo "<p>OS: $(lsb_release -d | cut -f2)</p>" >> $REPORT_FILE

# Функция для добавления результатов
add_check() {
    local title=$1
    local status=$2
    local details=$3
    
    echo "<div class='section'>" >> $REPORT_FILE
    echo "<h3>$title</h3>" >> $REPORT_FILE
    echo "<p class='$status'>Status: $status</p>" >> $REPORT_FILE
    echo "<pre>$details</pre>" >> $REPORT_FILE
    echo "</div>" >> $REPORT_FILE
}

# 1. Проверка обновлений системы
echo "Checking system updates..."
if apt list --upgradable 2>/dev/null | grep -q upgradable; then
    UPDATES=$(apt list --upgradable 2>/dev/null | grep upgradable | wc -l)
    add_check "System Updates" "warn" "$UPDATES security updates available"
else
    add_check "System Updates" "pass" "System is up to date"
fi

# 2. Проверка открытых портов
echo "Checking open ports..."
OPEN_PORTS=$(ss -tuln | grep LISTEN)
add_check "Open Ports" "warn" "$OPEN_PORTS"

# 3. Проверка SSH конфигурации
echo "Checking SSH configuration..."
SSH_ISSUES=""

if grep -q "^PermitRootLogin yes" /etc/ssh/sshd_config 2>/dev/null; then
    SSH_ISSUES+="❌ Root login is enabled\n"
fi

if grep -q "^PasswordAuthentication yes" /etc/ssh/sshd_config 2>/dev/null; then
    SSH_ISSUES+="❌ Password authentication is enabled\n"
fi

if [ -z "$SSH_ISSUES" ]; then
    add_check "SSH Configuration" "pass" "SSH is properly configured"
else
    add_check "SSH Configuration" "fail" "$SSH_ISSUES"
fi

# 4. Проверка firewall
echo "Checking firewall..."
if systemctl is-active --quiet ufw || systemctl is-active --quiet firewalld; then
    FIREWALL_STATUS=$(ufw status 2>/dev/null || firewall-cmd --state 2>/dev/null)
    add_check "Firewall" "pass" "Firewall is active: $FIREWALL_STATUS"
else
    add_check "Firewall" "fail" "No firewall is active"
fi

# 5. Проверка fail2ban
echo "Checking fail2ban..."
if systemctl is-active --quiet fail2ban; then
    FAIL2BAN_STATUS=$(fail2ban-client status 2>/dev/null || echo "Running")
    add_check "Fail2ban" "pass" "Fail2ban is active: $FAIL2BAN_STATUS"
else
    add_check "Fail2ban" "warn" "Fail2ban is not installed or not running"
fi

# 6. Проверка прав на критичные файлы
echo "Checking file permissions..."
PERM_ISSUES=""

if [ $(stat -c %a /etc/passwd) != "644" ]; then
    PERM_ISSUES+="❌ /etc/passwd has incorrect permissions\n"
fi

if [ $(stat -c %a /etc/shadow) != "640" ] && [ $(stat -c %a /etc/shadow) != "600" ]; then
    PERM_ISSUES+="❌ /etc/shadow has incorrect permissions\n"
fi

if [ -z "$PERM_ISSUES" ]; then
    add_check "File Permissions" "pass" "Critical files have correct permissions"
else
    add_check "File Permissions" "fail" "$PERM_ISSUES"
fi

# 7. Проверка SUID/SGID файлов
echo "Checking SUID/SGID files..."
SUID_FILES=$(find / -type f \( -perm -4000 -o -perm -2000 \) 2>/dev/null | head -20)
add_check "SUID/SGID Files" "warn" "Found SUID/SGID files (showing first 20):\n$SUID_FILES"

# 8. Проверка неудачных попыток входа
echo "Checking failed login attempts..."
FAILED_LOGINS=$(grep "Failed password" /var/log/auth.log 2>/dev/null | tail -10 || echo "No recent failed attempts")
add_check "Failed Login Attempts" "warn" "$FAILED_LOGINS"

# 9. Проверка запущенных сервисов
echo "Checking running services..."
SERVICES=$(systemctl list-units --type=service --state=running | grep running)
add_check "Running Services" "pass" "$SERVICES"

# 10. Проверка Docker безопасности (если установлен)
if command -v docker &> /dev/null; then
    echo "Checking Docker security..."
    
    DOCKER_ISSUES=""
    
    # Проверка privileged контейнеров
    PRIV_CONTAINERS=$(docker ps --filter "status=running" --format "{{.ID}}" | xargs -I {} docker inspect {} --format '{{.Name}}: Privileged={{.HostConfig.Privileged}}' | grep "Privileged=true" || echo "")
    
    if [ -n "$PRIV_CONTAINERS" ]; then
        DOCKER_ISSUES+="❌ Privileged containers found:\n$PRIV_CONTAINERS\n"
    fi
    
    # Проверка контейнеров с root
    ROOT_CONTAINERS=$(docker ps --format "{{.Names}}" | xargs -I {} docker exec {} whoami 2>/dev/null | grep -c "root" || echo "0")
    
    if [ "$ROOT_CONTAINERS" != "0" ]; then
        DOCKER_ISSUES+="⚠️  $ROOT_CONTAINERS containers running as root\n"
    fi
    
    if [ -z "$DOCKER_ISSUES" ]; then
        add_check "Docker Security" "pass" "Docker containers are properly configured"
    else
        add_check "Docker Security" "fail" "$DOCKER_ISSUES"
    fi
fi

# 11. Проверка последних изменений в /etc
echo "Checking recent /etc changes..."
RECENT_CHANGES=$(find /etc -type f -mtime -7 2>/dev/null | head -20)
add_check "Recent /etc Changes" "warn" "Files modified in last 7 days:\n$RECENT_CHANGES"

# 12. Проверка дискового пространства
echo "Checking disk space..."
DISK_USAGE=$(df -h | awk '$5+0 > 80 {print $0}')
if [ -n "$DISK_USAGE" ]; then
    add_check "Disk Space" "fail" "Partitions with >80% usage:\n$DISK_USAGE"
else
    add_check "Disk Space" "pass" "All partitions have sufficient space"
fi

# HTML Footer
cat >> $REPORT_FILE << 'EOF'
<div class='section'>
    <h3>Recommendations</h3>
    <ul>
        <li>Disable root SSH login</li>
        <li>Use SSH keys instead of passwords</li>
        <li>Keep system updated</li>
        <li>Enable and configure firewall</li>
        <li>Install fail2ban for brute-force protection</li>
        <li>Regular security audits</li>
        <li>Monitor logs for suspicious activity</li>
    </ul>
</div>
</body>
</html>
EOF

echo "Audit report generated: $REPORT_FILE"

# Отправка отчета по email (опционально)
if command -v mail &> /dev/null; then
    cat $REPORT_FILE | mail -s "Security Audit Report - $(hostname)" -a "Content-Type: text/html" security@company.com
fi

# Отправка в Slack (опционально)
if [ -n "${SLACK_WEBHOOK:-}" ]; then
    curl -X POST $SLACK_WEBHOOK \
        -H 'Content-Type: application/json' \
        -d "{\"text\":\"🔒 Security audit completed for $(hostname). Report: $REPORT_FILE\"}"
fi
```

**2. Автоматическое исправление уязвимостей:**
```bash
#!/bin/bash
# security_hardening.sh

set -euo pipefail

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

log "Starting security hardening..."

# 1. SSH Hardening
log "Hardening SSH..."
cat > /etc/ssh/sshd_config.d/99-hardening.conf << 'EOF'
# Disable root login
PermitRootLogin no

# Disable password authentication
PasswordAuthentication no
PubkeyAuthentication yes

# Disable empty passwords
PermitEmptyPasswords no

# Disable X11 forwarding
X11Forwarding no

# Use only strong ciphers
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512

# Set login grace time
LoginGraceTime 30

# Maximum authentication attempts
MaxAuthTries 3

# Maximum sessions
MaxSessions 2

# Client alive interval
ClientAliveInterval 300
ClientAliveCountMax 2
EOF

systemctl restart sshd
log "SSH hardened"

# 2. Установка и настройка firewall
log "Configuring firewall..."
apt install -y ufw

# Разрешить SSH
ufw allow 22/tcp

# Разрешить HTTP/HTTPS
ufw allow 80/tcp
ufw allow 443/tcp

# Включить firewall
ufw --force enable
log "Firewall configured"

# 3. Установка fail2ban
log "Installing fail2ban..."
apt install -y fail2ban

cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5
destemail = admin@company.com
sendername = Fail2Ban

[sshd]
enabled = true
port = 22
logpath = /var/log/auth.log

[nginx-http-auth]
enabled = true

[nginx-noscript]
enabled = true

[nginx-badbots]
enabled = true
EOF

systemctl enable fail2ban
systemctl restart fail2ban
log "Fail2ban configured"

# 4. Автоматические обновления безопасности
log "Enabling automatic security updates..."
apt install -y unattended-upgrades

cat > /etc/apt/apt.conf.d/50unattended-upgrades << 'EOF'
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
};
Unattended-Upgrade::AutoFixInterruptedDpkg "true";
Unattended-Upgrade::MinimalSteps "true";
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "false";
Unattended-Upgrade::Mail "admin@company.com";
EOF

systemctl enable unattended-upgrades
log "Automatic updates enabled"

# 5. Настройка файловых прав
log "Setting correct file permissions..."
chmod 644 /etc/passwd
chmod 640 /etc/shadow
chmod 644 /etc/group
chmod 640 /etc/gshadow
log "File permissions corrected"

# 6. Установка и настройка auditd
log "Installing auditd..."
apt install -y auditd audispd-plugins

# Мониторинг критичных файлов
auditctl -w /etc/passwd -p wa -k passwd_changes
auditctl -w /etc/shadow -p wa -k shadow_changes
auditctl -w /etc/sudoers -p wa -k sudoers_changes
auditctl -w /var/log/auth.log -p wa -k auth_log_changes

systemctl enable auditd
systemctl start auditd
log "Auditd configured"

# 7. Отключение неиспользуемых сервисов
log "Disabling unnecessary services..."
SERVICES_TO_DISABLE=(
    "bluetooth"
    "cups"
    "avahi-daemon"
)

for service in "${SERVICES_TO_DISABLE[@]}"; do
    if systemctl list-unit-files | grep -q "^$service"; then
        systemctl disable $service 2>/dev/null || true
        systemctl stop $service 2>/dev/null || true
    fi
done
log "Unnecessary services disabled"

# 8. Kernel hardening
log "Hardening kernel parameters..."
cat > /etc/sysctl.d/99-security.conf << 'EOF'
# IP Forwarding
net.ipv4.ip_forward = 0
net.ipv6.conf.all.forwarding = 0

# SYN cookies protection
net.ipv4.tcp_syncookies = 1

# Ignore ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0

# Ignore source routed packets
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0

# Ignore send redirects
net.ipv4.conf.all.send_redirects = 0

# Disable ICMP redirect acceptance
net.ipv4.conf.default.accept_redirects = 0

# Log Martians
net.ipv4.conf.all.log_martians = 1

# Increase system file descriptor limit
fs.file-max = 65535

# Protect against SYN flood attacks
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 5

# Disable IPv6 if not needed
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
EOF

sysctl -p /etc/sysctl.d/99-security.conf
log "Kernel parameters hardened"

log "Security hardening completed!"
```

**3. Ansible playbook для security hardening:**
```yaml
# playbooks/security_hardening.yml
---
- name: Security Hardening
  hosts: all
  become: yes
  
  vars:
    ssh_port: 22
    allowed_users: ["deploy", "admin"]
    fail2ban_bantime: 3600
    
  tasks:
    - name: Update all packages
      apt:
        upgrade: dist
        update_cache: yes
      when: ansible_os_family == "Debian"
    
    - name: Install security packages
      apt:
        name:
          - ufw
          - fail2ban
          - unattended-upgrades
          - auditd
          - aide
          - rkhunter
          - lynis
        state: present
    
    - name: Configure SSH
      template:
        src: templates/sshd_config.j2
        dest: /etc/ssh/sshd_config
        validate: '/usr/sbin/sshd -t -f %s'
      notify: restart sshd
    
    - name: Configure UFW default policies
      ufw:
        direction: "{{ item.direction }}"
        policy: "{{ item.policy }}"
      loop:
        - { direction: 'incoming', policy: 'deny' }
        - { direction: 'outgoing', policy: 'allow' }
    
    - name: Allow SSH through firewall
      ufw:
        rule: allow
        port: "{{ ssh_port }}"
        proto: tcp
    
    - name: Enable UFW
      ufw:
        state: enabled
    
    - name: Configure fail2ban
      template:
        src: templates/jail.local.j2
        dest: /etc/fail2ban/jail.local
      notify: restart fail2ban
    
    - name: Set correct file permissions
      file:
        path: "{{ item.path }}"
        mode: "{{ item.mode }}"
      loop:
        - { path: '/etc/passwd', mode: '0644' }
        - { path: '/etc/shadow', mode: '0640' }
        - { path: '/etc/group', mode: '0644' }
        - { path: '/etc/gshadow', mode: '0640' }
    
    - name: Configure kernel parameters
      sysctl:
        name: "{{ item.name }}"
        value: "{{ item.value }}"
        state: present
        reload: yes
      loop:
        - { name: 'net.ipv4.tcp_syncookies', value: '1' }
        - { name: 'net.ipv4.conf.all.accept_redirects', value: '0' }
        - { name: 'net.ipv4.conf.all.send_redirects', value: '0' }
        - { name: 'net.ipv4.conf.all.log_martians', value: '1' }
    
    - name: Run security audit
      command: lynis audit system --quick
      register: audit_result
      changed_when: false
    
    - name: Save audit report
      copy:
        content: "{{ audit_result.stdout }}"
        dest: "/var/log/security_audit_{{ ansible_date_time.date }}.log"
  
  handlers:
    - name: restart sshd
      service:
        name: sshd
        state: restarted
    
    - name: restart fail2ban
      service:
        name: fail2ban
        state: restarted
```

---

### Задача 14: Blue-Green Deployment с нулевым downtime
**Цель:** Реализовать стратегию деплоя без простоя сервиса

**Решение:**

**1. Скрипт Blue-Green деплоя:**
```bash
#!/bin/bash
# blue_green_deploy.sh

set -euo pipefail

APP_NAME="myapp"
BLUE_PORT=3000
GREEN_PORT=3001
HEALTH_CHECK_URL="http://localhost"
NGINX_UPSTREAM="/etc/nginx/conf.d/${APP_NAME}_upstream.conf"
DEPLOY_DIR="/opt/${APP_NAME}"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

error_exit() {
    log "ERROR: $1"
    exit 1
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
    
    error_exit "Health check failed on port $port after $max_attempts attempts"
}

# Деплой приложения
deploy_app() {
    local env=$1
    local port=$2
    
    log "Deploying to $env environment (port $port)..."
    
    # Остановить старую версию
    pm2 delete ${APP_NAME}-${env} 2>/dev/null || true
    
    # Обновить код (git pull, docker pull, etc.)
    cd $DEPLOY_DIR
    git pull origin main
    npm install --production
    
    # Запустить новую версию
    PORT=$port pm2 start app.js --name ${APP_NAME}-${env}
    
    # Подождать запуска
    sleep 5
    
    # Проверить здоровье
    health_check $port
}

# Переключить трафик
switch_traffic() {
    local target_env=$1
    local target_port=$2
    
    log "Switching traffic to $target_env environment..."
    
    # Создать новый upstream конфиг
    cat > $NGINX_UPSTREAM << EOF
upstream ${APP_NAME}_backend {
    server 127.0.0.1:${target_port} max_fails=3 fail_timeout=30s;
}
EOF
    
    # Проверить конфигурацию
    nginx -t || error_exit "Nginx configuration test failed"
    
    # Плавная перезагрузка
    nginx -s reload
    
    log "Traffic switched to $target_env"
}

# Откат
rollback() {
    local current_env=$1
    local previous_env=$2
    local previous_port=$3
    
    log "Rolling back from $current_env to $previous_env..."
    
    # Проверить что старая версия жива
    if ! health_check $previous_port; then
        error_exit "Previous environment is not healthy, manual intervention required"
    fi
    
    # Переключить обратно
    switch_traffic $previous_env $previous_port
    
    log "Rollback completed"
}

# Очистка старой среды
cleanup_old_env() {
    local env=$1
    
    log "Cleaning up $env environment..."
    pm2 delete ${APP_NAME}-${env} 2>/dev/null || true
}

# Основная логика
main() {
    local active_env=$(get_active_env)
    local target_env
    local target_port
    local previous_port
    
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
    
    # 1. Деплой в неактивную среду
    deploy_app $target_env $target_port
    
    # 2. Smoke tests на новой версии
    log "Running smoke tests..."
    if ! curl -sf "${HEALTH_CHECK_URL}:${target_port}/health" > /dev/null; then
        error_exit "Smoke tests failed"
    fi
    
    # 3. Переключить трафик
    switch_traffic $target_env $target_port
    
    # 4. Подождать и проверить
    sleep 10
    
    if ! health_check $target_port; then
        log "New version is unhealthy, rolling back..."
        rollback $target_env $active_env $previous_port
        error_exit "Deployment failed, rolled back"
    fi
    
    # 5. Очистить старую среду
    cleanup_old_env $active_env
    
    log "=== Deployment Completed Successfully ==="
    log "Active environment: $target_env"
    log "Port: $target_port"
}

# Запуск
main "$@"
```

**2. Kubernetes Blue-Green с использованием Services:**
```yaml
# blue-green-k8s.yml
---
# Blue Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
  labels:
    app: myapp
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: myapp
        image: myapp:v1.0
        ports:
        - containerPort: 3000
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 3
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"

---
# Green Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
  labels:
    app: myapp
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0
        ports:
        - containerPort: 3000
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 3
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"

---
# Production Service (initially pointing to blue)
apiVersion: v1
kind: Service
metadata:
  name: myapp-production
spec:
  type: LoadBalancer
  selector:
    app: myapp
    version: blue  # Change to 'green' to switch traffic
  ports:
  - port: 80
    targetPort: 3000

---
# Service for Blue environment
apiVersion: v1
kind: Service
metadata:
  name: myapp-blue
spec:
  selector:
    app: myapp
    version: blue
  ports:
  - port: 80
    targetPort: 3000

---
# Service for Green environment
apiVersion: v1
kind: Service
metadata:
  name: myapp-green
spec:
  selector:
    app: myapp
    version: green
  ports:
  - port: 80
    targetPort: 3000
```

**3. Скрипт для переключения в Kubernetes:**
```bash
#!/bin/bash
# k8s_blue_green_switch.sh

set -euo pipefail

APP_NAME="myapp"
NAMESPACE="default"
PRODUCTION_SERVICE="${APP_NAME}-production"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

# Получить текущую активную версию
get_active_version() {
    kubectl get service $PRODUCTION_SERVICE -n $NAMESPACE \
        -o jsonpath='{.spec.selector.version}'
}

# Проверить health deployments
check_deployment_health() {
    local version=$1
    local deployment="${APP_NAME}-${version}"
    
    log "Checking health of $deployment..."
    
    # Проверить что deployment готов
    kubectl rollout status deployment/$deployment -n $NAMESPACE --timeout=5m
    
    # Проверить что все поды ready
    local ready_replicas=$(kubectl get deployment $deployment -n $NAMESPACE \
        -o jsonpath='{.status.readyReplicas}')
    local desired_replicas=$(kubectl get deployment $deployment -n $NAMESPACE \
        -o jsonpath='{.spec.replicas}')
    
    if [ "$ready_replicas" != "$desired_replicas" ]; then
        log "ERROR: Only $ready_replicas/$desired_replicas replicas are ready"
        return 1
    fi
    
    log "$deployment is healthy ($ready_replicas/$desired_replicas replicas ready)"
    return 0
}

# Переключить трафик
switch_traffic() {
    local target_version=$1
    
    log "Switching traffic to $target_version version..."
    
    kubectl patch service $PRODUCTION_SERVICE -n $NAMESPACE \
        -p "{\"spec\":{\"selector\":{\"version\":\"$target_version\"}}}"
    
    log "Traffic switched to $target_version"
}

# Smoke tests
run_smoke_tests() {
    local service_url=$1
    
    log "Running smoke tests against $service_url..."
    
    # Получить IP сервиса
    local service_ip=$(kubectl get service $PRODUCTION_SERVICE -n $NAMESPACE \
        -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    
    # Простые smoke tests
    local tests_passed=0
    local tests_total=3
    
    # Test 1: Health check
    if curl -sf "http://${service_ip}/health" > /dev/null; then
        log "✓ Health check passed"
        tests_passed=$((tests_passed + 1))
    else
        log "✗ Health check failed"
    fi
    
    # Test 2: Response time
    local response_time=$(curl -sf -o /dev/null -w "%{time_total}" "http://${service_ip}/")
    if (( $(echo "$response_time < 2.0" | bc -l) )); then
        log "✓ Response time acceptable: ${response_time}s"
        tests_passed=$((tests_passed + 1))
    else
        log "✗ Response time too slow: ${response_time}s"
    fi
    
    # Test 3: HTTP status
    local http_status=$(curl -sf -o /dev/null -w "%{http_code}" "http://${service_ip}/")
    if [ "$http_status" = "200" ]; then
        log "✓ HTTP status OK: $http_status"
        tests_passed=$((tests_passed + 1))
    else
        log "✗ HTTP status failed: $http_status"
    fi
    
    if [ $tests_passed -eq $tests_total ]; then
        log "All smoke tests passed ($tests_passed/$tests_total)"
        return 0
    else
        log "Smoke tests failed ($tests_passed/$tests_total)"
        return 1
    fi
}

# Откат
rollback() {
    local previous_version=$1
    
    log "Rolling back to $previous_version version..."
    switch_traffic $previous_version
    log "Rollback completed"
}

# Главная функция
main() {
    local active_version=$(get_active_version)
    local target_version
    
    if [ "$active_version" = "blue" ]; then
        target_version="green"
    else
        target_version="blue"
    fi
    
    log "=== Kubernetes Blue-Green Deployment ==="
    log "Current active: $active_version"
    log "Target version: $target_version"
    
    # 1. Проверить что целевая версия здорова
    if ! check_deployment_health $target_version; then
        log "ERROR: Target deployment is not healthy"
        exit 1
    fi
    
    # 2. Переключить трафик
    switch_traffic $target_version
    
    # 3. Подождать стабилизации
    sleep 10
    
    # 4. Запустить smoke tests
    if ! run_smoke_tests $target_version; then
        log "Smoke tests failed, rolling back..."
        rollback $active_version
        exit 1
    fi
    
    # 5. Мониторинг после деплоя
    log "Monitoring new version for 60 seconds..."
    sleep 60
    
    if ! check_deployment_health $target_version; then
        log "New version became unhealthy, rolling back..."
        rollback $active_version
        exit 1
    fi
    
    log "=== Deployment Completed Successfully ==="
    log "Active version: $target_version"
    
    # 6. Опционально: уменьшить replicas старой версии
    read -p "Scale down $active_version deployment? (y/n) " -n 1 -r
    echo
    if [[ $REPLY =~ ^[Yy]$ ]]; then
        kubectl scale deployment ${APP_NAME}-${active_version} -n $NAMESPACE --replicas=0
        log "Scaled down $active_version deployment"
    fi
}

main "$@"
```

---

## 📚 ДОПОЛНИТЕЛЬНЫЕ ПРАКТИЧЕСКИЕ ЗАДАНИЯ

### Задача 15: Настройка CI/CD с тестированием
**Цель:** Полноценный pipeline с автоматическим тестированием

**GitHub Actions Pipeline:**
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  DOCKER_REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  lint:
    name: Code Linting
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run ESLint
        run: npm run lint
        
      - name: Run Prettier check
        run: npm run format:check

  test:
    name: Unit & Integration Tests
    runs-on: ubuntu-latest
    needs: lint
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run unit tests
        run: npm run test:unit
        env:
          DATABASE_URL: postgresql://postgres:testpass@localhost:5432/testdb
        
      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:testpass@localhost:5432/testdb
        
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json

  security:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: lint
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Run npm audit
        run: npm audit --audit-level=moderate
        
      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

  build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    needs: [test, security]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Log in to registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.DOCKER_REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=sha,prefix={{branch}}-
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://staging.myapp.com
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Set kubeconfig
        run: |
          mkdir -p ~/.kube
          echo "${{ secrets.KUBECONFIG_STAGING }}" | base64 -d > ~/.kube/config
      
      - name: Deploy to staging
        run: |
          kubectl set image deployment/myapp \
            myapp=${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}:develop \
            -n staging
          kubectl rollout status deployment/myapp -n staging
      
      - name: Run smoke tests
        run: |
          chmod +x ./scripts/smoke-tests.sh
          ./scripts/smoke-tests.sh https://staging.myapp.com

  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://myapp.com
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Set kubeconfig
        run: |
          mkdir -p ~/.kube
          echo "${{ secrets.KUBECONFIG_PRODUCTION }}" | base64 -d > ~/.kube/config
      
      - name: Blue-Green Deploy
        run: |
          chmod +x ./scripts/k8s_blue_green_switch.sh
          ./scripts/k8s_blue_green_switch.sh
      
      - name: Notify Slack
        if: always()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Production deployment ${{ job.status }}'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

---

### Задача 16: Observability Stack (Полный мониторинг)
**Цель:** Внедрить полноценный стек наблюдаемости

**Docker Compose для observability:**
```yaml
# docker-compose-observability.yml
version: '3.8'

services:
  # Metrics
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus/alerts.yml:/etc/prometheus/alerts.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.enable-lifecycle'
      - '--storage.tsdb.retention.time=30d'
    ports:
      - "9090:9090"
    networks:
      - monitoring
    restart: unless-stopped

  # Visualization
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
      - ./grafana/dashboards:/var/lib/grafana/dashboards
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD:-admin}
      - GF_USERS_ALLOW_SIGN_UP=false
      - GF_INSTALL_PLUGINS=grafana-piechart-panel
    ports:
      - "3000:3000"
    networks:
      - monitoring
    restart: unless-stopped
    depends_on:
      - prometheus

  # Logs
  loki:
    image: grafana/loki:latest
    container_name: loki
    volumes:
      - ./loki/loki-config.yml:/etc/loki/local-config.yaml
      - loki_data:/loki
    command: -config.file=/etc/loki/local-config.yaml
    ports:
      - "3100:3100"
    networks:
      - monitoring
    restart: unless-stopped

  # Log collector
  promtail:
    image: grafana/promtail:latest
    container_name: promtail
    volumes:
      - /var/log:/var/log:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - ./promtail/promtail-config.yml:/etc/promtail/config.yml
    command: -config.file=/etc/promtail/config.yml
    networks:
      - monitoring
    restart: unless-stopped

  # Traces
  tempo:
    image: grafana/tempo:latest
    container_name: tempo
    command: [ "-config.file=/etc/tempo.yaml" ]
    volumes:
      - ./tempo/tempo.yaml:/etc/tempo.yaml
      - tempo_data:/tmp/tempo
    ports:
      - "3200:3200"   # tempo
      - "4317:4317"   # otlp grpc
      - "4318:4318"   # otlp http
    networks:
      - monitoring
    restart: unless-stopped

  # Alerting
  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    volumes:
      - ./alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml
      - alertmanager_data:/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
    ports:
      - "9093:9093"
    networks:
      - monitoring
    restart: unless-stopped

  # Node exporter
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($|/)'
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    ports:
      - "9100:9100"
    networks:
      - monitoring
    restart: unless-stopped

  # cAdvisor for container metrics
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    privileged: true
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    ports:
      - "8080:8080"
    networks:
      - monitoring
    restart: unless-stopped

  # Blackbox exporter for endpoint monitoring
  blackbox-exporter:
    image: prom/blackbox-exporter:latest
    container_name: blackbox-exporter
    volumes:
      - ./blackbox/blackbox.yml:/etc/blackbox_exporter/config.yml
    command:
      - '--config.file=/etc/blackbox_exporter/config.yml'
    ports:
      - "9115:9115"
    networks:
      - monitoring
    restart: unless-stopped

volumes:
  prometheus_data:
  grafana_data:
  loki_data:
  tempo_data:
  alertmanager_data:

networks:
  monitoring:
    driver: bridge
```

**Prometheus конфигурация с сервис-дискавери:**
```yaml
# prometheus/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'production'
    replica: '1'

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

# Load rules
rule_files:
  - "alerts.yml"

# Scrape configurations
scrape_configs:
  # Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Node exporter
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
        labels:
          env: 'production'

  # cAdvisor
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']

  # Blackbox exporter for HTTP checks
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - https://myapp.com
          - https://api.myapp.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115

  # Docker service discovery (for Docker Swarm)
  - job_name: 'docker'
    dockerswarm_sd_configs:
      - host: unix:///var/run/docker.sock
        role: tasks
    relabel_configs:
      - source_labels: [__meta_dockerswarm_service_name]
        target_label: service

  # Kubernetes service discovery (if using K8s)
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
```

**Комплексные алерты:**
```yaml
# prometheus/alerts.yml
groups:
  - name: infrastructure
    interval: 30s
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
          component: system
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}% for 5 minutes"

      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 85
        for: 5m
        labels:
          severity: warning
          component: system
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is {{ $value }}%"

      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 < 15
        for: 5m
        labels:
          severity: critical
          component: system
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
          description: "Only {{ $value }}% disk space available"

      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
          component: availability
        annotations:
          summary: "Instance {{ $labels.instance }} is down"
          description: "{{ $labels.job }} on {{ $labels.instance }} has been down for more than 1 minute"

  - name: application
    interval: 30s
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
          component: application
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} requests/s"

      - alert: SlowResponseTime
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 2
        for: 5m
        labels:
          severity: warning
          component: performance
        annotations:
          summary: "Slow response time detected"
          description: "95th percentile response time is {{ $value }}s"

      - alert: DatabaseConnectionPoolExhausted
        expr: database_connection_pool_usage > 0.9
        for: 2m
        labels:
          severity: critical
          component: database
        annotations:
          summary: "Database connection pool nearly exhausted"
          description: "Connection pool usage is {{ $value }}%"

  - name: kubernetes
    interval: 30s
    rules:
      - alert: PodCrashLooping
        expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
        for: 5m
        labels:
          severity: warning
          component: kubernetes
        annotations:
          summary: "Pod {{ $labels.pod }} is crash looping"
          description: "Pod has restarted {{ $value }} times in the last 15 minutes"

      - alert: PodNotReady
        expr: kube_pod_status_ready{condition="false"} == 1
        for: 5m
        labels:
          severity: warning
          component: kubernetes
        annotations:
          summary: "Pod {{ $labels.pod }} is not ready"
          description: "Pod has been in non-ready state for 5 minutes"
```

**Alertmanager конфигурация:**
```yaml
# alertmanager/alertmanager.yml
global:
  resolve_timeout: 5m
  slack_api_url: '${SLACK_WEBHOOK_URL}'

# Templates
templates:
  - '/etc/alertmanager/templates/*.tmpl'

# Routes
route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'default'
  
  routes:
    # Critical alerts go to PagerDuty and Slack
    - match:
        severity: critical
      receiver: 'critical-alerts'
      continue: true
    
    # Warning alerts only to Slack
    - match:
        severity: warning
      receiver: 'warning-alerts'
    
    # Database alerts
    - match:
        component: database
      receiver: 'database-team'
    
    # During maintenance window, suppress all but critical
    - match:
        maintenance: 'true'
      receiver: 'null'
      match_re:
        severity: 'warning|info'

# Receivers
receivers:
  - name: 'default'
    slack_configs:
      - channel: '#alerts'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
        send_resolved: true

  - name: 'critical-alerts'
    pagerduty_configs:
      - service_key: '${PAGERDUTY_SERVICE_KEY}'
        description: '{{ .GroupLabels.alertname }}: {{ .CommonAnnotations.summary }}'
    slack_configs:
      - channel: '#critical-alerts'
        title: ':fire: CRITICAL: {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
        send_resolved: true
        color: 'danger'

  - name: 'warning-alerts'
    slack_configs:
      - channel: '#warnings'
        title: ':warning: {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
        send_resolved: true
        color: 'warning'

  - name: 'database-team'
    email_configs:
      - to: 'database-team@company.com'
        from: 'alertmanager@company.com'
        smarthost: 'smtp.company.com:587'
        auth_username: 'alertmanager@company.com'
        auth_password: '${SMTP_PASSWORD}'
        headers:
          Subject: 'Database Alert: {{ .GroupLabels.alertname }}'

  - name: 'null'

# Inhibition rules
inhibit_rules:
  # Inhibit warning if critical is firing
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']
```

---

### Задача 17: Автоматизация с Python для DevOps
**Цель:** Создать полезные Python скрипты для автоматизации

**1. Универсальный скрипт управления инфраструктурой:**
```python
#!/usr/bin/env python3
# infrastructure_manager.py

import argparse
import json
import subprocess
import sys
from dataclasses import dataclass
from typing import List, Dict
import logging

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)

@dataclass
class Server:
    hostname: str
    ip: str
    role: str
    environment: str

class InfrastructureManager:
    def __init__(self, inventory_file: str):
        self.inventory_file = inventory_file
        self.servers = self.load_inventory()
    
    def load_inventory(self) -> List[Server]:
        """Загрузить инвентарь серверов"""
        try:
            with open(self.inventory_file, 'r') as f:
                data = json.load(f)
            
            servers = []
            for server_data in data['servers']:
                servers.append(Server(**server_data))
            
            logger.info(f"Loaded {len(servers)} servers from inventory")
            return servers
        except Exception as e:
            logger.error(f"Failed to load inventory: {e}")
            sys.exit(1)
    
    def execute_command(self, server: Server, command: str) -> tuple:
        """Выполнить команду на сервере через SSH"""
        ssh_command = f"ssh -o StrictHostKeyChecking=no {server.ip} '{command}'"
        
        try:
            result = subprocess.run(
                ssh_command,
                shell=True,
                capture_output=True,
                text=True,
                timeout=30
            )
            return result.returncode, result.stdout, result.stderr
        except subprocess.TimeoutExpired:
            logger.error(f"Command timeout on {server.hostname}")
            return 1, "", "Timeout"
        except Exception as e:
            logger.error(f"Failed to execute on {server.hostname}: {e}")
            return 1, "", str(e)
    
    def health_check(self, environment: str = None):
        """Проверить здоровье серверов"""
        servers = self.filter_servers(environment=environment)
        
        logger.info(f"Running health check on {len(servers)} servers...")
        
        results = []
        for server in servers:
            checks = {
                'ping': self.check_ping(server),
                'disk': self.check_disk(server),
                'memory': self.check_memory(server),
                'cpu': self.check_cpu(server),
                'services': self.check_services(server)
            }
            
            results.append({
                'server': server.hostname,
                'ip': server.ip,
                'checks': checks,
                'status': 'healthy' if all(checks.values()) else 'unhealthy'
            })
        
        self.print_health_report(results)
        return results
    
    def check_ping(self, server: Server) -> bool:
        """Проверить доступность сервера"""
        result = subprocess.run(
            f"ping -c 1 -W 2 {server.ip}",
            shell=True,
            capture_output=True
        )
        return result.returncode == 0
    
    def check_disk(self, server: Server) -> bool:
        """Проверить дисковое пространство"""
        returncode, stdout, _ = self.execute_command(
            server,
            "df -h / | tail -1 | awk '{print $5}' | sed 's/%//'"
        )
        
        if returncode == 0 and stdout.strip():
            usage = int(stdout.strip())
            return usage < 85
        return False
    
    def check_memory(self, server: Server) -> bool:
        """Проверить использование памяти"""
        returncode, stdout, _ = self.execute_command(
            server,
            "free | grep Mem | awk '{print int($3/$2 * 100)}'"
        )
        
        if returncode == 0 and stdout.strip():
            usage = int(stdout.strip())
            return usage < 90
        return False
    
    def check_cpu(self, server: Server) -> bool:
        """Проверить загрузку CPU"""
        returncode, stdout, _ = self.execute_command(
            server,
            "top -bn1 | grep 'Cpu(s)' | awk '{print 100 - $8}'"
        )
        
        if returncode == 0 and stdout.strip():
            usage = float(stdout.strip())
            return usage < 80
        return False
    
    def check_services(self, server: Server) -> bool:
        """Проверить критичные сервисы"""
        services = ['nginx', 'postgresql', 'docker']
        
        for service in services:
            returncode, _, _ = self.execute_command(
                server,
                f"systemctl is-active {service}"
            )
            if returncode != 0:
                logger.warning(f"{service} is not running on {server.hostname}")
        
        return True  # Возвращаем True даже если какие-то сервисы не запущены
    
    def print_health_report(self, results: List[Dict]):
        """Вывести отчет о здоровье"""
        print("\n" + "=" * 80)
        print("HEALTH CHECK REPORT")
        print("=" * 80 + "\n")
        
        for result in results:
            status_icon = "✓" if result['status'] == 'healthy' else "✗"
            print(f"{status_icon} {result['server']} ({result['ip']}) - {result['status'].upper()}")
            
            for check, passed in result['checks'].items():
                check_icon = "  ✓" if passed else "  ✗"
                print(f"{check_icon} {check}")
            print()
    
    def deploy_application(self, environment: str, version: str):
        """Деплой приложения"""
        servers = self.filter_servers(environment=environment, role='webserver')
        
        logger.info(f"Deploying version {version} to {environment}...")
        
        for server in servers:
            logger.info(f"Deploying to {server.hostname}...")
            
            commands = [
                f"cd /opt/myapp && git fetch",
                f"cd /opt/myapp && git checkout {version}",
                "cd /opt/myapp && npm install --production",
                "pm2 restart myapp"
            ]
            
            for cmd in commands:
                returncode, stdout, stderr = self.execute_command(server, cmd)
                if returncode != 0:
                    logger.error(f"Failed on {server.hostname}: {stderr}")
                    return False
            
            logger.info(f"✓ Deployed to {server.hostname}")
        
        logger.info("Deployment completed successfully!")
        return True
    
    def backup_databases(self, environment: str):
        """Создать бэкапы баз данных"""
        servers = self.filter_servers(environment=environment, role='database')
        
        logger.info(f"Starting database backup for {environment}...")
        
        for server in servers:
            logger.info(f"Backing up database on {server.hostname}...")
            
            backup_cmd = (
                "sudo -u postgres pg_dumpall | "
                "gzip > /backup/postgres_$(date +%Y%m%d_%H%M%S).sql.gz"
            )
            
            returncode, stdout, stderr = self.execute_command(server, backup_cmd)
            
            if returncode == 0:
                logger.info(f"✓ Backup completed on {server.hostname}")
            else:
                logger.error(f"Backup failed on {server.hostname}: {stderr}")
        
        logger.info("Database backups completed!")
    
    def filter_servers(self, environment: str = None, role: str = None) -> List[Server]:
        """Фильтровать серверы по критериям"""
        filtered = self.servers
        
        if environment:
            filtered = [s for s in filtered if s.environment == environment]
        
        if role:
            filtered = [s for s in filtered if s.role == role]
        
        return filtered
    
    def list_servers(self, environment: str = None):
        """Показать список серверов"""
        servers = self.filter_servers(environment=environment)
        
        print("\n" + "=" * 80)
        print(f"SERVERS{' (' + environment.upper() + ')' if environment else ''}")
        print("=" * 80 + "\n")
        
        for server in servers:
            print(f"• {server.hostname:20} {server.ip:15} {server.role:15} {server.environment}")
        
        print(f"\nTotal: {len(servers)} servers\n")

def main():
    parser = argparse.ArgumentParser(description='Infrastructure Management Tool')
    parser.add_argument('--inventory', default='inventory.json', help='Inventory file')
    
    subparsers = parser.add_subparsers(dest='command', help='Commands')
    
    # List command
    list_parser = subparsers.add_parser('list', help='List servers')
    list_parser.add_argument('--env', help='Filter by environment')
    
    # Health check command
    health_parser = subparsers.add_parser('health', help='Run health check')
    health_parser.add_argument('--env', help='Filter by environment')
    
    # Deploy command
    deploy_parser = subparsers.add_parser('deploy', help='Deploy application')
    deploy_parser.add_argument('--env', required=True, help='Environment')
    deploy_parser.add_argument('--version', required=True, help='Version/tag to deploy')
    
    # Backup command
    backup_parser = subparsers.add_parser('backup', help='Backup databases')
    backup_parser.add_argument('--env', required=True, help='Environment')
    
    args = parser.parse_args()
    
    if not args.command:
        parser.print_help()
        sys.exit(1)
    
    manager = InfrastructureManager(args.inventory)
    
    if args.command == 'list':
        manager.list_servers(args.env)
    
    elif args.command == 'health':
        manager.health_check(args.env)
    
    elif args.command == 'deploy':
        manager.deploy_application(args.env, args.version)
    
    elif args.command == 'backup':
        manager.backup_databases(args.env)

if __name__ == '__main__':
    main()
```

**Пример inventory.json:**
```json
{
  "servers": [
    {
      "hostname": "web1.prod",
      "ip": "192.168.1.10",
      "role": "webserver",
      "environment": "production"
    },
    {
      "hostname": "web2.prod",
      "ip": "192.168.1.11",
      "role": "webserver",
      "environment": "production"
    },
    {
      "hostname": "db1.prod",
      "ip": "192.168.1.20",
      "role": "database",
      "environment": "production"
    },
    {
      "hostname": "web1.staging",
      "ip": "192.168.1.30",
      "role": "webserver",
      "environment": "staging"
    }
  ]
}
```

**Использование:**
```bash
# Список всех серверов
python3 infrastructure_manager.py list

# Список production серверов
python3 infrastructure_manager.py list --env production

# Health check production
python3 infrastructure_manager.py health --env production

# Деплой в staging
python3 infrastructure_manager.py deploy --env staging --version v1.2.3

# Бэкап production БД
python3 infrastructure_manager.py backup --env production
```

---

## 🎯 ПРАКТИЧЕСКИЕ УПРАЖНЕНИЯ ДЛЯ ТРЕНИРОВКИ

### Упражнение 1: Настройка локального Kubernetes на Proxmox
```bash
#!/bin/bash
# Создать 3 VM в Proxmox для K8s кластера

# Master node
qm create 200 --name k8s-master --memory 4096 --cores 2 --net0 virtio,bridge=vmbr0
qm set 200 --scsihw virtio-scsi-pci --scsi0 local-lvm:32
qm set 200 --ide2 local:iso/ubuntu-22.04-server.iso,media=cdrom
qm set 200 --boot order=scsi0;ide2

# Worker nodes
for i in 1 2; do
  qm create $((200 + i)) --name k8s-worker$i --memory 4096 --cores 2 --net0 virtio,bridge=vmbr0
  qm set $((200 + i)) --scsihw virtio-scsi-pci --scsi0 local-lvm:32
  qm set $((200 + i)) --ide2 local:iso/ubuntu-22.04-server.iso,media=cdrom
  qm set $((200 + i)) --boot order=scsi0;ide2
done
```

### Упражнение 2: Создать полный CI/CD для Python приложения
**Задача:**
1. Создать простое Flask приложение
2. Написать unit тесты
3. Настроить GitHub Actions
4. Автоматический деплой в Docker
5. Blue-Green deployment

### Упражнение 3: Настроить полный мониторинг стек
**Задача:**
1. Развернуть Prometheus + Grafana в Docker
2. Настроить мониторинг хоста (node-exporter)
3. Создать алерты для CPU, RAM, Disk
4. Настроить уведомления в Telegram/Slack
5. Создать дашборд в Grafana

### Упражнение 4: Автоматизация бэкапов
**Задача:**
1. Создать скрипт для бэкапа PostgreSQL
2. Загружать бэкапы в MinIO/S3
3. Ротация бэкапов (хранить последние 7 дней)
4. Тест восстановления
5. Уведомления о статусе

---

## 📖 РЕКОМЕНДАЦИИ ПО ОБУЧЕНИЮ

### Для Junior DevOps:
1. **Базовые навыки:**
   - Bash scripting (обязательно)
   - Git (базовые команды)
   - Linux администрирование
   - Docker основы
   - Базовый networking

2. **Практика:**
   - Поднять 2-3 VM в Proxmox
   - Настроить Nginx reverse proxy
   - Написать 10+ bash скриптов
   - Настроить базовый мониторинг
   - Создать простой CI/CD pipeline

3. **Проекты для практики:**
   - Веб-сервер с автоматическим деплоем
   - Система бэкапов с уведомлениями
   - Базовый мониторинг с алертами

### Для Middle DevOps:
1. **Продвинутые навыки:**
   - Terraform/Infrastructure as Code
   - Ansible/Configuration Management
   - Kubernetes basics
   - CI/CD (GitLab CI, GitHub Actions)
   - Python для автоматизации

2. **Практика:**
   - Написать Terraform модули
   - Настроить K8s кластер
   - Создать Ansible playbooks
   - GitOps с ArgoCD
   - Observability stack (Prometheus, Grafana, Loki)

3. **Проекты для практики:**
   - Multi-region infrastructure
   - HA PostgreSQL cluster
   - Full observability stack
   - GitOps workflow

### Для Senior DevOps:
1. **Экспертные навыки:**
   - Архитектура cloud-native приложений
   - Security & Compliance
   - Disaster Recovery planning
   - Performance optimization
   - Team leadership

2. **Практика:**
   - Проектирование отказоустойчивых систем
   - Миграция legacy систем
   - Cost optimization
   - SRE practices
   - Mentoring junior/middle

3. **Проекты для практики:**
   - Полная миграция в cloud
   - Disaster recovery система
   - Platform engineering
   - Security hardening

---

## 🔧 ПОЛЕЗНЫЕ ИНСТРУМЕНТЫ

### Обязательные инструменты:
- **Terminal:** tmux, zsh с oh-my-zsh
- **Version Control:** git, tig
- **IaC:** Terraform, Ansible, Packer
- **Containers:** Docker, docker-compose, Podman
- **Orchestration:** Kubernetes, K3s, Helm
- **CI/CD:** Jenkins, GitHub Actions, GitLab CI
- **Monitoring:** Prometheus, Grafana, Loki
- **Cloud CLI:** aws-cli, azure-cli, gcloud

### Полезные утилиты:
- **jq** - парсинг JSON
- **yq** - парсинг YAML
- **httpie** - HTTP клиент
- **k9s** - Kubernetes UI в терминале
- **lazydocker** - Docker UI в терминале
- **dive** - анализ Docker образов
- **trivy** - сканер уязвимостей

---

## 📝 ЧЕКЛИСТ ДЛЯ СОБЕСЕДОВАНИЯ

### Junior:
- ✅ Знание Linux команд
- ✅ Bash scripting
- ✅ Git basics
- ✅ Docker basics
- ✅ Basic networking
- ✅ CI/CD concepts

### Middle:
- ✅ Все из Junior +
- ✅ Terraform/Ansible
- ✅ Kubernetes basics
- ✅ Monitoring & Logging
- ✅ Python scripting
- ✅ Security basics

### Senior:
- ✅ Все из Middle +
- ✅ Architecture design
- ✅ HA & Disaster Recovery
- ✅ Performance tuning
- ✅ Cloud platforms (AWS/GCP/Azure)
- ✅ Cost optimization
- ✅ Team leadership

---

## 🎓 РЕСУРСЫ ДЛЯ ОБУЧЕНИЯ

### Документация:
- [Kubernetes Docs](https://kubernetes.io/docs/)
- [Terraform Docs](https://www.terraform.io/docs)
- [Ansible Docs](https://docs.ansible.com/)
- [Docker Docs](https://docs.docker.com/)
- [Prometheus Docs](https://prometheus.io/docs/)

### Практика:
- [KodeKloud](https://kodekloud.com/) - интерактивные лабы
- [Play with Docker](https://labs.play-with-docker.com/)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)
- [Katacoda](https://www.katacoda.com/) (архив)

### YouTube каналы:
- TechWorld with Nana
- DevOps Toolkit
- Just me and Opensource
- Cloud Advocate

---

## ✅ ЗАКЛЮЧЕНИЕ

Этот сборник задач покрывает реальные сценарии работы DevOps инженера от Junior до Senior уровня. 

**Важно:**
1. Все задачи можно выполнить на локальной машине с Proxmox или просто Linux desktop
2. Начинайте с простых задач и постепенно переходите к сложным
3. Документируйте все что делаете
4. Создавайте Git репозиторий для всех ваших скриптов
5. Практика важнее теории - делайте, а не только читайте

**Следующие шаги:**
1. Выберите задачу соответствующую вашему уровню
2. Создайте VM в Proxmox (или используйте локальный Linux)
3. Решите задачу
4. Задокументируйте решение
5. Переходите к следующей задаче

Удачи в изучении DevOps! 🚀

listen postgres-replicas
    bind *:5001
    option httpchk GET /replica
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server pg1 192.168.1.10:5432 maxconn 100 check port 8008
    server pg2 192.168.1.11:5432 maxconn 100 check port 8008
    server pg3 192.168.1.12:5432 maxconn 100 check port 8008