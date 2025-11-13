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
}
```

**Основные команды:**
```bash
terraform init          # Инициализация
terraform fmt           # Форматирование
terraform validate      # Проверка синтаксиса
terraform plan          # Просмотр изменений
terraform apply         # Применение
terraform destroy       # Удаление
```

### 💻 Задание

Создай базовую конфигурацию:
1. Создай `main.tf` с провайдером `local`
2. Создай ресурс `local_file` с файлом `servers.txt`
3. Создай `variables.tf` с переменной `servers` (list)
4. Создай `outputs.tf` для вывода пути к файлу
5. Примени конфигурацию

### 🚀 Бонус

- Используй `join("\n", var.servers)` для содержимого
- Создай local переменную `server_count`
- Добавь форматированный output

---

## Модуль 2: Управление состоянием и бэкенды (20 минут)

### 🎯 Напоминалка

**Terraform State:**
```hcl
# Remote backend (S3)
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

**Версионирование:**
```hcl
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

**Команды state:**
```bash
terraform state list
terraform state show aws_instance.web
terraform state mv SOURCE DEST
terraform state rm aws_instance.web
terraform import aws_instance.web i-123
```

**Workspaces:**
```bash
terraform workspace new dev
terraform workspace select dev
terraform workspace show
```

### 💻 Задание

1. Создай `versions.tf` с Terraform >= 1.0
2. Добавь провайдер `random` ~> 3.0
3. Создай `random_id` (8 байт)
4. Используй ID в имени `local_file`
5. Используй `terraform state` команды
6. Удали `random_id` из state
7. Проверь `terraform plan`

### 🚀 Бонус

Настрой workspace-based окружения с разными конфигами для dev/staging/prod.

---

## Модуль 3: Ресурсы и Data Sources (25 минут)

### 🎯 Напоминалка

**Lifecycle:**
```hcl
resource "aws_instance" "web" {
  lifecycle {
    create_before_destroy = true
    prevent_destroy       = true
    ignore_changes        = [tags]
  }
}
```

**Meta-arguments:**
```hcl
# count
resource "aws_instance" "web" {
  count = 3
  tags = {
    Name = "web-${count.index}"
  }
}

# for_each
resource "aws_instance" "web" {
  for_each = toset(["web", "api", "admin"])
  tags = {
    Name = each.key
  }
}

# depends_on
resource "aws_eip" "web" {
  depends_on = [aws_internet_gateway.main]
}
```

**Data Sources:**
```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  filter {
    name   = "name"
    values = ["ubuntu/*"]
  }
}

data "terraform_remote_state" "network" {
  backend = "s3"
  config = {
    bucket = "state-bucket"
    key    = "network/terraform.tfstate"
  }
}
```

**Dynamic блоки:**
```hcl
dynamic "ingress" {
  for_each = var.ingress_rules
  content {
    from_port   = ingress.value.from_port
    to_port     = ingress.value.to_port
    protocol    = ingress.value.protocol
  }
}
```

### 💻 Задание

1. Создай переменную `servers` (map)
2. Используй `for_each` для создания `local_file`
3. Имя файла: `${each.key}.txt`
4. Содержимое: значение из map
5. Output со всеми файлами
6. Примени и проверь

### 🚀 Бонус

Используй dynamic блоки и data sources для чтения созданных файлов.

---

## Модуль 4: Переменные, Locals и Outputs (25 минут)

### 🎯 Напоминалка

**Типы переменных:**
```hcl
variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "availability_zones" {
  type    = list(string)
  default = ["us-east-1a"]
}

variable "tags" {
  type = map(string)
  default = {}
}

variable "server_config" {
  type = object({
    instance_type = string
    disk_size     = number
  })
}
```

**Валидация:**
```hcl
variable "environment" {
  type = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Must be dev, staging, or prod"
  }
}

variable "db_password" {
  type      = string
  sensitive = true
}
```

**Locals:**
```hcl
locals {
  instance_name = "${var.project}-${var.environment}-web"
  instance_type = var.environment == "prod" ? "t2.large" : "t2.micro"
  
  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
  
  servers_by_type = {
    for server in var.servers :
    server.type => server.name...
  }
}
```

**Outputs:**
```hcl
output "instance_ip" {
  description = "Public IP"
  value       = aws_instance.web.public_ip
}

output "db_endpoint" {
  value     = aws_db_instance.main.endpoint
  sensitive = true
}
```

### 💻 Задание

1. Создай переменную `environment` с валидацией
2. Создай `servers` (list of objects)
3. Locals с `common_tags` и `servers_by_type`
4. Файл `terraform.tfvars`
5. Outputs для имен и количества серверов

### 🚀 Бонус

Conditional resources с `count` и тернарным оператором.

---

## Модуль 5: Модули и переиспользование (30 минут)

### 🎯 Напоминалка

**Структура модуля:**
```
modules/webserver/
├── main.tf
├── variables.tf
├── outputs.tf
└── README.md
```

**Использование:**
```hcl
module "web_server" {
  source = "./modules/webserver"
  
  server_name = "web1"
  port        = 8080
}

# Доступ к outputs
output "config" {
  value = module.web_server.config_file
}

# For_each с модулями
module "servers" {
  source   = "./modules/webserver"
  for_each = var.servers
  
  server_name = each.key
}

# Registry модули
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
}
```

**Специальные переменные:**
```hcl
path.module   # Путь к модулю
path.root     # Корневая директория
path.cwd      # Текущая директория
```

### 💻 Задание

1. Создай модуль `modules/service`
2. Ресурсы: config файл и metadata файл (JSON)
3. Переменные: service_name, port, config_content
4. Outputs: пути и service_info
5. Используй модуль 3 раза
6. Примени и проверь

### 🚀 Бонус

Система модулей с зависимостями (network → application) и templatefile.

---

## Модуль 6: Функции и выражения (25 минут)

### 🎯 Напоминалка

**Строковые:**
```hcl
format("web-%03d", 5)                    # "web-005"
lower("HELLO")                           # "hello"
upper("hello")                           # "HELLO"
split(",", "a,b,c")                      # ["a", "b", "c"]
join(", ", ["a", "b"])                   # "a, b"
replace("hello world", "world", "tf")    # "hello tf"
trim("  hello  ")                        # "hello"
```

**Численные:**
```hcl
abs(-5)                                  # 5
max(5, 12, 9)                           # 12
min(5, 12, 9)                           # 5
ceil(4.3)                               # 5
floor(4.9)                              # 4
```

**Коллекции:**
```hcl
length(["a", "b", "c"])                 # 3
concat(["a"], ["b"])                    # ["a", "b"]
contains(["a", "b"], "a")               # true
distinct(["a", "b", "a"])               # ["a", "b"]
flatten([["a"], ["b"]])                 # ["a", "b"]
keys({a = 1})                           # ["a"]
values({a = 1})                         # [1]
merge({a = 1}, {b = 2})                 # {a = 1, b = 2}
```

**For выражения:**
```hcl
# List comprehension
[for s in var.list : upper(s)]
[for i, v in var.list : "${i}: ${v}"]
[for s in var.list : upper(s) if s != ""]

# Map comprehension
{for s in var.list : s => upper(s)}
{for k, v in var.map : k => upper(v)}

# Группировка
{for s in var.servers : s.type => s.name...}
```

**Условные:**
```hcl
var.env == "prod" ? "t2.large" : "t2.micro"
var.value != null ? var.value : "default"
```

**Типы:**
```hcl
tostring(42)
tonumber("42")
tobool("true")
tolist(["a"])
tomap({a = 1})
toset(["a", "b", "a"])
```

**Сетевые:**
```hcl
cidrhost("10.0.0.0/24", 5)              # "10.0.0.5"
cidrnetmask("10.0.0.0/24")              # "255.255.255.0"
cidrsubnet("10.0.0.0/16", 8, 2)         # "10.0.2.0/24"
```

**Кодирование:**
```hcl
base64encode("hello")
jsonencode({a = 1})
yamlencode({a = 1})
```

**Файлы:**
```hcl
file("${path.module}/config.txt")
fileexists("file.txt")
templatefile("tmpl.tpl", {name = "web"})
```

### 💻 Задание

1. Список серверов в переменной
2. For expression для создания map с портами
3. Locals с uppercase именами и группировкой
4. Templatefile для конфигурации
5. Outputs: formatted список, count, JSON

### 🚀 Бонус

CIDR вычисления с автоматическим созданием subnets для разных типов сетей.

---

## Модуль 7: Работа с облаками (30 минут)

### 🎯 Напоминалка

**AWS:**
```hcl
provider "aws" {
  region = "us-east-1"
  
  default_tags {
    tags = {
      ManagedBy = "Terraform"
    }
  }
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.public[0].id
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/*"]
  }
}
```

**Azure:**
```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "main" {
  name     = "rg-${var.project}"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "vnet-main"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}
```

**GCP:**
```hcl
provider "google" {
  project = var.project_id
  region  = var.region
}

resource "google_compute_network" "vpc" {
  name = "${var.project}-vpc"
}
```

### 💻 Задание

Multi-cloud симуляция с local/random провайдерами:
1. Map переменная с "облаками"
2. For_each для создания ресурсов
3. Random ID для уникальности
4. Файлы конфигурации для каждого "облака"
5. Summary output

### 🚀 Бонус

Если есть AWS Free Tier - создай реальную VPC с subnets, IGW, Security Groups. **Не забудь destroy!**

---

## Модуль 8: Продвинутые техники (30 минут)

### 🎯 Напоминалка

**Provisioners:**
```hcl
resource "aws_instance" "web" {
  provisioner "local-exec" {
    command = "echo ${self.private_ip} >> ips.txt"
  }
  
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Destroyed' >> log.txt"
  }
  
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
    ]
    
    connection {
      type = "ssh"
      user = "ubuntu"
      host = self.public_ip
    }
  }
}
```

**Moved блоки:**
```hcl
moved {
  from = aws_instance.web
  to   = aws_instance.web_server
}

moved {
  from = aws_instance.web[0]
  to   = aws_instance.web["web-1"]
}
```

**Import:**
```hcl
# Terraform 1.5+
import {
  to = aws_instance.existing
  id = "i-1234567890abcdef0"
}
```

**Sensitive данные:**
```hcl
variable "db_password" {
  type      = string
  sensitive = true
}

output "connection_string" {
  value     = "postgresql://user:${var.db_password}@host/db"
  sensitive = true
}

locals {
  db_config = sensitive({
    password = var.db_password
  })
}
```

**Debugging:**
```bash
export TF_LOG=DEBUG
export TF_LOG_PATH=terraform.log

terraform graph | dot -Tsvg > graph.svg
terraform show -json plan.out | jq
```

**Security best practices:**
```hcl
# .gitignore
*.tfstate
*.tfstate.*
*.tfvars
.terraform/

# Remote state с шифрованием
terraform {
  backend "s3" {
    bucket         = "terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    kms_key_id     = "arn:aws:kms:..."
    dynamodb_table = "terraform-locks"
  }
}
```

### 💻 Задание

Production-ready конфигурация:
1. Versions с точными версиями
2. Sensitive переменная
3. Locals для вычислений
4. Moved блок
5. Lifecycle правила
6. Templatefile
7. Documented outputs
8. Правильный .gitignore

### 🚀 Бонус

CI/CD pipeline:
- GitHub Actions (fmt, validate, plan, apply)
- Pre-commit hooks
- Makefile
- README с документацией

---

## Финальный проект (60 минут)

### Задача: Production Infrastructure

**Архитектура:**
```
├── environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
├── modules/
│   ├── networking/
│   ├── compute/
│   └── database/
├── .gitignore
├── Makefile
└── README.md
```

**Требования:**

1. **Модуль Networking:**
   - Файлы конфигурации сети
   - Outputs: network_id, subnet_ids

2. **Модуль Compute:**
   - Файлы серверов
   - Dynamic блоки
   - Outputs: server_ips

3. **Модуль Database:**
   - Random passwords
   - Outputs: db_endpoint (sensitive)

4. **Окружения:**
   - Разные размеры ресурсов
   - Разные количества

5. **Глобально:**
   - Versioning
   - Variables с validation
   - Locals
   - Теги
   - Комментарии

**Шаблон:**

```hcl
# environments/dev/main.tf
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.4"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.5"
    }
  }
}

locals {
  environment = "dev"
  project     = "myapp"
  
  common_tags = {
    Project     = local.project
    Environment = local.environment
    ManagedBy   = "Terraform"
  }
}

module "networking" {
  source = "../../modules/networking"
  
  environment = local.environment
  vpc_cidr    = var.vpc_cidr
  tags        = local.common_tags
}

module "compute" {
  source = "../../modules/compute"
  
  environment    = local.environment
  instance_count = var.instance_count
  network_id     = module.networking.network_id
  tags           = local.common_tags
  
  depends_on = [module.networking]
}

module "database" {
  source = "../../modules/database"
  
  environment = local.environment
  db_name     = var.db_name
  network_id  = module.networking.network_id
  tags        = local.common_tags
}

output "network_info" {
  value = {
    network_id = module.networking.network_id
    subnet_ids = module.networking.subnet_ids
  }
}

output "database_endpoint" {
  value     = module.database.endpoint
  sensitive = true
}
```

**Makefile:**
```makefile
.PHONY: help init plan apply destroy fmt validate

ENV ?= dev

help:
	@echo "Available targets:"
	@echo "  init     - Initialize Terraform"
	@echo "  plan     - Run terraform plan"
	@echo "  apply    - Apply changes"
	@echo "  destroy  - Destroy infrastructure"

init:
	cd environments/$(ENV) && terraform init

plan:
	cd environments/$(ENV) && terraform plan

apply:
	cd environments/$(ENV) && terraform apply

destroy:
	cd environments/$(ENV) && terraform destroy

fmt:
	terraform fmt -recursive

validate:
	cd environments/$(ENV) && terraform validate
```

**.gitignore:**
```
*.tfstate
*.tfstate.*
*.tfvars
.terraform/
crash.log
*.tfplan
```

**README.md:**
```markdown
# MyApp Infrastructure

## Quick Start

```bash
make init ENV=dev
make plan ENV=dev
make apply ENV=dev
```

## Environments

- dev: 2 small instances
- staging: 2 medium instances
- prod: 4 large instances

## Modules

See individual module READMEs.
```

---

## Справочная секция

### Частые ошибки

1. Забыть `terraform init` после изменений
2. State lock при параллельном выполнении
3. Циклические зависимости
4. Изменение count на for_each без moved блоков
5. Не использовать terraform fmt

### Полезные команды

```bash
# Базовые
terraform init
terraform init -upgrade
terraform validate
terraform fmt
terraform plan
terraform apply
terraform destroy

# State
terraform state list
terraform state show aws_instance.web
terraform state mv SOURCE DEST
terraform state rm aws_instance.web
terraform import aws_instance.web i-123

# Другие
terraform output
terraform console
terraform graph
terraform workspace list
terraform providers
```

### Переменные окружения

```bash
export TF_LOG=DEBUG
export TF_LOG_PATH=terraform.log
export TF_VAR_instance_type=t2.micro
export TF_INPUT=0
export TF_PLUGIN_CACHE_DIR=$HOME/.terraform.d/plugin-cache
```

### Production модуль

```hcl
# versions.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 5.0"
    }
  }
}

# variables.tf
variable "name" {
  description = "Resource name"
  type        = string
  
  validation {
    condition     = length(var.name) > 0
    error_message = "Name cannot be empty"
  }
}

# locals.tf
locals {
  name_prefix = "${var.environment}-${var.name}"
  common_tags = merge(var.tags, {
    Module    = "example"
    ManagedBy = "Terraform"
  })
}

# outputs.tf
output "id" {
  description = "Resource ID"
  value       = aws_instance.this.id
}
```

### Registry модули

```hcl
# AWS VPC
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
}

# AWS EKS
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "19.0.0"
  
  cluster_name = "my-cluster"
  vpc_id       = module.vpc.vpc_id
}

# AWS RDS
module "db" {
  source  = "terraform-aws-modules/rds/aws"
  version = "6.0.0"
  
  identifier = "mydb"
  engine     = "postgres"
}
```

### Полезные инструменты

```bash
# Документация
terraform-docs markdown table . > README.md

# Линтеры
tflint --init && tflint

# Security
checkov -d .
tfsec .
terrascan scan

# Cost
infracost breakdown --path .

# Testing
terraform-compliance -f tests/ -p plan.out

# Wrapper
terragrunt plan
```

---

## План повторения

### Первое прохождение (2-3 часа):
- Все модули последовательно
- Все основные задания
- Бонусы по возможности

### Повторное прохождение (через 6 месяцев):
- Фокус на бонусные задания
- Реальные cloud ресурсы
- Улучшение времени
- Новые фичи Terraform

### Для закрепления:
- Перепиши ручные конфигурации в Terraform
- Создай модули для работы
- Изучи Registry модули
- Настрой CI/CD
- Попробуй CDK for Terraform

### Ресурсы:
- docs.terraform.io
- registry.terraform.io
- learn.hashicorp.com/terraform
- github.com/ozbillwang/terraform-best-practices

---

## Чек-лист навыков

После курса ты должен уметь:

- ✅ Писать базовые конфигурации
- ✅ Использовать переменные, locals, outputs
- ✅ Работать с state и backend
- ✅ Создавать модули
- ✅ Использовать count и for_each
- ✅ Применять функции Terraform
- ✅ Управлять версиями провайдеров
- ✅ Работать с remote state
- ✅ Использовать workspaces
- ✅ Импортировать ресурсы
- ✅ Рефакторить с moved блоками
- ✅ Организовывать проекты
- ✅ Настраивать CI/CD

Проходя курс раз в 6-12 месяцев, ты будешь в курсе всех best practices и новых фич! 🚀