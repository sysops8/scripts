# 100 Bash скриптов для DevOps инженера

## 🔧 Системное администрирование

### 1. Проверка использования диска
```bash
#!/bin/bash
# Проверка использования диска с предупреждением

THRESHOLD=80

df -h | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{ print $5 " " $1 }' | while read output; do
    usage=$(echo $output | awk '{print $1}' | sed 's/%//g')
    partition=$(echo $output | awk '{print $2}')
    
    if [ $usage -ge $THRESHOLD ]; then
        echo "⚠️  ВНИМАНИЕ: Раздел $partition заполнен на ${usage}%"
    else
        echo "✓ $partition: ${usage}%"
    fi
done
```

### 2. Мониторинг CPU и RAM
```bash
#!/bin/bash
# Мониторинг системных ресурсов

echo "=== Системные ресурсы ==="
echo "CPU Usage:"
top -bn1 | grep "Cpu(s)" | awk '{print "  User: " $2 ", System: " $4 ", Idle: " $8}'

echo ""
echo "Memory Usage:"
free -h | awk 'NR==2{printf "  Used: %s/%s (%.2f%%)\n", $3,$2,$3*100/$2}'

echo ""
echo "Load Average:"
uptime | awk -F'load average:' '{print "  " $2}'
```

### 3. Проверка работающих процессов
```bash
#!/bin/bash
# Проверка запущенных процессов

check_process() {
    local process_name=$1
    
    if pgrep -x "$process_name" > /dev/null; then
        echo "✓ $process_name запущен (PID: $(pgrep -x $process_name))"
        return 0
    else
        echo "✗ $process_name не найден"
        return 1
    fi
}

check_process "nginx"
check_process "mysql"
check_process "redis-server"
```

### 4. Автоматический restart сервиса
```bash
#!/bin/bash
# Перезапуск сервиса с проверкой статуса

SERVICE_NAME=$1

if [ -z "$SERVICE_NAME" ]; then
    echo "Использование: $0 <service_name>"
    exit 1
fi

echo "Перезапуск $SERVICE_NAME..."
sudo systemctl restart $SERVICE_NAME

if sudo systemctl is-active --quiet $SERVICE_NAME; then
    echo "✓ $SERVICE_NAME успешно перезапущен"
    sudo systemctl status $SERVICE_NAME --no-pager -l
else
    echo "✗ Ошибка перезапуска $SERVICE_NAME"
    sudo journalctl -u $SERVICE_NAME -n 20 --no-pager
    exit 1
fi
```

### 5. Проверка открытых портов
```bash
#!/bin/bash
# Сканирование открытых портов

check_port() {
    local host=$1
    local port=$2
    
    timeout 2 bash -c "</dev/tcp/$host/$port" 2>/dev/null
    
    if [ $? -eq 0 ]; then
        echo "✓ Порт $port открыт на $host"
        return 0
    else
        echo "✗ Порт $port закрыт на $host"
        return 1
    fi
}

# Проверка популярных портов
check_port "localhost" 80
check_port "localhost" 443
check_port "localhost" 22
check_port "localhost" 3306
```

### 6. Ротация логов с архивацией
```bash
#!/bin/bash
# Ротация и архивация логов

LOG_FILE="/var/log/application.log"
MAX_SIZE_MB=100
ARCHIVE_DIR="/var/log/archive"

mkdir -p "$ARCHIVE_DIR"

if [ -f "$LOG_FILE" ]; then
    size=$(du -m "$LOG_FILE" | cut -f1)
    
    if [ $size -gt $MAX_SIZE_MB ]; then
        timestamp=$(date +%Y%m%d_%H%M%S)
        archive_name="${ARCHIVE_DIR}/application_${timestamp}.log.gz"
        
        gzip -c "$LOG_FILE" > "$archive_name"
        > "$LOG_FILE"  # Очистка оригинала
        
        echo "✓ Лог заархивирован: $archive_name"
        
        # Удаление архивов старше 30 дней
        find "$ARCHIVE_DIR" -name "*.log.gz" -mtime +30 -delete
    else
        echo "Размер лога: ${size}MB (порог: ${MAX_SIZE_MB}MB)"
    fi
fi
```

### 7. Парсинг логов Nginx для поиска ошибок
```bash
#!/bin/bash
# Анализ логов Nginx

LOG_FILE="/var/log/nginx/access.log"
ERROR_LOG="/var/log/nginx/error.log"

echo "=== TOP 10 IP адресов ==="
awk '{print $1}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10

echo ""
echo "=== Статусы 4xx ==="
awk '$9 ~ /^4/ {print $9}' "$LOG_FILE" | sort | uniq -c | sort -rn

echo ""
echo "=== Статусы 5xx ==="
awk '$9 ~ /^5/ {print $9}' "$LOG_FILE" | sort | uniq -c | sort -rn

echo ""
echo "=== Последние ошибки ==="
tail -20 "$ERROR_LOG"
```

### 8. Мониторинг изменений файлов
```bash
#!/bin/bash
# Отслеживание изменений конфигурационных файлов

FILE_TO_WATCH="/etc/nginx/nginx.conf"
CHECK_INTERVAL=5

if [ ! -f "$FILE_TO_WATCH" ]; then
    echo "Файл $FILE_TO_WATCH не найден"
    exit 1
fi

last_checksum=$(md5sum "$FILE_TO_WATCH" | awk '{print $1}')

echo "Мониторинг $FILE_TO_WATCH (Ctrl+C для выхода)"

while true; do
    sleep $CHECK_INTERVAL
    current_checksum=$(md5sum "$FILE_TO_WATCH" | awk '{print $1}')
    
    if [ "$current_checksum" != "$last_checksum" ]; then
        echo "⚠️  $(date): Файл $FILE_TO_WATCH изменён!"
        logger "ALERT: $FILE_TO_WATCH was modified"
        last_checksum=$current_checksum
    fi
done
```

### 9. Очистка старых файлов
```bash
#!/bin/bash
# Очистка старых файлов и логов

DIRS_TO_CLEAN=(
    "/tmp"
    "/var/log"
    "/var/cache"
)

DAYS_OLD=30

for dir in "${DIRS_TO_CLEAN[@]}"; do
    echo "Очистка $dir (файлы старше $DAYS_OLD дней)..."
    
    find "$dir" -type f -mtime +$DAYS_OLD -print -delete 2>/dev/null | while read file; do
        echo "  Удалён: $file"
    done
done

echo "✓ Очистка завершена"
```

### 10. Проверка состояния служб
```bash
#!/bin/bash
# Проверка критичных сервисов

SERVICES=(
    "nginx"
    "mysql"
    "redis"
    "docker"
)

echo "=== Статус сервисов ==="

for service in "${SERVICES[@]}"; do
    if systemctl is-active --quiet "$service"; then
        echo "✓ $service: активен"
    else
        echo "✗ $service: неактивен"
        echo "  Попытка перезапуска..."
        sudo systemctl start "$service"
    fi
done
```

---

## 🐳 Docker & Containers

### 11. Список всех контейнеров с форматированием
```bash
#!/bin/bash
# Красивый вывод Docker контейнеров

echo "=== Docker Containers ==="
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}" | \
    awk 'NR==1 {print "\033[1m" $0 "\033[0m"; next} {print}'

echo ""
echo "=== Resource Usage ==="
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

### 12. Очистка Docker системы
```bash
#!/bin/bash
# Полная очистка Docker

echo "Текущее использование диска:"
docker system df

read -p "Выполнить очистку? (y/n) " -n 1 -r
echo

if [[ $REPLY =~ ^[Yy]$ ]]; then
    echo "Удаление остановленных контейнеров..."
    docker container prune -f
    
    echo "Удаление неиспользуемых образов..."
    docker image prune -a -f
    
    echo "Удаление неиспользуемых volumes..."
    docker volume prune -f
    
    echo "Удаление неиспользуемых сетей..."
    docker network prune -f
    
    echo ""
    echo "Использование после очистки:"
    docker system df
fi
```

### 13. Проверка здоровья контейнеров
```bash
#!/bin/bash
# Мониторинг health status контейнеров

echo "=== Container Health Check ==="

docker ps --format '{{.Names}}' | while read container; do
    health=$(docker inspect --format='{{.State.Health.Status}}' "$container" 2>/dev/null)
    status=$(docker inspect --format='{{.State.Status}}' "$container")
    
    if [ "$status" != "running" ]; then
        echo "✗ $container: $status"
    elif [ "$health" == "unhealthy" ]; then
        echo "⚠️  $container: unhealthy"
    elif [ "$health" == "healthy" ]; then
        echo "✓ $container: healthy"
    else
        echo "◦ $container: running (no healthcheck)"
    fi
done
```

### 14. Backup Docker volumes
```bash
#!/bin/bash
# Бэкап Docker volumes

VOLUME_NAME=$1
BACKUP_DIR="/backup/docker-volumes"

if [ -z "$VOLUME_NAME" ]; then
    echo "Использование: $0 <volume_name>"
    echo "Доступные volumes:"
    docker volume ls --format '{{.Name}}'
    exit 1
fi

mkdir -p "$BACKUP_DIR"
timestamp=$(date +%Y%m%d_%H%M%S)
backup_file="${BACKUP_DIR}/${VOLUME_NAME}_${timestamp}.tar.gz"

docker run --rm \
    -v "$VOLUME_NAME:/source:ro" \
    -v "$BACKUP_DIR:/backup" \
    alpine \
    tar czf "/backup/$(basename $backup_file)" -C /source .

echo "✓ Volume $VOLUME_NAME сохранён в $backup_file"
```

### 15. Автоматический restart упавших контейнеров
```bash
#!/bin/bash
# Перезапуск остановленных контейнеров

echo "Поиск остановленных контейнеров..."

docker ps -a --filter "status=exited" --format '{{.Names}}' | while read container; do
    echo "Перезапуск: $container"
    docker start "$container"
    
    if [ $? -eq 0 ]; then
        echo "✓ $container перезапущен"
    else
        echo "✗ Ошибка перезапуска $container"
    fi
done
```

### 16. Мониторинг логов Docker
```bash
#!/bin/bash
# Мониторинг логов контейнеров на наличие ошибок

CONTAINER=$1
LINES=${2:-100}

if [ -z "$CONTAINER" ]; then
    echo "Использование: $0 <container_name> [lines]"
    exit 1
fi

echo "=== Анализ логов $CONTAINER (последние $LINES строк) ==="

docker logs --tail $LINES "$CONTAINER" 2>&1 | \
    grep -iE "error|exception|fatal|warning" | \
    tail -20

echo ""
echo "=== Статистика ошибок ==="
docker logs --tail 1000 "$CONTAINER" 2>&1 | \
    grep -ioE "error|exception|fatal|warning" | \
    sort | uniq -c | sort -rn
```

### 17. Docker Compose управление
```bash
#!/bin/bash
# Управление Docker Compose проектами

PROJECT_DIR=${1:-.}
ACTION=${2:-status}

cd "$PROJECT_DIR" || exit 1

case $ACTION in
    up)
        echo "Запуск сервисов..."
        docker-compose up -d
        docker-compose ps
        ;;
    down)
        echo "Остановка сервисов..."
        docker-compose down
        ;;
    restart)
        echo "Перезапуск сервисов..."
        docker-compose restart
        docker-compose ps
        ;;
    logs)
        docker-compose logs --tail=100 -f
        ;;
    status|*)
        docker-compose ps
        ;;
esac
```

### 18. Проверка обновлений образов
```bash
#!/bin/bash
# Проверка наличия обновлений Docker образов

echo "=== Проверка обновлений образов ==="

docker images --format '{{.Repository}}:{{.Tag}}' | grep -v '<none>' | while read image; do
    echo "Проверка $image..."
    docker pull "$image" > /dev/null 2>&1
    
    if [ $? -eq 0 ]; then
        echo "✓ $image обновлён"
    else
        echo "◦ $image актуален"
    fi
done
```

### 19. Экспорт и импорт образов
```bash
#!/bin/bash
# Экспорт Docker образов для переноса

ACTION=$1
IMAGE=$2
FILE=$3

case $ACTION in
    export)
        if [ -z "$IMAGE" ] || [ -z "$FILE" ]; then
            echo "Использование: $0 export <image> <file.tar>"
            exit 1
        fi
        echo "Экспорт $IMAGE..."
        docker save -o "$FILE" "$IMAGE"
        gzip "$FILE"
        echo "✓ Образ сохранён в ${FILE}.gz"
        ;;
    import)
        if [ -z "$FILE" ]; then
            echo "Использование: $0 import <file.tar.gz>"
            exit 1
        fi
        echo "Импорт из $FILE..."
        gunzip -c "$FILE" | docker load
        echo "✓ Образ импортирован"
        ;;
    *)
        echo "Использование: $0 {export|import}"
        exit 1
        ;;
esac
```

### 20. Мониторинг Docker events
```bash
#!/bin/bash
# Мониторинг событий Docker в реальном времени

echo "Мониторинг Docker events (Ctrl+C для выхода)..."
echo ""

docker events --format '{{.Time}} | {{.Type}} | {{.Action}} | {{.Actor.Attributes.name}}' | \
while read line; do
    echo "[$(date +'%H:%M:%S')] $line"
    
    # Логирование критичных событий
    if echo "$line" | grep -qE "die|kill|stop"; then
        logger "DOCKER_EVENT: $line"
    fi
done
```

---

## ☸️ Kubernetes

### 21. Список всех подов с статусами
```bash
#!/bin/bash
# Просмотр подов Kubernetes

NAMESPACE=${1:-default}

echo "=== Pods в namespace: $NAMESPACE ==="
kubectl get pods -n "$NAMESPACE" -o custom-columns=\
NAME:.metadata.name,\
STATUS:.status.phase,\
RESTARTS:.status.containerStatuses[0].restartCount,\
AGE:.metadata.creationTimestamp
```

### 22. Проверка проблемных подов
```bash
#!/bin/bash
# Поиск подов с проблемами

NAMESPACE=${1:-default}

echo "=== Проблемные поды в $NAMESPACE ==="

kubectl get pods -n "$NAMESPACE" --field-selector=status.phase!=Running,status.phase!=Succeeded

echo ""
echo "=== Поды с большим количеством перезапусков ==="

kubectl get pods -n "$NAMESPACE" -o json | \
jq -r '.items[] | select(.status.containerStatuses[0].restartCount > 3) | 
    "\(.metadata.name): \(.status.containerStatuses[0].restartCount) restarts"'
```

### 23. Получение логов пода
```bash
#!/bin/bash
# Удобный просмотр логов Kubernetes

POD=$1
NAMESPACE=${2:-default}
LINES=${3:-100}

if [ -z "$POD" ]; then
    echo "Использование: $0 <pod_name> [namespace] [lines]"
    kubectl get pods -A
    exit 1
fi

# Если несколько контейнеров
containers=$(kubectl get pod "$POD" -n "$NAMESPACE" -o jsonpath='{.spec.containers[*].name}')
container_count=$(echo "$containers" | wc -w)

if [ $container_count -gt 1 ]; then
    echo "Контейнеры в поде: $containers"
    read -p "Выберите контейнер: " container
    kubectl logs -n "$NAMESPACE" "$POD" -c "$container" --tail="$LINES" -f
else
    kubectl logs -n "$NAMESPACE" "$POD" --tail="$LINES" -f
fi
```

### 24. Масштабирование deployment
```bash
#!/bin/bash
# Масштабирование Kubernetes deployment

DEPLOYMENT=$1
REPLICAS=$2
NAMESPACE=${3:-default}

if [ -z "$DEPLOYMENT" ] || [ -z "$REPLICAS" ]; then
    echo "Использование: $0 <deployment> <replicas> [namespace]"
    echo "Текущие deployments:"
    kubectl get deployments -n "$NAMESPACE"
    exit 1
fi

echo "Масштабирование $DEPLOYMENT до $REPLICAS реплик..."
kubectl scale deployment "$DEPLOYMENT" --replicas="$REPLICAS" -n "$NAMESPACE"

echo "Ожидание готовности..."
kubectl rollout status deployment "$DEPLOYMENT" -n "$NAMESPACE"

echo "✓ Масштабирование завершено"
kubectl get deployment "$DEPLOYMENT" -n "$NAMESPACE"
```

### 25. Проверка состояния нод
```bash
#!/bin/bash
# Мониторинг состояния узлов кластера

echo "=== Node Status ==="
kubectl get nodes -o wide

echo ""
echo "=== Node Resources ==="
kubectl top nodes

echo ""
echo "=== Node Conditions ==="
kubectl get nodes -o json | \
jq -r '.items[] | "\(.metadata.name):", 
    (.status.conditions[] | "  \(.type): \(.status)")' | \
grep -A5 "^"
```

### 26. Дамп конфигураций всех ресурсов
```bash
#!/bin/bash
# Backup конфигураций Kubernetes

BACKUP_DIR="k8s-backup-$(date +%Y%m%d_%H%M%S)"
NAMESPACE=${1:-default}

mkdir -p "$BACKUP_DIR"

echo "Сохранение конфигураций из namespace: $NAMESPACE"

resources=("deployments" "services" "configmaps" "secrets" "ingresses" "statefulsets")

for resource in "${resources[@]}"; do
    echo "Экспорт $resource..."
    kubectl get "$resource" -n "$NAMESPACE" -o yaml > "$BACKUP_DIR/${resource}.yaml"
done

tar czf "${BACKUP_DIR}.tar.gz" "$BACKUP_DIR"
rm -rf "$BACKUP_DIR"

echo "✓ Backup сохранён: ${BACKUP_DIR}.tar.gz"
```

### 27. Выполнение команд в поде
```bash
#!/bin/bash
# Exec в под Kubernetes

POD=$1
NAMESPACE=${2:-default}
COMMAND=${3:-/bin/sh}

if [ -z "$POD" ]; then
    echo "Использование: $0 <pod_name> [namespace] [command]"
    kubectl get pods -n "$NAMESPACE"
    exit 1
fi

echo "Подключение к $POD..."
kubectl exec -it -n "$NAMESPACE" "$POD" -- $COMMAND
```

### 28. Мониторинг событий кластера
```bash
#!/bin/bash
# Отслеживание событий Kubernetes

NAMESPACE=${1:---all-namespaces}

if [ "$NAMESPACE" == "--all-namespaces" ]; then
    echo "Мониторинг событий всех namespaces..."
    kubectl get events --all-namespaces --watch
else
    echo "Мониторинг событий namespace: $NAMESPACE..."
    kubectl get events -n "$NAMESPACE" --watch
fi
```

### 29. Проверка использования ресурсов
```bash
#!/bin/bash
# Анализ использования ресурсов в кластере

NAMESPACE=${1:-default}

echo "=== Top Pods by CPU ==="
kubectl top pods -n "$NAMESPACE" --sort-by=cpu | head -10

echo ""
echo "=== Top Pods by Memory ==="
kubectl top pods -n "$NAMESPACE" --sort-by=memory | head -10

echo ""
echo "=== Resource Requests vs Limits ==="
kubectl get pods -n "$NAMESPACE" -o json | \
jq -r '.items[] | "\(.metadata.name):",
    "  CPU Request: \(.spec.containers[0].resources.requests.cpu // "none")",
    "  CPU Limit: \(.spec.containers[0].resources.limits.cpu // "none")",
    "  Memory Request: \(.spec.containers[0].resources.requests.memory // "none")",
    "  Memory Limit: \(.spec.containers[0].resources.limits.memory // "none")"'
```

### 30. Rollout и rollback
```bash
#!/bin/bash
# Управление развёртыванием

DEPLOYMENT=$1
ACTION=$2
NAMESPACE=${3:-default}

if [ -z "$DEPLOYMENT" ]; then
    echo "Использование: $0 <deployment> {status|history|undo} [namespace]"
    exit 1
fi

case $ACTION in
    status)
        kubectl rollout status deployment "$DEPLOYMENT" -n "$NAMESPACE"
        ;;
    history)
        kubectl rollout history deployment "$DEPLOYMENT" -n "$NAMESPACE"
        ;;
    undo)
        echo "Откат deployment $DEPLOYMENT..."
        kubectl rollout undo deployment "$DEPLOYMENT" -n "$NAMESPACE"
        kubectl rollout status deployment "$DEPLOYMENT" -n "$NAMESPACE"
        ;;
    *)
        echo "Action: {status|history|undo}"
        exit 1
        ;;
esac
```

---

## 🔐 Security & SSL

### 31. Проверка срока действия SSL сертификата
```bash
#!/bin/bash
# Проверка SSL сертификатов

check_ssl() {
    local domain=$1
    local port=${2:-443}
    
    echo "Проверка $domain:$port..."
    
    expiry_date=$(echo | openssl s_client -servername "$domain" -connect "$domain:$port" 2>/dev/null | \
        openssl x509 -noout -enddate 2>/dev/null | cut -d= -f2)
    
    if [ -z "$expiry_date" ]; then
        echo "✗ Не удалось получить информацию о сертификате"
        return 1
    fi
    
    expiry_epoch=$(date -d "$expiry_date" +%s)
    current_epoch=$(date +%s)
    days_left=$(( ($expiry_epoch - $current_epoch) / 86400 ))
    
    echo "  Истекает: $expiry_date"
    echo "  Осталось дней: $days_left"
    
    if [ $days_left -lt 30 ]; then
        echo "  ⚠️  ВНИМАНИЕ: Сертификат скоро истечёт!"
    else
        echo "  ✓ Сертификат действителен"
    fi
}

# Проверка нескольких доменов
domains=("google.com" "github.com")

for domain in "${domains[@]}"; do
    check_ssl "$domain"
    echo ""
done
```

### 32. Сканирование портов (nmap альтернатива)
```bash
#!/bin/bash
# Простое сканирование портов

scan_ports() {
    local host=$1
    local start_port=${2:-1}
    local end_port=${3:-1000}
    
    echo "Сканирование $host портов $start_port-$end_port..."
    
    for port in $(seq $start_port $end_port); do
        timeout 1 bash -c "</dev/tcp/$host/$port" 2>/dev/null && \
            echo "✓ Порт $port открыт"
    done
}

scan_ports "localhost" 1 1024
```

### 33. Генерация сильных паролей
```bash
#!/bin/bash
# Генератор паролей

generate_password() {
    local length=${1:-16}
    
    # Метод 1: через /dev/urandom
    password=$(tr -dc 'A-Za-z0-9!@#$%^&*()_+' < /dev/urandom | head -c $length)
    echo "Password 1: $password"
    
    # Метод 2: через openssl
    password=$(openssl rand -base64 $length | tr -d '=' | head -c $length)
    echo "Password 2: $password"
    
    # Метод 3: только буквы и цифры
    password=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w $length | head -n 1)
    echo "Password 3: $password"
}

generate_password 20
```

### 34. Проверка SSH ключей
```bash
#!/bin/bash
# Аудит SSH ключей

SSH_DIR="$HOME/.ssh"

echo "=== SSH Key Audit ==="

for keyfile in "$SSH_DIR"/*.pub; do
    if [ -f "$keyfile" ]; then
        echo ""
        echo "Файл: $(basename $keyfile)"
        
        # Тип ключа
        key_type=$(ssh-keygen -l -f "$keyfile" | awk '{print $4}')
        echo "  Тип: $key_type"
        
        # Размер ключа
        key_size=$(ssh-keygen -l -f "$keyfile" | awk '{print $1}')
        echo "  Размер: $key_size бит"
        
        # Fingerprint
        fingerprint=$(ssh-keygen -l -f "$keyfile" | awk '{print $2}')
        echo "  Fingerprint: $fingerprint"
        
        # Проверка слабых ключей
        if [ "$key_type" == "(RSA)" ] && [ $key_size -lt 2048 ]; then
            echo "  ⚠️  ВНИМАНИЕ: Слабый ключ RSA < 2048 бит!"
        fi
    fi
done
```

### 35. Проверка прав доступа к файлам
```bash
#!/bin/bash
# Аудит прав доступа к критичным файлам

CRITICAL_FILES=(
    "/etc/passwd"
    "/etc/shadow"
    "/etc/ssh/sshd_config"
    "/root/.ssh/authorized_keys"
)

echo "=== File Permissions Audit ==="

for file in "${CRITICAL_FILES[@]}"; do
    if [ -e "$file" ]; then
        perms=$(stat -c "%a %U:%G" "$file" 2>/dev/null)
        echo "$file: $perms"
        
        # Проверка подозрительных прав
        perm_num=$(stat -c "%a" "$file")
        if [ "$perm_num" -gt 644 ] && [ "$file" != "/root/.ssh/authorized_keys" ]; then
            echo "  ⚠️  Подозрительные права доступа!"
        fi
    else
        echo "$file: не найден"
    fi
done
```

### 36. Проверка открытых соединений
```bash
#!/bin/bash
# Мониторинг сетевых соединений

echo "=== Активные соединения ==="
echo ""

echo "ESTABLISHED соединения:"
netstat -tn 2>/dev/null | grep ESTABLISHED | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn

echo ""
echo "Прослушиваемые порты:"
netstat -tuln 2>/dev/null | grep LISTEN | awk '{print $4}' | sed 's/.*://g' | sort -n | uniq

echo ""
echo "TOP подключающихся IP:"
netstat -tn 2>/dev/null | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head -10
```

### 37. Fail2ban мониторинг
```bash
#!/bin/bash
# Проверка статуса Fail2ban

if ! command -v fail2ban-client &> /dev/null; then
    echo "Fail2ban не установлен"
    exit 1
fi

echo "=== Fail2ban Status ==="
sudo fail2ban-client status

echo ""
echo "=== Banned IPs ==="

for jail in $(sudo fail2ban-client status | grep "Jail list" | sed "s/.*://g" | sed 's/,//g'); do
    echo ""
    echo "Jail: $jail"
    sudo fail2ban-client status $jail | grep "Banned IP"
done
```

### 38. Проверка обновлений безопасности
```bash
#!/bin/bash
# Проверка доступных обновлений безопасности

if [ -f /etc/debian_version ]; then
    # Debian/Ubuntu
    echo "Проверка обновлений безопасности (Debian/Ubuntu)..."
    apt-get update > /dev/null 2>&1
    security_updates=$(apt-get upgrade -s | grep -i security | wc -l)
    echo "Доступно обновлений безопасности: $security_updates"
    apt-get upgrade -s | grep -i security
elif [ -f /etc/redhat-release ]; then
    # RHEL/CentOS
    echo "Проверка обновлений безопасности (RHEL/CentOS)..."
    yum updateinfo list security | tail -n +2
fi
```

### 39. Шифрование файлов GPG
```bash
#!/bin/bash
# Шифрование и расшифровка файлов

ACTION=$1
FILE=$2

case $ACTION in
    encrypt)
        if [ -z "$FILE" ]; then
            echo "Использование: $0 encrypt <file>"
            exit 1
        fi
        gpg -c "$FILE"
        echo "✓ Файл зашифрован: ${FILE}.gpg"
        ;;
    decrypt)
        if [ -z "$FILE" ]; then
            echo "Использование: $0 decrypt <file.gpg>"
            exit 1
        fi
        gpg "$FILE"
        echo "✓ Файл расшифрован"
        ;;
    *)
        echo "Использование: $0 {encrypt|decrypt} <file>"
        exit 1
        ;;
esac
```

### 40. Проверка CVE уязвимостей
```bash
#!/bin/bash
# Сканирование системы на уязвимости

if command -v lynis &> /dev/null; then
    echo "Запуск Lynis аудита..."
    sudo lynis audit system --quick
elif command -v rkhunter &> /dev/null; then
    echo "Запуск Rootkit Hunter..."
    sudo rkhunter --check --skip-keypress
else
    echo "Установите lynis или rkhunter для сканирования"
    echo "Ubuntu: sudo apt install lynis"
fi
```

---

## 📊 Monitoring & Alerts

### 41. Проверка времени отклика сервисов
```bash
#!/bin/bash
# Мониторинг времени отклика HTTP

check_response_time() {
    local url=$1
    local threshold=${2:-2}
    
    response_time=$(curl -o /dev/null -s -w '%{time_total}\n' "$url")
    status_code=$(curl -o /dev/null -s -w '%{http_code}\n' "$url")
    
    echo "$url:"
    echo "  Status: $status_code"
    echo "  Time: ${response_time}s"
    
    if (( $(echo "$response_time > $threshold" | bc -l) )); then
        echo "  ⚠️  Медленный ответ (порог: ${threshold}s)"
        return 1
    else
        echo "  ✓ OK"
        return 0
    fi
}

urls=(
    "https://google.com"
    "https://github.com"
)

for url in "${urls[@]}"; do
    check_response_time "$url" 2
    echo ""
done
```

### 42. Отправка уведомлений в Slack
```bash
#!/bin/bash
# Отправка сообщений в Slack

WEBHOOK_URL="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"

send_slack_notification() {
    local message=$1
    local channel=${2:-#alerts}
    local username=${3:-"Monitoring Bot"}
    
    payload=$(cat <<EOF
{
    "channel": "$channel",
    "username": "$username",
    "text": "$message",
    "icon_emoji": ":robot_face:"
}
EOF
)
    
    curl -X POST -H 'Content-type: application/json' \
        --data "$payload" "$WEBHOOK_URL"
    
    echo "✓ Уведомление отправлено в Slack"
}

send_slack_notification "⚠️ CPU usage > 80% on server-01"
```

### 43. Telegram bot уведомления
```bash
#!/bin/bash
# Отправка уведомлений в Telegram

BOT_TOKEN="your_bot_token"
CHAT_ID="your_chat_id"

send_telegram() {
    local message=$1
    
    curl -s -X POST "https://api.telegram.org/bot${BOT_TOKEN}/sendMessage" \
        -d chat_id="$CHAT_ID" \
        -d text="$message" \
        -d parse_mode="HTML" > /dev/null
    
    if [ $? -eq 0 ]; then
        echo "✓ Сообщение отправлено в Telegram"
    else
        echo "✗ Ошибка отправки"
    fi
}

# Пример использования
cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
if (( $(echo "$cpu_usage > 80" | bc -l) )); then
    send_telegram "⚠️ <b>High CPU Alert</b>%0ACurrent usage: ${cpu_usage}%"
fi
```

### 44. Проверка доступности веб-сервисов
```bash
#!/bin/bash
# Healthcheck веб-сервисов

SERVICES_FILE="services.txt"  # URL на каждой строке

if [ ! -f "$SERVICES_FILE" ]; then
    cat > "$SERVICES_FILE" << EOF
https://google.com
https://github.com
https://stackoverflow.com
EOF
fi

echo "=== Service Health Check ==="
echo "Time: $(date)"
echo ""

while IFS= read -r url; do
    [ -z "$url" ] && continue
    
    status_code=$(curl -o /dev/null -s -w "%{http_code}" --max-time 10 "$url")
    response_time=$(curl -o /dev/null -s -w "%{time_total}" --max-time 10 "$url")
    
    if [ "$status_code" == "200" ]; then
        echo "✓ $url - ${status_code} (${response_time}s)"
    else
        echo "✗ $url - ${status_code}"
        # Отправка алерта
        echo "ALERT: $url is down!" | mail -s "Service Down" admin@example.com
    fi
done < "$SERVICES_FILE"
```

### 45. Мониторинг дискового пространства с алертами
```bash
#!/bin/bash
# Расширенный мониторинг дисков

THRESHOLD=80
CRITICAL=90
ALERT_EMAIL="admin@example.com"

check_disk_space() {
    df -h | grep -vE '^Filesystem|tmpfs|cdrom' | while read line; do
        partition=$(echo $line | awk '{print $1}')
        usage=$(echo $line | awk '{print $5}' | sed 's/%//g')
        mount_point=$(echo $line | awk '{print $6}')
        available=$(echo $line | awk '{print $4}')
        
        if [ $usage -ge $CRITICAL ]; then
            echo "🔴 CRITICAL: $mount_point на $partition: ${usage}% (осталось: $available)"
            echo "CRITICAL: Disk usage $usage% on $mount_point" | \
                mail -s "CRITICAL Disk Alert" "$ALERT_EMAIL"
        elif [ $usage -ge $THRESHOLD ]; then
            echo "⚠️  WARNING: $mount_point на $partition: ${usage}% (осталось: $available)"
        else
            echo "✓ $mount_point: ${usage}%"
        fi
    done
}

check_disk_space
```

### 46. Мониторинг базы данных
```bash
#!/bin/bash
# Проверка состояния PostgreSQL

check_postgres() {
    local host=${1:-localhost}
    local port=${2:-5432}
    local database=${3:-postgres}
    
    # Проверка доступности
    if pg_isready -h "$host" -p "$port" -d "$database" > /dev/null 2>&1; then
        echo "✓ PostgreSQL на $host:$port доступен"
    else
        echo "✗ PostgreSQL на $host:$port недоступен"
        return 1
    fi
    
    # Количество активных соединений
    connections=$(psql -h "$host" -p "$port" -d "$database" -t -c \
        "SELECT count(*) FROM pg_stat_activity;" 2>/dev/null | tr -d ' ')
    echo "  Активных соединений: $connections"
    
    # Размер базы данных
    db_size=$(psql -h "$host" -p "$port" -d "$database" -t -c \
        "SELECT pg_size_pretty(pg_database_size('$database'));" 2>/dev/null | tr -d ' ')
    echo "  Размер БД: $db_size"
}

check_postgres "localhost" 5432 "mydb"
```

### 47. Мониторинг процессов с высокой нагрузкой
```bash
#!/bin/bash
# Поиск процессов-пожирателей ресурсов

echo "=== TOP CPU Processes ==="
ps aux --sort=-%cpu | head -11

echo ""
echo "=== TOP Memory Processes ==="
ps aux --sort=-%mem | head -11

echo ""
echo "=== Процессы в состоянии D (uninterruptible sleep) ==="
ps aux | awk '$8 ~ /D/ {print $0}'

echo ""
echo "=== Zombie процессы ==="
ps aux | awk '$8 ~ /Z/ {print $0}'
```

### 48. Сбор системных метрик
```bash
#!/bin/bash
# Сбор метрик для мониторинга

METRICS_FILE="/var/log/system-metrics.log"

collect_metrics() {
    timestamp=$(date +"%Y-%m-%d %H:%M:%S")
    
    # CPU
    cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    
    # Memory
    mem_total=$(free -m | awk 'NR==2{print $2}')
    mem_used=$(free -m | awk 'NR==2{print $3}')
    mem_percent=$(awk "BEGIN {printf \"%.2f\", ($mem_used/$mem_total)*100}")
    
    # Disk
    disk_usage=$(df -h / | awk 'NR==2{print $5}' | sed 's/%//g')
    
    # Load Average
    load_avg=$(uptime | awk -F'load average:' '{print $2}' | xargs)
    
    # Network (RX/TX bytes)
    rx_bytes=$(cat /sys/class/net/eth0/statistics/rx_bytes 2>/dev/null || echo 0)
    tx_bytes=$(cat /sys/class/net/eth0/statistics/tx_bytes 2>/dev/null || echo 0)
    
    # Запись в лог
    echo "$timestamp|CPU:${cpu_usage}%|MEM:${mem_percent}%|DISK:${disk_usage}%|LOAD:${load_avg}|RX:${rx_bytes}|TX:${tx_bytes}" >> "$METRICS_FILE"
    
    echo "Метрики собраны: $timestamp"
}

# Запуск каждые 60 секунд
while true; do
    collect_metrics
    sleep 60
done
```

### 49. Проверка логов на ошибки
```bash
#!/bin/bash
# Анализ системных логов на наличие ошибок

LOG_FILES=(
    "/var/log/syslog"
    "/var/log/messages"
    "/var/log/kern.log"
)

PATTERNS=("error" "fail" "critical" "panic" "fatal")
TIME_RANGE="10 minutes"

echo "=== Анализ логов за последние $TIME_RANGE ==="

for log in "${LOG_FILES[@]}"; do
    if [ -f "$log" ]; then
        echo ""
        echo "Файл: $log"
        
        for pattern in "${PATTERNS[@]}"; do
            count=$(grep -i "$pattern" "$log" | wc -l)
            if [ $count -gt 0 ]; then
                echo "  $pattern: $count вхождений"
                grep -i "$pattern" "$log" | tail -5 | sed 's/^/    /'
            fi
        done
    fi
done
```

### 50. Создание дашборда статуса
```bash
#!/bin/bash
# Генерация HTML дашборда статуса системы

OUTPUT="/var/www/html/status.html"

generate_dashboard() {
    cat > "$OUTPUT" << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>System Status Dashboard</title>
    <meta http-equiv="refresh" content="60">
    <style>
        body { font-family: Arial; margin: 20px; background: #f5f5f5; }
        .container { max-width: 1200px; margin: 0 auto; }
        .metric { background: white; padding: 20px; margin: 10px 0; border-radius: 5px; }
        .ok { color: green; }
        .warning { color: orange; }
        .critical { color: red; }
        h1 { color: #333; }
    </style>
</head>
<body>
    <div class="container">
        <h1>System Status</h1>
        <p>Last Updated: $(date)</p>
EOF

    # CPU
    cpu=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    cpu_class="ok"
    [ ${cpu%.*} -gt 70 ] && cpu_class="warning"
    [ ${cpu%.*} -gt 90 ] && cpu_class="critical"
    
    echo "<div class='metric'><h2>CPU Usage</h2><p class='$cpu_class'>${cpu}%</p></div>" >> "$OUTPUT"
    
    # Memory
    mem=$(free | awk 'NR==2{printf "%.2f", $3*100/$2}')
    mem_class="ok"
    [ ${mem%.*} -gt 70 ] && mem_class="warning"
    [ ${mem%.*} -gt 90 ] && mem_class="critical"
    
    echo "<div class='metric'><h2>Memory Usage</h2><p class='$mem_class'>${mem}%</p></div>" >> "$OUTPUT"
    
    # Disk
    disk=$(df -h / | awk 'NR==2{print $5}' | sed 's/%//g')
    disk_class="ok"
    [ $disk -gt 70 ] && disk_class="warning"
    [ $disk -gt 90 ] && disk_class="critical"
    
    echo "<div class='metric'><h2>Disk Usage</h2><p class='$disk_class'>${disk}%</p></div>" >> "$OUTPUT"
    
    # Services
    echo "<div class='metric'><h2>Services</h2><ul>" >> "$OUTPUT"
    for service in nginx mysql redis docker; do
        if systemctl is-active --quiet $service 2>/dev/null; then
            echo "<li class='ok'>$service: Running</li>" >> "$OUTPUT"
        else
            echo "<li class='critical'>$service: Stopped</li>" >> "$OUTPUT"
        fi
    done
    echo "</ul></div>" >> "$OUTPUT"
    
    cat >> "$OUTPUT" << 'EOF'
    </div>
</body>
</html>
EOF

    echo "✓ Dashboard обновлён: $OUTPUT"
}

generate_dashboard
```

---

## 🚀 CI/CD & Automation

### 51. Git автоматизация
```bash
#!/bin/bash
# Автоматический коммит и пуш изменений

REPO_DIR=${1:-.}
COMMIT_MESSAGE=${2:-"Auto commit $(date +%Y-%m-%d_%H:%M:%S)"}

cd "$REPO_DIR" || exit 1

# Проверка изменений
if [ -n "$(git status --porcelain)" ]; then
    echo "Обнаружены изменения, создание коммита..."
    
    git add .
    git commit -m "$COMMIT_MESSAGE"
    git push origin $(git branch --show-current)
    
    echo "✓ Изменения отправлены"
else
    echo "Нет изменений для коммита"
fi
```

### 52. Деплой приложения
```bash
#!/bin/bash
# Простой скрипт деплоя

APP_DIR="/var/www/myapp"
REPO_URL="git@github.com:user/repo.git"
BRANCH="main"

deploy() {
    echo "=== Начало деплоя ==="
    
    # Backup текущей версии
    if [ -d "$APP_DIR" ]; then
        backup_dir="${APP_DIR}_backup_$(date +%Y%m%d_%H%M%S)"
        echo "Создание backup: $backup_dir"
        cp -r "$APP_DIR" "$backup_dir"
    fi
    
    # Клонирование или обновление
    if [ -d "$APP_DIR/.git" ]; then
        echo "Обновление репозитория..."
        cd "$APP_DIR"
        git fetch origin
        git checkout "$BRANCH"
        git pull origin "$BRANCH"
    else
        echo "Клонирование репозитория..."
        git clone -b "$BRANCH" "$REPO_URL" "$APP_DIR"
        cd "$APP_DIR"
    fi
    
    # Установка зависимостей
    if [ -f "package.json" ]; then
        echo "Установка npm пакетов..."
        npm install --production
    elif [ -f "requirements.txt" ]; then
        echo "Установка Python пакетов..."
        pip install -r requirements.txt
    fi
    
    # Перезапуск сервисов
    echo "Перезапуск сервисов..."
    sudo systemctl restart myapp
    
    # Проверка здоровья
    sleep 5
    if curl -f http://localhost:3000/health > /dev/null 2>&1; then
        echo "✓ Деплой успешен"
    else
        echo "✗ Приложение не отвечает, откат..."
        sudo systemctl stop myapp
        rm -rf "$APP_DIR"
        mv "$backup_dir" "$APP_DIR"
        sudo systemctl start myapp
        exit 1
    fi
}

deploy
```

### 53. Проверка статуса Jenkins jobs
```bash
#!/bin/bash
# Мониторинг Jenkins через API

JENKINS_URL="http://jenkins.local:8080"
JENKINS_USER="admin"
JENKINS_TOKEN="your_token"

get_job_status() {
    local job_name=$1
    
    response=$(curl -s -u "$JENKINS_USER:$JENKINS_TOKEN" \
        "$JENKINS_URL/job/$job_name/lastBuild/api/json")
    
    result=$(echo "$response" | jq -r '.result')
    number=$(echo "$response" | jq -r '.number')
    duration=$(echo "$response" | jq -r '.duration')
    
    echo "$job_name (Build #$number): $result (${duration}ms)"
}

jobs=("deploy-prod" "deploy-staging" "run-tests")

echo "=== Jenkins Jobs Status ==="
for job in "${jobs[@]}"; do
    get_job_status "$job"
done
```

### 54. Запуск GitLab CI pipeline
```bash
#!/bin/bash
# Trigger GitLab pipeline через API

GITLAB_URL="https://gitlab.com"
PROJECT_ID="12345"
GITLAB_TOKEN="your_private_token"
REF="main"

trigger_pipeline() {
    response=$(curl -s -X POST \
        -H "PRIVATE-TOKEN: $GITLAB_TOKEN" \
        "$GITLAB_URL/api/v4/projects/$PROJECT_ID/pipeline?ref=$REF")
    
    pipeline_id=$(echo "$response" | jq -r '.id')
    status=$(echo "$response" | jq -r '.status')
    
    if [ "$pipeline_id" != "null" ]; then
        echo "✓ Pipeline запущен: #$pipeline_id"
        echo "  Status: $status"
        echo "  URL: $GITLAB_URL/user/repo/-/pipelines/$pipeline_id"
    else
        echo "✗ Ошибка запуска pipeline"
        echo "$response"
    fi
}

trigger_pipeline
```

### 55. Создание релизных тегов
```bash
#!/bin/bash
# Автоматическое создание версионных тегов

REPO_DIR=${1:-.}

cd "$REPO_DIR" || exit 1

# Получение последнего тега
last_tag=$(git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0")
echo "Последний тег: $last_tag"

# Парсинг версии
version=${last_tag#v}
IFS='.' read -r major minor patch <<< "$version"

# Инкремент версии
read -p "Тип релиза [major/minor/patch]: " release_type

case $release_type in
    major)
        new_version="v$((major + 1)).0.0"
        ;;
    minor)
        new_version="v${major}.$((minor + 1)).0"
        ;;
    patch|*)
        new_version="v${major}.${minor}.$((patch + 1))"
        ;;
esac

echo "Новая версия: $new_version"
read -p "Создать тег? (y/n) " -n 1 -r
echo

if [[ $REPLY =~ ^[Yy]$ ]]; then
    git tag -a "$new_version" -m "Release $new_version"
    git push origin "$new_version"
    echo "✓ Тег $new_version создан и отправлен"
fi
```

### 56. Сравнение веток Git
```bash
#!/bin/bash
# Сравнение изменений между ветками

BRANCH1=${1:-main}
BRANCH2=${2:-develop}

echo "=== Сравнение $BRANCH1 и $BRANCH2 ==="
echo ""

echo "Коммиты в $BRANCH2, отсутствующие в $BRANCH1:"
git log $BRANCH1..$BRANCH2 --oneline --no-merges

echo ""
echo "Изменённые файлы:"
git diff --name-status $BRANCH1..$BRANCH2

echo ""
echo "Статистика:"
git diff --stat $BRANCH1..$BRANCH2
```

### 57. Проверка качества кода
```bash
#!/bin/bash
# Pre-commit проверки кода

echo "=== Code Quality Checks ==="

# Проверка синтаксиса Python
if ls *.py 1> /dev/null 2>&1; then
    echo "Проверка Python файлов..."
    for file in *.py; do
        python3 -m py_compile "$file"
        if [ $? -eq 0 ]; then
            echo "✓ $file"
        else
            echo "✗ $file содержит ошибки"
            exit 1
        fi
    done
fi

# Проверка синтаксиса Bash
if ls *.sh 1> /dev/null 2>&1; then
    echo "Проверка Bash скриптов..."
    for file in *.sh; do
        bash -n "$file"
        if [ $? -eq 0 ]; then
            echo "✓ $file"
        else
            echo "✗ $file содержит ошибки"
            exit 1
        fi
    done
fi

# Проверка YAML файлов
if command -v yamllint &> /dev/null && ls *.yaml *.yml 1> /dev/null 2>&1; then
    echo "Проверка YAML файлов..."
    yamllint *.yaml *.yml
fi

echo ""
echo "✓ Все проверки пройдены"
```

### 58. Создание changelog
```bash
#!/bin/bash
# Генерация changelog из Git коммитов

OUTPUT="CHANGELOG.md"
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "")

cat > "$OUTPUT" << EOF
# Changelog

## [Unreleased]

EOF

if [ -n "$LAST_TAG" ]; then
    echo "### Changes since $LAST_TAG" >> "$OUTPUT"
    echo "" >> "$OUTPUT"
    
    # Features
    echo "#### ✨ Features" >> "$OUTPUT"
    git log $LAST_TAG..HEAD --oneline --grep="^feat" >> "$OUTPUT" 2>/dev/null
    echo "" >> "$OUTPUT"
    
    # Fixes
    echo "#### 🐛 Bug Fixes" >> "$OUTPUT"
    git log $LAST_TAG..HEAD --oneline --grep="^fix" >> "$OUTPUT" 2>/dev/null
    echo "" >> "$OUTPUT"
    
    # Other
    echo "#### 📝 Other Changes" >> "$OUTPUT"
    git log $LAST_TAG..HEAD --oneline --grep="^chore\|^docs\|^refactor" >> "$OUTPUT" 2>/dev/null
else
    echo "Все коммиты:" >> "$OUTPUT"
    git log --oneline >> "$OUTPUT"
fi

echo "✓ Changelog создан: $OUTPUT"
```

### 59. Blue-Green deployment
```bash
#!/bin/bash
# Blue-Green deployment strategy

BLUE_PORT=8080
GREEN_PORT=8081
NGINX_CONFIG="/etc/nginx/sites-available/myapp"
ACTIVE_ENV_FILE="/tmp/active_env"

# Определение текущей активной среды
if [ -f "$ACTIVE_ENV_FILE" ]; then
    ACTIVE=$(cat "$ACTIVE_ENV_FILE")
else
    ACTIVE="blue"
fi

# Определение неактивной среды
if [ "$ACTIVE" == "blue" ]; then
    INACTIVE="green"
    INACTIVE_PORT=$GREEN_PORT
    ACTIVE_PORT=$BLUE_PORT
else
    INACTIVE="blue"
    INACTIVE_PORT=$BLUE_PORT
    ACTIVE_PORT=$GREEN_PORT
fi

echo "Текущая активная среда: $ACTIVE (порт $ACTIVE_PORT)"
echo "Деплой в среду: $INACTIVE (порт $INACTIVE_PORT)"

# Деплой в неактивную среду
echo "Запуск приложения на порту $INACTIVE_PORT..."
# docker run -d -p $INACTIVE_PORT:8080 myapp:latest

# Healthcheck
sleep 5
if curl -f http://localhost:$INACTIVE_PORT/health > /dev/null 2>&1; then
    echo "✓ Приложение в $INACTIVE среде работает"
    
    # Переключение Nginx
    echo "Переключение трафика на $INACTIVE..."
    sed -i "s/localhost:$ACTIVE_PORT/localhost:$INACTIVE_PORT/g" "$NGINX_CONFIG"
    sudo nginx -t && sudo systemctl reload nginx
    
    # Обновление активной среды
    echo "$INACTIVE" > "$ACTIVE_ENV_FILE"
    
    echo "✓ Трафик переключён на $INACTIVE среду"
    echo "Старая среда $ACTIVE (порт $ACTIVE_PORT) всё ещё запущена для отката"
else
    echo "✗ Healthcheck failed, отмена деплоя"
    exit 1
fi
```

### 60. Rollback deployment
```bash
#!/bin/bash
# Откат последнего деплоя

DEPLOY_LOG="/var/log/deploys.log"
BACKUP_DIR="/backups/app"

rollback() {
    if [ ! -f "$DEPLOY_LOG" ]; then
        echo "Нет истории деплоев"
        exit 1
    fi
    
    last_deploy=$(tail -1 "$DEPLOY_LOG")
    backup_path=$(echo "$last_deploy" | awk '{print $NF}')
    
    if [ ! -d "$backup_path" ]; then
        echo "Backup не найден: $backup_path"
        exit 1
    fi
    
    echo "Откат к: $backup_path"
    read -p "Продолжить? (y/n) " -n 1 -r
    echo
    
    if [[ $REPLY =~ ^[Yy]$ ]]; then
        sudo systemctl stop myapp
        rm -rf /var/www/myapp
        cp -r "$backup_path" /var/www/myapp
        sudo systemctl start myapp
        
        echo "✓ Rollback выполнен"
    fi
}

rollback
```

---

## 🌐 Network & DNS

### 61. DNS lookup с детальной информацией
```bash
#!/bin/bash
# Подробная информация о DNS

domain=$1

if [ -z "$domain" ]; then
    echo "Использование: $0 <domain>"
    exit 1
fi

echo "=== DNS Information for $domain ==="
echo ""

echo "A Records:"
dig +short A "$domain"

echo ""
echo "AAAA Records (IPv6):"
dig +short AAAA "$domain"

echo ""
echo "MX Records:"
dig +short MX "$domain"

echo ""
echo "NS Records:"
dig +short NS "$domain"

echo ""
echo "TXT Records:"
dig +short TXT "$domain"

echo ""
echo "CNAME:"
dig +short CNAME "$domain"
```

### 62. Массовый ping хостов
```bash
#!/bin/bash
# Проверка доступности множества хостов

HOSTS_FILE="hosts.txt"
TIMEOUT=2

if [ ! -f "$HOSTS_FILE" ]; then
    cat > "$HOSTS_FILE" << EOF
8.8.8.8
1.1.1.1
google.com
github.com
EOF
fi

echo "=== Ping Results ==="

while IFS= read -r host; do
    [ -z "$host" ] && continue
    
    if ping -c 1 -W $TIMEOUT "$host" > /dev/null 2>&1; then
        response_time=$(ping -c 1 -W $TIMEOUT "$host" | grep 'time=' | awk -F'time=' '{print $2}' | awk '{print $1}')
        echo "✓ $host - ${response_time}ms"
    else
        echo "✗ $host - недоступен"
    fi
done < "$HOSTS_FILE"
```

### 63. Трассировка маршрута с визуализацией
```bash
#!/bin/bash
# Расширенный traceroute

HOST=$1

if [ -z "$HOST" ]; then
    echo "Использование: $0 <host>"
    exit 1
fi

echo "=== Трассировка маршрута до $HOST ==="
echo ""

if command -v mtr &> /dev/null; then
    # Использование mtr если доступен
    mtr -r -c 10 "$HOST"
else
    # Стандартный traceroute
    traceroute -m 20 "$HOST" 2>&1 | while read line; do
        echo "$line"
        
        # Извлечение IP и проверка географии через whois
        ip=$(echo "$line" | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' | head -1)
        if [ -n "$ip" ]; then
            country=$(whois "$ip" 2>/dev/null | grep -i country | head -1)
            [ -n "$country" ] && echo "  └─ $country"
        fi
    done
fi
```

### 64. Проверка скорости интернета
```bash
#!/bin/bash
# Тест скорости интернет-соединения

TEST_FILE_URL="http://speedtest.tele2.net/100MB.zip"
TEST_FILE_SIZE=100  # MB

echo "=== Internet Speed Test ==="

# Download speed
echo "Тестирование скорости загрузки..."
download_time=$(curl -o /dev/null -w '%{time_total}' -s "$TEST_FILE_URL")
download_speed=$(echo "scale=2; $TEST_FILE_SIZE / $download_time" | bc)
echo "Download: ${download_speed} MB/s"

# Latency (ping)
echo ""
echo "Проверка задержки..."
ping_result=$(ping -c 5 8.8.8.8 | tail -1 | awk '{print $4}' | cut -d '/' -f 2)
echo "Average Ping: ${ping_result}ms"

# DNS resolution time
echo ""
echo "Время DNS резолвинга..."
dns_time=$(time -p nslookup google.com 2>&1 | grep real | awk '{print $2}')
echo "DNS Resolution: ${dns_time}s"
```

### 65. Мониторинг сетевого трафика
```bash
#!/bin/bash
# Мониторинг сетевого интерфейса

INTERFACE=${1:-eth0}
INTERVAL=${2:-1}

if [ ! -d "/sys/class/net/$INTERFACE" ]; then
    echo "Интерфейс $INTERFACE не найден"
    echo "Доступные интерфейсы:"
    ls /sys/class/net/
    exit 1
fi

echo "Мониторинг $INTERFACE (Ctrl+C для выхода)"
echo ""

rx_prev=$(cat /sys/class/net/$INTERFACE/statistics/rx_bytes)
tx_prev=$(cat /sys/class/net/$INTERFACE/statistics/tx_bytes)

while true; do
    sleep $INTERVAL
    
    rx_curr=$(cat /sys/class/net/$INTERFACE/statistics/rx_bytes)
    tx_curr=$(cat /sys/class/net/$INTERFACE/statistics/tx_bytes)
    
    rx_rate=$(( ($rx_curr - $rx_prev) / $INTERVAL / 1024 ))
    tx_rate=$(( ($tx_curr - $tx_prev) / $INTERVAL / 1024 ))
    
    echo "$(date +'%H:%M:%S') - RX: ${rx_rate} KB/s | TX: ${tx_rate} KB/s"
    
    rx_prev=$rx_curr
    tx_prev=$tx_curr
done
```

### 66. Проверка открытых портов на удалённом хосте
```bash
#!/bin/bash
# Сканирование портов удалённого хоста

HOST=$1
START_PORT=${2:-1}
END_PORT=${3:-1000}

if [ -z "$HOST" ]; then
    echo "Использование: $0 <host> [start_port] [end_port]"
    exit 1
fi

echo "Сканирование $HOST портов $START_PORT-$END_PORT..."
echo ""

open_ports=()

for port in $(seq $START_PORT $END_PORT); do
    timeout 1 bash -c "echo >/dev/tcp/$HOST/$port" 2>/dev/null && {
        echo "✓ Порт $port открыт"
        open_ports+=($port)
        
        # Определение сервиса
        service=$(grep -w "$port/tcp" /etc/services | awk '{print $1}' | head -1)
        [ -n "$service" ] && echo "  └─ Сервис: $service"
    }
done

echo ""
echo "Найдено открытых портов: ${#open_ports[@]}"
```

### 67. Проверка SSL/TLS конфигурации
```bash
#!/bin/bash
# Проверка SSL/TLS настроек сервера

HOST=$1
PORT=${2:-443}

if [ -z "$HOST" ]; then
    echo "Использование: $0 <host> [port]"
    exit 1
fi

echo "=== SSL/TLS Analysis for $HOST:$PORT ==="
echo ""

# Протоколы
echo "Supported Protocols:"
for protocol in ssl3 tls1 tls1_1 tls1_2 tls1_3; do
    result=$(echo | openssl s_client -connect $HOST:$PORT -$protocol 2>&1 | grep "Protocol")
    if [ -n "$result" ]; then
        echo "  ✓ $protocol"
    else
        echo "  ✗ $protocol"
    fi
done

echo ""
echo "Certificate Information:"
echo | openssl s_client -connect $HOST:$PORT 2>/dev/null | \
    openssl x509 -noout -issuer -subject -dates

echo ""
echo "Cipher Suites:"
nmap --script ssl-enum-ciphers -p $PORT $HOST 2>/dev/null | \
    grep -A 50 "TLS"
```

### 68. Мониторинг DNS изменений
```bash
#!/bin/bash
# Отслеживание изменений DNS записей

DOMAIN=$1
CHECK_INTERVAL=${2:-300}  # 5 минут
RECORD_FILE="/tmp/dns_${DOMAIN}.txt"

if [ -z "$DOMAIN" ]; then
    echo "Использование: $0 <domain> [interval_seconds]"
    exit 1
fi

echo "Мониторинг DNS записей для $DOMAIN (Ctrl+C для выхода)"

# Первоначальная запись
dig +short "$DOMAIN" > "$RECORD_FILE"

while true; do
    sleep $CHECK_INTERVAL
    
    current=$(dig +short "$DOMAIN")
    previous=$(cat "$RECORD_FILE")
    
    if [ "$current" != "$previous" ]; then
        echo "⚠️  $(date): DNS записи изменились!"
        echo "Было: $previous"
        echo "Стало: $current"
        echo "$current" > "$RECORD_FILE"
        
        # Отправка уведомления
        echo "DNS changed for $DOMAIN: $previous -> $current" | \
            mail -s "DNS Alert" admin@example.com
    else
        echo "$(date): DNS записи без изменений"
    fi
done
```

### 69. Анализ сетевых соединений
```bash
#!/bin/bash
# Детальный анализ сетевых соединений

echo "=== Network Connections Analysis ==="
echo ""

echo "TOP IP адресов по количеству соединений:"
netstat -tn 2>/dev/null | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head -10

echo ""
echo "Соединения по состояниям:"
netstat -tan 2>/dev/null | awk '{print $6}' | sort | uniq -c | sort -rn

echo ""
echo "TOP процессов с открытыми портами:"
lsof -i -n -P 2>/dev/null | awk 'NR>1 {print $1}' | sort | uniq -c | sort -rn | head -10

echo ""
echo "Listening порты и процессы:"
netstat -tlnp 2>/dev/null | grep LISTEN | awk '{print $4 " -> " $7}'
```

### 70. Тест пропускной способности сети
```bash
#!/bin/bash
# Тест пропускной способности между серверами (требует iperf3)

MODE=$1  # server или client
SERVER_IP=$2

case $MODE in
    server)
        echo "Запуск iperf3 сервера..."
        iperf3 -s
        ;;
    client)
        if [ -z "$SERVER_IP" ]; then
            echo "Использование: $0 client <server_ip>"
            exit 1
        fi
        
        echo "=== Network Bandwidth Test ==="
        echo "Сервер: $SERVER_IP"
        echo ""
        
        echo "TCP тест (10 секунд):"
        iperf3 -c "$SERVER_IP" -t 10
        
        echo ""
        echo "UDP тест (1 Gbps):"
        iperf3 -c "$SERVER_IP" -u -b 1G -t 10
        ;;
    *)
        echo "Использование: $0 {server|client} [server_ip]"
        exit 1
        ;;
esac
```

---

## 📦 Backup & Recovery

### 71. Полный бэкап системы
```bash
#!/bin/bash
# Полное резервное копирование системы

BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/system_backup_$DATE.tar.gz"
EXCLUDE_FILE="/tmp/backup_exclude.txt"

# Создание списка исключений
cat > "$EXCLUDE_FILE" << 'EOF'
/proc/*
/sys/*
/dev/*
/tmp/*
/run/*
/mnt/*
/media/*
/lost+found
/backup/*
EOF

echo "=== System Backup Started ==="
echo "Backup file: $BACKUP_FILE"

mkdir -p "$BACKUP_DIR"

# Создание архива
tar -czf "$BACKUP_FILE" \
    --exclude-from="$EXCLUDE_FILE" \
    --one-file-system \
    / 2>/dev/null

if [ $? -eq 0 ]; then
    size=$(du -h "$BACKUP_FILE" | cut -f1)
    echo "✓ Backup завершён: $size"
    
    # Удаление старых бэкапов (>7 дней)
    find "$BACKUP_DIR" -name "system_backup_*.tar.gz" -mtime +7 -delete
else
    echo "✗ Ошибка создания backup"
    exit 1
fi

rm -f "$EXCLUDE_FILE"
```

### 72. MySQL/MariaDB backup
```bash
#!/bin/bash
# Резервное копирование всех баз данных MySQL

BACKUP_DIR="/backup/mysql"
DATE=$(date +%Y%m%d_%H%M%S)
MYSQL_USER="root"
MYSQL_PASS="password"
RETENTION_DAYS=7

mkdir -p "$BACKUP_DIR"

echo "=== MySQL Backup Started ==="

# Получение списка баз данных
databases=$(mysql -u"$MYSQL_USER" -p"$MYSQL_PASS" -e "SHOW DATABASES;" | grep -Ev "(Database|information_schema|performance_schema|mysql|sys)")

for db in $databases; do
    echo "Backup базы: $db"
    
    backup_file="$BACKUP_DIR/${db}_${DATE}.sql.gz"
    
    mysqldump -u"$MYSQL_USER" -p"$MYSQL_PASS" \
        --single-transaction \
        --routines \
        --triggers \
        "$db" | gzip > "$backup_file"
    
    if [ $? -eq 0 ]; then
        echo "✓ $db - $(du -h $backup_file | cut -f1)"
    else
        echo "✗ Ошибка backup $db"
    fi
done

# Очистка старых бэкапов
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete

echo "✓ MySQL backup завершён"
```

### 73. PostgreSQL backup
```bash
#!/bin/bash
# Резервное копирование PostgreSQL

BACKUP_DIR="/backup/postgresql"
DATE=$(date +%Y%m%d_%H%M%S)
PG_USER="postgres"
PG_HOST="localhost"
RETENTION_DAYS=7

mkdir -p "$BACKUP_DIR"

echo "=== PostgreSQL Backup Started ==="

# Backup всех баз данных
databases=$(sudo -u postgres psql -t -c "SELECT datname FROM pg_database WHERE datistemplate = false;")

for db in $databases; do
    db=$(echo $db | xargs)  # trim whitespace
    [ -z "$db" ] && continue
    
    echo "Backup базы: $db"
    
    backup_file="$BACKUP_DIR/${db}_${DATE}.sql.gz"
    
    sudo -u postgres pg_dump -h "$PG_HOST" "$db" | gzip > "$backup_file"
    
    if [ $? -eq 0 ]; then
        echo "✓ $db - $(du -h $backup_file | cut -f1)"
    else
        echo "✗ Ошибка backup $db"
    fi
done

# Backup глобальных объектов (роли, tablespaces)
sudo -u postgres pg_dumpall --globals-only | \
    gzip > "$BACKUP_DIR/globals_${DATE}.sql.gz"

# Очистка старых бэкапов
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete

echo "✓ PostgreSQL backup завершён"
```

### 74. Инкрементальный backup с rsync
```bash
#!/bin/bash
# Инкрементальное резервное копирование

SOURCE_DIR="/var/www"
BACKUP_BASE="/backup/incremental"
DATE=$(date +%Y%m%d_%H%M%S)
CURRENT_BACKUP="$BACKUP_BASE/backup_$DATE"
LATEST_LINK="$BACKUP_BASE/latest"

mkdir -p "$BACKUP_BASE"

echo "=== Incremental Backup Started ==="
echo "Source: $SOURCE_DIR"
echo "Destination: $CURRENT_BACKUP"

# Инкрементальный backup с hardlinks
if [ -d "$LATEST_LINK" ]; then
    echo "Использование предыдущего backup для hardlinks"
    rsync -a --delete --link-dest="$LATEST_LINK" \
        "$SOURCE_DIR/" "$CURRENT_BACKUP/"
else
    echo "Создание полного backup"
    rsync -a "$SOURCE_DIR/" "$CURRENT_BACKUP/"
fi

if [ $? -eq 0 ]; then
    # Обновление ссылки на последний backup
    rm -f "$LATEST_LINK"
    ln -s "$CURRENT_BACKUP" "$LATEST_LINK"
    
    echo "✓ Backup завершён"
    echo "Размер: $(du -sh $CURRENT_BACKUP | cut -f1)"
else
    echo "✗ Ошибка backup"
    exit 1
fi

# Удаление бэкапов старше 30 дней
find "$BACKUP_BASE" -maxdepth 1 -type d -name "backup_*" -mtime +30 -exec rm -rf {} \;
```

### 75. Backup в облако (AWS S3)
```bash
#!/bin/bash
# Загрузка backup в AWS S3

LOCAL_BACKUP_DIR="/backup"
S3_BUCKET="s3://my-backup-bucket"
DATE=$(date +%Y%m%d)

if ! command -v aws &> /dev/null; then
    echo "AWS CLI не установлен"
    exit 1
fi

echo "=== S3 Backup Started ==="

# Создание архива для загрузки
ARCHIVE_NAME="backup_${DATE}.tar.gz"
tar -czf "/tmp/$ARCHIVE_NAME" -C "$LOCAL_BACKUP_DIR" .

# Загрузка в S3
echo "Загрузка в S3..."
aws s3 cp "/tmp/$ARCHIVE_NAME" "$S3_BUCKET/" \
    --storage-class STANDARD_IA

if [ $? -eq 0 ]; then
    echo "✓ Backup загружен в S3"
    rm -f "/tmp/$ARCHIVE_NAME"
    
    # Удаление старых бэкапов из S3 (>30 дней)
    cutoff_date=$(date -d "30 days ago" +%Y%m%d)
    aws s3 ls "$S3_BUCKET/" | while read -r line; do
        file=$(echo $line | awk '{print $4}')
        file_date=$(echo $file | grep -oP '\d{8}')
        
        if [ -n "$file_date" ] && [ "$file_date" -lt "$cutoff_date" ]; then
            echo "Удаление старого backup: $file"
            aws s3 rm "$S3_BUCKET/$file"
        fi
    done
else
    echo "✗ Ошибка загрузки в S3"
    exit 1
fi
```

### 76. Восстановление из backup
```bash
#!/bin/bash
# Восстановление системы из backup

BACKUP_FILE=$1
RESTORE_DIR=${2:-/}

if [ -z "$BACKUP_FILE" ] || [ ! -f "$BACKUP_FILE" ]; then
    echo "Использование: $0 <backup_file> [restore_dir]"
    echo ""
    echo "Доступные backup файлы:"
    ls -lh /backup/*.tar.gz
    exit 1
fi

echo "⚠️  ВНИМАНИЕ: Восстановление из $BACKUP_FILE в $RESTORE_DIR"
read -p "Продолжить? (yes/no) " -r
echo

if [ "$REPLY" != "yes" ]; then
    echo "Отменено"
    exit 0
fi

echo "=== Starting Restore ==="

# Создание точки восстановления
if [ "$RESTORE_DIR" == "/" ]; then
    snapshot_dir="/backup/pre_restore_$(date +%Y%m%d_%H%M%S)"
    mkdir -p "$snapshot_dir"
    echo "Создание snapshot критичных директорий..."
    tar -czf "$snapshot_dir/snapshot.tar.gz" /etc /var/www 2>/dev/null
fi

# Восстановление
echo "Восстановление файлов..."
tar -xzf "$BACKUP_FILE" -C "$RESTORE_DIR" --exclude='proc/*' --exclude='sys/*' --exclude='dev/*'

if [ $? -eq 0 ]; then
    echo "✓ Восстановление завершено"
    echo "Необходима перезагрузка системы"
else
    echo "✗ Ошибка восстановления"
    exit 1
fi
```

### 77. Проверка целостности backup
```bash
#!/bin/bash
# Проверка целостности архивов

BACKUP_DIR="/backup"

echo "=== Backup Integrity Check ==="

find "$BACKUP_DIR" -name "*.tar.gz" -o -name "*.sql.gz" | while read backup_file; do
    echo ""
    echo "Проверка: $(basename $backup_file)"
    
    # Проверка архива
    if gzip -t "$backup_file" 2>/dev/null; then
        echo "  ✓ Архив целостен"
        
        # Проверка содержимого tar
        if [[ "$backup_file" == *.tar.gz ]]; then
            file_count=$(tar -tzf "$backup_file" 2>/dev/null | wc -l)
            echo "  Файлов в архиве: $file_count"
        fi
        
        # Размер и дата
        size=$(du -h "$backup_file" | cut -f1)
        modified=$(stat -c %y "$backup_file" | cut -d'.' -f1)
        echo "  Размер: $size"
        echo "  Дата: $modified"
    else
        echo "  ✗ ОШИБКА: Архив повреждён!"
    fi
done
```

### 78. Синхронизация между серверами
```bash
#!/bin/bash
# Синхронизация данных между серверами

SOURCE_DIR="/var/www"
REMOTE_USER="backup"
REMOTE_HOST="backup-server.com"
REMOTE_DIR="/backup/web"

echo "=== Server Sync ==="
echo "Source: $SOURCE_DIR"
echo "Destination: $REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR"

# Проверка доступности удалённого сервера
if ! ssh -q -o BatchMode=yes -o ConnectTimeout=5 "$REMOTE_USER@$REMOTE_HOST" exit; then
    echo "✗ Невозможно подключиться к $REMOTE_HOST"
    exit 1
fi

# Синхронизация с rsync
rsync -avz --delete \
    --exclude='*.log' \
    --exclude='cache/*' \
    --exclude='tmp/*' \
    -e "ssh -o StrictHostKeyChecking=no" \
    "$SOURCE_DIR/" \
    "$REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/"

if [ $? -eq 0 ]; then
    echo "✓ Синхронизация завершена"
    
    # Логирование
    echo "$(date): Sync completed to $REMOTE_HOST" >> /var/log/sync.log
else
    echo "✗ Ошибка синхронизации"
    exit 1
fi
```

### 79. Автоматизация backup с cron
```bash
#!/bin/bash
# Установка cron jobs для автоматического backup

CRON_FILE="/etc/cron.d/auto-backup"

cat > "$CRON_FILE" << 'EOF'
# Daily MySQL backup at 2 AM
0 2 * * * root /usr/local/bin/mysql-backup.sh >> /var/log/backup.log 2>&1

# Weekly full system backup on Sunday at 3 AM
0 3 * * 0 root /usr/local/bin/system-backup.sh >> /var/log/backup.log 2>&1

# Hourly incremental backup
0 * * * * root /usr/local/bin/incremental-backup.sh >> /var/log/backup.log 2>&1

# Daily sync to remote server at 4 AM
0 4 * * * root /usr/local/bin/remote-sync.sh >> /var/log/backup.log 2>&1
EOF

chmod 644 "$CRON_FILE"

echo "✓ Cron jobs для backup установлены"
echo ""
echo "Расписание:"
cat "$CRON_FILE"
```

### 80. Мониторинг статуса backup
```bash
#!/bin/bash
# Проверка статуса последних backup

BACKUP_DIR="/backup"
LOG_FILE="/var/log/backup.log"

echo "=== Backup Status Report ==="
echo "Date: $(date)"
echo ""

# Проверка последних backup файлов
echo "Последние backup файлы:"
find "$BACKUP_DIR" -type f -name "*.tar.gz" -o -name "*.sql.gz" | \
    xargs ls -lht | head -10

echo ""
echo "Использование дискового пространства:"
du -sh "$BACKUP_DIR"
df -h "$BACKUP_DIR"

echo ""
echo "Последние записи в логе:"
if [ -f "$LOG_FILE" ]; then
    tail -20 "$LOG_FILE"
else
    echo "Лог-файл не найден"
fi

# Проверка старых backup (предупреждение)
echo ""
latest_backup=$(find "$BACKUP_DIR" -type f -name "*.tar.gz" -printf '%T@ %p\n' | \
    sort -n | tail -1 | cut -d' ' -f2)

if [ -n "$latest_backup" ]; then
    last_modified=$(stat -c %Y "$latest_backup")
    current_time=$(date +%s)
    hours_old=$(( ($current_time - $last_modified) / 3600 ))
    
    echo "Последний backup: $(basename $latest_backup)"
    echo "Создан: $hours_old часов назад"
    
    if [ $hours_old -gt 48 ]; then
        echo "⚠️  ВНИМАНИЕ: Последний backup старше 48 часов!"
    fi
fi
```

---

## 🔍 Log Analysis & Troubleshooting

### 81. Анализ Apache/Nginx логов
```bash
#!/bin/bash
# Детальный анализ веб-сервера логов

LOG_FILE=${1:-/var/log/nginx/access.log}

if [ ! -f "$LOG_FILE" ]; then
    echo "Лог файл не найден: $LOG_FILE"
    exit 1
fi

echo "=== Web Server Log Analysis ==="
echo "File: $LOG_FILE"
echo "Lines: $(wc -l < $LOG_FILE)"
echo ""

echo "TOP 10 IP адресов:"
awk '{print $1}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10

echo ""
echo "TOP 10 запрашиваемых URL:"
awk '{print $7}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10

echo ""
echo "Статусы ответов:"
awk '{print $9}' "$LOG_FILE" | sort | uniq -c | sort -rn

echo ""
echo "User Agents:"
awk -F'"' '{print $6}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10

echo ""
echo "Запросы по часам:"
awk '{print $4}' "$LOG_FILE" | cut -d: -f2 | sort | uniq -c

echo ""
echo "Ошибки 4xx:"
awk '$9 ~ /^4/ {print $7}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10

echo ""
echo "Ошибки 5xx:"
awk '$9 ~ /^5/ {print $7}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10
```

### 82. Поиск ошибок в системных логах
```bash
#!/bin/bash
# Поиск критичных ошибок в логах

HOURS_BACK=${1:-24}

echo "=== System Error Analysis (последние $HOURS_BACK часов) ==="
echo ""

# Поиск в journalctl
if command -v journalctl &> /dev/null; then
    echo "Критичные ошибки из journal:"
    journalctl --since "$HOURS_BACK hours ago" -p err -p crit --no-pager | tail -20
    
    echo ""
    echo "Ошибки по сервисам:"
    journalctl --since "$HOURS_BACK hours ago" -p err --no-pager | \
        awk '{print $5}' | sort | uniq -c | sort -rn | head -10
fi

echo ""
echo "Ошибки в /var/log/syslog:"
if [ -f /var/log/syslog ]; then
    grep -i "error\|fail\|critical" /var/log/syslog | tail -20
fi

echo ""
echo "Kernel ошибки:"
dmesg -T | grep -i "error\|fail" | tail -20

echo ""
echo "Segmentation faults:"
grep "segfault" /var/log/syslog /var/log/kern.log 2>/dev/null | tail -10
```

### 83. Анализ использования дисковой подсистемы
```bash
#!/bin/bash
# Поиск процессов с высокой дисковой активностью

echo "=== Disk I/O Analysis ==="
echo ""

if command -v iotop &> /dev/null; then
    echo "TOP процессы по I/O (5 секунд):"
    iotop -b -n 1 -o | head -20
else
    echo "iotop не установлен, используем альтернативный метод"
    
    echo "Процессы по чтению диска:"
    cat /proc/*/io 2>/dev/null | \
        awk '/read_bytes/ {sum+=$2} END {print sum/(1024*1024) " MB"}'
fi

echo ""
echo "Открытые файлы по процессам:"
lsof / 2>/dev/null | awk '{print $1}' | sort | uniq -c | sort -rn | head -10

echo ""
echo "Самые большие файлы:"
find / -type f -size +100M -exec ls -lh {} \;
    
