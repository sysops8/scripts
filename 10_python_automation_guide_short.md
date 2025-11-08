# 10 Python Automation Scripts - Пошаговая Инструкция

## Подготовка окружения

### 1. Установка Python
```bash
# Для Ubuntu/Debian
sudo apt update
sudo apt install python3 python3-pip python3-venv -y

# Для Windows - скачайте с python.org
```

### 2. Создание рабочей директории
```bash
mkdir ~/python-automation
cd ~/python-automation
```

### 3. Установка необходимых модулей
```bash
pip3 install requests
pip3 install beautifulsoup4
pip3 install schedule
pip3 install python-dotenv
pip3 install feedparser
pip3 install yt-dlp
pip3 install nltk
pip3 install kubernetes
```

---

## Script 1: Мониторинг веб-сайта + Slack уведомления

### Настройка Slack Webhook:
1. Перейдите на https://api.slack.com/apps
2. "Create New App" → "From scratch"
3. Введите имя приложения и выберите workspace
4. В меню слева: "Incoming Webhooks" → включите ON
5. "Add New Webhook to Workspace" → выберите канал
6. Скопируйте Webhook URL

### Код скрипта:
```python
# monitor_website.py
import requests
import json

def monitor_server_health(server_url, slack_webhook):
    try:
        response = requests.get(server_url, timeout=10)
        
        if response.status_code == 200:
            message = f"✅ Server is UP: {server_url}"
        else:
            message = f"⚠️ Server returned status: {response.status_code}"
    except Exception as e:
        message = f"❌ Server is DOWN: {server_url}\nError: {str(e)}"
    
    # Отправка в Slack
    slack_data = {"text": message}
    slack_response = requests.post(
        slack_webhook,
        data=json.dumps(slack_data),
        headers={'Content-Type': 'application/json'}
    )
    
    if slack_response.status_code == 200:
        print("Slack notification sent!")
    else:
        print("Failed to send Slack notification")

# Использование
SERVER_URL = "https://example.com"
SLACK_WEBHOOK = "your-webhook-url-here"

monitor_server_health(SERVER_URL, SLACK_WEBHOOK)
```

---

## Script 2: Автомасштабирование Kubernetes по погоде

### Предварительные требования:
1. Зарегистрируйтесь на https://www.weatherapi.com/
2. Получите API ключ
3. Установите kubectl и настройте доступ к кластеру

### Код скрипта:
```python
# weather_autoscaling.py
import requests
import subprocess

def get_weather_data(city):
    API_KEY = "your-weather-api-key"
    BASE_URL = f"http://api.weatherapi.com/v1/current.json?key={API_KEY}&q={city}"
    
    response = requests.get(BASE_URL)
    data = response.json()
    
    if 'error' not in data:
        weather_condition = data['current']['condition']['text']
        print(f"Weather in {city}: {weather_condition}")
        
        # Условие для масштабирования
        if "rain" in weather_condition.lower() or "heavy" in weather_condition.lower():
            print("Heavy rain detected! Scaling up pods...")
            scale_pods_using_kubectl("default", "your-deployment", 3)
        else:
            print("Weather condition is normal. No scaling needed.")
    else:
        print(f"Error fetching weather data: {data['error']['message']}")

def scale_pods_using_kubectl(namespace, deployment_name, replicas):
    try:
        command = [
            "kubectl", "scale", "deployment", 
            deployment_name, 
            f"--replicas={replicas}",
            f"-n={namespace}"
        ]
        result = subprocess.run(command, capture_output=True, text=True)
        
        if result.returncode == 0:
            print(f"Scaled {deployment_name} to {replicas} replicas")
        else:
            print(f"Error scaling: {result.stderr}")
    except Exception as e:
        print(f"Exception: {e}")

# Использование
get_weather_data("London")
```

---

## Script 3: Массовая рассылка email

### Настройка Gmail App Password:
1. Перейдите на https://myaccount.google.com/apppasswords
2. Войдите в аккаунт
3. Создайте новый App Password
4. Сохраните сгенерированный пароль

### Код скрипта:
```python
# email_campaign.py
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email import encoders
import os

def send_individual_emails(subject, body, recipients, pdf_file=None):
    EMAIL_ADDRESS = "your-email@gmail.com"
    EMAIL_PASSWORD = "your-app-password"
    
    for recipient in recipients:
        msg = MIMEMultipart('alternative')
        msg['From'] = EMAIL_ADDRESS
        msg['To'] = recipient
        msg['Subject'] = subject
        
        # HTML тело письма
        html_body = f"""
        <html>
            <body>
                <h2>Hello!</h2>
                <p>{body}</p>
                <p>Best regards,<br>Your Team</p>
            </body>
        </html>
        """
        
        msg.attach(MIMEText(html_body, 'html'))
        
        # Прикрепление PDF
        if pdf_file and os.path.exists(pdf_file):
            with open(pdf_file, 'rb') as f:
                part = MIMEBase('application', 'octet-stream')
                part.set_payload(f.read())
                encoders.encode_base64(part)
                part.add_header('Content-Disposition', f'attachment; filename={os.path.basename(pdf_file)}')
                msg.attach(part)
        
        try:
            with smtplib.SMTP_SSL('smtp.gmail.com', 465) as server:
                server.login(EMAIL_ADDRESS, EMAIL_PASSWORD)
                server.send_message(msg)
                print(f"Email sent to {recipient}")
        except Exception as e:
            print(f"Failed to send to {recipient}: {e}")

# Использование
recipients_list = [
    "user1@example.com",
    "user2@example.com"
]

send_individual_emails(
    subject="Important Update",
    body="This is a test email from Python automation.",
    recipients=recipients_list,
    pdf_file="document.pdf"
)
```

---

## Script 4: Автоматический бэкап с архивацией

### Код скрипта:
```python
# backup_automation.py
import os
import shutil
from datetime import datetime
import zipfile

def backup_and_zip_files(source_folder, backup_folder):
    if not os.path.exists(backup_folder):
        os.makedirs(backup_folder)
    
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    zip_filename = os.path.join(backup_folder, f"backup_{timestamp}.zip")
    
    try:
        with zipfile.ZipFile(zip_filename, 'w', zipfile.ZIP_DEFLATED) as zipf:
            for root, dirs, files in os.walk(source_folder):
                for file in files:
                    file_path = os.path.join(root, file)
                    arcname = os.path.relpath(file_path, source_folder)
                    zipf.write(file_path, arcname)
        
        print(f"Backup created: {zip_filename}")
    except Exception as e:
        print(f"Backup failed: {e}")

# Использование
SOURCE = r"C:\Users\YourUser\Documents\important_data"
BACKUP = r"C:\Users\YourUser\Documents\backups"

backup_and_zip_files(SOURCE, BACKUP)
```

---

## Script 5: Web Scraping (парсинг новостей)

### Код скрипта:
```python
# web_scraper.py
import requests
from bs4 import BeautifulSoup

def scrape_headlines(url):
    try:
        response = requests.get(url, timeout=10)
        soup = BeautifulSoup(response.content, 'html.parser')
        
        # Поиск заголовков (h2 теги)
        headlines = soup.find_all('h2')
        
        print(f"Found {len(headlines)} headlines:\n")
        for i, headline in enumerate(headlines[:20], 1):
            text = headline.get_text().strip()
            if text:
                print(f"{i}. {text}")
    except Exception as e:
        print(f"Error scraping: {e}")

# Использование
scrape_headlines("https://www.bbc.com/news")
```

---

## Script 6: Простой чатбот

### Код скрипта:
```python
# chatbot.py
import nltk
from nltk.chat.util import Chat, reflections

# Определение пар вопрос-ответ
pairs = [
    [
        r"(.*)help(.*)",
        ["Sure! How can I assist you?", "I'm here to help!"]
    ],
    [
        r"(.*)price(.*)",
        ["The price is $50.", "Our standard price is $50."]
    ],
    [
        r"(.*)course(.*)",
        ["We offer DevOps and Cloud courses. Visit our website!"]
    ],
    [
        r"quit",
        ["Goodbye! Have a great day!"]
    ],
]

def start_chatbot():
    print("Welcome to Customer Support Chatbot!")
    print("Type 'quit' to exit\n")
    
    chatbot = Chat(pairs, reflections)
    chatbot.converse()

# Использование
if __name__ == "__main__":
    start_chatbot()
```

---

## Script 7: Очистка старых файлов

### Код скрипта:
```python
# cleanup_directory.py
import os
import time

def cleanup_old_files(directory, days_old):
    current_time = time.time()
    days_in_seconds = days_old * 24 * 60 * 60
    
    for filename in os.listdir(directory):
        file_path = os.path.join(directory, filename)
        
        if os.path.isfile(file_path):
            file_age = current_time - os.path.getmtime(file_path)
            
            if file_age > days_in_seconds:
                try:
                    os.remove(file_path)
                    print(f"Deleted: {filename}")
                except Exception as e:
                    print(f"Error deleting {filename}: {e}")

# Использование
cleanup_old_files(r"C:\Users\YourUser\Downloads", 90)
```

---

## Script 8: Скачивание YouTube видео

### Код скрипта:
```python
# youtube_downloader.py
import yt_dlp
import feedparser

def download_latest_videos(subscription_feed_url, output_folder):
    feed = feedparser.parse(subscription_feed_url)
    
    ydl_opts = {
        'outtmpl': f'{output_folder}/%(title)s.%(ext)s',
        'format': 'best',
    }
    
    for entry in feed.entries[:5]:  # Скачать 5 последних видео
        video_url = entry.link
        
        try:
            with yt_dlp.YoutubeDL(ydl_opts) as ydl:
                ydl.download([video_url])
                print(f"Downloaded: {video_url}")
        except Exception as e:
            print(f"Failed to download {video_url}: {e}")

# Использование (замените CHANNEL_ID на ID канала)
FEED_URL = "https://www.youtube.com/feeds/videos.xml?channel_id=CHANNEL_ID"
OUTPUT = r"C:\Users\YourUser\Videos\Downloads"

download_latest_videos(FEED_URL, OUTPUT)
```

---

## Script 9: Организация файлов по типу

### Код скрипта:
```python
# file_organizer.py
import os
import shutil

def move_files_by_type(source_directory, destination_directory, file_extension):
    if not os.path.exists(destination_directory):
        os.makedirs(destination_directory)
    
    for filename in os.listdir(source_directory):
        if filename.endswith(file_extension):
            source_path = os.path.join(source_directory, filename)
            dest_path = os.path.join(destination_directory, filename)
            
            try:
                shutil.move(source_path, dest_path)
                print(f"Moved: {filename}")
            except Exception as e:
                print(f"Error moving {filename}: {e}")

# Использование
SOURCE = r"C:\Users\YourUser\Downloads"

# Перемещение PDF в папку Docs
move_files_by_type(SOURCE, f"{SOURCE}\\Docs", ".pdf")

# Перемещение изображений в папку Images
move_files_by_type(SOURCE, f"{SOURCE}\\Images", ".jpg")
move_files_by_type(SOURCE, f"{SOURCE}\\Images", ".png")

# Перемещение текстовых файлов в папку Text
move_files_by_type(SOURCE, f"{SOURCE}\\Text", ".txt")
```

---

## Script 10: Синхронизация Git репозиториев

### Код скрипта:
```python
# git_sync.py
import os
import subprocess

def clone_or_pull_repo(repo_url, local_directory):
    if not os.path.exists(local_directory):
        # Клонирование репозитория
        print(f"Cloning {repo_url}...")
        subprocess.run(["git", "clone", repo_url, local_directory])
    else:
        # Обновление существующего репозитория
        print(f"Pulling latest changes for {local_directory}...")
        os.chdir(local_directory)
        subprocess.run(["git", "pull"])
        print(f"Updated repository in {local_directory}")

# Использование
repositories = [
    ("https://github.com/user/repo1.git", r"C:\Projects\repo1"),
    ("https://github.com/user/repo2.git", r"C:\Projects\repo2"),
]

for repo_url, local_dir in repositories:
    clone_or_pull_repo(repo_url, local_dir)
```

---

## Автоматизация скриптов

### Для Linux (Cron):
```bash
# Редактировать crontab
crontab -e

# Запускать backup каждый день в 18:00
0 18 * * * /usr/bin/python3 /path/to/backup_automation.py

# Запускать мониторинг каждые 5 минут
*/5 * * * * /usr/bin/python3 /path/to/monitor_website.py
```

### Для Windows (Task Scheduler):
```powershell
# Создать задачу через PowerShell
$action = New-ScheduledTaskAction -Execute "python.exe" -Argument "C:\path\to\script.py"
$trigger = New-ScheduledTaskTrigger -Daily -At 6PM
Register-ScheduledTask -Action $action -Trigger $trigger -TaskName "PythonBackup"

# Удалить задачу
Unregister-ScheduledTask -TaskName "PythonBackup" -Confirm:$false
```

### Или через GUI Task Scheduler:
1. Откройте Task Scheduler (Планировщик заданий)
2. "Create Basic Task" (Создать простую задачу)
3. Укажите имя задачи
4. Выберите триггер (Daily/Weekly/etc)
5. Action: "Start a program"
6. Program: путь к python.exe (найти: `where python`)
7. Arguments: полный путь к скрипту
8. Finish

---

## Решение типичных проблем

### ModuleNotFoundError:
```bash
pip3 install <module-name>
```

### Ошибки с путями в Windows:
Используйте raw strings:
```python
path = r"C:\Users\Name\folder"
```

### Проблемы с kubectl:
```bash
# Установка kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Проверка подключения
kubectl get nodes
```

### Git не найден:
```bash
# Ubuntu/Debian
sudo apt install git

# Windows - скачайте с git-scm.com
```

---

## Настройка в Proxmox VM

### Создание VM:
1. Создайте Ubuntu 22.04 VM
2. RAM: минимум 2GB
3. Disk: 20GB
4. Network: подключите к bridge

### После установки:
```bash
# Обновление системы
sudo apt update && sudo apt upgrade -y

# Установка необходимого ПО
sudo apt install python3 python3-pip git curl wget -y

# Клонирование репозитория со скриптами
git clone https://github.com/yourusername/python-automation.git
cd python-automation

# Установка зависимостей
pip3 install -r requirements.txt
```

---

## Полезные команды

### Проверка версии Python:
```bash
python3 --version
```

### Создание виртуального окружения:
```bash
python3 -m venv venv
source venv/bin/activate  # Linux
venv\Scripts\activate     # Windows
```

### Список установленных модулей:
```bash
pip3 list
```

### Экспорт зависимостей:
```bash
pip3 freeze > requirements.txt
```

---

## Безопасность

⚠️ **ВАЖНО:**
- Никогда не храните API ключи и пароли в коде
- Используйте переменные окружения или .env файлы
- Добавьте .env в .gitignore
- Для продакшена используйте секреты Kubernetes или AWS Secrets Manager

### Пример .env файла:
```
WEATHER_API_KEY=your_key_here
SLACK_WEBHOOK=your_webhook_here
EMAIL_ADDRESS=your_email@gmail.com
EMAIL_PASSWORD=your_app_password
```

### Загрузка из .env:
```python
from dotenv import load_dotenv
import os

load_dotenv()
API_KEY = os.getenv('WEATHER_API_KEY')
```

---

## Дополнительные ресурсы

- [Python Documentation](https://docs.python.org/3/)
- [Kubernetes Python Client](https://github.com/kubernetes-client/python)
- [Requests Library](https://requests.readthedocs.io/)
- [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/)
