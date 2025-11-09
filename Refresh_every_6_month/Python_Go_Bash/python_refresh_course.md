# Python Refresh: Ежегодный/Полугодовой курс для DevOps/SysAdmin

**Цель:** Освежить в памяти ключевые концепции Python за 2-3 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый раздел состоит из:
1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Небольшой скрипт, который нужно написать с нуля
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

---

## Модуль 1: Основы и типы данных (15 минут)

### 🎯 Напоминалка

**Базовый синтаксис:**
```python
#!/usr/bin/env python3

# Переменные (динамическая типизация)
name = "Alice"
age = 30
is_admin = True

# Множественное присваивание
x, y, z = 1, 2, 3

# f-strings (Python 3.6+)
message = f"Hello, {name}! You are {age} years old."

# Ввод от пользователя
user_input = input("Enter something: ")

# Преобразование типов
num = int("42")
text = str(100)
```

**Коллекции:**
```python
# Список (изменяемый)
servers = ["web1", "web2", "db1"]
servers.append("cache1")

# Кортеж (неизменяемый)
config = ("localhost", 8080)

# Словарь
server_info = {
    "hostname": "web1",
    "ip": "192.168.1.10",
    "port": 80
}

# Множество (уникальные значения)
ports = {80, 443, 8080}
```

**Аргументы командной строки:**
```python
import sys

script_name = sys.argv[0]
first_arg = sys.argv[1] if len(sys.argv) > 1 else "default"
all_args = sys.argv[1:]
```

### 💻 Задание

Напиши скрипт, который:
1. Запрашивает у пользователя имя сервера
2. Запрашивает IP-адрес
3. Запрашивает порт (с значением по умолчанию 22)
4. Создает словарь с этими данными
5. Выводит красиво отформатированную информацию: `"Server: [имя] at [IP]:[порт]"`

### 🚀 Бонус (новое)

Сделай так, чтобы скрипт принимал данные как аргументы командной строки (`python script.py web1 192.168.1.10 80`). Используй `argparse` для парсинга аргументов с помощью и значениями по умолчанию.

---

## Модуль 2: Условия и циклы (25 минут)

### 🎯 Напоминалка

**Условные операторы:**
```python
# Базовый синтаксис
if condition:
    print("true")
elif another_condition:
    print("also true")
else:
    print("false")

# Тернарный оператор
result = "even" if num % 2 == 0 else "odd"

# Проверка вхождения
if "key" in dictionary:
    print("exists")

# Проверка типа
if isinstance(variable, str):
    print("It's a string")
```

**Операторы сравнения:**
```python
# Стандартные
==, !=, <, >, <=, >=

# Логические
and, or, not

# Специальные
is, is not  # Проверка идентичности объектов
in, not in  # Проверка вхождения
```

**Циклы:**
```python
# For по списку
for item in items:
    print(item)

# For с индексом
for index, item in enumerate(items):
    print(f"{index}: {item}")

# For по словарю
for key, value in dictionary.items():
    print(f"{key}: {value}")

# Range
for i in range(1, 11):  # от 1 до 10
    print(i)

# While
counter = 0
while counter < 5:
    print(counter)
    counter += 1

# Управление циклом
break     # Выход из цикла
continue  # Следующая итерация
```

**List Comprehensions:**
```python
# Базовый синтаксис
squares = [x**2 for x in range(10)]

# С условием
evens = [x for x in range(20) if x % 2 == 0]

# Вложенный
matrix = [[i*j for j in range(5)] for i in range(5)]
```

### 💻 Задание

Напиши скрипт для проверки доступности портов:
1. Создай список портов: `[22, 80, 443, 8080, 3306]`
2. Создай список "открытых" портов (для теста используй случайные): `[22, 443, 8080]`
3. Пройдись по всем портам и выведи для каждого:
   - Если порт открыт: `"Port 22: OPEN"`
   - Если закрыт: `"Port 80: CLOSED"`
4. В конце выведи общее количество открытых портов

### 🚀 Бонус (новое)

Используй list comprehension для создания двух списков за один проход: открытые и закрытые порты. Затем выведи их в красивом формате с использованием `join()`.

---

## Модуль 3: Функции и модули (30 минут)

### 🎯 Напоминалка

**Функции:**
```python
# Базовая функция
def greet(name):
    return f"Hello, {name}!"

# Значения по умолчанию
def connect(host, port=22):
    return f"Connecting to {host}:{port}"

# Множественные возвращаемые значения
def get_stats():
    return cpu_usage, memory_usage, disk_usage

cpu, mem, disk = get_stats()

# *args и **kwargs
def log_message(*args, **kwargs):
    level = kwargs.get('level', 'INFO')
    print(f"[{level}]", *args)

# Lambda функции
square = lambda x: x**2
sorted_servers = sorted(servers, key=lambda s: s['load'])

# Декораторы (базово)
def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"Time: {time.time() - start}")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
```

**Импорт модулей:**
```python
# Различные способы импорта
import os
import sys
from pathlib import Path
from datetime import datetime, timedelta
import json
import subprocess

# Собственные модули
from my_module import my_function
from utils.helpers import *  # Не рекомендуется

# Условный импорт
try:
    import requests
except ImportError:
    print("Please install requests: pip install requests")
```

**Полезные встроенные функции:**
```python
len(collection)           # Длина
max(iterable)            # Максимум
min(iterable)            # Минимум
sum(iterable)            # Сумма
sorted(iterable)         # Отсортированный список
any(iterable)            # True если хотя бы один True
all(iterable)            # True если все True
enumerate(iterable)      # Индекс + значение
zip(iter1, iter2)        # Объединение итерируемых объектов
map(function, iterable)  # Применить функцию ко всем
filter(function, iter)   # Фильтровать по условию
```

### 💻 Задание

Создай модуль для работы с логами:
1. Напиши функцию `parse_log_line(line)`, которая принимает строку лога вида:
   ```
   2025-01-15 10:30:45 ERROR Connection timeout
   ```
   И возвращает словарь: `{"timestamp": ..., "level": ..., "message": ...}`

2. Напиши функцию `filter_logs(logs, level)`, которая фильтрует логи по уровню

3. Напиши функцию `count_errors(logs)`, которая считает количество ошибок

4. В основной части создай тестовый список логов и примени все функции

### 🚀 Бонус (новое)

Создай декоратор `@retry`, который автоматически повторяет функцию 3 раза при ошибке, с задержкой между попытками. Примени его к функции, которая имитирует нестабильное соединение с сервером.

---

## Модуль 4: Работа с файлами и путями (30 минут)

### 🎯 Напоминалка

**Чтение и запись файлов:**
```python
# Чтение всего файла
with open('file.txt', 'r') as f:
    content = f.read()

# Чтение построчно
with open('file.txt', 'r') as f:
    for line in f:
        print(line.strip())

# Чтение всех строк в список
with open('file.txt', 'r') as f:
    lines = f.readlines()

# Запись
with open('file.txt', 'w') as f:
    f.write("Hello World\n")

# Дозапись
with open('file.txt', 'a') as f:
    f.write("New line\n")

# Работа с JSON
import json

# Чтение JSON
with open('config.json', 'r') as f:
    config = json.load(f)

# Запись JSON
with open('output.json', 'w') as f:
    json.dump(data, f, indent=2)

# Работа с CSV
import csv

with open('data.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row['column_name'])
```

**Работа с путями (pathlib):**
```python
from pathlib import Path

# Создание пути
path = Path('/home/user/file.txt')
path = Path.home() / 'documents' / 'file.txt'

# Проверки
path.exists()        # Существует?
path.is_file()       # Это файл?
path.is_dir()        # Это директория?

# Свойства
path.name           # 'file.txt'
path.stem           # 'file'
path.suffix         # '.txt'
path.parent         # Path('/home/user')

# Операции
path.mkdir(parents=True, exist_ok=True)  # Создать директорию
path.unlink()                             # Удалить файл
path.rename('new_name.txt')               # Переименовать

# Поиск файлов
for file in Path('/var/log').glob('*.log'):
    print(file)

# Рекурсивный поиск
for file in Path('/var/log').rglob('*.log'):
    print(file)
```

**Работа с ОС:**
```python
import os
import shutil

# Переменные окружения
home = os.environ.get('HOME')
os.environ['MY_VAR'] = 'value'

# Работа с директориями
os.makedirs('/path/to/dir', exist_ok=True)
os.chdir('/path')
current_dir = os.getcwd()

# Копирование и перемещение
shutil.copy('source.txt', 'dest.txt')
shutil.move('source.txt', '/new/location/')
shutil.rmtree('/path/to/dir')  # Удалить директорию с содержимым
```

### 💻 Задание

Напиши скрипт для очистки логов:
1. Создай директорию `logs/` (если не существует)
2. Создай 5 тестовых лог-файлов с именами `app_2025-01-XX.log` (где XX - разные дни)
3. Напиши функцию, которая находит все `.log` файлы в директории
4. Напиши функцию, которая удаляет файлы старше 3 дней (для теста используй имена файлов)
5. Выведи список удаленных файлов

### 🚀 Бонус (новое)

Добавь возможность архивировать старые логи перед удалением. Используй модуль `tarfile` или `zipfile` для создания архива `old_logs_YYYY-MM-DD.tar.gz`, а затем удаляй оригинальные файлы.

---

## Модуль 5: Работа с процессами и системой (30 минут)

### 🎯 Напоминалка

**subprocess - выполнение команд:**
```python
import subprocess

# Простой запуск (Python 3.5+)
result = subprocess.run(['ls', '-l'], capture_output=True, text=True)
print(result.stdout)
print(result.returncode)

# Проверка ошибок
try:
    subprocess.run(['false'], check=True)
except subprocess.CalledProcessError as e:
    print(f"Command failed with code {e.returncode}")

# С shell (осторожно с безопасностью!)
result = subprocess.run('echo $HOME', shell=True, capture_output=True, text=True)

# Получить вывод
output = subprocess.check_output(['uname', '-a']).decode()

# Pipe между командами
p1 = subprocess.Popen(['ps', 'aux'], stdout=subprocess.PIPE)
p2 = subprocess.Popen(['grep', 'python'], stdin=p1.stdout, stdout=subprocess.PIPE)
output = p2.communicate()[0]
```

**psutil - мониторинг системы:**
```python
import psutil

# CPU
cpu_percent = psutil.cpu_percent(interval=1)
cpu_count = psutil.cpu_count()

# Память
mem = psutil.virtual_memory()
print(f"Total: {mem.total}, Used: {mem.used}, Percent: {mem.percent}%")

# Диск
disk = psutil.disk_usage('/')
print(f"Total: {disk.total}, Free: {disk.free}, Percent: {disk.percent}%")

# Сеть
net = psutil.net_io_counters()
print(f"Sent: {net.bytes_sent}, Received: {net.bytes_recv}")

# Процессы
for proc in psutil.process_iter(['pid', 'name', 'cpu_percent']):
    print(proc.info)

# Конкретный процесс
proc = psutil.Process(pid)
print(proc.name(), proc.cpu_percent(), proc.memory_info())
```

**Работа с датой и временем:**
```python
from datetime import datetime, timedelta
import time

# Текущее время
now = datetime.now()
print(now.strftime('%Y-%m-%d %H:%M:%S'))

# Парсинг
date = datetime.strptime('2025-01-15', '%Y-%m-%d')

# Арифметика
tomorrow = now + timedelta(days=1)
week_ago = now - timedelta(weeks=1)

# Unix timestamp
timestamp = time.time()
date_from_ts = datetime.fromtimestamp(timestamp)

# Задержка
time.sleep(5)  # 5 секунд
```

### 💻 Задание

Напиши скрипт мониторинга системы:
1. Создай функцию `get_system_info()`, которая собирает:
   - Использование CPU (%)
   - Использование памяти (%)
   - Использование диска для `/` (%)
   - Количество запущенных процессов
2. Создай функцию `check_thresholds()`, которая проверяет, не превышены ли пороговые значения:
   - CPU > 80%
   - Memory > 85%
   - Disk > 90%
3. Если пороги превышены, выведи предупреждение
4. Запиши результаты в JSON файл с timestamp

### 🚀 Бонус (новое)

Добавь функцию для поиска процессов, потребляющих больше всего CPU. Используй `subprocess` для выполнения команды `ps` или `psutil` для анализа процессов. Выведи топ-5 процессов с их PID, именем и использованием CPU.

---

## Модуль 6: Обработка ошибок и логирование (25 минут)

### 🎯 Напоминалка

**Try-Except:**
```python
# Базовая обработка
try:
    result = risky_operation()
except Exception as e:
    print(f"Error: {e}")

# Несколько типов ошибок
try:
    file = open('file.txt')
    content = file.read()
except FileNotFoundError:
    print("File not found")
except PermissionError:
    print("Permission denied")
except Exception as e:
    print(f"Unexpected error: {e}")
finally:
    file.close()  # Выполнится всегда

# Else блок
try:
    result = safe_operation()
except Exception:
    print("Error occurred")
else:
    print("Success!")  # Выполнится только если не было ошибок

# Raise
if value < 0:
    raise ValueError("Value must be positive")

# Custom exceptions
class ConfigError(Exception):
    pass

raise ConfigError("Invalid configuration")
```

**Logging:**
```python
import logging

# Базовая настройка
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('app.log'),
        logging.StreamHandler()  # Вывод в консоль
    ]
)

# Использование
logging.debug("Debug message")
logging.info("Info message")
logging.warning("Warning message")
logging.error("Error message")
logging.critical("Critical message")

# С переменными
logging.info(f"Processing file: {filename}")
logging.error(f"Failed to connect: {error}", exc_info=True)  # С traceback

# Создание logger
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# Rotating файлы
from logging.handlers import RotatingFileHandler
handler = RotatingFileHandler('app.log', maxBytes=1024*1024, backupCount=5)
logger.addHandler(handler)
```

**Context Managers:**
```python
# Встроенные
with open('file.txt') as f:
    content = f.read()
# Файл автоматически закроется

# Собственный context manager
from contextlib import contextmanager

@contextmanager
def timer():
    start = time.time()
    yield
    print(f"Elapsed: {time.time() - start}")

with timer():
    # Код для измерения
    time.sleep(1)
```

### 💻 Задание

Улучши скрипт резервного копирования с правильной обработкой ошибок:
1. Создай функцию `backup_file(source, destination)`, которая копирует файл
2. Добавь обработку ошибок:
   - Исходный файл не найден
   - Нет прав на чтение/запись
   - Недостаточно места на диске (имитируй через условие)
3. Настрой логирование в файл `backup.log`
4. Логируй начало операции (INFO), успех (INFO) и все ошибки (ERROR)
5. Тестируй с различными сценариями (файл существует, не существует и т.д.)

### 🚀 Бонус (новое)

Создай декоратор `@retry_with_backoff`, который повторяет функцию при ошибке с экспоненциальной задержкой (1s, 2s, 4s, 8s). Логируй каждую попытку. После N попыток пробрасывай исключение дальше.

---

## Модуль 7: Работа с API и HTTP (30 минут)

### 🎯 Напоминалка

**requests - HTTP клиент:**
```python
import requests

# GET запрос
response = requests.get('https://api.example.com/users')
print(response.status_code)
print(response.json())  # Автоматический парсинг JSON

# С параметрами
params = {'page': 1, 'limit': 10}
response = requests.get('https://api.example.com/users', params=params)

# POST запрос
data = {'username': 'admin', 'password': 'secret'}
response = requests.post('https://api.example.com/login', json=data)

# Headers
headers = {'Authorization': 'Bearer token123'}
response = requests.get('https://api.example.com/protected', headers=headers)

# Timeout
try:
    response = requests.get('https://slow-api.com', timeout=5)
except requests.Timeout:
    print("Request timed out")

# Проверка статуса
response.raise_for_status()  # Выбросит ошибку при 4xx или 5xx

# Сессии (для cookies и connection pooling)
session = requests.Session()
session.headers.update({'Authorization': 'Bearer token'})
response = session.get('https://api.example.com/data')
```

**Работа с JSON:**
```python
import json

# Сериализация
data = {'name': 'server1', 'status': 'running'}
json_string = json.dumps(data, indent=2)

# Десериализация
data = json.loads(json_string)

# Красивый вывод
print(json.dumps(data, indent=2, sort_keys=True))

# Работа с файлами
with open('config.json', 'w') as f:
    json.dump(data, f, indent=2)
```

**urllib (встроенный, без requests):**
```python
from urllib.request import urlopen
from urllib.parse import urlencode, urlparse

# GET запрос
with urlopen('https://api.example.com/data') as response:
    data = response.read().decode('utf-8')

# POST с данными
params = urlencode({'key': 'value'})
with urlopen('https://api.example.com/submit', params.encode()) as response:
    result = response.read()
```

### 💻 Задание

Напиши скрипт для мониторинга статуса веб-сервисов:
1. Создай список URL для проверки: `['https://google.com', 'https://github.com', 'https://invalid-url-12345.com']`
2. Напиши функцию `check_url(url)`, которая:
   - Делает GET запрос с timeout=5
   - Возвращает словарь: `{"url": ..., "status": ..., "response_time": ...}`
   - Обрабатывает ошибки (timeout, connection error и т.д.)
3. Проверь все URL из списка
4. Сохрани результаты в JSON файл `health_check_<timestamp>.json`

### 🚀 Бонус (новое)

Используй публичный API (например, `https://api.github.com/users/octocat` или `https://jsonplaceholder.typicode.com/users`). Напиши скрипт, который:
1. Получает список пользователей/объектов
2. Фильтрует их по какому-то критерию
3. Создает красивый текстовый отчет
4. Сохраняет отчет в файл Markdown

---

## Модуль 8: Регулярные выражения (20 минут)

### 🎯 Напоминалка

**re - модуль регулярных выражений:**
```python
import re

# Поиск
match = re.search(r'pattern', text)
if match:
    print(match.group())  # Найденный текст
    print(match.start())  # Позиция начала

# Найти все вхождения
matches = re.findall(r'\d+', text)  # Все числа
matches = re.finditer(r'pattern', text)  # Iterator объектов Match

# Замена
new_text = re.sub(r'old', 'new', text)
new_text = re.sub(r'\d+', lambda m: str(int(m.group())*2), text)

# Разделение
parts = re.split(r'[,;]', text)  # По запятой или точке с запятой

# Компиляция (для многократного использования)
pattern = re.compile(r'\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}')
ips = pattern.findall(text)

# Группы
match = re.search(r'(\d+)-(\d+)-(\d+)', '2025-01-15')
year, month, day = match.groups()

# Именованные группы
match = re.search(r'(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})', '2025-01-15')
print(match.group('year'))
```

**Частые паттерны:**
```python
# IP адрес
ip_pattern = r'\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}'

# Email (простой)
email_pattern = r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'

# URL
url_pattern = r'https?://[^\s]+'

# Дата (YYYY-MM-DD)
date_pattern = r'\d{4}-\d{2}-\d{2}'

# Телефон (различные форматы)
phone_pattern = r'(\+?\d{1,3})?[\s.-]?\(?\d{3}\)?[\s.-]?\d{3}[\s.-]?\d{4}'

# Лог-запись
log_pattern = r'(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) (\w+) (.+)'
```

**Флаги:**
```python
re.IGNORECASE  # Игнорировать регистр
re.MULTILINE   # ^ и $ для каждой строки
re.DOTALL      # . включает \n
re.VERBOSE     # Позволяет комментарии в паттерне

# Использование
re.search(r'pattern', text, re.IGNORECASE | re.MULTILINE)
```

### 💻 Мини-задание

Напиши парсер логов Apache/Nginx:
1. Создай строку лога: `'192.168.1.100 - - [15/Jan/2025:10:30:45 +0000] "GET /api/users HTTP/1.1" 200 1234'`
2. Используй регулярное выражение для извлечения:
   - IP адреса
   - Даты и времени
   - HTTP метода
   - URL
   - Статус кода
   - Размера ответа
3. Выведи результаты в структурированном виде

---

## Финальный проект (40 минут)

### Задача: Система мониторинга и алертинга сервисов

Напиши полноценный скрипт, который объединяет все изученные модули.

**Требования:**

1. **Конфигурация (JSON файл):**
   ```json
   {
     "services": [
       {"name": "Web Server", "url": "https://example.com", "timeout": 5},
       {"name": "API", "url": "https://api.example.com/health", "timeout": 3}
     ],
     "thresholds": {
       "cpu": 80,
       "memory": 85,
       "disk": 90
     },
     "alert_email": "admin@example.com"
   }
   ```

2. **Функционал:**
   - Проверка доступности веб-сервисов из конфига
   - Проверка системных метрик (CPU, Memory, Disk)
   - Логирование всех проверок
   - Сохранение результатов в JSON с timestamp
   - При превышении порогов - вывод алертов

3. **Технические требования:**
   - Используй `argparse` для опций `--config`, `--verbose`, `--alert`
   - Логирование в файл с ротацией
   - Обработка всех ошибок с понятными сообщениями
   - Модульная структура (функции для каждой задачи)
   - Используй `pathlib` для работы с путями

**Структура скрипта:**
```python
#!/usr/bin/env python3
"""
System and Service Monitoring Tool
"""

import argparse
import json
import logging
from pathlib import Path
from datetime import datetime
import requests
import psutil

# Константы
DEFAULT_CONFIG = Path.home() / '.monitoring' / 'config.json'
LOG_DIR = Path.home() / '.monitoring' / 'logs'

# Настройка логирования
def setup_logging(verbose=False):
    """Configure logging"""
    pass

# Загрузка конфигурации
def load_config(config_path):
    """Load and validate configuration"""
    pass

# Проверка сервисов
def check_service(service_config):
    """Check single service availability"""
    pass

def check_all_services(services):
    """Check all configured services"""
    pass

# Проверка системных метрик
def check_system_metrics(thresholds):
    """Check CPU, Memory, Disk usage"""
    pass

# Генерация алертов
def generate_alerts(results, thresholds):
    """Generate alerts for failed checks"""
    pass

# Сохранение результатов
def save_results(results, output_dir):
    """Save results to JSON file"""
    pass

# Основная логика
def main():
    parser = argparse.ArgumentParser(description='System and Service Monitor')
    parser.add_argument('--config', type=Path, default=DEFAULT_CONFIG,
                        help='Path to config file')
    parser.add_argument('--verbose', action='store_true',
                        help='Verbose output')
    parser.add_argument('--alert', action='store_true',
                        help='Enable alerts')
    
    args = parser.parse_args()
    
    # Ваша реализация здесь
    setup_logging(args.verbose)
    config = load_config(args.config)
    
    # Проверки
    service_results = check_all_services(config['services'])
    system_results = check_system_metrics(config['thresholds'])
    
    # Алерты
    if args.alert:
        generate_alerts(service_results, config['thresholds'])
    
    # Сохранение
    save_results({
        'timestamp': datetime.now().isoformat(),
        'services': service_results,
        'system': system_results
    }, LOG_DIR)
    
    logging.info("Monitoring completed successfully")

if __name__ == '__main__':
    main()
```

**Дополнительные улучшения (опционально):**
- Отправка email-алертов через SMTP
- Интеграция с Slack/Telegram через webhooks
- Графики метрик с использованием matplotlib
- База данных для истории метрик (SQLite)
- Веб-интерфейс для отображения статуса (Flask)

---

## Справочная секция: Быстрые шпаргалки

### Частые ошибки

1. **Изменяемые значения по умолчанию:**
   ```python
   # ❌ Неправильно
   def add_item(item, list=[]):
       list.append(item)
       return list
   
   # ✅ Правильно
   def add_item(item, list=None):
       if list is None:
           list = []
       list.append(item)
       return list
   ```

2. **Забыть про индентацию** - Python чувствителен к отступам

3. **Изменение списка во время итерации:**
   ```python
   # ❌ Неправильно
   for item in items:
       if condition:
           items.remove(item)
   
   # ✅ Правильно
   items = [item for item in items if not condition]
   ```

4. **Не закрывать файлы** - всегда используй `with open()`

5. **Игнорирование исключений:**
   ```python
   # ❌ Неправильно
   try:
       risky_operation()
   except:
       pass
   
   # ✅ Правильно
   try:
       risky_operation()
   except SpecificError as e:
       logging.error(f"Failed: {e}")
   ```

### Полезные встроенные модули для DevOps

```python
os              # Работа с ОС
sys             # Системные параметры
subprocess      # Запуск команд
pathlib         # Работа с путями (современный)
shutil          # Файловые операции высокого уровня
json            # JSON парсинг
argparse        # Парсинг аргументов CLI
logging         # Логирование
datetime        # Дата и время
time            # Время и задержки
re              # Регулярные выражения
glob            # Поиск файлов по маске
tempfile        # Временные файлы
configparser    # INI файлы
csv             # CSV файлы
xml.etree       # XML парсинг
hashlib         # Хэширование
base64          # Base64 кодирование
urllib          # HTTP без внешних зависимостей
socket          # Сетевые сокеты
threading       # Потоки
multiprocessing # Процессы
queue           # Очереди
collections     # Специальные коллекции
itertools       # Инструменты для итераторов
functools       # Функциональное программирование
```

### Полезные внешние библиотеки

```python
requests        # HTTP клиент
psutil          # Системная информация
paramiko        # SSH клиент
fabric          # Автоматизация по SSH
ansible         # Автоматизация инфраструктуры
docker          # Docker API
kubernetes      # Kubernetes API
boto3           # AWS SDK
google-cloud    # GCP SDK
azure           # Azure SDK
pyyaml          # YAML парсинг
python-dotenv   # .env файлы
click           # CLI фреймворк (альтернатива argparse)
rich            # Красивый вывод в терминал
tabulate        # Таблицы в терминале
colorama        # Цветной вывод
tqdm            # Progress bars
schedule        # Планировщик задач
```

### Шаблон профессионального скрипта

```python
#!/usr/bin/env python3
"""
Script description here.

Usage:
    ./script.py [OPTIONS] ARGUMENTS

Examples:
    ./script.py --verbose input.txt
"""

import argparse
import logging
import sys
from pathlib import Path
from typing import Optional, List, Dict

# Константы
VERSION = '1.0.0'
DEFAULT_CONFIG = Path.home() / '.config' / 'script.conf'

# Настройка логирования
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('/tmp/script.log'),
        logging.StreamHandler()
    ]
)
logger = logging.getLogger(__name__)


class ScriptError(Exception):
    """Custom exception for script errors"""
    pass


def parse_arguments() -> argparse.Namespace:
    """Parse command line arguments"""
    parser = argparse.ArgumentParser(
        description=__doc__,
        formatter_class=argparse.RawDescriptionHelpFormatter
    )
    
    parser.add_argument('input', type=Path, help='Input file')
    parser.add_argument('-o', '--output', type=Path, help='Output file')
    parser.add_argument('-v', '--verbose', action='store_true',
                        help='Verbose output')
    parser.add_argument('--version', action='version',
                        version=f'%(prog)s {VERSION}')
    
    args = parser.parse_args()
    
    if args.verbose:
        logger.setLevel(logging.DEBUG)
    
    return args


def validate_input(input_path: Path) -> None:
    """Validate input file"""
    if not input_path.exists():
        raise ScriptError(f"Input file not found: {input_path}")
    
    if not input_path.is_file():
        raise ScriptError(f"Not a file: {input_path}")
    
    logger.debug(f"Input validated: {input_path}")


def process_data(input_path: Path) -> Dict:
    """Main processing logic"""
    logger.info(f"Processing {input_path}")
    
    try:
        with open(input_path, 'r') as f:
            # Ваша логика здесь
            pass
    except Exception as e:
        raise ScriptError(f"Processing failed: {e}")
    
    return {}


def main() -> int:
    """Main entry point"""
    try:
        args = parse_arguments()
        
        logger.info("Script started")
        
        validate_input(args.input)
        result = process_data(args.input)
        
        if args.output:
            # Сохранить результат
            pass
        
        logger.info("Script completed successfully")
        return 0
        
    except ScriptError as e:
        logger.error(f"Script error: {e}")
        return 1
    except KeyboardInterrupt:
        logger.warning("Interrupted by user")
        return 130
    except Exception as e:
        logger.exception(f"Unexpected error: {e}")
        return 1


if __name__ == '__main__':
    sys.exit(main())
```

---

## Советы по прохождению курса

1. **Не подглядывай!** Сначала попробуй вспомнить/написать сам, и только потом гугли или смотри в шпаргалки

2. **Используй виртуальное окружение:**
   ```bash
   python -m venv venv
   source venv/bin/activate  # Linux/Mac
   venv\Scripts\activate     # Windows
   ```

3. **Тестируй в изолированной среде** - Docker контейнер или VM

4. **Используй линтеры:**
   ```bash
   pip install pylint flake8 black mypy
   pylint script.py
   black script.py  # Автоформатирование
   mypy script.py   # Проверка типов
   ```

5. **Фиксируй время** - засеки, сколько времени уходит на каждый модуль

6. **Пиши тесты** - используй `pytest` для проверки функций

7. **Читай документацию** - https://docs.python.org

8. **Используй REPL** - `python` или `ipython` для быстрых экспериментов

---

## План повторения

### При первом прохождении (2-3 часа):
- Пройди все модули последовательно
- Обязательно выполни все основные задания
- Бонусные задания — по желанию

### При повторном прохождении (через 6-12 месяцев):
- Сфокусируйся на бонусных заданиях
- Добавь свои усложнения к заданиям
- Засеки время и попробуй улучшить результат
- Обрати внимание на то, что забылось больше всего

### Для закрепления:
- Автоматизируй рутинные задачи на работе с помощью Python
- Напиши утилиту для личных нужд (backup, парсинг данных, мониторинг)
- Изучи реальные production скрипты в open-source проектах на GitHub
- Почитай PEP 8 (Style Guide) и PEP 20 (Zen of Python)
- Попробуй продвинутые темы: async/await, type hints, dataclasses

### Дополнительные ресурсы:
- **Real Python** - отличные туториалы
- **Python Cookbook** - паттерны и рецепты
- **Effective Python** - best practices
- **Awesome Python** - список библиотек и инструментов

---

## Чек-лист навыков

После прохождения курса ты должен уметь:

- ✅ Писать скрипты с аргументами командной строки
- ✅ Обрабатывать файлы и директории
- ✅ Работать с JSON, CSV и другими форматами
- ✅ Выполнять системные команды через subprocess
- ✅ Мониторить систему через psutil
- ✅ Делать HTTP запросы к API
- ✅ Обрабатывать ошибки правильно
- ✅ Настраивать логирование
- ✅ Использовать регулярные выражения
- ✅ Писать функции и модули
- ✅ Работать с датой и временем
- ✅ Создавать CLI утилиты

Проходя этот курс раз в 6-12 месяцев, ты не только не забудешь Python, но и постепенно прокачаешь свои навыки на продвинутых бонусных заданиях. Удачи! 🐍🚀