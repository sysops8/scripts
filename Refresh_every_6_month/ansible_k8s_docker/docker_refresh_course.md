# Docker Refresh: Ежегодный/Полугодовой курс для DevOps/SysAdmin

**Цель:** Освежить в памяти ключевые концепции Docker, Docker Compose и Docker Swarm за 2-3 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый раздел состоит из:
1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Небольшой проект, который нужно создать с нуля
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

**Требования:** Docker Engine 20.10+, Docker Compose v2+, Linux/macOS/WSL2

---

## Модуль 1: Основы Docker и работа с образами (20 минут)

### 🎯 Напоминалка

**Базовые команды Docker:**
```bash
# Информация о системе
docker version
docker info
docker system df  # Использование диска

# Работа с образами
docker images
docker pull nginx:alpine
docker build -t myapp:latest .
docker tag myapp:latest myapp:v1.0
docker rmi image_name
docker image prune  # Удалить неиспользуемые образы

# Работа с контейнерами
docker ps                    # Запущенные
docker ps -a                 # Все контейнеры
docker run -d -p 80:80 nginx
docker stop container_id
docker start container_id
docker restart container_id
docker rm container_id
docker logs container_id
docker exec -it container_id /bin/bash

# Очистка системы
docker system prune -a       # Удалить всё неиспользуемое
docker container prune       # Удалить остановленные контейнеры
docker volume prune          # Удалить неиспользуемые volumes
```

**Dockerfile - основной синтаксис:**
```dockerfile
# Базовый образ
FROM ubuntu:22.04

# Метаданные
LABEL maintainer="admin@example.com"
LABEL version="1.0"

# Переменные окружения
ENV APP_HOME=/app \
    PYTHON_VERSION=3.11

# Рабочая директория
WORKDIR /app

# Копирование файлов
COPY requirements.txt .
COPY . /app

# Установка зависимостей
RUN apt-get update && \
    apt-get install -y python3 && \
    rm -rf /var/lib/apt/lists/*

# Открытие портов
EXPOSE 8080

# Volumes
VOLUME ["/data"]

# Пользователь
USER appuser

# Команда по умолчанию
CMD ["python3", "app.py"]
# или
ENTRYPOINT ["python3"]
CMD ["app.py"]
```

**Multi-stage build (важно!):**
```dockerfile
# Стадия сборки
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Финальный образ
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/myapp .
CMD ["./myapp"]
```

**Полезные флаги docker run:**
```bash
-d                    # Detached режим
-p 8080:80           # Проброс портов (host:container)
-v /host:/container  # Монтирование volume
-e VAR=value         # Переменные окружения
--name mycontainer   # Имя контейнера
--rm                 # Удалить после остановки
--network mynet      # Подключить к сети
--restart unless-stopped  # Политика перезапуска
--memory 512m        # Лимит памяти
--cpus 1.5          # Лимит CPU
```

### 💻 Задание

Создай простое веб-приложение в контейнере:

1. Создай `Dockerfile` для простого веб-сервера:
   - Базовый образ: `nginx:alpine`
   - Скопируй кастомный `index.html` с текстом "Docker Refresh Course v1.0"
   - Добавь healthcheck для проверки `/` endpoint
   
2. Создай `index.html`:
   ```html
   <!DOCTYPE html>
   <html>
   <head><title>Docker Test</title></head>
   <body>
       <h1>Docker Refresh Course v1.0</h1>
       <p>Container ID: ${HOSTNAME}</p>
   </body>
   </html>
   ```

3. Собери образ с тегом `web-app:v1`

4. Запусти контейнер:
   - На порту 8080
   - С именем `web-test`
   - С автоматическим перезапуском
   - Проверь доступность через `curl localhost:8080`

### 🚀 Бонус (новое)

Создай multi-stage Dockerfile для Node.js приложения:
- Стадия 1: Сборка с `node:18` (npm install, npm build)
- Стадия 2: Production с `node:18-alpine` (только runtime файлы)
- Сравни размер образов до и после оптимизации

---

## Модуль 2: Docker Networks и Volumes (25 минут)

### 🎯 Напоминалка

**Docker Networks:**
```bash
# Типы сетей
bridge      # По умолчанию, изолированная сеть
host        # Использует сеть хоста
none        # Без сети
overlay     # Для Swarm, multi-host сеть

# Управление сетями
docker network ls
docker network create mynetwork
docker network create --driver bridge --subnet 172.20.0.0/16 custom-net
docker network inspect mynetwork
docker network rm mynetwork

# Подключение контейнеров
docker run --network mynetwork nginx
docker network connect mynetwork container_name
docker network disconnect mynetwork container_name

# DNS между контейнерами
# В одной сети контейнеры видят друг друга по имени!
docker run --name web --network mynet nginx
docker run --network mynet alpine ping web  # Работает!
```

**Docker Volumes:**
```bash
# Типы монтирования
-v volume_name:/path           # Named volume (рекомендуется)
-v /host/path:/container/path  # Bind mount
--mount type=tmpfs,dst=/tmp    # tmpfs (в памяти)

# Управление volumes
docker volume ls
docker volume create mydata
docker volume inspect mydata
docker volume rm mydata
docker volume prune  # Удалить неиспользуемые

# Где хранятся volumes
/var/lib/docker/volumes/  # На Linux

# Резервное копирование volume
docker run --rm -v mydata:/data -v $(pwd):/backup \
  alpine tar czf /backup/backup.tar.gz /data

# Восстановление
docker run --rm -v mydata:/data -v $(pwd):/backup \
  alpine tar xzf /backup/backup.tar.gz -C /
```

**Пример с volumes и networks:**
```bash
# Создать сеть и volume
docker network create app-net
docker volume create app-data

# Запустить БД
docker run -d \
  --name postgres \
  --network app-net \
  -v app-data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15-alpine

# Запустить приложение
docker run -d \
  --name app \
  --network app-net \
  -e DB_HOST=postgres \
  -e DB_PASSWORD=secret \
  myapp:latest
```

### 💻 Задание

Создай изолированное окружение для веб-приложения с БД:

1. Создай кастомную сеть `app-network` (bridge, subnet 172.25.0.0/16)

2. Создай два named volumes:
   - `postgres-data` для БД
   - `app-logs` для логов

3. Запусти PostgreSQL контейнер:
   - Имя: `db`
   - Сеть: `app-network`
   - Volume: `postgres-data` → `/var/lib/postgresql/data`
   - Переменные: `POSTGRES_USER=appuser`, `POSTGRES_PASSWORD=secret123`, `POSTGRES_DB=appdb`

4. Запусти Nginx контейнер:
   - Имя: `web`
   - Сеть: `app-network`
   - Порт: 8080:80
   - Volume: `app-logs` → `/var/log/nginx`

5. Проверь:
   - `docker network inspect app-network` - оба контейнера должны быть в списке
   - `docker exec web ping db` - должен пинговаться по имени
   - Посмотри логи nginx: `docker exec web ls -la /var/log/nginx`

### 🚀 Бонус (новое)

Настрой read-only filesystem для контейнера с исключениями:
```bash
docker run -d --read-only \
  --tmpfs /tmp:rw,size=64m \
  -v app-data:/data:rw \
  nginx:alpine
```

Объясни, зачем это нужно для безопасности. Создай резервную копию volume `postgres-data` в файл `db-backup.tar.gz`.

---

## Модуль 3: Docker Compose - Основы (30 минут)

### 🎯 Напоминалка

**Docker Compose v2 (новый синтаксис):**
```yaml
# docker-compose.yml или compose.yaml
version: '3.8'  # Опционально в v2

services:
  web:
    image: nginx:alpine
    # или build: .
    # или build:
    #       context: .
    #       dockerfile: Dockerfile
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
      - logs:/var/log/nginx
    environment:
      - NGINX_HOST=example.com
      - NGINX_PORT=80
    # или
    env_file:
      - .env
    networks:
      - frontend
    restart: unless-stopped
    depends_on:
      - db
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
  
  db:
    image: postgres:15-alpine
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD:-secret}
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # Без доступа наружу

volumes:
  postgres-data:
    driver: local
  logs:
```

**Команды Docker Compose:**
```bash
# Запуск
docker compose up                 # Foreground
docker compose up -d              # Background
docker compose up --build         # Пересобрать образы

# Управление
docker compose ps                 # Статус сервисов
docker compose logs               # Логи всех сервисов
docker compose logs -f web        # Follow логи одного сервиса
docker compose exec web sh        # Войти в контейнер
docker compose stop               # Остановить
docker compose start              # Запустить остановленные
docker compose restart web        # Перезапустить сервис
docker compose down               # Остановить и удалить
docker compose down -v            # + удалить volumes

# Масштабирование
docker compose up -d --scale web=3

# Валидация
docker compose config             # Проверить синтаксис
docker compose config --services  # Список сервисов

# Работа с образами
docker compose build              # Собрать все
docker compose build --no-cache web
docker compose pull               # Скачать образы
docker compose push               # Загрузить в registry
```

**Продвинутые фичи:**
```yaml
services:
  app:
    # Расширение другого сервиса
    extends:
      file: common.yml
      service: base-app
    
    # Лимиты ресурсов
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          memory: 256M
    
    # Healthcheck
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    
    # Зависимости с условиями
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    
    # Кастомные DNS
    dns:
      - 8.8.8.8
      - 8.8.4.4
    
    # Extra hosts
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

### 💻 Задание

Создай полноценный стек LEMP (Linux, Nginx, MySQL, PHP):

1. Создай структуру проекта:
   ```
   lemp-stack/
   ├── docker-compose.yml
   ├── nginx/
   │   ├── Dockerfile
   │   └── default.conf
   ├── php/
   │   └── Dockerfile
   └── www/
       └── index.php
   ```

2. `docker-compose.yml`:
   ```yaml
   services:
     nginx:
       build: ./nginx
       ports:
         - "8080:80"
       volumes:
         - ./www:/var/www/html
       depends_on:
         - php
       networks:
         - frontend
     
     php:
       build: ./php
       volumes:
         - ./www:/var/www/html
       networks:
         - frontend
         - backend
       environment:
         - DB_HOST=mysql
         - DB_NAME=testdb
         - DB_USER=user
         - DB_PASSWORD=password
     
     mysql:
       image: mysql:8.0
       environment:
         MYSQL_ROOT_PASSWORD: rootpass
         MYSQL_DATABASE: testdb
         MYSQL_USER: user
         MYSQL_PASSWORD: password
       volumes:
         - mysql-data:/var/lib/mysql
       networks:
         - backend
   
   networks:
     frontend:
     backend:
   
   volumes:
     mysql-data:
   ```

3. `nginx/Dockerfile`:
   ```dockerfile
   FROM nginx:alpine
   COPY default.conf /etc/nginx/conf.d/default.conf
   ```

4. `nginx/default.conf`:
   ```nginx
   server {
       listen 80;
       root /var/www/html;
       index index.php index.html;
       
       location ~ \.php$ {
           fastcgi_pass php:9000;
           fastcgi_index index.php;
           include fastcgi_params;
           fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
       }
   }
   ```

5. `php/Dockerfile`:
   ```dockerfile
   FROM php:8.2-fpm-alpine
   RUN docker-php-ext-install pdo pdo_mysql
   ```

6. `www/index.php`:
   ```php
   <?php
   phpinfo();
   echo "<h1>LEMP Stack Working!</h1>";
   try {
       $pdo = new PDO(
           "mysql:host=" . getenv('DB_HOST') . ";dbname=" . getenv('DB_NAME'),
           getenv('DB_USER'),
           getenv('DB_PASSWORD')
       );
       echo "<p>Database connection: OK</p>";
   } catch (PDOException $e) {
       echo "<p>Database connection: FAILED</p>";
   }
   ?>
   ```

7. Запусти и проверь работу на `http://localhost:8080`

### 🚀 Бонус (новое)

Добавь в стек:
1. **PHPMyAdmin** для управления БД (порт 8081)
2. **Redis** для кеширования
3. **Healthchecks** для всех сервисов
4. Создай `.env` файл для всех паролей и настроек

Используй `docker compose config` для проверки итоговой конфигурации с подставленными переменными.

---

## Модуль 4: Docker Compose - Продвинутое использование (30 минут)

### 🎯 Напоминалка

**Profiles (для разных окружений):**
```yaml
services:
  web:
    image: nginx
    # Всегда запускается
  
  db:
    image: postgres
    profiles: ["dev"]
    # Только с docker compose --profile dev up
  
  backup:
    image: backup-tool
    profiles: ["prod", "backup"]
    # docker compose --profile prod up
```

**Override файлы:**
```yaml
# docker-compose.yml - базовая конфигурация
services:
  app:
    image: myapp:latest
    
# docker-compose.override.yml - локальная разработка (автоматически применяется)
services:
  app:
    build: .
    volumes:
      - ./src:/app
    environment:
      - DEBUG=true

# docker-compose.prod.yml - продакшн
services:
  app:
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
    environment:
      - DEBUG=false

# Использование
docker compose -f docker-compose.yml -f docker-compose.prod.yml up
```

**Секреты (для Swarm, но можно имитировать):**
```yaml
services:
  app:
    secrets:
      - db_password
      - api_key

secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    external: true  # Для Swarm
```

**Расширение конфигурации:**
```yaml
# common.yml
x-common-service: &common
  restart: unless-stopped
  logging:
    driver: "json-file"
    options:
      max-size: "10m"
      max-file: "3"

services:
  web:
    <<: *common
    image: nginx
  
  app:
    <<: *common
    image: myapp
```

**Переменные и подстановка:**
```yaml
services:
  web:
    image: ${DOCKER_REGISTRY:-docker.io}/nginx:${NGINX_VERSION:-alpine}
    ports:
      - "${WEB_PORT:-8080}:80"
    environment:
      - API_URL=${API_URL}
      - DB_PASSWORD=${DB_PASSWORD:?error message}  # Обязательная переменная
```

### 💻 Задание

Создай многоокруженческий проект с Wordpress:

1. Структура:
   ```
   wordpress-multi-env/
   ├── docker-compose.yml           # Базовая конфигурация
   ├── docker-compose.dev.yml       # Development override
   ├── docker-compose.prod.yml      # Production override
   ├── .env.example
   └── nginx/
       └── nginx.conf
   ```

2. `docker-compose.yml` (базовый):
   ```yaml
   services:
     wordpress:
       image: wordpress:${WP_VERSION:-latest}
       environment:
         WORDPRESS_DB_HOST: db
         WORDPRESS_DB_USER: ${DB_USER}
         WORDPRESS_DB_PASSWORD: ${DB_PASSWORD}
         WORDPRESS_DB_NAME: ${DB_NAME}
       volumes:
         - wp-content:/var/www/html/wp-content
       depends_on:
         - db
     
     db:
       image: mysql:8.0
       environment:
         MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
         MYSQL_DATABASE: ${DB_NAME}
         MYSQL_USER: ${DB_USER}
         MYSQL_PASSWORD: ${DB_PASSWORD}
       volumes:
         - db-data:/var/lib/mysql
   
   volumes:
     wp-content:
     db-data:
   ```

3. `docker-compose.dev.yml`:
   ```yaml
   services:
     wordpress:
       ports:
         - "8080:80"
       environment:
         WORDPRESS_DEBUG: 1
       volumes:
         - ./themes:/var/www/html/wp-content/themes
         - ./plugins:/var/www/html/wp-content/plugins
     
     db:
       ports:
         - "3306:3306"  # Для локального доступа
     
     phpmyadmin:
       image: phpmyadmin:latest
       ports:
         - "8081:80"
       environment:
         PMA_HOST: db
         PMA_USER: ${DB_USER}
         PMA_PASSWORD: ${DB_PASSWORD}
   ```

4. `docker-compose.prod.yml`:
   ```yaml
   services:
     nginx:
       image: nginx:alpine
       ports:
         - "80:80"
         - "443:443"
       volumes:
         - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
         - wp-content:/var/www/html/wp-content:ro
       depends_on:
         - wordpress
     
     wordpress:
       deploy:
         replicas: 2
         resources:
           limits:
             cpus: '1'
             memory: 512M
       restart: always
       environment:
         WORDPRESS_DEBUG: 0
     
     db:
       deploy:
         resources:
           limits:
             memory: 1G
       restart: always
   ```

5. `.env.example`:
   ```env
   WP_VERSION=6.4-php8.2-apache
   DB_NAME=wordpress
   DB_USER=wpuser
   DB_PASSWORD=changeme
   DB_ROOT_PASSWORD=rootchangeme
   ```

6. Создай реальный `.env` из примера

7. Запусти разные окружения:
   ```bash
   # Development
   docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d
   
   # Production (только валидация)
   docker compose -f docker-compose.yml -f docker-compose.prod.yml config
   ```

8. Проверь доступность:
   - Wordpress: http://localhost:8080
   - PHPMyAdmin: http://localhost:8081

### 🚀 Бонус (новое)

Добавь:
1. **Profiles** для дополнительных сервисов:
   - `monitoring`: Prometheus + Grafana
   - `backup`: Automated MySQL backup script
   
2. **Healthchecks** для всех сервисов

3. Создай Makefile с командами:
   ```makefile
   dev-up:
       docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d
   
   prod-up:
       docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
   
   backup:
       docker compose --profile backup up
   ```

---

## Модуль 5: Docker Swarm - Основы (35 минут)

### 🎯 Напоминалка

**Docker Swarm - встроенный оркестратор:**
```bash
# Инициализация Swarm
docker swarm init
docker swarm init --advertise-addr 192.168.1.10

# Информация
docker info  # Смотри Swarm: active
docker node ls  # Список нод

# Добавление нод
docker swarm join-token worker    # Получить команду для worker
docker swarm join-token manager   # Получить команду для manager

# На другой ноде выполнить:
docker swarm join --token TOKEN IP:2377

# Управление нодами
docker node ls
docker node inspect node_id
docker node update --availability drain node_id  # Вывести из работы
docker node update --availability active node_id  # Вернуть в работу
docker node update --label-add type=database node_id  # Добавить label

# Покинуть Swarm
docker swarm leave          # С ноды
docker swarm leave --force  # С manager
```

**Services в Swarm:**
```bash
# Создание сервиса
docker service create \
  --name web \
  --replicas 3 \
  --publish 8080:80 \
  nginx:alpine

# Просмотр сервисов
docker service ls
docker service ps web          # Таски (контейнеры) сервиса
docker service inspect web
docker service logs web

# Масштабирование
docker service scale web=5
docker service update --replicas 10 web

# Обновление сервиса
docker service update --image nginx:latest web
docker service update --env-add KEY=VALUE web
docker service update --publish-add 443:443 web

# Откат
docker service rollback web

# Удаление
docker service rm web
```

**Deploy configuration в docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    deploy:
      # Количество реплик
      replicas: 3
      
      # Стратегия обновления
      update_config:
        parallelism: 2        # Обновлять по 2 за раз
        delay: 10s            # Задержка между обновлениями
        failure_action: rollback
        monitor: 60s
        order: start-first    # Или stop-first
      
      # Откат
      rollback_config:
        parallelism: 2
        delay: 5s
      
      # Политика перезапуска
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
      
      # Ресурсы
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
      
      # Размещение
      placement:
        constraints:
          - node.role == worker
          - node.labels.type == webserver
        preferences:
          - spread: node.labels.datacenter
      
      # Метки
      labels:
        - "com.example.description=Web service"
    
    networks:
      - webnet
    
    ports:
      - "8080:80"
    
    volumes:
      - web-data:/usr/share/nginx/html

networks:
  webnet:
    driver: overlay  # Для multi-host

volumes:
  web-data:
```

**Stack - деплой Compose файла в Swarm:**
```bash
# Деплой стека
docker stack deploy -c docker-compose.yml mystack

# Просмотр стеков
docker stack ls
docker stack services mystack
docker stack ps mystack

# Удаление стека
docker stack rm mystack
```

### 💻 Задание

Создай Swarm кластер и задеплой приложение (можно на одной ноде для теста):

1. Инициализируй Swarm:
   ```bash
   docker swarm init
   ```

2. Создай `swarm-stack.yml`:
   ```yaml
   version: '3.8'
   
   services:
     visualizer:
       image: dockersamples/visualizer:latest
       ports:
         - "8080:8080"
       volumes:
         - "/var/run/docker.sock:/var/run/docker.sock"
       deploy:
         placement:
           constraints:
             - node.role == manager
       networks:
         - webnet
     
     web:
       image: nginx:alpine
       deploy:
         replicas: 3
         update_config:
           parallelism: 1
           delay: 10s
         restart_policy:
           condition: on-failure
       ports:
         - "80:80"
       networks:
         - webnet
     
     whoami:
       image: traefik/whoami
       deploy:
         replicas: 5
         resources:
           limits:
             cpus: '0.1'
             memory: 64M
       networks:
         - webnet
   
   networks:
     webnet:
       driver: overlay
   ```

3. Задеплой стек:
   ```bash
   docker stack deploy -c swarm-stack.yml myapp
   ```

4. Проверь:
   ```bash
   docker stack ls
   docker stack services myapp
   docker stack ps myapp
   docker service ls
   ```

5. Открой визуализатор: http://localhost:8080

6. Помасштабируй сервис:
   ```bash
   docker service scale myapp_whoami=10
   ```

7. Обнови образ nginx:
   ```bash
   docker service update --image nginx:latest myapp_web
   ```

8. Посмотри логи:
   ```bash
   docker service logs myapp_web
   ```

### 🚀 Бонус (новое)

Добавь в стек:
1. **Traefik** как reverse proxy с автоматическим service discovery
2. **Rolling update** с health checks
3. **Secrets** для чувствительных данных

Пример с Traefik:
```yaml
services:
  traefik:
    image: traefik:v2.10
    command:
      - "--api.insecure=true"
      - "--providers.docker.swarmMode=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    deploy:
      placement:
        constraints:
          - node.role == manager
    networks:
      - traefik-public

  whoami:
    image: traefik/whoami
    deploy:
      replicas: 3
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.whoami.rule=Host(`whoami.localhost`)"
        - "traefik.http.services.whoami.loadbalancer.server.port=80"
    networks:
      - traefik-public

networks:
  traefik-public:
    driver: overlay
```

---

## Модуль 6: Docker Swarm - Продвинутое использование (35 минут)

### 🎯 Напоминалка

**Secrets в Swarm:**
```bash
# Создание secret из файла
echo "my_secret_password" | docker secret create db_password -

# Из файла
docker secret create db_password ./password.txt

# Просмотр
docker secret ls
docker secret inspect db_password

# Использование в сервисе
docker service create \
  --name db \
  --secret db_password \
  postgres:15-alpine

# В контейнере secret доступен в /run/secrets/db_password
```

**Secrets в docker-compose для Swarm:**
```yaml
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    secrets:
      - db_password
      - db_user
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_USER_FILE: /run/secrets/db_user

secrets:
  db_password:
    external: true  # Создан заранее
  db_user:
    file: ./secrets/db_user.txt  # Из файла при деплое
```

**Configs (для не-секретных конфигов):**
```bash
# Создание config
docker config create nginx_config nginx.conf

# Использование
docker service create \
  --name web \
  --config source=nginx_config,target=/etc/nginx/nginx.conf \
  nginx:alpine
```

**Configs в compose:**
```yaml
services:
  web:
    image: nginx:alpine
    configs:
      - source: nginx_config
        target: /etc/nginx/nginx.conf
        mode: 0440

configs:
  nginx_config:
    file: ./nginx.conf
```

**Overlay networks с шифрованием:**
```bash
# Создание overlay сети
docker network create \
  --driver overlay \
  --subnet 10.0.9.0/24 \
  --opt encrypted \
  my-overlay-net

# Attachable для standalone контейнеров
docker network create \
  --driver overlay \
  --attachable \
  my-network
```

**Health checks:**
```yaml
services:
  web:
    image: nginx:alpine
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    deploy:
      replicas: 3
      update_config:
        failure_action: rollback
        monitor: 60s  # Мониторить health после обновления
```

**Constraints и Preferences:**
```yaml
services:
  db:
    image: postgres:15
    deploy:
      placement:
        constraints:
          # Только на manager нодах
          - node.role == manager
          # На нодах с конкретным label
          - node.labels.disk == ssd
          # На конкретной ноде
          - node.hostname == node1
        preferences:
          # Распределить по датацентрам
          - spread: node.labels.datacenter
          # Или по зонам доступности
          - spread: node.labels.zone
```

**Global mode (по одному на каждой ноде):**
```yaml
services:
  monitoring-agent:
    image: prom/node-exporter
    deploy:
      mode: global  # Вместо replicas
      restart_policy:
        condition: on-failure
```

### 💻 Задание

Создай production-ready стек с WordPress, используя все фичи Swarm:

1. Создай secrets:
   ```bash
   echo "wordpress_db_password_123" | docker secret create wp_db_password -
   echo "mysql_root_pass_456" | docker secret create mysql_root_password -
   echo "wpuser" | docker secret create wp_db_user -
   ```

2. Создай configs:
   
   `nginx.conf`:
   ```nginx
   server {
       listen 80;
       server_name _;
       client_max_body_size 64M;
       
       location / {
           proxy_pass http://wordpress:80;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       }
   }
   ```
   
   ```bash
   docker config create nginx_wp_config nginx.conf
   ```

3. Создай `wordpress-prod-stack.yml`:
   ```yaml
   version: '3.8'
   
   services:
     nginx:
       image: nginx:alpine
       ports:
         - "80:80"
       configs:
         - source: nginx_config
           target: /etc/nginx/conf.d/default.conf
       networks:
         - frontend
       deploy:
         replicas: 2
         update_config:
           parallelism: 1
           delay: 10s
         restart_policy:
           condition: on-failure
         placement:
           constraints:
             - node.role == worker
       healthcheck:
         test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost"]
         interval: 30s
         timeout: 5s
         retries: 3
     
     wordpress:
       image: wordpress:latest
       secrets:
         - wp_db_password
         - wp_db_user
       environment:
         WORDPRESS_DB_HOST: db:3306
         WORDPRESS_DB_USER_FILE: /run/secrets/wp_db_user
         WORDPRESS_DB_PASSWORD_FILE: /run/secrets/wp_db_password
         WORDPRESS_DB_NAME: wordpress
       volumes:
         - wp-content:/var/www/html/wp-content
       networks:
         - frontend
         - backend
       deploy:
         replicas: 3
         update_config:
           parallelism: 1
           delay: 10s
           failure_action: rollback
           monitor: 60s
         restart_policy:
           condition: on-failure
           delay: 5s
           max_attempts: 3
         resources:
           limits:
             cpus: '0.5'
             memory: 512M
           reservations:
             cpus: '0.25'
             memory: 256M
       healthcheck:
         test: ["CMD", "curl", "-f", "http://localhost/wp-admin/install.php"]
         interval: 30s
         timeout: 10s
         retries: 3
         start_period: 60s
     
     db:
       image: mysql:8.0
       secrets:
         - mysql_root_password
         - wp_db_password
         - wp_db_user
       environment:
         MYSQL_ROOT_PASSWORD_FILE: /run/secrets/mysql_root_password
         MYSQL_DATABASE: wordpress
         MYSQL_USER_FILE: /run/secrets/wp_db_user
         MYSQL_PASSWORD_FILE: /run/secrets/wp_db_password
       volumes:
         - db-data:/var/lib/mysql
       networks:
         - backend
       deploy:
         replicas: 1
         placement:
           constraints:
             - node.labels.database == true
         restart_policy:
           condition: on-failure
         resources:
           limits:
             cpus: '1'
             memory: 1G
           reservations:
             cpus: '0.5'
             memory: 512M
       healthcheck:
         test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
         interval: 30s
         timeout: 10s
         retries: 5
         start_period: 60s
     
     backup:
       image: alpine:latest
       command: >
         sh -c "while true; do
           echo 'Backup running...';
           sleep 86400;
         done"
       volumes:
         - db-data:/data:ro
         - backups:/backups
       networks:
         - backend
       deploy:
         mode: replicated
         replicas: 1
         restart_policy:
           condition: on-failure
   
   networks:
     frontend:
       driver: overlay
     backend:
       driver: overlay
       internal: true
   
   volumes:
     wp-content:
       driver: local
     db-data:
       driver: local
     backups:
       driver: local
   
   configs:
     nginx_config:
       external: true
       name: nginx_wp_config
   
   secrets:
     wp_db_password:
       external: true
     wp_db_user:
       external: true
     mysql_root_password:
       external: true
   ```

4. Добавь label на ноду для БД:
   ```bash
   docker node update --label-add database=true $(docker node ls -q)
   ```

5. Задеплой стек:
   ```bash
   docker stack deploy -c wordpress-prod-stack.yml wp
   ```

6. Проверь статус:
   ```bash
   docker stack services wp
   docker stack ps wp
   docker service logs wp_wordpress
   ```

7. Протестируй rolling update:
   ```bash
   docker service update --image wordpress:6.4 wp_wordpress
   ```

8. Помониторь обновление:
   ```bash
   watch docker service ps wp_wordpress
   ```

### 🚀 Бонус (новое)

Добавь полноценный мониторинг и логирование:

1. **Prometheus + Grafana для метрик:**
   ```yaml
   services:
     prometheus:
       image: prom/prometheus:latest
       configs:
         - source: prometheus_config
           target: /etc/prometheus/prometheus.yml
       ports:
         - "9090:9090"
       networks:
         - monitoring
       deploy:
         mode: replicated
         replicas: 1
         placement:
           constraints:
             - node.role == manager
     
     grafana:
       image: grafana/grafana:latest
       ports:
         - "3000:3000"
       volumes:
         - grafana-data:/var/lib/grafana
       networks:
         - monitoring
       deploy:
         mode: replicated
         replicas: 1
     
     node-exporter:
       image: prom/node-exporter:latest
       volumes:
         - /proc:/host/proc:ro
         - /sys:/host/sys:ro
         - /:/rootfs:ro
       command:
         - '--path.procfs=/host/proc'
         - '--path.sysfs=/host/sys'
         - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($|/)'
       networks:
         - monitoring
       deploy:
         mode: global  # На каждой ноде
   
   networks:
     monitoring:
       driver: overlay
   ```

2. **Automated backup script** (создай как отдельный сервис)

3. **Traefik с Let's Encrypt** для HTTPS

---

## Модуль 7: CI/CD интеграция (30 минут)

### 🎯 Напоминалка

**Docker Registry (приватный):**
```bash
# Запуск локального registry
docker run -d -p 5000:5000 --name registry registry:2

# Тегирование и push
docker tag myapp:latest localhost:5000/myapp:latest
docker push localhost:5000/myapp:latest

# Pull с приватного registry
docker pull localhost:5000/myapp:latest

# Registry с аутентификацией
docker run -d -p 5000:5000 \
  -v $(pwd)/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
  registry:2

# Создание пользователя
docker run --rm --entrypoint htpasswd registry:2 \
  -Bbn myuser mypassword > auth/htpasswd

# Логин
docker login localhost:5000
```

**Multi-stage builds для CI/CD:**
```dockerfile
# syntax=docker/dockerfile:1

# Стадия сборки
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Стадия тестирования
FROM builder AS tester
RUN npm ci
RUN npm test

# Финальный образ
FROM nginx:alpine AS production
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1
CMD ["nginx", "-g", "daemon off;"]
```

**BuildKit для оптимизации:**
```bash
# Включение BuildKit
export DOCKER_BUILDKIT=1

# Или в daemon.json
{
  "features": {
    "buildkit": true
  }
}

# Использование cache mounts
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.* ./
RUN --mount=type=cache,target=/go/pkg/mod \
    go mod download
COPY . .
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    go build -o app .
```

**Docker Compose для CI:**
```yaml
# docker-compose.test.yml
version: '3.8'

services:
  test:
    build:
      context: .
      target: tester
    command: npm test
    environment:
      - CI=true
  
  integration:
    build: .
    depends_on:
      - db
      - redis
    command: npm run test:integration
    environment:
      - DB_HOST=db
      - REDIS_HOST=redis
  
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: test
  
  redis:
    image: redis:alpine
```

**GitHub Actions пример:**
```yaml
# .github/workflows/docker.yml
name: Docker Build and Push

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: docker.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=sha
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=registry,ref=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:buildcache
          cache-to: type=registry,ref=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:buildcache,mode=max
      
      - name: Run tests
        run: |
          docker compose -f docker-compose.test.yml up --abort-on-container-exit
```

### 💻 Задание

Настрой полный CI/CD пайплайн для приложения:

1. Создай структуру проекта:
   ```
   app-cicd/
   ├── Dockerfile
   ├── docker-compose.yml
   ├── docker-compose.test.yml
   ├── docker-compose.prod.yml
   ├── .dockerignore
   ├── Makefile
   └── app/
       ├── server.js
       ├── package.json
       └── test/
   ```

2. `Dockerfile` (multi-stage):
   ```dockerfile
   FROM node:18-alpine AS base
   WORKDIR /app
   COPY package*.json ./
   
   FROM base AS development
   RUN npm install
   COPY . .
   CMD ["npm", "run", "dev"]
   
   FROM base AS builder
   RUN npm ci --only=production
   COPY . .
   
   FROM base AS tester
   RUN npm ci
   COPY . .
   RUN npm test
   
   FROM node:18-alpine AS production
   WORKDIR /app
   COPY --from=builder /app/node_modules ./node_modules
   COPY --from=builder /app/package*.json ./
   COPY --from=builder /app/app ./app
   USER node
   EXPOSE 3000
   HEALTHCHECK --interval=30s --timeout=3s --start-period=5s \
     CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"
   CMD ["node", "app/server.js"]
   ```

3. `docker-compose.test.yml`:
   ```yaml
   version: '3.8'
   
   services:
     unit-tests:
       build:
         context: .
         target: tester
       command: npm test
       environment:
         - NODE_ENV=test
     
     integration-tests:
       build:
         context: .
         target: tester
       command: npm run test:integration
       depends_on:
         - redis
         - postgres
       environment:
         - NODE_ENV=test
         - REDIS_URL=redis://redis:6379
         - DATABASE_URL=postgresql://test:test@postgres:5432/testdb
     
     redis:
       image: redis:alpine
     
     postgres:
       image: postgres:15-alpine
       environment:
         POSTGRES_USER: test
         POSTGRES_PASSWORD: test
         POSTGRES_DB: testdb
   ```

4. `Makefile`:
   ```makefile
   .PHONY: build test push deploy clean
   
   IMAGE_NAME := myapp
   VERSION := $(shell git rev-parse --short HEAD)
   REGISTRY := localhost:5000
   
   build:
   	docker build -t $(IMAGE_NAME):$(VERSION) .
   	docker tag $(IMAGE_NAME):$(VERSION) $(IMAGE_NAME):latest
   
   test:
   	docker compose -f docker-compose.test.yml up --abort-on-container-exit
   	docker compose -f docker-compose.test.yml down -v
   
   push:
   	docker tag $(IMAGE_NAME):$(VERSION) $(REGISTRY)/$(IMAGE_NAME):$(VERSION)
   	docker tag $(IMAGE_NAME):latest $(REGISTRY)/$(IMAGE_NAME):latest
   	docker push $(REGISTRY)/$(IMAGE_NAME):$(VERSION)
   	docker push $(REGISTRY)/$(IMAGE_NAME):latest
   
   deploy:
   	docker stack deploy -c docker-compose.prod.yml $(IMAGE_NAME)
   
   clean:
   	docker compose down -v
   	docker rmi $(IMAGE_NAME):$(VERSION) $(IMAGE_NAME):latest || true
   
   all: clean build test push deploy
   ```

5. `.dockerignore`:
   ```
   node_modules
   npm-debug.log
   .git
   .gitignore
   README.md
   .env
   .env.local
   .DS_Store
   coverage
   .vscode
   ```

6. Создай простое приложение `app/server.js`:
   ```javascript
   const http = require('http');
   
   const server = http.createServer((req, res) => {
     if (req.url === '/health') {
       res.writeHead(200);
       res.end('OK');
     } else {
       res.writeHead(200);
       res.end('Hello from Docker!\n');
     }
   });
   
   server.listen(3000, () => {
     console.log('Server running on port 3000');
   });
   ```

7. `app/package.json`:
   ```json
   {
     "name": "docker-app",
     "version": "1.0.0",
     "scripts": {
       "dev": "node server.js",
       "test": "echo 'Tests passed' && exit 0",
       "test:integration": "echo 'Integration tests passed' && exit 0"
     }
   }
   ```

8. Запусти полный цикл:
   ```bash
   make build
   make test
   ```

### 🚀 Бонус (новое)

1. Настрой **локальный Docker Registry** с UI:
   ```yaml
   services:
     registry:
       image: registry:2
       ports:
         - "5000:5000"
       volumes:
         - registry-data:/var/lib/registry
     
     registry-ui:
       image: joxit/docker-registry-ui:latest
       ports:
         - "8080:80"
       environment:
         - REGISTRY_URL=http://registry:5000
         - DELETE_IMAGES=true
       depends_on:
         - registry
   ```

2. Добавь **Hadolint** для линтинга Dockerfile в CI

3. Создай **GitLab CI** или **Jenkins** пайплайн

---

## Финальный проект (60 минут)

### Задача: Production-ready микросервисная архитектура с мониторингом

Создай полноценный стек из нескольких микросервисов с Swarm деплоем, мониторингом, логированием и CI/CD.

**Архитектура:**
- Frontend (Nginx + React build)
- API Gateway (Nginx reverse proxy)
- Auth Service (Node.js)
- Users API (Python FastAPI)
- Posts API (Node.js Express)
- PostgreSQL (с репликацией)
- Redis (кеш и сессии)
- Prometheus + Grafana (мониторинг)
- Loki + Promtail (логи)
- Traefik (load balancer + SSL)

**Требования:**

1. **Структура проекта:**
   ```
   microservices-stack/
   ├── docker-compose.yml           # Development
   ├── docker-compose.prod.yml      # Production (Swarm)
   ├── Makefile
   ├── .env.example
   ├── services/
   │   ├── frontend/
   │   │   ├── Dockerfile
   │   │   └── nginx.conf
   │   ├── auth-service/
   │   │   ├── Dockerfile
   │   │   ├── package.json
   │   │   └── src/
   │   ├── users-api/
   │   │   ├── Dockerfile
   │   │   ├── requirements.txt
   │   │   └── app/
   │   └── posts-api/
   │       ├── Dockerfile
   │       └── src/
   ├── monitoring/
   │   ├── prometheus/
   │   │   └── prometheus.yml
   │   ├── grafana/
   │   │   └── dashboards/
   │   └── loki/
   │       └── loki-config.yml
   ├── proxy/
   │   └── traefik.yml
   └── scripts/
       ├── deploy.sh
       ├── backup.sh
       └── rollback.sh
   ```

2. **docker-compose.prod.yml для Swarm:**
   ```yaml
   version: '3.8'
   
   x-deploy-policy: &default-deploy
     restart_policy:
       condition: on-failure
       delay: 5s
       max_attempts: 3
     update_config:
       parallelism: 1
       delay: 10s
       failure_action: rollback
       monitor: 60s
   
   services:
     traefik:
       image: traefik:v2.10
       command:
         - "--api.dashboard=true"
         - "--providers.docker.swarmMode=true"
         - "--providers.docker.exposedbydefault=false"
         - "--entrypoints.web.address=:80"
         - "--entrypoints.websecure.address=:443"
         - "--metrics.prometheus=true"
       ports:
         - "80:80"
         - "443:443"
         - "8080:8080"
       volumes:
         - /var/run/docker.sock:/var/run/docker.sock:ro
       networks:
         - traefik-public
       deploy:
         placement:
           constraints:
             - node.role == manager
         labels:
           - "traefik.enable=true"
           - "traefik.http.routers.traefik.rule=Host(`traefik.localhost`)"
           - "traefik.http.services.traefik.loadbalancer.server.port=8080"
     
     frontend:
       image: ${REGISTRY}/frontend:${VERSION:-latest}
       networks:
         - traefik-public
       deploy:
         <<: *default-deploy
         replicas: 2
         labels:
           - "traefik.enable=true"
           - "traefik.http.routers.frontend.rule=Host(`app.localhost`)"
           - "traefik.http.services.frontend.loadbalancer.server.port=80"
       healthcheck:
         test: ["CMD", "wget", "-q", "--spider", "http://localhost"]
         interval: 30s
         timeout: 5s
         retries: 3
     
     auth-service:
       image: ${REGISTRY}/auth-service:${VERSION:-latest}
       secrets:
         - jwt_secret
       environment:
         - REDIS_URL=redis://redis:6379
         - JWT_SECRET_FILE=/run/secrets/jwt_secret
       networks:
         - traefik-public
         - backend
       deploy:
         <<: *default-deploy
         replicas: 3
         labels:
           - "traefik.enable=true"
           - "traefik.http.routers.auth.rule=Host(`app.localhost`) && PathPrefix(`/api/auth`)"
           - "traefik.http.services.auth.loadbalancer.server.port=3000"
       healthcheck:
         test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
         interval: 30s
         timeout: 10s
         retries: 3
     
     users-api:
       image: ${REGISTRY}/users-api:${VERSION:-latest}
       secrets:
         - db_password
       environment:
         - DATABASE_URL=postgresql://users:@db:5432/usersdb
         - DB_PASSWORD_FILE=/run/secrets/db_password
         - REDIS_URL=redis://redis:6379
       networks:
         - traefik-public
         - backend
       deploy:
         <<: *default-deploy
         replicas: 2
         labels:
           - "traefik.enable=true"
           - "traefik.http.routers.users.rule=Host(`app.localhost`) && PathPrefix(`/api/users`)"
           - "traefik.http.services.users.loadbalancer.server.port=8000"
       healthcheck:
         test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
         interval: 30s
         timeout: 10s
         retries: 3
     
     posts-api:
       image: ${REGISTRY}/posts-api:${VERSION:-latest}
       secrets:
         - db_password
       environment:
         - DATABASE_URL=postgresql://posts:@db:5432/postsdb
         - DB_PASSWORD_FILE=/run/secrets/db_password
         - REDIS_URL=redis://redis:6379
       networks:
         - traefik-public
         - backend
       deploy:
         <<: *default-deploy
         replicas: 2
         labels:
           - "traefik.enable=true"
           - "traefik.http.routers.posts.rule=Host(`app.localhost`) && PathPrefix(`/api/posts`)"
           - "traefik.http.services.posts.loadbalancer.server.port=4000"
     
     db:
       image: postgres:15-alpine
       secrets:
         - db_password
       environment:
         - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
       volumes:
         - postgres-data:/var/lib/postgresql/data
       networks:
         - backend
       deploy:
         replicas: 1
         placement:
           constraints:
             - node.labels.database == true
         resources:
           limits:
             cpus: '2'
             memory: 2G
           reservations:
             cpus: '1'
             memory: 1G
       healthcheck:
         test: ["CMD", "pg_isready", "-U", "postgres"]
         interval: 10s
         timeout: 5s
         retries: 5
     
     redis:
       image: redis:7-alpine
       command: redis-server --appendonly yes
       volumes:
         - redis-data:/data
       networks:
         - backend
       deploy:
         replicas: 1
         resources:
           limits:
             cpus: '0.5'
             memory: 512M
       healthcheck:
         test: ["CMD", "redis-cli", "ping"]
         interval: 10s
         timeout: 3s
         retries: 3
     
     prometheus:
       image: prom/prometheus:latest
       command:
         - '--config.file=/etc/prometheus/prometheus.yml'
         - '--storage.tsdb.path=/prometheus'
         - '--storage.tsdb.retention.time=30d'
       configs:
         - source: prometheus_config
           target: /etc/prometheus/prometheus.yml
       volumes:
         - prometheus-data:/prometheus
       networks:
         - monitoring
         - backend
       deploy:
         replicas: 1
         placement:
           constraints:
             - node.role == manager
       healthcheck:
         test: ["CMD", "wget", "-q", "--spider", "http://localhost:9090/-/healthy"]
         interval: 30s
         timeout: 10s
         retries: 3
     
     grafana:
       image: grafana/grafana:latest
       environment:
         - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD:-admin}
         - GF_USERS_ALLOW_SIGN_UP=false
       volumes:
         - grafana-data:/var/lib/grafana
       networks:
         - monitoring
         - traefik-public
       deploy:
         replicas: 1
         labels:
           - "traefik.enable=true"
           - "traefik.http.routers.grafana.rule=Host(`grafana.localhost`)"
           - "traefik.http.services.grafana.loadbalancer.server.port=3000"
     
     loki:
       image: grafana/loki:latest
       command: -config.file=/etc/loki/loki-config.yml
       configs:
         - source: loki_config
           target: /etc/loki/loki-config.yml
       volumes:
         - loki-data:/loki
       networks:
         - monitoring
       deploy:
         replicas: 1
     
     promtail:
       image: grafana/promtail:latest
       command: -config.file=/etc/promtail/promtail-config.yml
       configs:
         - source: promtail_config
           target: /etc/promtail/promtail-config.yml
       volumes:
         - /var/lib/docker/containers:/var/lib/docker/containers:ro
         - /var/run/docker.sock:/var/run/docker.sock:ro
       networks:
         - monitoring
       deploy:
         mode: global
   
   networks:
     traefik-public:
       driver: overlay
       attachable: true
     backend:
       driver: overlay
       internal: true
     monitoring:
       driver: overlay
   
   volumes:
     postgres-data:
     redis-data:
     prometheus-data:
     grafana-data:
     loki-data:
   
   configs:
     prometheus_config:
       file: ./monitoring/prometheus/prometheus.yml
     loki_config:
       file: ./monitoring/loki/loki-config.yml
     promtail_config:
       file: ./monitoring/promtail/promtail-config.yml
   
   secrets:
     jwt_secret:
       external: true
     db_password:
       external: true
   ```

3. **Makefile для автоматизации:**
   ```makefile
   .PHONY: help build test push deploy rollback clean
   
   VERSION := $(shell git rev-parse --short HEAD)
   REGISTRY := localhost:5000
   STACK_NAME := myapp
   
   help:
   	@echo "Available commands:"
   	@echo "  make build     - Build all Docker images"
   	@echo "  make test      - Run tests"
   	@echo "  make push      - Push images to registry"
   	@echo "  make deploy    - Deploy stack to Swarm"
   	@echo "  make rollback  - Rollback to previous version"
   	@echo "  make clean     - Clean up resources"
   
   build:
   	@echo "Building images..."
   	docker build -t $(REGISTRY)/frontend:$(VERSION) services/frontend/
   	docker build -t $(REGISTRY)/auth-service:$(VERSION) services/auth-service/
   	docker build -t $(REGISTRY)/users-api:$(VERSION) services/users-api/
   	docker build -t $(REGISTRY)/posts-api:$(VERSION) services/posts-api/
   	@echo "Tagging as latest..."
   	docker tag $(REGISTRY)/frontend:$(VERSION) $(REGISTRY)/frontend:latest
   	docker tag $(REGISTRY)/auth-service:$(VERSION) $(REGISTRY)/auth-service:latest
   	docker tag $(REGISTRY)/users-api:$(VERSION) $(REGISTRY)/users-api:latest
   	docker tag $(REGISTRY)/posts-api:$(VERSION) $(REGISTRY)/posts-api:latest
   
   test:
   	@echo "Running tests..."
   	docker compose -f docker-compose.test.yml up --abort-on-container-exit
   	docker compose -f docker-compose.test.yml down -v
   
   push:
   	@echo "Pushing images to registry..."
   	docker push $(REGISTRY)/frontend:$(VERSION)
   	docker push $(REGISTRY)/auth-service:$(VERSION)
   	docker push $(REGISTRY)/users-api:$(VERSION)
   	docker push $(REGISTRY)/posts-api:$(VERSION)
   	docker push $(REGISTRY)/frontend:latest
   	docker push $(REGISTRY)/auth-service:latest
   	docker push $(REGISTRY)/users-api:latest
   	docker push $(REGISTRY)/posts-api:latest
   
   secrets:
   	@echo "Creating secrets..."
   	@echo "my_jwt_secret_key_123" | docker secret create jwt_secret - 2>/dev/null || true
   	@echo "postgres_password_456" | docker secret create db_password - 2>/dev/null || true
   
   labels:
   	@echo "Adding node labels..."
   	docker node update --label-add database=true $(docker node ls -q)
   
   deploy: secrets labels
   	@echo "Deploying stack $(STACK_NAME)..."
   	REGISTRY=$(REGISTRY) VERSION=$(VERSION) \
   	  docker stack deploy -c docker-compose.prod.yml $(STACK_NAME)
   	@echo "Waiting for services to start..."
   	sleep 10
   	docker stack services $(STACK_NAME)
   
   status:
   	@echo "Stack status:"
   	docker stack services $(STACK_NAME)
   	@echo "\nTasks:"
   	docker stack ps $(STACK_NAME) --no-trunc
   
   logs:
   	@read -p "Service name: " service; \
   	docker service logs -f $(STACK_NAME)_$service
   
   rollback:
   	@read -p "Service to rollback: " service; \
   	docker service rollback $(STACK_NAME)_$service
   
   scale:
   	@read -p "Service name: " service; \
   	read -p "Replicas: " replicas; \
   	docker service scale $(STACK_NAME)_$service=$replicas
   
   clean:
   	@echo "Removing stack..."
   	docker stack rm $(STACK_NAME)
   	@echo "Waiting for cleanup..."
   	sleep 10
   	docker volume prune -f
   	docker network prune -f
   
   ci: build test push deploy
   
   .DEFAULT_GOAL := help
   ```

4. **monitoring/prometheus/prometheus.yml:**
   ```yaml
   global:
     scrape_interval: 15s
     evaluation_interval: 15s
   
   scrape_configs:
     - job_name: 'traefik'
       static_configs:
         - targets: ['traefik:8080']
     
     - job_name: 'auth-service'
       dns_sd_configs:
         - names: ['tasks.auth-service']
           type: 'A'
           port: 3000
     
     - job_name: 'users-api'
       dns_sd_configs:
         - names: ['tasks.users-api']
           type: 'A'
           port: 8000
     
     - job_name: 'posts-api'
       dns_sd_configs:
         - names: ['tasks.posts-api']
           type: 'A'
           port: 4000
   ```

5. **scripts/deploy.sh:**
   ```bash
   #!/bin/bash
   set -e
   
   echo "=== Microservices Deployment ==="
   
   # Check if Swarm is initialized
   if ! docker info | grep -q "Swarm: active"; then
     echo "Initializing Docker Swarm..."
     docker swarm init
   fi
   
   # Get current version
   VERSION=$(git rev-parse --short HEAD)
   echo "Deploying version: $VERSION"
   
   # Build and push
   make build push
   
   # Deploy
   make deploy
   
   # Wait and show status
   echo "Waiting for services to stabilize..."
   sleep 30
   make status
   
   echo "=== Deployment complete ==="
   echo "Access points:"
   echo "  - Application: http://app.localhost"
   echo "  - Traefik Dashboard: http://traefik.localhost:8080"
   echo "  - Grafana: http://grafana.localhost:3000"
   ```

6. **Задачи:**
   - Создай все директории и базовые файлы
   - Напиши простые mock-сервисы (достаточно простого HTTP сервера)
   - Настрой Prometheus для сбора метрик
   - Настрой Grafana дашборд для визуализации
   - Протестируй rolling updates
   - Протестируй rollback
   - Создай скрипт для резервного копирования volumes

### 🚀 Бонус задания

1. **Blue-Green Deployment:**
   - Создай два стека (blue и green)
   - Настрой переключение между ними через Traefik

2. **Automated Backups:**
   - Создай сервис для автоматического бэкапа PostgreSQL
   - Загрузка в S3 или локальное хранилище

3. **Health Monitoring & Alerts:**
   - Настрой Alertmanager для Prometheus
   - Интеграция со Slack/Telegram для уведомлений

4. **Service Mesh (Опционально):**
   - Попробуй добавить Consul для service discovery
   - Или Linkerd для advanced networking

---

## Справочная секция: Быстрые шпаргалки

### Docker Commands Cheatsheet

```bash
# === Образы ===
docker images                          # Список образов
docker pull nginx:alpine              # Скачать образ
docker build -t myapp:v1 .           # Собрать образ
docker tag myapp:v1 myapp:latest     # Тегировать
docker rmi myapp:v1                  # Удалить образ
docker image prune -a                # Удалить неиспользуемые
docker history myapp:v1              # История слоев
docker inspect myapp:v1              # Детальная информация

# === Контейнеры ===
docker ps                             # Запущенные
docker ps -a                          # Все контейнеры
docker run -d -p 80:80 nginx         # Запустить
docker stop container_id              # Остановить
docker start container_id             # Запустить
docker restart container_id           # Перезапустить
docker rm container_id                # Удалить
docker rm -f $(docker ps -aq)        # Удалить все
docker logs -f container_id          # Логи (follow)
docker exec -it container_id bash    # Войти в контейнер
docker cp file.txt container:/path   # Скопировать в контейнер
docker stats                          # Статистика ресурсов
docker top container_id              # Процессы в контейнере

# === Networks ===
docker network ls                     # Список сетей
docker network create mynet          # Создать сеть
docker network inspect mynet         # Информация
docker network connect mynet cont    # Подключить контейнер
docker network rm mynet              # Удалить сеть

# === Volumes ===
docker volume ls                      # Список volumes
docker volume create mydata          # Создать volume
docker volume inspect mydata         # Информация
docker volume rm mydata              # Удалить
docker volume prune                  # Удалить неиспользуемые

# === System ===
docker system df                     # Использование диска
docker system prune                  # Очистить всё неиспользуемое
docker system prune -a --volumes    # Очистить всё включая образы
```

### Docker Compose Commands

```bash
# === Основные команды ===
docker compose up                    # Запустить (foreground)
docker compose up -d                 # Запустить (background)
docker compose up --build           # Пересобрать и запустить
docker compose down                  # Остановить и удалить
docker compose down -v              # + удалить volumes
docker compose stop                  # Просто остановить
docker compose start                 # Запустить остановленные
docker compose restart              # Перезапустить

# === Управление сервисами ===
docker compose ps                    # Статус сервисов
docker compose logs                  # Логи всех сервисов
docker compose logs -f web          # Follow логи сервиса
docker compose exec web sh          # Выполнить команду
docker compose run web bash         # Разовый запуск
docker compose top                   # Процессы всех сервисов

# === Масштабирование ===
docker compose up -d --scale web=3  # Запустить 3 реплики

# === Сборка ===
docker compose build                 # Собрать все сервисы
docker compose build --no-cache web # Без кеша
docker compose pull                  # Скачать образы
docker compose push                  # Загрузить в registry

# === Валидация ===
docker compose config               # Проверить синтаксис
docker compose config --services    # Список сервисов
docker compose version              # Версия Compose
```

### Docker Swarm Commands

```bash
# === Инициализация ===
docker swarm init                    # Создать Swarm
docker swarm join-token worker      # Токен для worker
docker swarm join-token manager     # Токен для manager
docker swarm leave                   # Покинуть Swarm
docker swarm leave --force          # Принудительно (manager)

# === Управление нодами ===
docker node ls                       # Список нод
docker node inspect node_id         # Информация о ноде
docker node update --availability drain node_id  # Drain
docker node update --availability active node_id # Active
docker node update --label-add type=db node_id  # Добавить label
docker node rm node_id              # Удалить ноду

# === Services ===
docker service create --name web --replicas 3 nginx  # Создать
docker service ls                    # Список сервисов
docker service ps web               # Таски сервиса
docker service inspect web          # Информация
docker service logs web             # Логи
docker service scale web=5          # Масштабирование
docker service update --image nginx:latest web  # Обновить
docker service rollback web         # Откатить
docker service rm web               # Удалить

# === Stacks ===
docker stack deploy -c compose.yml mystack  # Деплой
docker stack ls                     # Список стеков
docker stack services mystack       # Сервисы стека
docker stack ps mystack             # Таски стека
docker stack rm mystack             # Удалить стек

# === Secrets & Configs ===
echo "secret" | docker secret create my_secret -
docker secret ls                     # Список секретов
docker secret inspect my_secret     # Информация
docker secret rm my_secret          # Удалить

docker config create nginx_conf nginx.conf
docker config ls                     # Список конфигов
docker config inspect nginx_conf    # Информация
docker config rm nginx_conf         # Удалить
```

### Dockerfile Best Practices

```dockerfile
# ✅ Используй конкретные версии
FROM node:18-alpine

# ✅ Один слой для установки пакетов
RUN apt-get update && apt-get install -y \
    package1 \
    package2 \
    && rm -rf /var/lib/apt/lists/*

# ✅ Используй .dockerignore
# node_modules, .git, tests/

# ✅ Multi-stage для уменьшения размера
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o app

FROM alpine:latest
COPY --from=builder /app/app /usr/local/bin/

# ✅ Не запускай от root
RUN adduser -D appuser
USER appuser

# ✅ Используй COPY вместо ADD (кроме tar архивов)
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

# ✅ Healthcheck
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1

# ✅ Правильный порядок (от редко меняющегося к часто)
COPY package*.json ./
RUN npm install
COPY . .

# ❌ Избегай
# Не копируй всё в начале: COPY . .
# Не используй latest в production
# Не храни секреты в образе
# Не устанавливай лишние пакеты
```

### Docker Compose Best Practices

```yaml
# ✅ Используй version 3.8+
version: '3.8'

# ✅ Используй переменные окружения
services:
  web:
    image: nginx:${NGINX_VERSION:-alpine}
    environment:
      - API_URL=${API_URL}

# ✅ Healthchecks для всех сервисов
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s

# ✅ Именованные volumes вместо bind mounts в production
volumes:
  - postgres-data:/var/lib/postgresql/data

# ✅ Зависимости с условиями
depends_on:
  db:
    condition: service_healthy

# ✅ Resource limits
deploy:
  resources:
    limits:
      cpus: '0.5'
      memory: 512M
    reservations:
      memory: 256M

# ✅ Логирование
logging:
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"

# ✅ Restart policies
restart: unless-stopped

# ❌ Избегай
# Не используй links (deprecated)
# Не экспозь порты без необходимости
# Не используй depends_on без healthcheck
```

### Полезные алиасы и функции

Добавь в `~/.bashrc` или `~/.zshrc`:

```bash
# Docker алиасы
alias d='docker'
alias dc='docker compose'
alias dps='docker ps'
alias dpsa='docker ps -a'
alias di='docker images'
alias dv='docker volume ls'
alias dn='docker network ls'
alias ds='docker stats'

# Docker Compose алиасы
alias dcup='docker compose up -d'
alias dcdown='docker compose down'
alias dclogs='docker compose logs -f'
alias dcps='docker compose ps'
alias dcbuild='docker compose build'

# Docker Swarm алиасы
alias dss='docker stack services'
alias dsps='docker stack ps'
alias dsl='docker stack ls'
alias dnl='docker node ls'
alias dsvc='docker service'

# Полезные функции
# Войти в контейнер
dexec() {
    docker exec -it $1 ${2:-sh}
}

# Остановить все контейнеры
dstopall() {
    docker stop $(docker ps -aq)
}

# Удалить все контейнеры
drmall() {
    docker rm $(docker ps -aq)
}

# Очистка всего
dclean() {
    docker system prune -a --volumes -f
}

# Логи контейнера
dlogs() {
    docker logs -f --tail 100 $1
}

# Размер образов
dsize() {
    docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
}

# Проверить healthcheck
dhealth() {
    docker inspect --format='{{json .State.Health}}' $1 | jq
}
```

### Troubleshooting Guide

**Проблема: Контейнер не запускается**
```bash
# Проверить логи
docker logs container_id

# Проверить события
docker events --since 1h

# Инспектировать контейнер
docker inspect container_id | jq '.[0].State'

# Попробовать запустить вручную
docker run -it --entrypoint sh image_name
```

**Проблема: Проблемы с сетью**
```bash
# Проверить сети контейнера
docker inspect container_id | jq '.[0].NetworkSettings'

# Проверить DNS
docker exec container_id nslookup service_name

# Проверить соединение
docker exec container_id ping other_container

# Проверить порты
docker port container_id
```

**Проблема: Мало места на диске**
```bash
# Посмотреть использование
docker system df

# Детальная информация
docker system df -v

# Очистить
docker system prune -a --volumes

# Удалить конкретные volumes
docker volume rm $(docker volume ls -qf dangling=true)
```

**Проблема: Медленная сборка**
```bash
# Использовать BuildKit
export DOCKER_BUILDKIT=1

# Cache mounts
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt

# Правильный порядок в Dockerfile
# Сначала зависимости, потом код

# Использовать .dockerignore
```

**Проблема: Контейнер потребляет много ресурсов**
```bash
# Статистика в реальном времени
docker stats

# Ограничить ресурсы
docker run -m 512m --cpus=0.5 image_name

# В Compose
deploy:
  resources:
    limits:
      memory: 512M
      cpus: '0.5'
```

**Проблема: Service не обновляется в Swarm**
```bash
# Проверить статус обновления
docker service ps service_name

# Посмотреть логи
docker service logs service_name

# Форсировать обновление
docker service update --force service_name

# Откатить
docker service rollback service_name
```

---

## Продвинутые темы для дальнейшего изучения

### 1. Security Best Practices
- Использование USER в Dockerfile
- Scan образов: `docker scan image_name`
- Secrets management (не хранить в ENV)
- Read-only filesystems
- Использование non-root пользователей
- AppArmor/SELinux профили
- Сканирование уязвимостей: Trivy, Clair, Snyk

### 2. Performance Optimization
- Multi-stage builds
- BuildKit cache mounts
- Layer caching стратегии
- Использование Alpine images
- Оптимизация размера образов
- Правильное использование volumes vs bind mounts

### 3. Monitoring & Logging
- Prometheus + Grafana
- ELK/EFK Stack (Elasticsearch, Logstash/Fluentd, Kibana)
- Loki + Promtail
- Jaeger для distributed tracing
- cAdvisor для метрик контейнеров

### 4. Advanced Networking
- Custom bridge networks
- Overlay networks в деталях
- Network policies
- Service mesh (Linkerd, Istio)
- Ingress controllers

### 5. Storage Solutions
- Volume drivers (NFS, GlusterFS)
- Distributed storage (Ceph, Longhorn)
- Backup стратегии
- Stateful applications в Swarm/Kubernetes

### 6. CI/CD Integration
- GitLab CI/CD
- Jenkins pipelines
- GitHub Actions
- ArgoCD для GitOps
- Automated testing в контейнерах

### 7. Orchestration Alternatives
- Kubernetes (следующий уровень)
- Nomad
- Amazon ECS
- Google Cloud Run

---

## Полезные ресурсы

### Официальная документация
- **Docker Docs**: https://docs.docker.com
- **Docker Compose**: https://docs.docker.com/compose
- **Docker Swarm**: https://docs.docker.com/engine/swarm
- **Dockerfile Reference**: https://docs.docker.com/engine/reference/builder

### Инструменты
- **Dive**: Анализ слоев образов - https://github.com/wagoodman/dive
- **Hadolint**: Линтер для Dockerfile - https://github.com/hadolint/hadolint
- **Docker Bench Security**: Проверка безопасности - https://github.com/docker/docker-bench-security
- **ctop**: Top для контейнеров - https://github.com/bcicen/ctop
- **lazydocker**: TUI для Docker - https://github.com/jesseduffield/lazydocker

### Обучающие ресурсы
- **Play with Docker**: https://labs.play-with-docker.com
- **Docker Curriculum**: https://docker-curriculum.com
- **Awesome Docker**: https://github.com/veggiemonk/awesome-docker
- **Docker Mastery Course**: Udemy

### Блоги и каналы
- Docker Blog: https://www.docker.com/blog
- Container Journal
- YouTube: TechWorld with Nana, NetworkChuck

---

## Чек-лист навыков

После прохождения курса ты должен уметь:

### Основы Docker
- ✅ Создавать эффективные Dockerfile с multi-stage builds
- ✅ Управлять образами и контейнерами
- ✅ Работать с networks и volumes
- ✅ Понимать и использовать .dockerignore
- ✅ Оптимизировать размер образов

### Docker Compose
- ✅ Создавать docker-compose.yml для различных окружений
- ✅ Использовать override файлы
- ✅ Настраивать health checks
- ✅ Работать с secrets и configs
- ✅ Управлять зависимостями между сервисами

### Docker Swarm
- ✅ Инициализировать и управлять Swarm кластером
- ✅ Деплоить стеки в Swarm
- ✅ Настраивать rolling updates и rollbacks
- ✅ Использовать placement constraints
- ✅ Работать с secrets в production
- ✅ Настраивать overlay networks

### Production Ready
- ✅ Настраивать мониторинг (Prometheus + Grafana)
- ✅ Собирать логи (Loki + Promtail)
- ✅ Реализовывать health checks
- ✅ Настраивать resource limits
- ✅ Создавать automated backups
- ✅ Интегрировать с CI/CD

### Troubleshooting
- ✅ Диагностировать проблемы с контейнерами
- ✅ Анализировать логи и метрики
- ✅ Решать сетевые проблемы
- ✅ Оптимизировать производительность

---

## План повторения

### При первом прохождении (2-3 часа):
1. Пройди модули 1-3 последовательно
2. Выполни все основные задания
3. Попробуй хотя бы одно бонусное задание
4. Создай простой стек с Compose

### При втором прохождении (через 6 месяцев):
1. Сфокусируйся на модулях 4-6 (Swarm)
2. Выполни все бонусные задания
3. Начни финальный проект
4. Добавь мониторинг в свой проект

### При третьем прохождении (через 12 месяцев):
1. Сделай весь финальный проект с нуля
2. Добавь свои улучшения
3. Попробуй продвинутые темы
4. Засеки время выполнения

### Для закрепления:
- Используй Docker для всех новых проектов
- Контейнеризуй существующие приложения
- Изучи production кейсы на GitHub
- Читай Docker blog и следи за новыми фичами
- Попробуй Kubernetes как следующий шаг

### Практика в реальных условиях:
- Создай локальную копию production окружения
- Автоматизируй деплой личных проектов
- Настрой мониторинг для своих сервисов
- Участвуй в open-source проектах с Docker
- Поделись знаниями с коллегами

---

**Удачи в освоении Docker! 🐳🚀**

*Не забывай: практика важнее теории. Запускай, ломай, чини, учись!*