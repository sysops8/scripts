# Terraform Refresh: Ежегодный/Полугодовой курс для DevOps

**Цель:** Освежить в памяти ключевые концепции Terraform за 2-3 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый раздел состоит из:
1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Небольшая конфигурация, которую нужно написать с нуля
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

---

## Модуль 1: Основы и базовый синтаксис HCL (15 минут)

### 🎯 Напоминалка

**Структура Terraform проекта:**
```
project/
├── main.tf           # Основные ресурсы
├── variables.tf      # Объявление переменных
├── outputs.tf        # Выходные значения
├── terraform.tfvars  # Значения переменных (не коммитить!)
├── versions.tf       # Версии провайдеров
└── .terraform/       # Локальный кэш (в .gitignore)
```

**Базовый синтаксис HCL:**
```hcl
# Блок провайдера
provider "aws" {
  region = "us-east-1"
}

# Блок ресурса
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name        = "web-server"
    Environment = "dev"
  }
}

# Блок data source
data "aws_ami" "ubuntu" {
  most_recent = true
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
  
  owners = ["099720109477"]
}

# Переменные
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "availability_zones" {
  description = "List of AZs"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {}
}

# Outputs
output "instance_ip" {
  description = "Public IP of instance"
  value       = aws_instance.web.public_ip
}

# Локальные переменные
locals {
  common_tags = {
    Project     = "MyApp"
    ManagedBy   = "Terraform"
    Environment = var.environment
  }
  
  instance_name = "${var.project}-${var.environment}-web"
}
```

**Ссылки на ресурсы:**
```hcl
# Атрибуты ресурса
aws_instance.web.id
aws_instance.web.public_ip

# Атрибуты data source
data.aws_ami.ubuntu.id

# Переменные
var.instance_type

# Локальные переменные
local.common_tags
```

**Основные команды:**
```bash
terraform init          # Инициализация (скачивание провайдеров)
terraform fmt           # Форматирование кода
terraform validate      # Проверка синтаксиса
terraform plan          # Просмотр изменений
terraform apply         # Применение изменений
terraform destroy       # Удаление всех ресурсов
terraform show          # Показать текущее состояние
terraform output        # Показать outputs
```

### 💻 Задание

Создай базовую конфигурацию для локального провайдера (без облака):
1. Создай файл `main.tf` с блоком `terraform` и провайдером `local`
2. Создай ресурс `local_file` который создаст файл `servers.txt` с содержимым:
   ```
   web1.example.com
   web2.example.com
   db1.example.com
   ```
3. Создай файл `variables.tf` с переменной `servers` типа `list(string)`
4. Создай файл `outputs.tf` который выводит путь к созданному файлу
5. Примени конфигурацию и проверь результат

### 🚀 Бонус (новое)

Добавь использование функций Terraform:
- Используй `join("\n", var.servers)` для формирования содержимого файла
- Создай локальную переменную `server_count` с использованием `length(var.servers)`
- Добавь output с форматированной строкой: "Created file with X servers" (используй `format()`)

---

## Модуль 2: Управление состоянием и бэкенды (20 минут)

### 🎯 Напоминалка

**Terraform State:**
```hcl
# Файл terraform.tfstate - НЕ коммитить в git!
# Содержит текущее состояние инфраструктуры
# Используется для сравнения с desired state

# Remote backend (S3 + DynamoDB)
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

# Local backend (по умолчанию)
terraform {
  backend "local" {
    path = "terraform.tfstate"
  }
}

# Terraform Cloud backend
terraform {
  cloud {
    organization = "my-org"
    
    workspaces {
      name = "my-workspace"
    }
  }
}
```

**Команды для работы со state:**
```bash
terraform state list                    # Список ресурсов
terraform state show aws_instance.web   # Детали ресурса
terraform state mv SOURCE DEST          # Переименование
terraform state rm aws_instance.web     # Удаление из state
terraform state pull > backup.tfstate   # Бэкап state
terraform state push backup.tfstate     # Восстановление

# Import существующих ресурсов
terraform import aws_instance.web i-1234567890abcdef0

# Обновление state без изменения ресурсов
terraform refresh
```

**Версионирование провайдеров:**
```hcl
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"  # >= 5.0, < 6.0
    }
    
    random = {
      source  = "hashicorp/random"
      version = ">= 3.0"
    }
  }
}

# Операторы версий:
# = 1.0.0     - точная версия
# != 1.0.0    - любая кроме этой
# > 1.0.0     - больше
# >= 1.0.0    - больше или равно
# < 2.0.0     - меньше
# <= 2.0.0    - меньше или равно
# ~> 1.0      - >= 1.0, < 2.0
# ~> 1.0.0    - >= 1.0.0, < 1.1.0
```

**Workspaces (для мультиокружений):**
```bash
terraform workspace list              # Список workspace
terraform workspace new dev           # Создать
terraform workspace select dev        # Переключиться
terraform workspace show              # Текущий
terraform workspace delete dev        # Удалить

# В коде можно использовать
terraform.workspace  # "dev", "staging", "prod"
```

### 💻 Задание

Создай конфигурацию с proper state management:
1. Создай файл `versions.tf` с требованиями к Terraform >= 1.0
2. Добавь провайдер `random` версии ~> 3.0
3. Создай ресурс `random_id` для генерации уникального ID (длина 8 байт)
4. Создай ресурс `local_file` который использует этот ID в имени файла
5. Примени конфигурацию
6. Используй команды `terraform state list` и `terraform state show` для просмотра
7. Удали ресурс `random_id` из state (но не из файла): `terraform state rm random_id.server`
8. Попробуй `terraform plan` - что увидишь?

### 🚀 Бонус (новое)

Настрой workspace-based окружения:
1. Создай три workspace: dev, staging, prod
2. Используй переменную `terraform.workspace` для выбора instance_type:
   - dev: t2.micro
   - staging: t2.small
   - prod: t2.medium
3. Создай locals блок который задает разные теги для каждого окружения
4. Переключайся между workspace и делай `terraform plan` - наблюдай разницу

---

## Модуль 3: Ресурсы и Data Sources (25 минут)

### 🎯 Напоминалка

**Жизненный цикл ресурсов:**
```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  
  lifecycle {
    create_before_destroy = true    # Сначала создать новый, потом удалить старый
    prevent_destroy       = true    # Запретить удаление через terraform destroy
    ignore_changes        = [        # Игнорировать изменения этих атрибутов
      tags["CreatedDate"],
      ami,
    ]
  }
}
```

**Meta-arguments:**
```hcl
# count - создание нескольких копий
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-abc123"
  instance_type = "t2.micro"
  
  tags = {
    Name = "web-${count.index}"  # web-0, web-1, web-2
  }
}

# for_each - создание на основе map или set
resource "aws_instance" "web" {
  for_each = toset(["web", "api", "admin"])
  
  ami           = "ami-abc123"
  instance_type = "t2.micro"
  
  tags = {
    Name = each.key          # "web", "api", "admin"
  }
}

# for_each с map
resource "aws_instance" "servers" {
  for_each = {
    web   = "t2.micro"
    api   = "t2.small"
    admin = "t2.micro"
  }
  
  ami           = "ami-abc123"
  instance_type = each.value
  
  tags = {
    Name = each.key
  }
}

# depends_on - явная зависимость
resource "aws_eip" "web" {
  depends_on = [aws_internet_gateway.main]
}

# provider - использование альтернативного провайдера
resource "aws_instance" "replica" {
  provider = aws.us-west-2
  # ...
}
```

**Data Sources - чтение существующих данных:**
```hcl
# Получение информации о VPC по тегу
data "aws_vpc" "selected" {
  filter {
    name   = "tag:Name"
    values = ["production"]
  }
}

# Получение списка availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

# Использование data source
resource "aws_subnet" "public" {
  count             = length(data.aws_availability_zones.available.names)
  availability_zone = data.aws_availability_zones.available.names[count.index]
  vpc_id            = data.aws_vpc.selected.id
  cidr_block        = cidrsubnet(data.aws_vpc.selected.cidr_block, 8, count.index)
}

# Remote state data source
data "terraform_remote_state" "network" {
  backend = "s3"
  config = {
    bucket = "terraform-state"
    key    = "network/terraform.tfstate"
    region = "us-east-1"
  }
}

# Использование данных из другого state
vpc_id = data.terraform_remote_state.network.outputs.vpc_id
```

**Динамические блоки:**
```hcl
resource "aws_security_group" "web" {
  name = "web-sg"
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}

# Переменная для динамического блока
variable "ingress_rules" {
  type = list(object({
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
  }))
  default = [
    {
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  ]
}
```

### 💻 Задание

Создай конфигурацию с множественными ресурсами:
1. Создай переменную `servers` типа `map(string)`:
   ```hcl
   {
     "web"   = "Hello from Web"
     "api"   = "Hello from API"
     "admin" = "Hello from Admin"
   }
   ```
2. Используй `for_each` для создания `local_file` ресурсов
3. Имя файла: `${each.key}.txt`
4. Содержимое файла: значение из map
5. Создай output который выводит все созданные файлы (используй `for` в output)
6. Примени конфигурацию и проверь файлы

### 🚀 Бонус (новое)

Используй динамические блоки и data sources:
1. Создай переменную со списком environment'ов: `["dev", "staging", "prod"]`
2. Используй `count` для создания файлов для каждого environment
3. Добавь data source `local_file` для чтения одного из созданных файлов
4. Создай динамический блок для генерации конфигурации на основе переменной
5. В output выведи содержимое прочитанного файла через data source

---

## Модуль 4: Переменные, Locals и Outputs (25 минут)

### 🎯 Напоминалка

**Типы переменных:**
```hcl
# Примитивные типы
variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "instance_count" {
  type    = number
  default = 1
}

variable "enable_monitoring" {
  type    = bool
  default = true
}

# Коллекции
variable "availability_zones" {
  type    = list(string)
  default = ["us-east-1a", "us-east-1b"]
}

variable "tags" {
  type = map(string)
  default = {
    Environment = "dev"
    Project     = "myapp"
  }
}

variable "allowed_ports" {
  type    = set(number)
  default = [22, 80, 443]
}

# Структурные типы
variable "server_config" {
  type = object({
    instance_type = string
    ami           = string
    disk_size     = number
    tags          = map(string)
  })
  default = {
    instance_type = "t2.micro"
    ami           = "ami-abc123"
    disk_size     = 20
    tags          = { Name = "server" }
  }
}

variable "databases" {
  type = list(object({
    name    = string
    size    = number
    backup  = bool
  }))
}

# Tuple (фиксированная длина и типы)
variable "network_config" {
  type = tuple([string, number, bool])
  # ["10.0.0.0/16", 3, true]
}

# Any - любой тип
variable "custom_config" {
  type    = any
  default = {}
}
```

**Валидация и чувствительные данные:**
```hcl
variable "instance_type" {
  type = string
  
  validation {
    condition     = can(regex("^t[2-3]\\.", var.instance_type))
    error_message = "Instance type must be t2.* or t3.*"
  }
}

variable "environment" {
  type = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod"
  }
}

variable "db_password" {
  type      = string
  sensitive = true
  # Не будет показываться в логах и output
}
```

**Способы задания значений переменных (приоритет сверху вниз):**
```bash
# 1. Командная строка
terraform apply -var="instance_type=t2.small"
terraform apply -var-file="prod.tfvars"

# 2. Переменные окружения
export TF_VAR_instance_type=t2.small

# 3. terraform.tfvars или terraform.tfvars.json (автоматически)
# 4. *.auto.tfvars или *.auto.tfvars.json (автоматически)
# 5. default значение в variables.tf
# 6. Интерактивный prompt
```

**Файл terraform.tfvars:**
```hcl
instance_type = "t2.small"
environment   = "production"

tags = {
  Project = "MyApp"
  Owner   = "DevOps"
}

servers = [
  {
    name = "web1"
    size = "small"
  },
  {
    name = "web2"
    size = "medium"
  }
]
```

**Locals - вычисляемые значения:**
```hcl
locals {
  # Простые значения
  region = "us-east-1"
  
  # Вычисляемые
  instance_name = "${var.project}-${var.environment}-web"
  
  # Условные
  instance_type = var.environment == "prod" ? "t2.large" : "t2.micro"
  
  # Списки и map
  availability_zones = ["${local.region}a", "${local.region}b", "${local.region}c"]
  
  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "Terraform"
    CreatedAt   = timestamp()
  }
  
  # Слияние тегов
  tags = merge(local.common_tags, var.additional_tags)
  
  # Сложные трансформации
  servers_by_type = {
    for server in var.servers :
    server.type => server.name...
  }
}
```

**Outputs - экспорт значений:**
```hcl
output "instance_ip" {
  description = "Public IP address of the instance"
  value       = aws_instance.web.public_ip
}

output "instance_ips" {
  description = "All instance IPs"
  value       = aws_instance.web[*].public_ip
}

output "db_connection_string" {
  description = "Database connection string"
  value       = "postgresql://${aws_db_instance.main.endpoint}/${aws_db_instance.main.name}"
  sensitive   = true  # Не показывать в логах
}

output "server_info" {
  description = "Complete server information"
  value = {
    id         = aws_instance.web.id
    public_ip  = aws_instance.web.public_ip
    private_ip = aws_instance.web.private_ip
  }
}

# Output с зависимостями
output "vpc_info" {
  depends_on = [aws_vpc.main]
  value      = data.aws_vpc.main.id
}
```

### 💻 Задание

Создай сложную систему переменных для мультиокружения:
1. Создай переменную `environment` с валидацией (только "dev", "staging", "prod")
2. Создай переменную `servers` типа `list(object)` с полями: name, type, disk_size
3. Создай locals блок который:
   - Формирует `common_tags` с Project, Environment, ManagedBy
   - Вычисляет `server_prefix` на основе окружения
   - Создает map `servers_by_type` группируя серверы по типу
4. Создай файл `terraform.tfvars` с тестовыми данными
5. Создай outputs для:
   - Списка всех имен серверов
   - Количества серверов
   - Серверов сгруппированных по типу
6. Запусти `terraform plan` и проверь outputs

### 🚀 Бонус (новое)

Создай систему с conditional resources:
1. Добавь переменную `enable_monitoring` (bool)
2. Используй `count` с условием: `count = var.enable_monitoring ? 1 : 0`
3. Создай ресурс `local_file` для мониторинга который создается только если флаг true
4. Добавь переменную `backup_enabled` и создай условный output
5. Используй тернарный оператор в locals для выбора instance_type на основе окружения:
   - prod: "large"
   - staging: "medium"
   - dev: "small"

---

## Модуль 5: Модули и переиспользование кода (30 минут)

### 🎯 Напоминалка

**Структура модуля:**
```
modules/
└── webserver/
    ├── main.tf       # Основные ресурсы
    ├── variables.tf  # Входные переменные
    ├── outputs.tf    # Выходные значения
    └── README.md     # Документация
```

**Создание модуля (modules/webserver/main.tf):**
```hcl
resource "local_file" "config" {
  filename = "${var.config_dir}/${var.server_name}.conf"
  content  = templatefile("${path.module}/templates/server.conf.tpl", {
    server_name = var.server_name
    port        = var.port
  })
}

resource "local_file" "log" {
  filename = "${var.log_dir}/${var.server_name}.log"
  content  = "Server ${var.server_name} initialized at ${timestamp()}"
}
```

**Переменные модуля (modules/webserver/variables.tf):**
```hcl
variable "server_name" {
  description = "Name of the server"
  type        = string
}

variable "port" {
  description = "Server port"
  type        = number
  default     = 80
}

variable "config_dir" {
  description = "Configuration directory"
  type        = string
  default     = "./config"
}

variable "log_dir" {
  description = "Log directory"
  type        = string
  default     = "./logs"
}

variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default     = {}
}
```

**Outputs модуля (modules/webserver/outputs.tf):**
```hcl
output "config_file" {
  description = "Path to configuration file"
  value       = local_file.config.filename
}

output "log_file" {
  description = "Path to log file"
  value       = local_file.log.filename
}

output "server_info" {
  description = "Complete server information"
  value = {
    name       = var.server_name
    port       = var.port
    config     = local_file.config.filename
  }
}
```

**Использование модуля:**
```hcl
# Использование локального модуля
module "web_server" {
  source = "./modules/webserver"
  
  server_name = "web1"
  port        = 8080
  config_dir  = "/etc/nginx"
  
  tags = {
    Environment = "production"
    Team        = "platform"
  }
}

# Доступ к outputs модуля
output "web_config" {
  value = module.web_server.config_file
}

# Множественные экземпляры модуля
module "web_servers" {
  source   = "./modules/webserver"
  for_each = toset(["web1", "web2", "web3"])
  
  server_name = each.key
  port        = 8080 + index(toset(["web1", "web2", "web3"]), each.key)
}

# Использование модуля из Registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
}

# Модуль из Git
module "consul" {
  source = "git::https://github.com/hashicorp/terraform-aws-consul.git?ref=v0.11.0"
}
```

**Meta-arguments для модулей:**
```hcl
# count
module "server" {
  source = "./modules/webserver"
  count  = 3
  
  server_name = "web-${count.index}"
}

# for_each
module "server" {
  source   = "./modules/webserver"
  for_each = var.servers
  
  server_name = each.key
  port        = each.value.port
}

# depends_on
module "app" {
  source = "./modules/app"
  
  depends_on = [module.database]
}

# providers
module "west_servers" {
  source = "./modules/webserver"
  
  providers = {
    aws = aws.us-west-2
  }
}
```

**Специальные переменные в модулях:**
```hcl
path.module   # Путь к директории модуля
path.root     # Путь к корневой директории
path.cwd      # Текущая рабочая директория
```

### 💻 Задание

Создай переиспользуемый модуль:
1. Создай директорию `modules/service`
2. В модуле должны быть ресурсы:
   - `local_file` для конфигурации сервиса
   - `local_file` для файла с метаданными (JSON с именем, портом, timestamp)
3. Входные переменные:
   - service_name (обязательная)
   - port (по умолчанию 8080)
   - config_content (по умолчанию "default config")
4. Outputs:
   - config_path
   - metadata_path
   - service_info (object)
5. В корневом main.tf используй модуль 3 раза для разных сервисов
6. Выведи информацию о всех созданных сервисах
7. Примени конфигурацию и проверь файлы

### 🚀 Бонус (новое)

Создай систему модулей с зависимостями:
1. Модуль `network` - создает файл с конфигурацией сети
2. Модуль `application` - зависит от `network`, использует его outputs
3. В корневом main.tf вызови оба модуля с явной зависимостью
4. Добавь в модуль `application` использование `templatefile()` для генерации конфига
5. Создай шаблон файла `app.conf.tpl` с переменными
6. Используй `for_each` для создания нескольких application модулей

---

## Модуль 6: Функции и выражения (25 минут)

### 🎯 Напоминалка

**Строковые функции:**
```hcl
# Форматирование
format("web-%03d", 5)                    # "web-005"
lower("HELLO")                           # "hello"
upper("hello")                           # "HELLO"
title("hello world")                     # "Hello World"
trim("  hello  ")                        # "hello"
trimprefix("www.example.com", "www.")    # "example.com"
trimsuffix("file.txt", ".txt")           # "file"

# Разделение и объединение
split(",", "a,b,c")                      # ["a", "b", "c"]
join(", ", ["a", "b", "c"])              # "a, b, c"

# Замена
replace("hello world", "world", "terraform")  # "hello terraform"
regex("^([a-z]+)-([0-9]+)$", "web-123")      # ["web", "123"]
regexall("\\d+", "a1b2c3")                   # ["1", "2", "3"]

# Работа с path
basename("/path/to/file.txt")            # "file.txt"
dirname("/path/to/file.txt")             # "/path/to"
```