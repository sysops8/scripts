# 50 Python Scripts для DevOps инженера

## 🔧 Системное администрирование

### 1. Проверка использования диска
```python
import shutil

def check_disk_usage(path="/"):
    usage = shutil.disk_usage(path)
    percent = (usage.used / usage.total) * 100
    print(f"Диск: {percent:.2f}% использовано")
    if percent > 80:
        print("⚠️ Предупреждение: мало места!")

check_disk_usage()
```

### 2. Мониторинг CPU и RAM
```python
import psutil

def system_health():
    cpu = psutil.cpu_percent(interval=1)
    ram = psutil.virtual_memory().percent
    print(f"CPU: {cpu}% | RAM: {ram}%")
    return cpu, ram

system_health()
```

### 3. Проверка работающих процессов
```python
import psutil

def check_process(process_name):
    for proc in psutil.process_iter(['name']):
        if process_name.lower() in proc.info['name'].lower():
            print(f"✓ {process_name} запущен")
            return True
    print(f"✗ {process_name} не найден")
    return False

check_process("nginx")
```

### 4. Автоматический restart сервиса
```python
import subprocess

def restart_service(service_name):
    try:
        subprocess.run(["systemctl", "restart", service_name], check=True)
        print(f"✓ {service_name} перезапущен")
    except subprocess.CalledProcessError:
        print(f"✗ Ошибка перезапуска {service_name}")

restart_service("nginx")
```

### 5. Проверка открытых портов
```python
import socket

def check_port(host, port):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(2)
    result = sock.connect_ex((host, port))
    sock.close()
    
    if result == 0:
        print(f"✓ Порт {port} открыт на {host}")
        return True
    else:
        print(f"✗ Порт {port} закрыт на {host}")
        return False

check_port("localhost", 80)
check_port("localhost", 443)
```

### 6. Логирование с ротацией
```python
import logging
from logging.handlers import RotatingFileHandler

def setup_logger(name, log_file, level=logging.INFO):
    handler = RotatingFileHandler(log_file, maxBytes=10*1024*1024, backupCount=5)
    formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
    handler.setFormatter(formatter)
    
    logger = logging.getLogger(name)
    logger.setLevel(level)
    logger.addHandler(handler)
    return logger

logger = setup_logger('app', '/var/log/myapp.log')
logger.info("Приложение запущено")
```

### 7. Парсинг логов Nginx
```python
import re

def parse_nginx_log(log_file):
    pattern = r'(\d+\.\d+\.\d+\.\d+).*\[(.+)\].*"(GET|POST) (.+?) HTTP.*" (\d+)'
    
    with open(log_file, 'r') as f:
        for line in f:
            match = re.search(pattern, line)
            if match:
                ip, date, method, path, status = match.groups()
                if status == '404':
                    print(f"404: {ip} -> {path}")

parse_nginx_log('/var/log/nginx/access.log')
```

### 8. Мониторинг изменений файлов
```python
import time
import os

def watch_file(filepath, interval=5):
    last_modified = os.path.getmtime(filepath)
    
    while True:
        time.sleep(interval)
        current_modified = os.path.getmtime(filepath)
        
        if current_modified != last_modified:
            print(f"⚠️ Файл {filepath} изменён!")
            last_modified = current_modified

watch_file('/etc/nginx/nginx.conf')
```

---

## 🐳 Docker & Containers

### 9. Список всех контейнеров
```python
import subprocess
import json

def list_containers():
    result = subprocess.run(
        ["docker", "ps", "-a", "--format", "{{json .}}"],
        capture_output=True, text=True
    )
    
    for line in result.stdout.strip().split('\n'):
        if line:
            container = json.loads(line)
            print(f"{container['Names']}: {container['Status']}")

list_containers()
```

### 10. Очистка неиспользуемых образов
```python
import subprocess

def cleanup_docker():
    commands = [
        ["docker", "system", "prune", "-f"],
        ["docker", "image", "prune", "-a", "-f"],
        ["docker", "volume", "prune", "-f"]
    ]
    
    for cmd in commands:
        subprocess.run(cmd)
        print(f"✓ Выполнено: {' '.join(cmd)}")

cleanup_docker()
```

### 11. Проверка health контейнеров
```python
import subprocess
import json

def check_container_health():
    result = subprocess.run(
        ["docker", "inspect", "--format={{.Name}} {{.State.Health.Status}}", 
         "$(docker ps -q)"],
        shell=True, capture_output=True, text=True
    )
    
    for line in result.stdout.strip().split('\n'):
        if line and 'unhealthy' in line:
            print(f"⚠️ Нездоровый контейнер: {line}")

check_container_health()
```

### 12. Автоматический restart упавших контейнеров
```python
import subprocess

def restart_stopped_containers():
    result = subprocess.run(
        ["docker", "ps", "-a", "--filter", "status=exited", 
         "--format", "{{.Names}}"],
        capture_output=True, text=True
    )
    
    for container in result.stdout.strip().split('\n'):
        if container:
            subprocess.run(["docker", "start", container])
            print(f"✓ Перезапущен: {container}")

restart_stopped_containers()
```

### 13. Мониторинг ресурсов Docker
```python
import subprocess
import json

def docker_stats():
    result = subprocess.run(
        ["docker", "stats", "--no-stream", "--format", 
         "{{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"],
        capture_output=True, text=True
    )
    
    print("Container\t\tCPU\tMemory")
    print(result.stdout)

docker_stats()
```

---

## ☸️ Kubernetes

### 14. Список всех подов
```python
from kubernetes import client, config

def list_pods(namespace="default"):
    config.load_kube_config()
    v1 = client.CoreV1Api()
    
    pods = v1.list_namespaced_pod(namespace)
    for pod in pods.items:
        status = pod.status.phase
        print(f"{pod.metadata.name}: {status}")

list_pods()
```

### 15. Проверка failing подов
```python
from kubernetes import client, config

def check_failing_pods(namespace="default"):
    config.load_kube_config()
    v1 = client.CoreV1Api()
    
    pods = v1.list_namespaced_pod(namespace)
    for pod in pods.items:
        if pod.status.phase != "Running":
            print(f"⚠️ {pod.metadata.name}: {pod.status.phase}")

check_failing_pods()
```

### 16. Масштабирование deployment
```python
from kubernetes import client, config

def scale_deployment(name, replicas, namespace="default"):
    config.load_kube_config()
    apps_v1 = client.AppsV1Api()
    
    body = {"spec": {"replicas": replicas}}
    apps_v1.patch_namespaced_deployment_scale(name, namespace, body)
    print(f"✓ {name} масштабирован до {replicas} реплик")

scale_deployment("nginx-deployment", 5)
```

### 17. Получение логов пода
```python
from kubernetes import client, config

def get_pod_logs(pod_name, namespace="default", lines=50):
    config.load_kube_config()
    v1 = client.CoreV1Api()
    
    logs = v1.read_namespaced_pod_log(
        pod_name, namespace, tail_lines=lines
    )
    print(logs)

get_pod_logs("my-pod-123")
```

### 18. Проверка статуса нод
```python
from kubernetes import client, config

def check_nodes():
    config.load_kube_config()
    v1 = client.CoreV1Api()
    
    nodes = v1.list_node()
    for node in nodes.items:
        conditions = {c.type: c.status for c in node.status.conditions}
        ready = conditions.get("Ready", "Unknown")
        print(f"{node.metadata.name}: Ready={ready}")

check_nodes()
```

---

## 🔐 Security & Secrets

### 19. Проверка SSL сертификата
```python
import ssl
import socket
from datetime import datetime

def check_ssl_expiry(hostname, port=443):
    context = ssl.create_default_context()
    with socket.create_connection((hostname, port)) as sock:
        with context.wrap_socket(sock, server_hostname=hostname) as ssock:
            cert = ssock.getpeercert()
            expires = datetime.strptime(cert['notAfter'], '%b %d %H:%M:%S %Y %Z')
            days_left = (expires - datetime.now()).days
            print(f"{hostname}: истекает через {days_left} дней")
            
            if days_left < 30:
                print("⚠️ Сертификат скоро истечёт!")

check_ssl_expiry("google.com")
```

### 20. Сканирование открытых портов
```python
import socket
from concurrent.futures import ThreadPoolExecutor

def scan_ports(host, ports=range(1, 1025)):
    open_ports = []
    
    def check(port):
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(0.5)
            if sock.connect_ex((host, port)) == 0:
                open_ports.append(port)
            sock.close()
        except:
            pass
    
    with ThreadPoolExecutor(max_workers=100) as executor:
        executor.map(check, ports)
    
    print(f"Открытые порты на {host}: {open_ports}")
    return open_ports

scan_ports("localhost")
```

### 21. Генерация случайных паролей
```python
import secrets
import string

def generate_password(length=16):
    chars = string.ascii_letters + string.digits + string.punctuation
    password = ''.join(secrets.choice(chars) for _ in range(length))
    return password

print(f"Пароль: {generate_password()}")
```

### 22. Шифрование конфигурационных файлов
```python
from cryptography.fernet import Fernet

def encrypt_file(filename, key):
    fernet = Fernet(key)
    with open(filename, 'rb') as f:
        data = f.read()
    encrypted = fernet.encrypt(data)
    
    with open(f"{filename}.encrypted", 'wb') as f:
        f.write(encrypted)

# Генерация ключа
key = Fernet.generate_key()
print(f"Ключ: {key.decode()}")

encrypt_file("config.yaml", key)
```

---

## 📊 Monitoring & Alerts

### 23. Отправка метрик в Prometheus
```python
from prometheus_client import CollectorRegistry, Gauge, push_to_gateway

def send_metric(job_name, metric_name, value):
    registry = CollectorRegistry()
    g = Gauge(metric_name, 'Description', registry=registry)
    g.set(value)
    push_to_gateway('localhost:9091', job=job_name, registry=registry)

send_metric('my_job', 'disk_usage', 75.5)
```

### 24. Проверка времени отклика сервиса
```python
import requests
import time

def check_response_time(url, threshold=2.0):
    start = time.time()
    try:
        response = requests.get(url, timeout=10)
        elapsed = time.time() - start
        
        print(f"{url}: {elapsed:.2f}s (status: {response.status_code})")
        
        if elapsed > threshold:
            print(f"⚠️ Медленный ответ! Порог: {threshold}s")
        
        return elapsed
    except Exception as e:
        print(f"✗ Ошибка: {e}")
        return None

check_response_time("https://google.com")
```

### 25. Мониторинг доступности множества сервисов
```python
import requests
from concurrent.futures import ThreadPoolExecutor

def check_services(services):
    def check(service):
        try:
            r = requests.get(service['url'], timeout=5)
            status = "✓" if r.status_code == 200 else "✗"
            print(f"{status} {service['name']}: {r.status_code}")
        except Exception as e:
            print(f"✗ {service['name']}: {e}")
    
    with ThreadPoolExecutor(max_workers=10) as executor:
        executor.map(check, services)

services = [
    {"name": "API", "url": "https://api.example.com/health"},
    {"name": "Web", "url": "https://example.com"},
    {"name": "DB", "url": "https://db.example.com:5432"}
]

check_services(services)
```

### 26. Telegram уведомления
```python
import requests

def send_telegram(token, chat_id, message):
    url = f"https://api.telegram.org/bot{token}/sendMessage"
    data = {"chat_id": chat_id, "text": message}
    
    response = requests.post(url, data=data)
    if response.status_code == 200:
        print("✓ Сообщение отправлено")
    else:
        print("✗ Ошибка отправки")

TOKEN = "your_bot_token"
CHAT_ID = "your_chat_id"
send_telegram(TOKEN, CHAT_ID, "⚠️ Сервер упал!")
```

### 27. Проверка доступности базы данных
```python
import psycopg2
import pymysql

def check_postgres(host, database, user, password):
    try:
        conn = psycopg2.connect(
            host=host, database=database, 
            user=user, password=password, connect_timeout=5
        )
        conn.close()
        print(f"✓ PostgreSQL {host} доступен")
        return True
    except Exception as e:
        print(f"✗ PostgreSQL {host}: {e}")
        return False

check_postgres("localhost", "mydb", "user", "pass")
```

---

## 🚀 CI/CD

### 28. Проверка статуса Jenkins job
```python
import requests
from requests.auth import HTTPBasicAuth

def check_jenkins_job(url, job_name, user, token):
    api_url = f"{url}/job/{job_name}/lastBuild/api/json"
    
    response = requests.get(api_url, auth=HTTPBasicAuth(user, token))
    if response.status_code == 200:
        data = response.json()
        result = data.get('result', 'UNKNOWN')
        print(f"Job {job_name}: {result}")
        return result
    else:
        print(f"✗ Ошибка получения статуса")

check_jenkins_job("http://jenkins.local", "deploy-prod", "admin", "token")
```

### 29. Автоматический деплой через GitLab API
```python
import requests

def trigger_gitlab_pipeline(project_id, token, ref="main"):
    url = f"https://gitlab.com/api/v4/projects/{project_id}/pipeline"
    headers = {"PRIVATE-TOKEN": token}
    data = {"ref": ref}
    
    response = requests.post(url, headers=headers, json=data)
    if response.status_code == 201:
        print(f"✓ Pipeline запущен: {response.json()['id']}")
    else:
        print(f"✗ Ошибка: {response.text}")

trigger_gitlab_pipeline(12345, "your_token")
```

### 30. Проверка успешности последнего коммита
```python
import subprocess

def check_last_commit():
    result = subprocess.run(
        ["git", "log", "-1", "--pretty=format:%h - %s (%an)"],
        capture_output=True, text=True
    )
    print(f"Последний коммит: {result.stdout}")

check_last_commit()
```

### 31. Автоматическое создание релизных тегов
```python
import subprocess
from datetime import datetime

def create_release_tag():
    version = datetime.now().strftime("v%Y.%m.%d")
    
    subprocess.run(["git", "tag", "-a", version, "-m", f"Release {version}"])
    subprocess.run(["git", "push", "origin", version])
    print(f"✓ Создан тег: {version}")

create_release_tag()
```

---

## 🌐 Networking

### 32. Проверка DNS резолвинга
```python
import socket

def check_dns(hostname):
    try:
        ip = socket.gethostbyname(hostname)
        print(f"{hostname} -> {ip}")
        return ip
    except socket.gaierror:
        print(f"✗ Не удалось разрешить {hostname}")
        return None

check_dns("google.com")
```

### 33. Ping множества хостов
```python
import subprocess
from concurrent.futures import ThreadPoolExecutor

def ping_hosts(hosts):
    def ping(host):
        result = subprocess.run(
            ["ping", "-c", "1", "-W", "1", host],
            stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL
        )
        status = "✓" if result.returncode == 0 else "✗"
        print(f"{status} {host}")
    
    with ThreadPoolExecutor(max_workers=10) as executor:
        executor.map(ping, hosts)

ping_hosts(["8.8.8.8", "1.1.1.1", "google.com"])
```

### 34. Трассировка маршрута
```python
import subprocess

def traceroute(host):
    result = subprocess.run(
        ["traceroute", "-m", "15", host],
        capture_output=True, text=True
    )
    print(result.stdout)

traceroute("google.com")
```

### 35. Проверка скорости соединения
```python
import speedtest

def test_speed():
    st = speedtest.Speedtest()
    
    print("Тестирование скорости...")
    download = st.download() / 1_000_000  # Mbps
    upload = st.upload() / 1_000_000
    ping = st.results.ping
    
    print(f"Download: {download:.2f} Mbps")
    print(f"Upload: {upload:.2f} Mbps")
    print(f"Ping: {ping:.2f} ms")

test_speed()
```

---

## 📦 Package Management

### 36. Проверка устаревших pip пакетов
```python
import subprocess
import json

def check_outdated_packages():
    result = subprocess.run(
        ["pip", "list", "--outdated", "--format=json"],
        capture_output=True, text=True
    )
    
    packages = json.loads(result.stdout)
    for pkg in packages:
        print(f"{pkg['name']}: {pkg['version']} -> {pkg['latest_version']}")

check_outdated_packages()
```

### 37. Массовое обновление пакетов
```python
import subprocess

def update_all_packages():
    result = subprocess.run(
        ["pip", "list", "--outdated", "--format=freeze"],
        capture_output=True, text=True
    )
    
    for line in result.stdout.strip().split('\n'):
        if line:
            package = line.split('==')[0]
            subprocess.run(["pip", "install", "--upgrade", package])
            print(f"✓ Обновлён: {package}")

update_all_packages()
```

### 38. Проверка безопасности зависимостей
```python
import subprocess

def security_audit():
    # pip-audit или safety
    result = subprocess.run(
        ["pip-audit", "--json"],
        capture_output=True, text=True
    )
    
    print("Проверка безопасности...")
    print(result.stdout)

security_audit()
```

---

## 🗄️ Database Operations

### 39. Бэкап PostgreSQL
```python
import subprocess
from datetime import datetime

def backup_postgres(host, database, user, output_dir):
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filename = f"{output_dir}/{database}_{timestamp}.sql"
    
    subprocess.run([
        "pg_dump",
        "-h", host,
        "-U", user,
        "-d", database,
        "-f", filename
    ])
    print(f"✓ Бэкап создан: {filename}")

backup_postgres("localhost", "mydb", "postgres", "/backups")
```

### 40. Проверка размера таблиц
```python
import psycopg2

def check_table_sizes(host, database, user, password):
    conn = psycopg2.connect(
        host=host, database=database, user=user, password=password
    )
    cursor = conn.cursor()
    
    cursor.execute("""
        SELECT tablename, pg_size_pretty(pg_total_relation_size(tablename::regclass))
        FROM pg_tables WHERE schemaname = 'public'
        ORDER BY pg_total_relation_size(tablename::regclass) DESC;
    """)
    
    for table, size in cursor.fetchall():
        print(f"{table}: {size}")
    
    conn.close()

check_table_sizes("localhost", "mydb", "user", "pass")
```

### 41. Мониторинг медленных запросов
```python
import psycopg2

def slow_queries(host, database, user, password, min_duration=1000):
    conn = psycopg2.connect(
        host=host, database=database, user=user, password=password
    )
    cursor = conn.cursor()
    
    cursor.execute(f"""
        SELECT query, calls, total_time, mean_time 
        FROM pg_stat_statements 
        WHERE mean_time > {min_duration}
        ORDER BY mean_time DESC LIMIT 10;
    """)
    
    for query, calls, total, mean in cursor.fetchall():
        print(f"Mean: {mean:.2f}ms | Calls: {calls}")
        print(f"Query: {query[:100]}...\n")
    
    conn.close()

slow_queries("localhost", "mydb", "user", "pass")
```

---

## 📈 Performance Testing

### 42. Простой load test
```python
import requests
import time
from concurrent.futures import ThreadPoolExecutor

def load_test(url, requests_count=100, workers=10):
    def make_request(i):
        start = time.time()
        try:
            r = requests.get(url, timeout=10)
            elapsed = time.time() - start
            return {"status": r.status_code, "time": elapsed}
        except Exception as e:
            return {"status": "error", "time": 0}
    
    print(f"Load testing {url}...")
    with ThreadPoolExecutor(max_workers=workers) as executor:
        results = list(executor.map(make_request, range(requests_count)))
    
    success = sum(1 for r in results if r['status'] == 200)
    avg_time = sum(r['time'] for r in results) / len(results)
    
    print(f"Успешных: {success}/{requests_count}")
    print(f"Среднее время: {avg_time:.3f}s")

load_test("https://google.com")
```

### 43. Бенчмарк API endpoint
```python
import requests
import time
import statistics

def benchmark_api(url, iterations=50):
    times = []
    
    for i in range(iterations):
        start = time.time()
        requests.get(url)
        elapsed = time.time() - start
        times.append(elapsed)
    
    print(f"Min: {min(times):.3f}s")
    print(f"Max: {max(times):.3f}s")
    print(f"Avg: {statistics.mean(times):.3f}s")
    print(f"Median: {statistics.median(times):.3f}s")

benchmark_api("https://api.github.com")
```

---

## 🔄 Automation Helpers

### 44. Выполнение команд на удалённом сервере
```python
import paramiko

def remote_execute(host, username, password, command):
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    
    ssh.connect(host, username=username, password=password)
    stdin, stdout, stderr = ssh.exec_command(command)
    
    print(stdout.read().decode())
    ssh.close()

remote_execute("192.168.1.100", "user", "pass", "df -h")
```

### 45. Копирование файлов по SSH
```python
import paramiko

def scp_file(host, username, password, local_path, remote_path):
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    ssh.connect(host, username=username, password=password)
    
    sftp = ssh.open_sftp()
    sftp.put(local_path, remote_path)
    sftp.close()
    ssh.close()
    
    print(f"✓ Файл скопирован на {host}")

scp_file("192.168.1.100", "user", "pass", "local.txt", "/tmp/remote.txt")
```

### 46. Парсинг конфигурационных файлов
```python
import yaml
import json

def parse_config(filename):
    with open(filename, 'r') as f:
        if filename.endswith('.yaml') or filename.endswith('.yml'):
            config = yaml.safe_load(f)
        elif filename.endswith('.json'):
            config = json.load(f)
        else:
            raise ValueError("Неподдерживаемый формат")
    
    return config

config = parse_config("config.yaml")
print(config)
```

### 47. Генерация отчётов в HTML
```python
from datetime import datetime

def generate_report(data, filename="report.html"):
    html = f"""
    <!DOCTYPE html>
    <html>
    <head><title>System Report</title></head>
    <body>
        <h1>System Report - {datetime.now()}</h1>
        <table border="1">
            <tr><th>Metric</th><th>Value</th></tr>
    """
    
    for key, value in data.items():
        html += f"<tr><td>{key}</td><td>{value}</td></tr>"
    
    html += """
        </table>
    </body>
    </html>
    """
    
    with open(filename, 'w') as f:
        f.write(html)
    
    print(f"✓ Отчёт создан: {filename}")

data = {"CPU": "45%", "RAM": "70%", "Disk": "55%"}
generate_report(data)
```

### 48. Ротация логов
```python
import os
import gzip
import shutil
from datetime import datetime

def rotate_logs(log_file, max_size_mb=100):
    if not os.path.exists(log_file):
        return
    
    size_mb = os.path.getsize(log_file) / (1024 * 1024)
    
    if size_mb > max_size_mb:
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        archived = f"{log_file}.{timestamp}.gz"
        
        with open(log_file, 'rb') as f_in:
            with gzip.open(archived, 'wb') as f_out:
                shutil.copyfileobj(f_in, f_out)
        
        open(log_file, 'w').close()  # Очистить оригинал
        print(f"✓ Лог заархивирован: {archived}")

rotate_logs("/var/log/app.log")
```

### 49. Проверка соответствия версий
```python
import subprocess
import re

def check_version(command, required_version):
    result = subprocess.run(command, capture_output=True, text=True)
    
    # Извлечение версии из вывода
    match = re.search(r'(\d+)\.(\d+)\.(\d+)', result.stdout)
    if match:
        current = tuple(map(int, match.groups()))
        required = tuple(map(int, required_version.split('.')))
        
        if current >= required:
            print(f"✓ Версия соответствует: {'.'.join(map(str, current))}")
            return True
        else:
            print(f"✗ Требуется обновление: {'.'.join(map(str, current))} < {required_version}")
            return False

check_version(["docker", "--version"], "20.10.0")
check_version(["kubectl", "version", "--client", "--short"], "1.20.0")
```

### 50. Комплексная проверка инфраструктуры
```python
import requests
import psutil
import subprocess

def infrastructure_health_check():
    report = {"timestamp": datetime.now().isoformat(), "checks": {}}
    
    # Системные ресурсы
    report["checks"]["cpu"] = {
        "value": psutil.cpu_percent(interval=1),
        "status": "OK" if psutil.cpu_percent() < 80 else "WARNING"
    }
    
    report["checks"]["memory"] = {
        "value": psutil.virtual_memory().percent,
        "status": "OK" if psutil.virtual_memory().percent < 80 else "WARNING"
    }
    
    report["checks"]["disk"] = {
        "value": psutil.disk_usage('/').percent,
        "status": "OK" if psutil.disk_usage('/').percent < 80 else "WARNING"
    }
    
    # Проверка сервисов
    services_to_check = [
        {"name": "nginx", "url": "http://localhost:80"},
        {"name": "api", "url": "http://localhost:8000/health"}
    ]
    
    for service in services_to_check:
        try:
            r = requests.get(service["url"], timeout=5)
            report["checks"][service["name"]] = {
                "status": "OK" if r.status_code == 200 else "ERROR",
                "response_time": r.elapsed.total_seconds()
            }
        except Exception as e:
            report["checks"][service["name"]] = {
                "status": "ERROR",
                "error": str(e)
            }
    
    # Вывод отчёта
    print("=" * 50)
    print("INFRASTRUCTURE HEALTH CHECK")
    print("=" * 50)
    for check, data in report["checks"].items():
        status_icon = "✓" if data["status"] == "OK" else "✗"
        print(f"{status_icon} {check}: {data['status']}")
        if "value" in data:
            print(f"  Value: {data['value']}")
        if "error" in data:
            print(f"  Error: {data['error']}")
    
    return report

infrastructure_health_check()
```

---

## 📚 Дополнительные полезные скрипты

### 51. Проверка истечения доменов
```python
import whois
from datetime import datetime

def check_domain_expiry(domain):
    try:
        w = whois.whois(domain)
        expiry_date = w.expiration_date
        
        if isinstance(expiry_date, list):
            expiry_date = expiry_date[0]
        
        days_left = (expiry_date - datetime.now()).days
        print(f"{domain} истекает через {days_left} дней")
        
        if days_left < 30:
            print("⚠️ Домен скоро истечёт!")
    except Exception as e:
        print(f"✗ Ошибка: {e}")

check_domain_expiry("example.com")
```

### 52. Мониторинг изменений в Git репозитории
```python
import subprocess
import time

def watch_git_changes(repo_path, interval=60):
    os.chdir(repo_path)
    
    while True:
        result = subprocess.run(
            ["git", "fetch", "origin"],
            capture_output=True, text=True
        )
        
        status = subprocess.run(
            ["git", "status", "-uno"],
            capture_output=True, text=True
        )
        
        if "behind" in status.stdout:
            print("⚠️ Обнаружены новые коммиты!")
            print(status.stdout)
        
        time.sleep(interval)

watch_git_changes("/path/to/repo")
```

### 53. Автоматическое создание issue в Jira
```python
import requests

def create_jira_issue(url, email, api_token, project_key, summary, description):
    endpoint = f"{url}/rest/api/3/issue"
    
    headers = {
        "Accept": "application/json",
        "Content-Type": "application/json"
    }
    
    payload = {
        "fields": {
            "project": {"key": project_key},
            "summary": summary,
            "description": {
                "type": "doc",
                "version": 1,
                "content": [{
                    "type": "paragraph",
                    "content": [{"type": "text", "text": description}]
                }]
            },
            "issuetype": {"name": "Bug"}
        }
    }
    
    response = requests.post(
        endpoint,
        json=payload,
        headers=headers,
        auth=(email, api_token)
    )
    
    if response.status_code == 201:
        issue = response.json()
        print(f"✓ Issue создан: {issue['key']}")
    else:
        print(f"✗ Ошибка: {response.text}")

create_jira_issue(
    "https://your-domain.atlassian.net",
    "email@example.com",
    "api_token",
    "PROJ",
    "Server Down",
    "Production server is not responding"
)
```

### 54. Мониторинг использования API квот
```python
import requests

def check_api_quota(api_url, headers):
    response = requests.get(api_url, headers=headers)
    
    if 'X-RateLimit-Remaining' in response.headers:
        remaining = int(response.headers['X-RateLimit-Remaining'])
        limit = int(response.headers['X-RateLimit-Limit'])
        
        percent_used = ((limit - remaining) / limit) * 100
        print(f"API Quota: {remaining}/{limit} осталось ({percent_used:.1f}% использовано)")
        
        if percent_used > 80:
            print("⚠️ Квота почти исчерпана!")

headers = {"Authorization": "Bearer YOUR_TOKEN"}
check_api_quota("https://api.github.com/rate_limit", headers)
```

### 55. Автоматическая очистка старых Docker образов
```python
import subprocess
from datetime import datetime, timedelta

def cleanup_old_images(days=30):
    result = subprocess.run(
        ["docker", "images", "--format", "{{.ID}}|{{.CreatedAt}}"],
        capture_output=True, text=True
    )
    
    cutoff_date = datetime.now() - timedelta(days=days)
    
    for line in result.stdout.strip().split('\n'):
        if line:
            image_id, created_at = line.split('|')
            # Парсинг даты (формат может отличаться)
            try:
                created = datetime.strptime(created_at.split()[0], "%Y-%m-%d")
                if created < cutoff_date:
                    subprocess.run(["docker", "rmi", "-f", image_id])
                    print(f"✓ Удалён старый образ: {image_id}")
            except:
                pass

cleanup_old_images(30)
```

---

## 🛠️ Установка всех зависимостей

### requirements.txt
```text
# Системные и мониторинг
psutil>=5.9.0
requests>=2.31.0
python-dotenv>=1.0.0

# Docker & Kubernetes
docker>=6.1.0
kubernetes>=27.2.0

# Базы данных
psycopg2-binary>=2.9.0
pymysql>=1.1.0
redis>=4.6.0

# Security
cryptography>=41.0.0
paramiko>=3.3.0

# Web & API
beautifulsoup4>=4.12.0
selenium>=4.12.0
feedparser>=6.0.10

# Messaging
python-telegram-bot>=20.5
slack-sdk>=3.22.0

# Monitoring
prometheus-client>=0.17.0
speedtest-cli>=2.1.3

# YAML & Config
pyyaml>=6.0.1
python-whois>=0.8.0

# Testing
yt-dlp>=2023.7.6
nltk>=3.8.1

# CI/CD
jira>=3.5.0
```

### Установка:
```bash
pip install -r requirements.txt
```

---

## 🚀 Готовые шаблоны для автоматизации

### Универсальный скрипт мониторинга
```python
#!/usr/bin/env python3
"""
Универсальный скрипт мониторинга инфраструктуры
Запуск: python monitor.py --config config.yaml
"""

import yaml
import requests
import psutil
import argparse
from datetime import datetime

class InfrastructureMonitor:
    def __init__(self, config_file):
        with open(config_file, 'r') as f:
            self.config = yaml.safe_load(f)
    
    def check_system_resources(self):
        """Проверка системных ресурсов"""
        return {
            "cpu": psutil.cpu_percent(interval=1),
            "memory": psutil.virtual_memory().percent,
            "disk": psutil.disk_usage('/').percent
        }
    
    def check_services(self):
        """Проверка HTTP сервисов"""
        results = {}
        for service in self.config.get('services', []):
            try:
                r = requests.get(service['url'], timeout=5)
                results[service['name']] = {
                    "status": "OK" if r.status_code == 200 else "ERROR",
                    "response_time": r.elapsed.total_seconds()
                }
            except Exception as e:
                results[service['name']] = {"status": "ERROR", "error": str(e)}
        return results
    
    def send_alert(self, message):
        """Отправка алерта"""
        if 'slack_webhook' in self.config:
            requests.post(
                self.config['slack_webhook'],
                json={"text": message}
            )
    
    def run(self):
        """Запуск мониторинга"""
        print(f"[{datetime.now()}] Starting monitoring...")
        
        resources = self.check_system_resources()
        services = self.check_services()
        
        # Проверка порогов
        alerts = []
        if resources['cpu'] > self.config.get('cpu_threshold', 80):
            alerts.append(f"⚠️ High CPU: {resources['cpu']}%")
        
        if resources['memory'] > self.config.get('memory_threshold', 80):
            alerts.append(f"⚠️ High Memory: {resources['memory']}%")
        
        for name, data in services.items():
            if data['status'] != "OK":
                alerts.append(f"⚠️ Service {name} is down!")
        
        # Отправка алертов
        if alerts:
            for alert in alerts:
                print(alert)
                self.send_alert(alert)
        else:
            print("✓ All systems operational")

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument('--config', default='monitor_config.yaml')
    args = parser.parse_args()
    
    monitor = InfrastructureMonitor(args.config)
    monitor.run()
```

### Конфигурационный файл (monitor_config.yaml)
```yaml
# Пороги для алертов
cpu_threshold: 80
memory_threshold: 80
disk_threshold: 85

# Сервисы для проверки
services:
  - name: webapp
    url: https://example.com
  - name: api
    url: https://api.example.com/health
  - name: database
    url: http://localhost:5432

# Slack webhook для уведомлений
slack_webhook: https://hooks.slack.com/services/YOUR/WEBHOOK/URL
```

---

## 🔧 Cron задачи для автоматизации

```bash
# Редактировать crontab
crontab -e

# Каждые 5 минут - проверка сервисов
*/5 * * * * /usr/bin/python3 /opt/scripts/monitor.py

# Каждый час - очистка логов
0 * * * * /usr/bin/python3 /opt/scripts/cleanup_logs.py

# Каждый день в 2:00 - бэкап базы данных
0 2 * * * /usr/bin/python3 /opt/scripts/backup_db.py

# Каждый день в 3:00 - проверка SSL сертификатов
0 3 * * * /usr/bin/python3 /opt/scripts/check_ssl.py

# Каждую неделю в воскресенье - очистка Docker
0 1 * * 0 /usr/bin/python3 /opt/scripts/docker_cleanup.py

# Каждый месяц 1-го числа - проверка доменов
0 9 1 * * /usr/bin/python3 /opt/scripts/check_domains.py
```

---

## 📊 Создание dashboard для мониторинга

```python
from flask import Flask, render_template, jsonify
import psutil
import subprocess

app = Flask(__name__)

@app.route('/')
def dashboard():
    return render_template('dashboard.html')

@app.route('/api/metrics')
def metrics():
    return jsonify({
        "cpu": psutil.cpu_percent(interval=1),
        "memory": psutil.virtual_memory().percent,
        "disk": psutil.disk_usage('/').percent,
        "containers": len(subprocess.run(
            ["docker", "ps", "-q"],
            capture_output=True, text=True
        ).stdout.strip().split('\n'))
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

## 💡 Best Practices

1. **Логирование**: Всегда логируйте действия скриптов
2. **Обработка ошибок**: Используйте try-except блоки
3. **Конфигурация**: Выносите настройки в отдельные файлы
4. **Секреты**: Используйте переменные окружения для паролей
5. **Мониторинг**: Добавляйте метрики в Prometheus
6. **Алерты**: Настройте уведомления в Slack/Telegram
7. **Документация**: Документируйте каждый скрипт
8. **Версионирование**: Храните скрипты в Git
9. **Тестирование**: Тестируйте на dev окружении
10. **Автоматизация**: Используйте cron или systemd timers

---

## 🎯 Полезные команды

### Проверка синтаксиса Python
```bash
python3 -m py_compile script.py
```

### Запуск с виртуальным окружением
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Создание исполняемого файла
```bash
chmod +x script.py
# Добавить в начало: #!/usr/bin/env python3
```

### Фоновый запуск
```bash
nohup python3 script.py &
```

### Запуск через systemd
```ini
[Unit]
Description=My Python Script
After=network.target

[Service]
Type=simple
User=devops
ExecStart=/usr/bin/python3 /opt/scripts/monitor.py
Restart=always

[Install]
WantedBy=multi-user.target
```

---

## 🚨 Troubleshooting

### ModuleNotFoundError
```bash
pip3 install <module_name>
# или
python3 -m pip install <module_name>
```

### Permission Denied
```bash
chmod +x script.py
# или запуск с sudo
sudo python3 script.py
```

### Encoding Issues
```python
# Добавить в начало файла
# -*- coding: utf-8 -*-
```

### Memory Issues
```python
# Использовать генераторы вместо списков
# Освобождать ресурсы явно
import gc
gc.collect()
```

---

Все скрипты готовы к использованию! 🎉