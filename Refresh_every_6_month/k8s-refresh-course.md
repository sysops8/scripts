# Kubernetes Refresh: Ежегодный/Полугодовой курс для DevOps

**Цель:** Освежить в памяти ключевые концепции Kubernetes за 2-3 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый раздел состоит из:
1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Реальная задача, которую нужно решить
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

**Предварительные требования:**
- Доступ к K8s кластеру (minikube, kind, k3s или облачный)
- kubectl установлен и настроен
- Базовые знания YAML

---

## Модуль 1: Базовая архитектура и kubectl (20 минут)

### 🎯 Напоминалка

**Архитектура Kubernetes:**
```
Control Plane:
├── API Server      # Точка входа для всех операций
├── etcd            # Распределенное хранилище состояния
├── Scheduler       # Распределение Pod'ов по нодам
├── Controller Mgr  # Контроллеры для различных ресурсов
└── Cloud Ctrl Mgr  # Интеграция с облачными провайдерами

Worker Nodes:
├── kubelet         # Агент на каждой ноде
├── kube-proxy      # Сетевой прокси
└── Container RT    # Docker/containerd/CRI-O
```

**Основные команды kubectl:**
```bash
# Информация о кластере
kubectl cluster-info
kubectl get nodes
kubectl version

# Работа с ресурсами
kubectl get pods                    # Список Pod'ов
kubectl get pods -A                 # Все namespace'ы
kubectl get pods -o wide            # Больше информации
kubectl get pods -o yaml            # YAML формат
kubectl describe pod <name>         # Детальная информация

# Создание и удаление
kubectl apply -f manifest.yaml      # Создать/обновить
kubectl delete -f manifest.yaml     # Удалить
kubectl delete pod <name>           # Удалить по имени
kubectl delete pods --all           # Удалить все Pod'ы

# Logs и debugging
kubectl logs <pod-name>             # Логи
kubectl logs -f <pod-name>          # Следить за логами
kubectl logs <pod> -c <container>   # Логи конкретного контейнера
kubectl exec -it <pod> -- /bin/bash # Войти в контейнер
kubectl port-forward <pod> 8080:80  # Пробросить порт

# Работа с namespace
kubectl get ns                      # Список namespace'ов
kubectl create ns dev               # Создать namespace
kubectl config set-context --current --namespace=dev  # Переключить

# Краткие алиасы
k get po                           # pods
k get svc                          # services
k get deploy                       # deployments
k get rs                           # replicasets
k get cm                           # configmaps
k get secrets                      # secrets
k get ing                          # ingress

# Полезные флаги
-n <namespace>                     # Указать namespace
-A, --all-namespaces              # Все namespace'ы
-l key=value                       # Фильтр по label
--field-selector                   # Фильтр по полю
-w, --watch                        # Следить за изменениями
```

**kubectl конфигурация:**
```bash
# Контексты
kubectl config get-contexts        # Список контекстов
kubectl config current-context     # Текущий контекст
kubectl config use-context <name>  # Переключить контекст

# Просмотр конфига
kubectl config view                # Весь конфиг
cat ~/.kube/config                 # Файл конфигурации
```

**Алиасы для ускорения (добавь в ~/.bashrc или ~/.zshrc):**
```bash
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deploy'
alias kaf='kubectl apply -f'
alias kdel='kubectl delete'
alias kl='kubectl logs'
alias kx='kubectl exec -it'
alias kctx='kubectl config use-context'
```

### 💻 Задание

Подготовь тестовое окружение:
1. Убедись, что kubectl работает: `kubectl version`
2. Проверь состояние кластера: `kubectl get nodes`
3. Создай namespace `test-refresh`: `kubectl create ns test-refresh`
4. Переключись на созданный namespace
5. Запусти тестовый Pod с nginx:
   ```bash
   kubectl run nginx --image=nginx:alpine -n test-refresh
   ```
6. Проверь, что Pod запустился: `kubectl get pods -n test-refresh`
7. Посмотри логи: `kubectl logs nginx -n test-refresh`
8. Войди в Pod: `kubectl exec -it nginx -n test-refresh -- sh`
9. Проброс порта: `kubectl port-forward nginx 8080:80 -n test-refresh`
10. Открой браузер на `localhost:8080` и проверь nginx

### 🚀 Бонус (новое)

Настрой быстрое переключение между namespace с помощью утилиты **kubens** (из пакета kubectx):
```bash
# Установка
brew install kubectx  # Mac
# или
curl -LO https://github.com/ahmetb/kubectx/releases/download/v0.9.5/kubens
chmod +x kubens && sudo mv kubens /usr/local/bin/

# Использование
kubens                    # Список namespace'ов
kubens test-refresh      # Переключиться
kubens -                 # Вернуться к предыдущему
```

Настрой kubectl plugins: установи **krew** и попробуй плагины `tree`, `ctx`, `ns`:
```bash
kubectl krew install tree ctx ns
kubectl tree deployment nginx
```

---

## Модуль 2: Pods и основные манифесты (25 минут)

### 🎯 Напоминалка

**Pod - минимальная единица развертывания:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: default
  labels:
    app: myapp
    env: dev
  annotations:
    description: "My test pod"
spec:
  containers:
  - name: main-container
    image: nginx:1.21
    ports:
    - containerPort: 80
      name: http
    env:
    - name: ENV_VAR
      value: "some-value"
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 5
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 3
  - name: sidecar
    image: busybox
    command: ['sh', '-c', 'while true; do echo hello; sleep 10; done']
  restartPolicy: Always  # Always, OnFailure, Never
```

**Основные поля манифеста:**
```yaml
apiVersion: v1              # API версия
kind: Pod                   # Тип ресурса
metadata:                   # Метаданные
  name: string              # Имя (обязательно)
  namespace: string         # Namespace
  labels:                   # Labels для селекторов
    key: value
  annotations:              # Аннотации (не используются для селекторов)
    key: value
spec:                       # Спецификация
  containers: []            # Список контейнеров (обязательно)
  volumes: []               # Volumes
  initContainers: []        # Init контейнеры
  nodeSelector: {}          # Селектор нод
  affinity: {}              # Affinity правила
  tolerations: []           # Tolerations
```

**Ресурсы (requests/limits):**
```yaml
resources:
  requests:           # Минимум, гарантированный контейнеру
    memory: "64Mi"    # 64 мегабайта
    cpu: "250m"       # 0.25 CPU (250 milliCPU)
  limits:             # Максимум, который может использовать
    memory: "128Mi"
    cpu: "500m"
```

**Единицы измерения:**
```yaml
Memory: Ki, Mi, Gi, Ti  # KiB, MiB, GiB, TiB
CPU: m (milliCPU)       # 1000m = 1 CPU
```

**Health Checks:**
```yaml
livenessProbe:        # Проверка живости (restart при сбое)
  httpGet:            # HTTP проба
    path: /health
    port: 8080
  # или
  exec:               # Команда
    command: ['cat', '/tmp/healthy']
  # или
  tcpSocket:          # TCP проба
    port: 8080
  initialDelaySeconds: 10  # Задержка перед первой проверкой
  periodSeconds: 5         # Период между проверками
  timeoutSeconds: 1        # Таймаут
  failureThreshold: 3      # Количество неудач для рестарта

readinessProbe:       # Проверка готовности (исключение из Service)
  # Аналогичная структура
```

**Multi-container Pod паттерны:**
```yaml
# Sidecar - вспомогательный контейнер (логи, прокси)
# Ambassador - прокси для внешних сервисов
# Adapter - преобразование данных
```

### 💻 Задание

Создай манифест `web-app-pod.yaml`:
1. Pod с именем `web-app`
2. Два контейнера:
   - **Основной**: nginx:alpine
     - Port: 80
     - Liveness probe: HTTP GET на / каждые 10 секунд
     - Readiness probe: HTTP GET на / каждые 5 секунд
     - Resources: requests (cpu: 100m, memory: 64Mi), limits (cpu: 200m, memory: 128Mi)
   - **Sidecar**: busybox
     - Команда: бесконечный цикл, который каждые 30 секунд выводит дату в логи
3. Labels: `app=web`, `tier=frontend`
4. Примени манифест и проверь:
   ```bash
   kubectl apply -f web-app-pod.yaml
   kubectl get pods -l app=web
   kubectl describe pod web-app
   kubectl logs web-app -c <container-name>
   ```

### 🚀 Бонус (новое)

Добавь **Init Container**, который:
1. Ждет доступности внешнего сервиса перед стартом основных контейнеров
2. Использует `busybox` и команду:
   ```yaml
   initContainers:
   - name: init-check
     image: busybox:1.35
     command: ['sh', '-c', 'until nslookup kubernetes.default; do echo waiting for k8s; sleep 2; done']
   ```
3. Наблюдай за последовательностью запуска: `kubectl get pods -w`

Добавь **Startup Probe** для медленно стартующих приложений (доступно с K8s 1.20+):
```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

---

## Модуль 3: Deployments и ReplicaSets (30 минут)

### 🎯 Напоминалка

**Deployment - декларативное управление Pod'ами:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3                    # Количество реплик
  selector:                      # Селектор Pod'ов (обязательно)
    matchLabels:
      app: nginx
  strategy:                      # Стратегия обновления
    type: RollingUpdate          # RollingUpdate или Recreate
    rollingUpdate:
      maxSurge: 1                # Максимум дополнительных Pod'ов
      maxUnavailable: 0          # Максимум недоступных Pod'ов
  template:                      # Шаблон Pod'а
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
```

**Основные команды:**
```bash
# Создание
kubectl create deployment nginx --image=nginx:1.21 --replicas=3
kubectl apply -f deployment.yaml

# Просмотр
kubectl get deployments
kubectl get rs                   # ReplicaSets
kubectl describe deployment nginx

# Масштабирование
kubectl scale deployment nginx --replicas=5
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=80

# Обновление
kubectl set image deployment/nginx nginx=nginx:1.22
kubectl rollout status deployment/nginx
kubectl rollout history deployment/nginx

# Откат
kubectl rollout undo deployment/nginx
kubectl rollout undo deployment/nginx --to-revision=2

# Пауза/возобновление
kubectl rollout pause deployment/nginx
kubectl rollout resume deployment/nginx

# Удаление
kubectl delete deployment nginx
```

**Стратегии обновления:**
```yaml
# RollingUpdate (по умолчанию)
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1           # Можно создать +1 Pod сверх replicas
    maxUnavailable: 0     # 0 Pod'ов может быть недоступно

# Recreate (удалить все, затем создать новые)
strategy:
  type: Recreate
```

**Селекторы:**
```yaml
# matchLabels - точное совпадение
selector:
  matchLabels:
    app: nginx
    env: prod

# matchExpressions - более сложные условия
selector:
  matchExpressions:
  - key: app
    operator: In        # In, NotIn, Exists, DoesNotExist
    values: [nginx, apache]
  - key: tier
    operator: Exists
```

**ReplicaSet** (обычно не используется напрямую):
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    # Pod template
```

### 💻 Задание

Создай production-ready Deployment:
1. Создай файл `api-deployment.yaml`:
   - Название: `api-server`
   - Replicas: 3
   - Image: `nginx:1.21-alpine` (будем имитировать API сервер)
   - Labels: `app=api`, `version=v1`
   - Resources: requests (cpu: 100m, memory: 128Mi), limits (cpu: 200m, memory: 256Mi)
   - Liveness probe: HTTP GET /health на порту 80
   - Readiness probe: HTTP GET /ready на порту 80
   - Rolling update: maxSurge=1, maxUnavailable=0

2. Примени Deployment: `kubectl apply -f api-deployment.yaml`

3. Проверь статус:
   ```bash
   kubectl get deployments
   kubectl get rs
   kubectl get pods -l app=api
   ```

4. Выполни rolling update:
   ```bash
   kubectl set image deployment/api-server nginx=nginx:1.22-alpine
   kubectl rollout status deployment/api-server
   ```

5. Посмотри историю: `kubectl rollout history deployment/api-server`

6. Откати обновление: `kubectl rollout undo deployment/api-server`

7. Масштабируй: `kubectl scale deployment api-server --replicas=5`

### 🚀 Бонус (новое)

Реализуй **Blue-Green deployment** вручную:
```bash
# Создай "blue" версию
kubectl apply -f api-deployment-blue.yaml  # version=blue, replicas=3

# Переключи Service на blue (мы создадим Service в следующем модуле)

# Создай "green" версию
kubectl apply -f api-deployment-green.yaml  # version=green, replicas=3

# Проверь обе версии
kubectl get pods -l app=api

# Переключи Service на green
kubectl patch service api-service -p '{"spec":{"selector":{"version":"green"}}}'

# Удали blue
kubectl delete deployment api-server-blue
```

Используй **Deployment Annotations** для автоматизации:
```yaml
metadata:
  annotations:
    kubernetes.io/change-cause: "Update to version 1.22"
```

Попробуй **kubectl diff** перед применением изменений:
```bash
kubectl diff -f deployment.yaml
```

---

## Модуль 4: Services и сетевое взаимодействие (30 минут)

### 🎯 Напоминалка

**Service - абстракция для доступа к Pod'ам:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:              # Селектор Pod'ов
    app: myapp
  type: ClusterIP        # ClusterIP, NodePort, LoadBalancer, ExternalName
  ports:
  - name: http
    protocol: TCP
    port: 80             # Порт Service
    targetPort: 8080     # Порт контейнера
  sessionAffinity: None  # None или ClientIP
```

**Типы Service:**

**1. ClusterIP** (по умолчанию) - внутренний IP:
```yaml
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 8080
```

**2. NodePort** - доступ через порт на каждой ноде:
```yaml
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080    # 30000-32767, опционально
```

**3. LoadBalancer** - внешний балансировщик (облачные провайдеры):
```yaml
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
```

**4. ExternalName** - CNAME для внешнего сервиса:
```yaml
spec:
  type: ExternalName
  externalName: external-service.example.com
```

**Headless Service** (без ClusterIP):
```yaml
spec:
  clusterIP: None      # Для прямого доступа к Pod'ам
  selector:
    app: myapp
```

**Основные команды:**
```bash
# Создание
kubectl expose deployment nginx --port=80 --target-port=8080
kubectl apply -f service.yaml

# Просмотр
kubectl get services
kubectl get svc
kubectl describe service nginx
kubectl get endpoints    # Список endpoints

# Тестирование
kubectl run test --image=busybox -it --rm -- wget -O- http://service-name
kubectl port-forward svc/my-service 8080:80
```

**DNS в Kubernetes:**
```bash
# Формат DNS имени
<service-name>.<namespace>.svc.cluster.local

# Примеры
my-service.default.svc.cluster.local
api-server.prod.svc.cluster.local

# Короткие формы (в том же namespace)
my-service
my-service.default
```

**Endpoints:**
```yaml
# Автоматически создаются для Service
# Содержат IP адреса Pod'ов
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
- addresses:
  - ip: 10.244.1.5
  - ip: 10.244.2.7
  ports:
  - port: 8080
```

### 💻 Задание

Настрой сетевое взаимодействие между сервисами:

1. **Создай backend Deployment** (`backend-deployment.yaml`):
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: backend
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: backend
     template:
       metadata:
         labels:
           app: backend
       spec:
         containers:
         - name: backend
           image: nginx:alpine
           ports:
           - containerPort: 80
   ```

2. **Создай backend Service** (`backend-service.yaml`):
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: backend-service
   spec:
     selector:
       app: backend
     ports:
     - port: 80
       targetPort: 80
   ```

3. **Создай frontend Pod** для тестирования:
   ```bash
   kubectl run frontend --image=busybox -it --rm -- sh
   # Внутри Pod'а:
   wget -O- http://backend-service
   nslookup backend-service
   ```

4. **Создай NodePort Service** для внешнего доступа:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: backend-nodeport
   spec:
     type: NodePort
     selector:
       app: backend
     ports:
     - port: 80
       targetPort: 80
       nodePort: 30080
   ```

5. Проверь доступ:
   ```bash
   # Получи IP ноды
   kubectl get nodes -o wide
   
   # Обратись к сервису
   curl http://<node-ip>:30080
   
   # Или через minikube
   minikube service backend-nodeport --url
   ```

6. Проверь endpoints:
   ```bash
   kubectl get endpoints backend-service
   kubectl describe endpoints backend-service
   ```

### 🚀 Бонус (новое)

Настрой **Service Mesh light** с помощью встроенных возможностей:

1. **Session Affinity** (sticky sessions):
   ```yaml
   spec:
     sessionAffinity: ClientIP
     sessionAffinityConfig:
       clientIP:
         timeoutSeconds: 10800
   ```

2. **Topology Aware Routing** (K8s 1.21+):
   ```yaml
   spec:
     topologyKeys:
     - "kubernetes.io/hostname"
     - "topology.kubernetes.io/zone"
     - "*"
   ```

3. **ExternalTrafficPolicy** для сохранения source IP:
   ```yaml
   spec:
     type: LoadBalancer
     externalTrafficPolicy: Local  # или Cluster
   ```

4. Создай **Multi-Port Service**:
   ```yaml
   spec:
     ports:
     - name: http
       port: 80
       targetPort: 8080
     - name: https
       port: 443
       targetPort: 8443
     - name: metrics
       port: 9090
       targetPort: 9090
   ```

---

## Модуль 5: ConfigMaps и Secrets (25 минут)

### 🎯 Напоминалка

**ConfigMap - конфигурация приложений:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  # Простые key-value
  database_url: "postgres://db:5432/mydb"
  log_level: "info"
  
  # Файлы конфигурации
  nginx.conf: |
    server {
      listen 80;
      server_name localhost;
      location / {
        root /usr/share/nginx/html;
      }
    }
  
  app.properties: |
    server.port=8080
    spring.datasource.url=jdbc:postgresql://db:5432/mydb
```

**Создание ConfigMap:**
```bash
# Из литералов
kubectl create configmap app-config \
  --from-literal=database_url=postgres://db:5432/mydb \
  --from-literal=log_level=info

# Из файла
kubectl create configmap nginx-config --from-file=nginx.conf

# Из директории
kubectl create configmap app-configs --from-file=configs/

# Из env файла
kubectl create configmap env-config --from-env-file=app.env

# Из YAML
kubectl apply -f configmap.yaml
```

**Использование ConfigMap в Pod:**
```yaml
# Как переменные окружения
spec:
  containers:
  - name: app
    image: myapp
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    
    # Все ключи из ConfigMap
    envFrom:
    - configMapRef:
        name: app-config
    
    # Как volume
    volumeMounts:
    - name: config
      mountPath: /etc/config
      readOnly: true
  
  volumes:
  - name: config
    configMap:
      name: app-config
      items:                    # Опционально: выбрать конкретные ключи
      - key: nginx.conf
        path: nginx.conf
```

**Secret - чувствительные данные:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque                    # Opaque, kubernetes.io/tls, docker-registry
data:
  # Base64 encoded values
  username: YWRtaW4=            # admin
  password: cGFzc3dvcmQxMjM=    # password123
stringData:                     # Автоматически кодируется в base64
  api-key: "my-secret-api-key"
```

**Типы Secret:**
```yaml
# Generic (Opaque)
type: Opaque

# TLS сертификаты
type: kubernetes.io/tls
data:
  tls.crt: <base64 cert>
  tls.key: <base64 key>

# Docker registry
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64 json>

# Basic auth
type: kubernetes.io/basic-auth
data:
  username: <base64>
  password: <base64>
```

**Создание Secret:**
```bash
# Из литералов
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=password123

# Из файлов
kubectl create secret generic tls-secret \
  --from-file=tls.crt \
  --from-file=tls.key

# TLS secret
kubectl create secret tls tls-secret \
  --cert=tls.crt \
  --key=tls.key

# Docker registry
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass \
  --docker-email=user@example.com

# Из YAML
kubectl apply -f secret.yaml
```

**Использование Secret:**
```yaml
# Как переменные окружения
spec:
  containers:
  - name: app
    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    
    # Как volume
    volumeMounts:
    - name: secret
      mountPath: /etc/secret
      readOnly: true
  
  volumes:
  - name: secret
    secret:
      secretName: db-secret
  
  # ImagePullSecrets
  imagePullSecrets:
  - name: regcred
```

**Команды для работы:**
```bash
# Просмотр
kubectl get configmaps
kubectl get secrets
kubectl describe configmap app-config
kubectl get secret db-secret -o yaml

# Декодирование secret
kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 -d

# Редактирование
kubectl edit configmap app-config
kubectl edit secret db-secret

# Удаление
kubectl delete configmap app-config
kubectl delete secret db-secret
```

### 💻 Задание

Создай приложение с конфигурацией и секретами:

1. **Создай ConfigMap** (`app-config.yaml`):
   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: app-config
   data:
     APP_ENV: "production"
     LOG_LEVEL: "info"
     REDIS_HOST: "redis-service"
     config.json: |
       {
         "server": {
           "port": 8080,
           "timeout": 30
         },
         "features": {
           "auth": true,
           "cache": true
         }
       }
   ```

2. **Создай Secret** (`db-secret.yaml`):
   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: db-secret
   type: Opaque
   stringData:
     DB_USERNAME: "admin"
     DB_PASSWORD: "SuperSecret123!"
     DB_HOST: "postgres-service"
   ```

3. **Создай Deployment**, использующий ConfigMap и Secret:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: myapp
     template:
       metadata:
         labels:
           app: myapp
       spec:
         containers:
         - name: app
           image: nginx:alpine
           # Env переменные из ConfigMap
           env:
           - name: APP_ENV
             valueFrom:
               configMapKeyRef:
                 name: app-config
                 key: APP_ENV
           # Env переменные из Secret
           - name: DB_USERNAME
             valueFrom:
               secretKeyRef:
                 name: db-secret
                 key: DB_USERNAME
           - name: DB_PASSWORD
             valueFrom:
               secretKeyRef:
                 name: db-secret
                 key: DB_PASSWORD
           # ConfigMap как volume
           volumeMounts:
           - name: config
             mountPath: /etc/config
             readOnly: true
         volumes:
         - name: config
           configMap:
             name: app-config
             items:
             - key: config.json
               path: config.json
   ```

4. Примени все манифесты:
   ```bash
   kubectl apply -f app-config.yaml
   kubectl apply -f db-secret.yaml
   kubectl apply -f app-deployment.yaml
   ```

5. Проверь:
   ```bash
   # Войди в Pod
   kubectl exec -it <pod-name> -- sh
   
   # Проверь env переменные
   env | grep APP_ENV
   env | grep DB_
   
   # Проверь смонтированный файл
   cat /etc/config/config.json
   ```

6. Обнови ConfigMap и проверь, как изменения применяются:
   ```bash
   kubectl edit configmap app-config
   # Измени LOG_LEVEL на "debug"
   
   # Переменные окружения НЕ обновятся автоматически
   # Файлы в volume обновятся через ~60 секунд
   ```

### 🚀 Бонус (новое)

**1. Используй Kustomize для управления конфигурацией:**

Создай структуру:
```
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   └── configmap.yaml
├── overlays/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   └── configmap.yaml
│   └── prod/
│       ├── kustomization.yaml
│       └── configmap.yaml
```

`base/kustomization.yaml`:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- configmap.yaml
```

`overlays/dev/kustomization.yaml`:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
- ../../base
configMapGenerator:
- name: app-config
  behavior: merge
  literals:
  - APP_ENV=development
  - LOG_LEVEL=debug
```

Применить:
```bash
kubectl apply -k overlays/dev
kubectl apply -k overlays/prod
```

**2. Используй External Secrets Operator** для интеграции с внешними хранилищами (AWS Secrets Manager, HashiCorp Vault):
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: vault-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: db-secret
  data:
  - secretKey: username
    remoteRef:
      key: database/credentials
      property: username
```

**3. Используй Sealed Secrets** для безопасного хранения секретов в Git:
```bash
# Установка
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml

# Создание sealed secret
kubeseal --format yaml < secret.yaml > sealed-secret.yaml

# Sealed secret можно коммитить в Git
kubectl apply -f sealed-secret.yaml
```

---

## Модуль 6: Volumes и Persistent Storage (30 минут)

### 🎯 Напоминалка

**Типы Volumes:**

**1. emptyDir** - временное хранилище на ноде:
```yaml
volumes:
- name: cache
  emptyDir: {}
```

**2. hostPath** - директория на ноде (не рекомендуется для prod):
```yaml
volumes:
- name: logs
  hostPath:
    path: /var/log/myapp
    type: DirectoryOrCreate
```

**3. configMap/secret** - смонтировать как файлы:
```yaml
volumes:
- name: config
  configMap:
    name: app-config
```

**4. persistentVolumeClaim** - постоянное хранилище:
```yaml
volumes:
- name: data
  persistentVolumeClaim:
    claimName: my-pvc
```

**PersistentVolume (PV)** - абстракция хранилища:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain  # Retain, Delete, Recycle
  storageClassName: slow
  nfs:
    server: nfs-server.example.com
    path: /exported/path
```

**PersistentVolumeClaim (PVC)** - запрос на хранилище:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce         # RWO, ROX, RWX
  resources:
    requests:
      storage: 5Gi
  storageClassName: fast  # Опционально
```

**Access Modes:**
```yaml
ReadWriteOnce (RWO)  # Одна нода, чтение-запись
ReadOnlyMany (ROX)   # Множество нод, только чтение
ReadWriteMany (RWX)  # Множество нод, чтение-запись
```

**StorageClass** - динамическое создание PV:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iopsPerGB: "10"
  fsType: ext4
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

**Использование в Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: data
      mountPath: /data
    - name: cache
      mountPath: /cache
    - name: config
      mountPath: /etc/config
      readOnly: true
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: my-pvc
  - name: cache
    emptyDir: {}
  - name: config
    configMap:
      name: app-config
```

**Команды:**
```bash
# PersistentVolumes
kubectl get pv
kubectl describe pv <name>

# PersistentVolumeClaims
kubectl get pvc
kubectl describe pvc <name>

# StorageClasses
kubectl get storageclass
kubectl get sc

# Посмотреть, какой PVC используется Pod'ом
kubectl get pods -o custom-columns=NAME:.metadata.name,PVC:.spec.volumes[*].persistentVolumeClaim.claimName
```

### 💻 Задание

Создай StatefulSet с постоянным хранилищем:

1. **Создай StorageClass** (для локального тестирования):
   ```yaml
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: manual
   provisioner: kubernetes.io/no-provisioner
   volumeBindingMode: WaitForFirstConsumer
   ```

2. **Создай PersistentVolume**:
   ```yaml
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: pv-local
   spec:
     capacity:
       storage: 1Gi
     accessModes:
     - ReadWriteOnce
     persistentVolumeReclaimPolicy: Retain
     storageClassName: manual
     hostPath:
       path: /tmp/k8s-data
       type: DirectoryOrCreate
   ```

3. **Создай StatefulSet с PVC**:
   ```yaml
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: web
   spec:
     serviceName: web
     replicas: 2
     selector:
       matchLabels:
         app: web
     template:
       metadata:
         labels:
           app: web
       spec:
         containers:
         - name: nginx
           image: nginx:alpine
           ports:
           - containerPort: 80
           volumeMounts:
           - name: www
             mountPath: /usr/share/nginx/html
     volumeClaimTemplates:
     - metadata:
         name: www
       spec:
         accessModes: [ "ReadWriteOnce" ]
         storageClassName: manual
         resources:
           requests:
             storage: 100Mi
   ```

4. **Создай Headless Service** для StatefulSet:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: web
   spec:
     clusterIP: None
     selector:
       app: web
     ports:
     - port: 80
   ```

5. Примени и проверь:
   ```bash
   kubectl apply -f storageclass.yaml
   kubectl apply -f pv.yaml
   kubectl apply -f statefulset.yaml
   
   # Проверь создание
   kubectl get statefulset
   kubectl get pods -l app=web
   kubectl get pvc
   
   # Запиши данные в первый Pod
   kubectl exec web-0 -- sh -c 'echo "Hello from web-0" > /usr/share/nginx/html/index.html'
   
   # Проверь данные
   kubectl exec web-0 -- cat /usr/share/nginx/html/index.html
   
   # Удали Pod и проверь, что данные сохранились
   kubectl delete pod web-0
   kubectl exec web-0 -- cat /usr/share/nginx/html/index.html
   ```

### 🚀 Бонус (новое)

**1. Volume Snapshots** (если поддерживается):
```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: my-snapshot
spec:
  volumeSnapshotClassName: csi-snapclass
  source:
    persistentVolumeClaimName: my-pvc
```

**2. Расширение PVC** (если StorageClass позволяет):
```bash
kubectl patch pvc my-pvc -p '{"spec":{"resources":{"requests":{"storage":"10Gi"}}}}'
```

**3. CSI Drivers** для специфичных хранилищ (AWS EBS, GCE PD, Azure Disk):
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
```

**4. Ephemeral Volumes** (K8s 1.23+):
```yaml
volumes:
- name: scratch
  ephemeral:
    volumeClaimTemplate:
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 1Gi
```

---

## Модуль 7: Ingress и управление трафиком (30 минут)

### 🎯 Напоминалка

**Ingress** - HTTP/HTTPS маршрутизация:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
  tls:
  - hosts:
    - example.com
    secretName: tls-secret
```

**PathType:**
```yaml
Prefix      # Совпадение префикса (/api совпадет с /api/users)
Exact       # Точное совпадение
ImplementationSpecific  # Зависит от Ingress Controller
```

**Популярные Ingress Controllers:**
- **NGINX Ingress Controller** (самый популярный)
- **Traefik**
- **HAProxy Ingress**
- **Kong**
- **Istio Gateway**
- **AWS ALB Ingress Controller**

**Установка NGINX Ingress Controller:**
```bash
# Для minikube
minikube addons enable ingress

# Для обычного кластера
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml

# Проверка
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

**Аннотации NGINX Ingress:**
```yaml
metadata:
  annotations:
    # Rewrite
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    
    # CORS
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-methods: "GET, POST, PUT"
    
    # Rate limiting
    nginx.ingress.kubernetes.io/limit-rps: "10"
    
    # Auth
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    
    # SSL
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    
    # Timeout
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
    
    # WebSocket
    nginx.ingress.kubernetes.io/websocket-services: "ws-service"
    
    # Custom headers
    nginx.ingress.kubernetes.io/configuration-snippet: |
      more_set_headers "X-Custom-Header: value";
```

**TLS/SSL:**
```bash
# Создать TLS secret
kubectl create secret tls tls-secret \
  --cert=tls.crt \
  --key=tls.key

# Или с Let's Encrypt (cert-manager)
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml
```

**IngressClass:**
```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx
spec:
  controller: k8s.io/ingress-nginx
```

**Команды:**
```bash
# Просмотр
kubectl get ingress
kubectl describe ingress my-ingress
kubectl get ingressclass

# Тестирование
curl -H "Host: example.com" http://<ingress-ip>/

# Логи NGINX Ingress
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx -f
```

### 💻 Задание

Настрой Ingress для микросервисной архитектуры:

1. **Создай два Deployment + Service**:

Frontend:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
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
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
  - port: 80
```

Backend:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
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
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - port: 80
```

2. **Создай Ingress**:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /()(.*)
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 80
```

3. Примени все манифесты:
```bash
kubectl apply -f frontend.yaml
kubectl apply -f backend.yaml
kubectl apply -f ingress.yaml
```

4. Проверь Ingress:
```bash
kubectl get ingress
kubectl describe ingress app-ingress

# Получи IP Ingress Controller
kubectl get svc -n ingress-nginx

# Добавь в /etc/hosts
echo "<ingress-ip> myapp.local" | sudo tee -a /etc/hosts

# Тестируй
curl http://myapp.local/
curl http://myapp.local/api/health
```

5. Добавь второй host в Ingress:
```yaml
rules:
- host: myapp.local
  # ... существующие правила
- host: admin.myapp.local
  http:
    paths:
    - path: /
      pathType: Prefix
      backend:
        service:
          name: admin-service
          port:
            number: 80
```

### 🚀 Бонус (новое)

**1. Настрой автоматические TLS сертификаты с cert-manager:**
```bash
# Установка cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml

# ClusterIssuer для Let's Encrypt
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
EOF

# Добавь аннотацию в Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
```

**2. Canary Deployments с NGINX Ingress:**
```yaml
# Production Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-prod
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        backend:
          service:
            name: app-v1
            port:
              number: 80

---
# Canary Ingress (10% трафика)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-canary
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        backend:
          service:
            name: app-v2
            port:
              number: 80
```

**3. Rate Limiting и Auth:**
```yaml
metadata:
  annotations:
    # Rate limiting
    nginx.ingress.kubernetes.io/limit-rps: "10"
    nginx.ingress.kubernetes.io/limit-connections: "5"
    
    # Basic Auth
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"

# Создание basic auth secret
htpasswd -c auth admin
kubectl create secret generic basic-auth --from-file=auth
```

---

## Модуль 8: Jobs, CronJobs и DaemonSets (25 минут)

### 🎯 Напоминалка

**Job** - одноразовая задача:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: backup-job
spec:
  template:
    spec:
      containers:
      - name: backup
        image: postgres:14
        command: ['sh', '-c', 'pg_dump -h db -U user mydb > /backup/dump.sql']
        volumeMounts:
        - name: backup
          mountPath: /backup
      restartPolicy: OnFailure  # Never или OnFailure
      volumes:
      - name: backup
        persistentVolumeClaim:
          claimName: backup-pvc
  backoffLimit: 4              # Максимум попыток
  completions: 1               # Сколько Pod'ов должно успешно завершиться
  parallelism: 1               # Сколько Pod'ов запускать параллельно
  ttlSecondsAfterFinished: 86400  # Удалить через 24 часа
```

**Параллельные Job:**
```yaml
spec:
  completions: 10     # Нужно завершить 10 Pod'ов
  parallelism: 3      # Запускать по 3 одновременно
```

**CronJob** - периодические задачи:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-cronjob
spec:
  schedule: "0 2 * * *"        # Каждый день в 2:00 (cron формат)
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: postgres:14
            command: ['sh', '-c', 'pg_dump -h db -U user mydb']
          restartPolicy: OnFailure
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  concurrencyPolicy: Forbid    # Allow, Forbid, Replace
  startingDeadlineSeconds: 100
  suspend: false               # Приостановить CronJob
```

**Cron schedule формат:**
```
# ┌───────────── минута (0 - 59)
# │ ┌───────────── час (0 - 23)
# │ │ ┌───────────── день месяца (1 - 31)
# │ │ │ ┌───────────── месяц (1 - 12)
# │ │ │ │ ┌───────────── день недели (0 - 6) (воскресенье = 0)
# │ │ │ │ │
# * * * * *

"0 2 * * *"         # Каждый день в 2:00
"*/15 * * * *"      # Каждые 15 минут
"0 */2 * * *"       # Каждые 2 часа
"0 0 * * 0"         # Каждое воскресенье в полночь
"0 9 1 * *"         # Первое число каждого месяца в 9:00
"0 0 1 1 *"         # 1 января в полночь
```

**DaemonSet** - Pod на каждой ноде:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd:v1.14
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      tolerations:              # Запускать даже на master нодах
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
```

**Команды:**
```bash
# Jobs
kubectl get jobs
kubectl describe job backup-job
kubectl logs job/backup-job
kubectl delete job backup-job

# CronJobs
kubectl get cronjobs
kubectl describe cronjob backup-cronjob
kubectl get jobs --watch                    # Смотреть на создаваемые Jobs
kubectl create job --from=cronjob/backup-cronjob manual-backup  # Ручной запуск

# DaemonSets
kubectl get daemonsets -A
kubectl describe daemonset fluentd -n kube-system
kubectl rollout status daemonset fluentd -n kube-system
```

### 💻 Задание

Создай систему бэкапов и мониторинга:

1. **Создай Job для одноразового бэкапа**:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-backup-manual
spec:
  template:
    spec:
      containers:
      - name: backup
        image: busybox
        command:
        - /bin/sh
        - -c
        - |
          echo "Starting backup at $(date)"
          echo "Backing up database..."
          sleep 10
          echo "Backup completed at $(date)"
          echo "Backup saved to /backups/db-$(date +%Y%m%d-%H%M%S).sql"
      restartPolicy: Never
  backoffLimit: 3
```

2. **Создай CronJob для автоматических бэкапов**:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-backup-daily
spec:
  schedule: "0 2 * * *"  # Каждый день в 2:00
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: busybox
            command:
            - /bin/sh
            - -c
            - |
              echo "Automated backup at $(date)"
              echo "Database backup in progress..."
              sleep 5
              echo "Backup completed"
          restartPolicy: OnFailure
  successfulJobsHistoryLimit: 5
  failedJobsHistoryLimit: 2
  concurrencyPolicy: Forbid
```

3. **Создай DaemonSet для Node Exporter**:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      hostNetwork: true
      hostPID: true
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
          hostPort: 9100
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 100Mi
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
```

4. Примени и проверь:
```bash
# Создай namespace для мониторинга
kubectl create ns monitoring

# Примени манифесты
kubectl apply -f job-manual.yaml
kubectl apply -f cronjob-daily.yaml
kubectl apply -f daemonset-exporter.yaml

# Проверь Job
kubectl get jobs
kubectl logs job/db-backup-manual

# Проверь CronJob
kubectl get cronjobs
kubectl get jobs --watch

# Ручной запуск CronJob
kubectl create job --from=cronjob/db-backup-daily manual-backup-1

# Проверь DaemonSet
kubectl get daemonsets -n monitoring
kubectl get pods -n monitoring -o wide
```

### 🚀 Бонус (новое)

**1. Используй IndexedJob** (K8s 1.24+) для обработки очереди:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: indexed-job
spec:
  completions: 5
  parallelism: 3
  completionMode: Indexed  # Каждому Pod присваивается индекс
  template:
    spec:
      containers:
      - name: worker
        image: busybox
        command:
        - /bin/sh
        - -c
        - |
          echo "Processing item ${JOB_COMPLETION_INDEX}"
          # Обработка элемента по индексу
      restartPolicy: Never
```

**2. Suspend/Resume CronJob:**
```bash
# Приостановить
kubectl patch cronjob db-backup-daily -p '{"spec":{"suspend":true}}'

# Возобновить
kubectl patch cronjob db-backup-daily -p '{"spec":{"suspend":false}}'
```

**3. DaemonSet Update Strategies:**
```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1  # Обновлять по одной ноде
```

**4. Node Selector для DaemonSet** (запуск только на определенных нодах):
```yaml
spec:
  template:
    spec:
      nodeSelector:
        disktype: ssd
      # Или с помощью affinity
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/worker
                operator: Exists
```

---

## Модуль 9: RBAC и безопасность (30 минут)

### 🎯 Напоминалка

**RBAC (Role-Based Access Control):**

**ServiceAccount** - идентификатор для Pod'ов:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: default
```

**Role** - права в namespace:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]           # "" для core API
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "update", "patch"]
```

**ClusterRole** - права на уровне кластера:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["namespaces"]
  verbs: ["get", "list"]
```

**RoleBinding** - привязка Role к пользователю/SA:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**ClusterRoleBinding** - привязка ClusterRole:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: default
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

**Verbs (действия):**
```yaml
get, list, watch        # Чтение
create                  # Создание
update, patch           # Обновление
delete, deletecollection # Удаление
*                       # Все действия
```

**Resources:**
```yaml
pods, services, deployments, replicasets, statefulsets
configmaps, secrets, serviceaccounts
nodes, namespaces, persistentvolumes, persistentvolumeclaims
ingresses, networkpolicies
events, logs
```

**Использование ServiceAccount в Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: app-sa
  containers:
  - name: app
    image: myapp
```

**SecurityContext** - настройки безопасности Pod/Container:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:                # Pod-level
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: nginx
    securityContext:              # Container-level
      allowPrivilegeEscalation: false
      runAsNonRoot: true
      runAsUser: 1000
      capabilities:
        drop:
        - ALL
        add:
        - NET_BIND_SERVICE
      readOnlyRootFilesystem: true
    volumeMounts:
    - name: cache
      mountPath: /var/cache/nginx
    - name: run
      mountPath: /var/run
  volumes:
  - name: cache
    emptyDir: {}
  - name: run
    emptyDir: {}
```

**NetworkPolicy** - контроль сетевого трафика:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-network-policy
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
```

**PodSecurityPolicy** (deprecated в K8s 1.25+, используй Pod Security Standards):
```yaml
# Pod Security Standards (замена PSP)
# Уровни: privileged, baseline, restricted
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

**Команды:**
```bash
# ServiceAccounts
kubectl get serviceaccounts
kubectl describe sa app-sa

# RBAC
kubectl get roles
kubectl get rolebindings
kubectl get clusterroles
kubectl get clusterrolebindings

# Проверка прав
kubectl auth can-i get pods
kubectl auth can-i create deployments --as=system:serviceaccount:default:app-sa
kubectl auth can-i --list --as=system:serviceaccount:default:app-sa

# Создание
kubectl create serviceaccount app-sa
kubectl create role pod-reader --verb=get,list --resource=pods
kubectl create rolebinding read-pods --role=pod-reader --serviceaccount=default:app-sa

# NetworkPolicies
kubectl get networkpolicies
kubectl describe networkpolicy api-network-policy
```

### 💻 Задание

Настрой безопасный доступ для приложения:

1. **Создай ServiceAccount**:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-reader
  namespace: default
```

2. **Создай Role** с правами только на чтение:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: configmap-reader
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

3. **Создай RoleBinding**:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-configs
  namespace: default
subjects:
- kind: ServiceAccount
  name: app-reader
  namespace: default
roleRef:
  kind: Role
  name: configmap-reader
  apiGroup: rbac.authorization.k8s.io
```

4. **Создай Pod с SecurityContext**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  serviceAccountName: app-reader
  securityContext:
    runAsUser: 1000
    runAsNonRoot: true
    fsGroup: 2000
  containers:
  - name: app
    image: nginx:alpine
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
    volumeMounts:
    - name: cache
      mountPath: /var/cache/nginx
    - name: run
      mountPath: /var/run
  volumes:
  - name: cache
    emptyDir: {}
  - name: run
    emptyDir: {}
```

5. **Создай NetworkPolicy**:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app-traffic
spec:
  podSelector:
    matchLabels:
      app: secure-app
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 80
```

6. Примени и проверь:
```bash
kubectl apply -f serviceaccount.yaml
kubectl apply -f role.yaml
kubectl apply -f rolebinding.yaml
kubectl apply -f pod-secure.yaml
kubectl apply -f networkpolicy.yaml

# Проверь права ServiceAccount
kubectl auth can-i get configmaps --as=system:serviceaccount:default:app-reader
kubectl auth can-i delete pods --as=system:serviceaccount:default:app-reader

# Зайди в Pod и проверь
kubectl exec -it secure-app -- sh
id  # Должен показать uid=1000
```

### 🚀 Бонус (новое)

**1. Используй Pod Security Admission** (K8s 1.25+):
```yaml
# Создай namespace с restricted режимом
apiVersion: v1
kind: Namespace
metadata:
  name: restricted-ns
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: latest
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

**2. OPA Gatekeeper** для сложных политик:
```bash
# Установка
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/master/deploy/gatekeeper.yaml

# Создание ConstraintTemplate
kubectl apply -f - <<EOF
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        violation[{"msg": msg}] {
          not input.review.object.metadata.labels.owner
          msg := "All pods must have an 'owner' label"
        }
EOF
```

**3. Audit Logging** - включение аудита:
```yaml
# /etc/kubernetes/audit-policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
  resources:
  - group: ""
    resources: ["secrets", "configmaps"]
- level: RequestResponse
  verbs: ["create", "update", "patch", "delete"]
```

**4. Использование External Secrets Operator**:
```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-backend
spec:
  provider:
    vault:
      server: "https://vault.example.com"
      path: "secret"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "my-app"
```

---

## Модуль 10: Мониторинг и Observability (30 минут)

### 🎯 Напоминалка

**Metrics Server** - базовые метрики:
```bash
# Установка
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Использование
kubectl top nodes
kubectl top pods
kubectl top pods --containers
kubectl top pods -n kube-system --sort-by=memory
```

**Prometheus + Grafana** - полноценный мониторинг:
```bash
# Установка через Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
```

**ServiceMonitor** для Prometheus Operator:
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: app-metrics
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

**Логирование:**

EFK Stack (Elasticsearch + Fluentd + Kibana):
```yaml
# Fluentd DaemonSet
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: fluentd-logging
  template:
    metadata:
      labels:
        k8s-app: fluentd-logging
    spec:
      serviceAccountName: fluentd
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1-debian-elasticsearch
        env:
        - name: FLUENT_ELASTICSEARCH_HOST
          value: "elasticsearch.logging.svc.cluster.local"
        - name: FLUENT_ELASTICSEARCH_PORT
          value: "9200"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

**Loki + Promtail** (легковесная альтернатива):
```bash
# Установка через Helm
helm repo add grafana https://grafana.github.io/helm-charts
helm install loki grafana/loki-stack -n monitoring --set promtail.enabled=true
```

**Трейсинг с Jaeger:**
```bash
# Установка
kubectl create namespace observability
kubectl create -f https://github.com/jaegertracing/jaeger-operator/releases/download/v1.42.0/jaeger-operator.yaml -n observability
```

**Probes подробно:**
```yaml
# Liveness Probe - перезапуск при сбое
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
    httpHeaders:
    - name: Custom-Header
      value: Awesome
  initialDelaySeconds: 15
  periodSeconds: 10
  timeoutSeconds: 5
  successThreshold: 1
  failureThreshold: 3

# Readiness Probe - исключение из Service
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
  failureThreshold: 3

# Startup Probe - для медленных приложений
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

**Events и логи:**
```bash
# Events
kubectl get events
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events --field-selector involvedObject.name=my-pod

# Логи
kubectl logs <pod>
kubectl logs <pod> -c <container>
kubectl logs <pod> --previous  # Логи предыдущего контейнера
kubectl logs -f <pod>          # Follow
kubectl logs <pod> --since=1h
kubectl logs <pod> --tail=100
kubectl logs -l app=myapp --all-containers=true

# Stern для удобного просмотра логов
stern <pod-query>
stern -n kube-system .
stern --all-namespaces -l app=nginx
```

**Horizontal Pod Autoscaler (HPA):**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
```

**Vertical Pod Autoscaler (VPA):**
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  updatePolicy:
    updateMode: "Auto"  # Off, Initial, Recreate, Auto
```

**Команды:**
```bash
# Metrics
kubectl top nodes
kubectl top pods -A

# HPA
kubectl get hpa
kubectl describe hpa app-hpa
kubectl autoscale deployment app --cpu-percent=70 --min=2 --max=10

# VPA
kubectl get vpa
kubectl describe vpa app-vpa
```

### 💻 Задание

Настрой мониторинг и автоскейлинг:

1. **Установи Metrics Server** (если еще не установлен):
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Для minikube
minikube addons enable metrics-server
```

2. **Создай Deployment с метриками**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: stress-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: stress-app
  template:
    metadata:
      labels:
        app: stress-app
    spec:
      containers:
      - name: stress
        image: polinux/stress
        command: ["stress"]
        args: ["--cpu", "1", "--timeout", "3600"]
        resources:
          requests:
            cpu: 100m
            memory: 50Mi
          limits:
            cpu: 200m
            memory: 100Mi
        livenessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - "ps aux | grep stress | grep -v grep"
          initialDelaySeconds: 5
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - "ps aux | grep stress | grep -v grep"
          initialDelaySeconds: 3
          periodSeconds: 5
```

3. **Создай HPA**:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: stress-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: stress-app
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

4. Примени и наблюдай:
```bash
kubectl apply -f deployment.yaml
kubectl apply -f hpa.yaml

# Наблюдай за метриками
kubectl top pods
watch -n 1 kubectl get hpa

# Наблюдай за Pod'ами
watch -n 1 kubectl get pods

# Посмотри логи
kubectl logs -f <pod-name>

# События
kubectl get events --watch
```

5. **Создай простой экспорт метрик**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: metrics-config
data:
  metrics.txt: |
    # HELP app_requests_total Total number of requests
    # TYPE app_requests_total counter
    app_requests_total 12345
    
    # HELP app_request_duration_seconds Request duration
    # TYPE app_request_duration_seconds histogram
    app_request_duration_seconds_bucket{le="0.1"} 1000
    app_request_duration_seconds_bucket{le="0.5"} 5000
    app_request_duration_seconds_bucket{le="1.0"} 8000
    app_request_duration_seconds_sum 4500.5
    app_request_duration_seconds_count 8000
---
apiVersion: v1
kind: Pod
metadata:
  name: metrics-exporter
  labels:
    app: metrics-exporter
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - name: metrics
      containerPort: 80
    volumeMounts:
    - name: metrics
      mountPath: /usr/share/nginx/html
  volumes:
  - name: metrics
    configMap:
      name: metrics-config
---
apiVersion: v1
kind: Service
metadata:
  name: metrics-service
  labels:
    app: metrics-exporter
spec:
  selector:
    app: metrics-exporter
  ports:
  - name: metrics
    port: 80
    targetPort: 80
```

### 🚀 Бонус (новое)

**1. Установи Prometheus Stack**:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring --create-namespace

# Доступ к Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80

# Логин: admin
# Пароль:
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 -d
```

**2. Создай custom metrics с Prometheus Adapter**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: adapter-config
  namespace: monitoring
data:
  config.yaml: |
    rules:
    - seriesQuery: 'http_requests_total'
      resources:
        overrides:
          namespace: {resource: "namespace"}
          pod: {resource: "pod"}
      name:
        matches: "^(.*)_total$"
        as: "${1}_per_second"
      metricsQuery: 'rate(<<.Series>>{<<.LabelMatchers>>}[2m])'
```

**3. Используй kubectl-debug** для продвинутой отладки:
```bash
# Установка
kubectl krew install debug

# Использование
kubectl debug -it <pod> --image=busybox --target=<container>
```

**4. Создай PodDisruptionBudget** для высокой доступности:
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 2
  # или maxUnavailable: 1
  selector:
    matchLabels:
      app: myapp
```

---

## Финальный проект (60 минут)

### Задача: Развернуть полноценное микросервисное приложение

Создай production-ready окружение для трехуровневого приложения.

**Архитектура:**
- Frontend (React/Nginx)
- Backend API (Node.js/Python)
- Database (PostgreSQL)
- Redis Cache
- Ingress для внешнего доступа
- Мониторинг и логирование

**Требования:**

1. **Namespace и организация:**
   - Namespace: `prod-app`
   - Labels для всех ресурсов
   - Используй Kustomize для управления

2. **Database (StatefulSet):**
   - PostgreSQL StatefulSet с 1 репликой
   - PersistentVolumeClaim 5Gi
   - ConfigMap для init scripts
   - Secret для credentials
   - Headless Service

3. **Redis (Deployment):**
   - Redis Deployment с 1 репликой
   - ConfigMap для конфигурации
   - ClusterIP Service

4. **Backend API (Deployment):**
   - 3 реплики
   - HPA (min=2, max=10, cpu=70%)
   - ConfigMap для конфигурации приложения
   - Secret для DB credentials и API keys
   - Liveness и Readiness probes
   - Resource requests/limits
   - ServiceAccount с RBAC
   - ClusterIP Service

5. **Frontend (Deployment):**
   - 2 реплики
   - ConfigMap для nginx.conf
   - Liveness и Readiness probes
   - ClusterIP Service

6. **Ingress:**
   - SSL/TLS (self-signed для теста)
   - Маршрутизация:
     - `/` → Frontend
     - `/api` → Backend
   - Rate limiting
   - CORS headers

7. **Мониторинг:**
   - ServiceMonitor для Prometheus
   - Metrics endpoints
   - Grafana dashboard

8. **Безопасность:**
   - NetworkPolicies
   - PodSecurityContext
   - RBAC
   - Secrets management

9. **Backup:**
   - CronJob для backup БД (каждый день в 2:00)
   - Job для manual backup

10. **Документация:**
    - README с инструкциями
    - Architecture diagram
    - Deployment guide

**Структура проекта:**
```
k8s-app/
├── base/
│   ├── kustomization.yaml
│   ├── namespace.yaml
│   ├── database/
│   │   ├── statefulset.yaml
│   │   ├── service.yaml
│   │   ├── pvc.yaml
│   │   ├── configmap.yaml
│   │   └── secret.yaml
│   ├── redis/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── configmap.yaml
│   ├── backend/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── hpa.yaml
│   │   ├── configmap.yaml
│   │   ├── secret.yaml
│   │   └── serviceaccount.yaml
│   ├── frontend/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── configmap.yaml
│   ├── ingress/
│   │   ├── ingress.yaml
│   │   └── tls-secret.yaml
│   ├── monitoring/
│   │   └── servicemonitor.yaml
│   ├── backup/
│   │   └── cronjob.yaml
│   └── security/
│       ├── networkpolicy.yaml
│       └── rbac.yaml
├── overlays/
│   ├── dev/
│   │   └── kustomization.yaml
│   └── prod/
│       └── kustomization.yaml
├── scripts/
│   ├── deploy.sh
│   ├── backup.sh
│   └── rollback.sh
└── README.md
```

**Начни с базового манифеста для namespace:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: prod-app
  labels:
    name: prod-app
    environment: production
```

**Примеры команд для развертывания:**
```bash
# Создание namespace
kubectl apply -f base/namespace.yaml

# Применение базовой конфигурации
kubectl apply -k base/

# Или для конкретного окружения
kubectl apply -k overlays/prod/

# Проверка
kubectl get all -n prod-app
kubectl get pvc -n prod-app
kubectl get ingress -n prod-app

# Мониторинг
watch -n 2 kubectl get pods -n prod-app

# Логи
stern -n prod-app .

# Тестирование
kubectl run test -n prod-app --image=busybox -it --rm -- sh
```

**Дополнительные улучшения (опционально):**
- GitOps с ArgoCD/Flux
- Service Mesh (Istio/Linkerd)
- Canary deployments
- Blue-Green deployments
- External Secrets Operator
- Cert-manager для Let's Encrypt
- Backup в S3/GCS
- Disaster Recovery plan
- Load testing с k6/Locust

---

## Справочная секция: Быстрые шпаргалки

### Kubectl shortcuts

```bash
# Алиасы
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deploy'
alias kga='kubectl get all'
alias kdp='kubectl describe pod'
alias kl='kubectl logs'
alias kx='kubectl exec -it'
alias kaf='kubectl apply -f'
alias kdel='kubectl delete -f'

# Быстрое создание ресурсов
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl create deployment nginx --image=nginx --replicas=3 --dry-run=client -o yaml > deployment.yaml
kubectl expose deployment nginx --port=80 --target-port=8080 --dry-run=client -o yaml > service.yaml
kubectl create configmap app-config --from-literal=key=value --dry-run=client -o yaml > configmap.yaml

# Редактирование
kubectl edit deployment nginx
kubectl patch deployment nginx -p '{"spec":{"replicas":5}}'
kubectl set image deployment/nginx nginx=nginx:1.22

# Debugging
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl describe pod <pod> | grep -A 10 Events
kubectl logs <pod> --previous
kubectl debug -it <pod> --image=busybox --target=<container>

# JSON/YAML манипуляции
kubectl get pod nginx -o json | jq '.spec.containers[0].image'
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase

# Контекст и namespace
kubectl config get-contexts
kubectl config use-context <context>
kubectl config set-context --current --namespace=<namespace>
```

### YAML шаблоны

**Минимальный Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: nginx
```

**Минимальный Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: nginx
```

**Минимальный Service:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
```

### Troubleshooting Guide

**Pod не запускается:**
```bash
# Проверка статуса
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get events --field-selector involvedObject.name=<pod>

# Типичные причины:
# - ImagePullBackOff: неверный image или нет доступа к registry
# - CrashLoopBackOff: приложение падает при запуске
# - Pending: нет ресурсов или проблемы с scheduling
# - CreateContainerConfigError: проблемы с ConfigMap/Secret
```

**Service недоступен:**
```bash
# Проверка Service и Endpoints
kubectl get svc
kubectl get endpoints <service>
kubectl describe svc <service>

# Проверка labels
kubectl get pods --show-labels
kubectl get pods -l app=my-app

# Тест изнутри кластера
kubectl run test --image=busybox -it --rm -- wget -O- http://service-name

# Проверка NetworkPolicy
kubectl get networkpolicies
```

**Ingress не работает:**
```bash
# Проверка Ingress
kubectl get ingress
kubectl describe ingress <ingress>

# Проверка Ingress Controller
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx <controller-pod>

# Проверка DNS
nslookup <hostname>

# Проверка TLS
kubectl get secret <tls-secret> -o yaml
```

**Проблемы с хранилищем:**
```bash
# Проверка PVC
kubectl get pvc
kubectl describe pvc <pvc>

# Проверка PV
kubectl get pv
kubectl describe pv <pv>

# Проверка StorageClass
kubectl get storageclass
kubectl describe storageclass <sc>
```

### Полезные kubectl плагины (krew)

```bash
# Установка krew
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)

export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"

# Полезные плагины
kubectl krew install ctx          # Быстрое переключение контекстов
kubectl krew install ns           # Быстрое переключение namespace
kubectl krew install tree         # Иерархия ресурсов
kubectl krew install tail         # Tail логов
kubectl krew install debug        # Расширенная отладка
kubectl krew install resource-capacity  # Использование ресурсов
kubectl krew install whoami       # Текущий пользователь
kubectl krew install view-secret  # Просмотр secrets
kubectl krew install get-all      # Все ресурсы

# Использование
kubectx                    # Список контекстов
kubectx <context>          # Переключить контекст
kubens                     # Список namespace
kubens <namespace>         # Переключить namespace
kubectl tree deployment nginx
kubectl tail <pod>
kubectl view-secret <secret> <key>
```

### Helm шпаргалка

```bash
# Установка Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Работа с репозиториями
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx

# Установка chart
helm install my-release stable/nginx
helm install my-release stable/nginx -n my-namespace --create-namespace
helm install my-release stable/nginx --set service.type=LoadBalancer
helm install my-release stable/nginx -f values.yaml

# Управление releases
helm list
helm list -A
helm status my-release
helm get values my-release
helm upgrade my-release stable/nginx
helm rollback my-release 1
helm uninstall my-release

# Создание своего chart
helm create my-chart
helm template my-chart
helm lint my-chart
helm package my-chart
```

### Производительность и оптимизация

**Resource Quotas:**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    persistentvolumeclaims: "10"
    pods: "50"
```

**LimitRange:**
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-memory-limit
  namespace: dev
spec:
  limits:
  - default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 200m
      memory: 256Mi
    max:
      cpu: 2
      memory: 2Gi
    min:
      cpu: 100m
      memory: 128Mi
    type: Container
```

**PriorityClass:**
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
description: "High priority class for critical pods"
```

### Best Practices

**1. Labels и Annotations:**
```yaml
metadata:
  labels:
    app: myapp
    version: v1.2.3
    environment: production
    tier: backend
    team: platform
  annotations:
    description: "Backend API service"
    maintainer: "platform-team@example.com"
    version: "1.2.3"
    last-updated: "2025-01-15"
```

**2. Resource Requests и Limits:**
```yaml
# Всегда указывай requests и limits
resources:
  requests:
    cpu: 100m      # Минимум для scheduling
    memory: 128Mi
  limits:
    cpu: 500m      # Максимум для предотвращения OOM
    memory: 512Mi
```

**3. Health Checks:**
```yaml
# Обязательно используй probes
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

**4. Security:**
```yaml
# Минимальные привилегии
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
    - ALL
```

**5. ConfigMaps и Secrets:**
```yaml
# Не хардкодь конфигурацию
env:
- name: DATABASE_URL
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: url
- name: LOG_LEVEL
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: log_level
```

**6. Pod Disruption Budgets:**
```yaml
# Для критичных сервисов
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```

### Production Checklist

**Перед деплоем:**
- ✅ Resource requests и limits установлены
- ✅ Liveness и Readiness probes настроены
- ✅ Логирование настроено и централизовано
- ✅ Метрики экспортируются
- ✅ Secrets не в plain text
- ✅ RBAC настроен с минимальными правами
- ✅ NetworkPolicies определены
- ✅ PodSecurityContext применен
- ✅ HPA настроен для автоскейлинга
- ✅ PDB установлен для высокой доступности
- ✅ Backup стратегия определена
- ✅ Monitoring и alerting настроены
- ✅ Ingress с TLS сертификатом
- ✅ Resource quotas на namespace
- ✅ Anti-affinity для распределения по нодам
- ✅ Graceful shutdown настроен
- ✅ Rolling update strategy определена
- ✅ Документация актуальна

---

## План повторения

### При первом прохождении (2-3 часа):
- Пройди модули 1-5 обязательно
- Модули 6-8 по желанию
- Финальный проект упрощенный

### При повторном прохождении (через 6-12 месяцев):
- Бегло просмотри теорию
- Сфокусируйся на бонусных заданиях
- Пройди модули 9-10 обязательно
- Финальный проект полностью
- Добавь свои кастомизации

### Для закрепления:
- Автоматизируй деплой своих проектов в K8s
- Настрой CI/CD с ArgoCD или Flux
- Попробуй разные окружения (EKS, GKE, AKS)
- Изучи Service Mesh (Istio/Linkerd)
- Поучаствуй в Kubernetes the Hard Way
- Получи сертификацию CKA/CKAD

### Дополнительные ресурсы:
- **Kubernetes Documentation** - официальная документация
- **Kubernetes Patterns** - книга с паттернами
- **Production Kubernetes** - best practices
- **The Kubernetes Book** - Nigel Poulton
- **Play with Kubernetes** - онлайн playground
- **KillerCoda** - интерактивные сценарии
- **Kubernetes Slack** - сообщество

---

## Чек-лист навыков

После прохождения курса ты должен уметь:

### Базовые навыки:
- ✅ Работать с kubectl уверенно
- ✅ Создавать и управлять Pod'ами
- ✅ Настраивать Deployments и ReplicaSets
- ✅ Работать с Services (ClusterIP, NodePort, LoadBalancer)
- ✅ Использовать ConfigMaps и Secrets
- ✅ Работать с Volumes и PersistentVolumes

### Продвинутые навыки:
- ✅ Настраивать Ingress с TLS
- ✅ Создавать Jobs и CronJobs
- ✅ Управлять DaemonSets
- ✅ Настраивать RBAC
- ✅ Применять NetworkPolicies
- ✅ Использовать SecurityContext

### Expert навыки:
- ✅ Настраивать мониторинг (Prometheus/Grafana)
- ✅ Настраивать логирование (EFK/Loki)
- ✅ Использовать HPA и VPA
- ✅ Работать с Helm charts
- ✅ Применять GitOps практики
- ✅ Troubleshooting сложных проблем

### Архитектурные навыки:
- ✅ Проектировать микросервисные архитектуры
- ✅ Планировать disaster recovery
- ✅ Оптимизировать использование ресурсов
- ✅ Обеспечивать безопасность кластера
- ✅ Настраивать CI/CD пайплайны
- ✅ Масштабировать приложения

---

## Полезные команды для экзаменов CKA/CKAD

```bash
# Императивные команды (для скорости на экзамене)
kubectl run nginx --image=nginx
kubectl run nginx --image=nginx --dry-run=client -o yaml
kubectl create deployment nginx --image=nginx --replicas=3
kubectl expose deployment nginx --port=80 --target-port=8080
kubectl create service clusterip redis --tcp=6379:6379
kubectl create configmap app-config --from-literal=key=value
kubectl create secret generic db-secret --from-literal=password=secret

# Быстрое редактирование
kubectl edit pod nginx
kubectl replace -f pod.yaml --force  # Для immutable полей

# Масштабирование
kubectl scale deployment nginx --replicas=5

# Rollout
kubectl set image deployment/nginx nginx=nginx:1.22
kubectl rollout status deployment/nginx
kubectl rollout undo deployment/nginx

# Labels
kubectl label pod nginx env=prod
kubectl label pod nginx env=prod --overwrite

# Shortcuts
k run nginx --image=nginx $do > pod.yaml  # $do = --dry-run=client -o yaml
k create deploy nginx --image=nginx $do > deploy.yaml
```

**Для экзамена добавь в ~/.bashrc:**
```bash
alias k='kubectl'
alias kgp='kubectl get pods'
alias kaf='kubectl apply -f'
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"

# Быстрое создание YAML
function krun() {
  kubectl run "$1" --image="$2" --dry-run=client -o yaml
}

function kcreate() {
  kubectl create deployment "$1" --image="$2" --dry-run=client -o yaml
}
```

---

## Что нового в последних версиях Kubernetes

**Kubernetes 1.25:**
- Pod Security Standards (замена PSP)
- Ephemeral Containers стабильная фича
- CSI Migration завершена

**Kubernetes 1.26:**
- CPUManager улучшения
- DynamicResourceAllocation (alpha)
- ValidatingAdmissionPolicy (alpha)

**Kubernetes 1.27:**
- SeccompDefault включен по умолчанию
- StatefulSet auto-delete PVCs
- ReadWriteOncePod access mode

**Kubernetes 1.28:**
- Sidecar containers (alpha)
- Job success/failure policies
- VolumeAttributesClass (alpha)

**Kubernetes 1.29:**
- MatchLabelKeys для PodAffinity
- ReadWriteOncePod graduated to stable
- AppArmor support improvements

**Kubernetes 1.30+ (upcoming):**
- Следи за release notes на kubernetes.io

---

## Заключение

Поздравляю! Ты прошел курс по освежению знаний Kubernetes. 

**Следующие шаги:**
1. Практикуйся регулярно - создай свой homelab кластер
2. Автоматизируй всё с помощью K8s в своих проектах
3. Изучай смежные технологии: Service Mesh, GitOps, Observability
4. Получи сертификацию CKA или CKAD
5. Делись знаниями - пиши посты, помогай новичкам

**Помни:**
- Kubernetes - это инструмент, а не самоцель
- Начинай с простого, усложняй постепенно
- Документация - твой лучший друг
- Community очень дружелюбное и готово помочь

Проходи этот курс каждые 6-12 месяцев, чтобы оставаться в форме. Каждый раз ты будешь узнавать что-то новое и замечать, как выросли твои навыки! 

Happy Kubernetes learning! ☸️🚀