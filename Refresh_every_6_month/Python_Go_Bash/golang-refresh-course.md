# Go Refresh: Ежегодный/Полугодовой курс для DevOps/SysAdmin

**Цель:** Освежить в памяти ключевые концепции Go за 3-4 часа практики и узнать 1-2 новые продвинутые техники.

**Формат:** Каждый модуль состоит из:
1. **Краткой теории (Напоминалка)**: Самое главное, что вы могли забыть
2. **Практического задания**: Небольшая программа, которую нужно написать с нуля
3. **Бонусного задания (для роста)**: Задача посложнее или с использованием новой фичи

---

## Модуль 1: Основы и структура программы (20 минут)

### 🎯 Напоминалка

**Базовая структура:**
```go
package main

import (
    "fmt"
    "os"
)

func main() {
    fmt.Println("Hello, World!")
}
```

**Переменные и константы:**
```go
// Объявление
var name string = "John"
var age int = 30
var enabled bool

// Короткая форма (только внутри функций)
name := "John"
age := 30

// Множественное присваивание
x, y := 10, 20

// Константы
const Pi = 3.14
const (
    StatusOK = 200
    StatusNotFound = 404
)
```

**Типы данных:**
```go
int, int8, int16, int32, int64
uint, uint8, uint16, uint32, uint64
float32, float64
string
bool
byte (alias для uint8)
rune (alias для int32, для Unicode)
```

**Работа с аргументами командной строки:**
```go
// os.Args - слайс строк
if len(os.Args) < 2 {
    fmt.Println("Usage: program <arg>")
    os.Exit(1)
}

arg := os.Args[1]
```

**Форматированный вывод:**
```go
fmt.Printf("Имя: %s, Возраст: %d\n", name, age)
fmt.Sprintf("Форматированная строка: %v", value)
```

### 💻 Задание

Напиши программу, которая:
1. Принимает имя пользователя как аргумент командной строки
2. Если аргумент не передан, использует значение по умолчанию "Guest"
3. Объявляет константу с названием компании/сервиса
4. Выводит приветствие: "Добро пожаловать, [Имя]! Сервис: [Название]."
5. Выводит версию программы (используй константу)

### 🚀 Бонус (новое)

Добавь флаги командной строки с помощью пакета `flag`:
- `-name` для имени пользователя
- `-verbose` для детального вывода
- При `-verbose` выводи дополнительную информацию (количество аргументов, время запуска)

**Подсказка:**
```go
import "flag"

name := flag.String("name", "Guest", "User name")
verbose := flag.Bool("verbose", false, "Verbose output")
flag.Parse()
```

---

## Модуль 2: Условия, циклы и функции (30 минут)

### 🎯 Напоминалка

**Условные операторы:**
```go
// Базовый if
if x > 10 {
    fmt.Println("Больше 10")
} else if x > 5 {
    fmt.Println("Больше 5")
} else {
    fmt.Println("5 или меньше")
}

// If с инициализацией
if err := doSomething(); err != nil {
    return err
}

// Switch
switch day {
case "Monday":
    fmt.Println("Понедельник")
case "Tuesday", "Wednesday":
    fmt.Println("Вторник или Среда")
default:
    fmt.Println("Другой день")
}

// Switch без условия (как if-else chain)
switch {
case x < 0:
    fmt.Println("Отрицательное")
case x == 0:
    fmt.Println("Ноль")
default:
    fmt.Println("Положительное")
}
```

**Циклы (только for!):**
```go
// Классический цикл
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

// While-подобный цикл
i := 0
for i < 10 {
    fmt.Println(i)
    i++
}

// Бесконечный цикл
for {
    // break для выхода
    // continue для следующей итерации
}

// Range для массивов/слайсов
numbers := []int{1, 2, 3, 4, 5}
for index, value := range numbers {
    fmt.Printf("Index: %d, Value: %d\n", index, value)
}

// Игнорирование значений
for _, value := range numbers {
    fmt.Println(value)
}
```

**Функции:**
```go
// Простая функция
func greet(name string) {
    fmt.Printf("Hello, %s!\n", name)
}

// Функция с возвратом
func add(a, b int) int {
    return a + b
}

// Множественный возврат
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("деление на ноль")
    }
    return a / b, nil
}

// Именованные возвращаемые значения
func calculate(x int) (result int, err error) {
    if x < 0 {
        err = fmt.Errorf("отрицательное число")
        return
    }
    result = x * 2
    return
}

// Вариадические функции
func sum(numbers ...int) int {
    total := 0
    for _, n := range numbers {
        total += n
    }
    return total
}
```

### 💻 Задание

Напиши программу, которая:
1. Принимает число N как аргумент командной строки
2. Конвертирует строку в число (используй `strconv.Atoi`)
3. Создает функцию `isPrime(n int) bool`, проверяющую, является ли число простым
4. Находит и выводит все простые числа от 2 до N

### 🚀 Бонус (новое)

Добавь функцию `primeFactors(n int) []int`, которая возвращает разложение числа на простые множители. Например, для 12 должна вернуть `[2, 2, 3]`.

---

## Модуль 3: Слайсы, массивы и мапы (30 минут)

### 🎯 Напоминалка

**Массивы (фиксированный размер):**
```go
var arr [5]int              // Массив из 5 нулей
arr := [5]int{1, 2, 3, 4, 5}
arr := [...]int{1, 2, 3}    // Размер вычисляется автоматически
```

**Слайсы (динамический размер):**
```go
// Создание
slice := []int{1, 2, 3}
slice := make([]int, 5)      // Длина 5, нули
slice := make([]int, 5, 10)  // Длина 5, capacity 10

// Добавление
slice = append(slice, 4)
slice = append(slice, 5, 6, 7)

// Срезы
sub := slice[1:4]   // Элементы с индекса 1 до 3
sub := slice[:3]    // Первые 3 элемента
sub := slice[2:]    // С 2-го до конца

// Длина и capacity
len(slice)
cap(slice)

// Копирование
newSlice := make([]int, len(slice))
copy(newSlice, slice)
```

**Мапы (словари):**
```go
// Создание
m := make(map[string]int)
m := map[string]int{
    "apple": 5,
    "banana": 3,
}

// Операции
m["orange"] = 7          // Добавление
value := m["apple"]      // Получение
delete(m, "banana")      // Удаление

// Проверка существования
value, exists := m["key"]
if exists {
    fmt.Println("Найдено:", value)
}

// Итерация
for key, value := range m {
    fmt.Printf("%s: %d\n", key, value)
}
```

**Строки:**
```go
import "strings"

strings.Contains(s, substr)
strings.HasPrefix(s, prefix)
strings.HasSuffix(s, suffix)
strings.Split(s, sep)
strings.Join(slice, sep)
strings.ToLower(s)
strings.ToUpper(s)
strings.TrimSpace(s)
```

### 💻 Задание

Напиши программу "Анализатор логов", которая:
1. Создает слайс строк с "логами" (можно захардкодить):
   ```
   "2024-11-09 10:00:00 INFO Server started"
   "2024-11-09 10:05:15 ERROR Connection failed"
   "2024-11-09 10:10:30 INFO Request processed"
   "2024-11-09 10:15:45 ERROR Database timeout"
   "2024-11-09 10:20:00 WARNING High memory usage"
   ```
2. Подсчитывает количество каждого уровня лога (INFO, ERROR, WARNING) используя мапу
3. Выводит статистику: сколько логов каждого типа

### 🚀 Бонус (новое)

Расширь программу:
- Прочитай логи из файла (используй `os.ReadFile` или `bufio.Scanner`)
- Извлеки и выведи только ERROR логи с временными метками
- Найди самый частый тип ошибки (если есть несколько ERROR логов с разными сообщениями)

---

## Модуль 4: Работа с файлами и ошибками (30 минут)

### 🎯 Напоминалка

**Обработка ошибок:**
```go
// Базовая проверка
result, err := doSomething()
if err != nil {
    return err
}

// Создание ошибок
import "errors"
err := errors.New("что-то пошло не так")
err := fmt.Errorf("ошибка: %v", someValue)

// Обертывание ошибок (Go 1.13+)
if err != nil {
    return fmt.Errorf("не удалось выполнить операцию: %w", err)
}

// Проверка типа ошибки
if errors.Is(err, os.ErrNotExist) {
    // Файл не существует
}

// Panic и recover (использовать осторожно!)
defer func() {
    if r := recover(); r != nil {
        fmt.Println("Recovered from:", r)
    }
}()
```

**Работа с файлами:**
```go
import (
    "io"
    "os"
    "bufio"
)

// Чтение всего файла
data, err := os.ReadFile("file.txt")
if err != nil {
    return err
}
content := string(data)

// Запись в файл
err := os.WriteFile("file.txt", []byte(content), 0644)

// Построчное чтение
file, err := os.Open("file.txt")
if err != nil {
    return err
}
defer file.Close()

scanner := bufio.NewScanner(file)
for scanner.Scan() {
    line := scanner.Text()
    fmt.Println(line)
}
if err := scanner.Err(); err != nil {
    return err
}

// Создание/запись
file, err := os.Create("output.txt")
if err != nil {
    return err
}
defer file.Close()

writer := bufio.NewWriter(file)
fmt.Fprintln(writer, "Hello, World!")
writer.Flush()

// Проверка существования
if _, err := os.Stat("file.txt"); os.IsNotExist(err) {
    fmt.Println("Файл не существует")
}

// Создание директории
os.Mkdir("mydir", 0755)
os.MkdirAll("path/to/nested/dir", 0755)

// Список файлов
entries, err := os.ReadDir(".")
for _, entry := range entries {
    fmt.Println(entry.Name(), entry.IsDir())
}
```

**Работа с путями:**
```go
import "path/filepath"

filepath.Join("path", "to", "file.txt")
filepath.Base("/path/to/file.txt")  // "file.txt"
filepath.Dir("/path/to/file.txt")   // "/path/to"
filepath.Ext("file.txt")            // ".txt"
filepath.Abs("relative/path")
```

### 💻 Задание

Напиши программу "Конфигурационный менеджер", которая:
1. Проверяет существование файла `config.txt` в текущей директории
2. Если файл не существует, создает его с дефолтными настройками:
   ```
   host=localhost
   port=8080
   debug=false
   ```
3. Если файл существует, читает его и парсит настройки в map[string]string
4. Выводит все настройки в формате "Ключ: Значение"
5. Обрабатывает все возможные ошибки с понятными сообщениями

### 🚀 Бонус (новое)

Расширь функционал:
- Добавь функцию для обновления конфигурации: принимает ключ и значение, обновляет файл
- Используй формат JSON вместо простого текста (пакет `encoding/json`)
- Добавь валидацию: порт должен быть числом от 1 до 65535

**Подсказка для JSON:**
```go
type Config struct {
    Host  string `json:"host"`
    Port  int    `json:"port"`
    Debug bool   `json:"debug"`
}

// Маршаллинг
data, err := json.MarshalIndent(config, "", "  ")

// Анмаршаллинг
var config Config
err := json.Unmarshal(data, &config)
```

---

## Модуль 5: Структуры и методы (30 минут)

### 🎯 Напоминалка

**Структуры:**
```go
// Определение
type Person struct {
    Name string
    Age  int
    Email string
}

// Создание
p1 := Person{Name: "John", Age: 30, Email: "john@example.com"}
p2 := Person{"Jane", 25, "jane@example.com"}  // По порядку полей
p3 := &Person{Name: "Bob"}  // Остальные поля - нулевые значения

// Доступ
fmt.Println(p1.Name)
p1.Age = 31

// Анонимные структуры
config := struct {
    Host string
    Port int
}{
    Host: "localhost",
    Port: 8080,
}
```

**Методы:**
```go
// Метод с value receiver
func (p Person) Greet() string {
    return fmt.Sprintf("Hello, I'm %s", p.Name)
}

// Метод с pointer receiver (может изменять структуру)
func (p *Person) HaveBirthday() {
    p.Age++
}

// Использование
person := Person{Name: "John", Age: 30}
fmt.Println(person.Greet())
person.HaveBirthday()
```

**Встраивание (Embedding):**
```go
type Address struct {
    City    string
    Country string
}

type Employee struct {
    Person          // Встроенная структура
    Address         // Еще одна встроенная структура
    Position string
    Salary   float64
}

emp := Employee{
    Person:   Person{Name: "John", Age: 30},
    Address:  Address{City: "Moscow", Country: "Russia"},
    Position: "DevOps",
    Salary:   100000,
}

// Прямой доступ к полям встроенных структур
fmt.Println(emp.Name)     // Из Person
fmt.Println(emp.City)     // Из Address
```

**Интерфейсы:**
```go
// Определение
type Writer interface {
    Write(data string) error
}

// Реализация (неявная!)
type FileWriter struct {
    filename string
}

func (fw FileWriter) Write(data string) error {
    return os.WriteFile(fw.filename, []byte(data), 0644)
}

// Использование
var w Writer = FileWriter{filename: "output.txt"}
w.Write("Hello")

// Пустой интерфейс (любой тип)
func PrintAnything(v interface{}) {
    fmt.Println(v)
}

// Type assertion
if str, ok := v.(string); ok {
    fmt.Println("Это строка:", str)
}

// Type switch
switch v := v.(type) {
case string:
    fmt.Println("Строка:", v)
case int:
    fmt.Println("Число:", v)
default:
    fmt.Println("Неизвестный тип")
}
```

### 💻 Задание

Создай систему мониторинга серверов:

1. Определи структуру `Server`:
   ```go
   type Server struct {
       Name      string
       IP        string
       Status    string  // "up", "down", "unknown"
       LastCheck time.Time
   }
   ```

2. Реализуй методы:
   - `Check() error` - "проверяет" сервер (можно просто рандомно устанавливать статус)
   - `String() string` - возвращает строковое представление сервера

3. Создай структуру `ServerPool`:
   ```go
   type ServerPool struct {
       Servers []Server
   }
   ```

4. Реализуй методы для `ServerPool`:
   - `Add(server Server)` - добавить сервер
   - `CheckAll()` - проверить все серверы
   - `GetHealthy() []Server` - вернуть только работающие серверы
   - `Report()` - вывести статистику (сколько up/down/unknown)

### 🚀 Бонус (новое)

Добавь интерфейс и несколько его реализаций:
```go
type HealthChecker interface {
    Check(server Server) error
}

type PingChecker struct {}
type HTTPChecker struct {
    Endpoint string
}
```

Модифицируй `Server`, чтобы он использовал `HealthChecker` для проверки состояния. Это позволит разным серверам использовать разные методы проверки.

---

## Модуль 6: Конкурентность и горутины (45 минут)

### 🎯 Напоминалка

**Горутины:**
```go
// Запуск горутины
go func() {
    fmt.Println("Выполняется в горутине")
}()

// С параметрами
go processData(data)

// Анонимная функция с замыканием
for i := 0; i < 5; i++ {
    go func(n int) {
        fmt.Println(n)
    }(i)  // Важно передать i как параметр!
}
```

**Каналы:**
```go
// Создание
ch := make(chan int)          // Небуферизованный
ch := make(chan int, 5)       // Буфер на 5 элементов

// Отправка и получение
ch <- 42          // Отправка
value := <-ch     // Получение

// Закрытие канала
close(ch)

// Проверка закрытия
value, ok := <-ch
if !ok {
    fmt.Println("Канал закрыт")
}

// Итерация по каналу
for value := range ch {
    fmt.Println(value)
}

// Однонаправленные каналы (в параметрах функций)
func send(ch chan<- int) {    // Только для отправки
    ch <- 42
}

func receive(ch <-chan int) { // Только для получения
    value := <-ch
}
```

**Select:**
```go
select {
case msg := <-ch1:
    fmt.Println("Получено из ch1:", msg)
case msg := <-ch2:
    fmt.Println("Получено из ch2:", msg)
case ch3 <- 42:
    fmt.Println("Отправлено в ch3")
case <-time.After(time.Second):
    fmt.Println("Таймаут")
default:
    fmt.Println("Нет доступных операций")
}
```

**WaitGroup:**
```go
import "sync"

var wg sync.WaitGroup

for i := 0; i < 5; i++ {
    wg.Add(1)
    go func(n int) {
        defer wg.Done()
        // Работа
        fmt.Println(n)
    }(i)
}

wg.Wait()  // Ждем завершения всех горутин
```

**Mutex (для защиты данных):**
```go
import "sync"

type SafeCounter struct {
    mu    sync.Mutex
    count int
}

func (c *SafeCounter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

func (c *SafeCounter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}
```

**Контекст:**
```go
import "context"

// Создание
ctx := context.Background()
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
defer cancel()

ctx, cancel := context.WithCancel(ctx)

// Использование
select {
case <-ctx.Done():
    return ctx.Err()
case result := <-ch:
    return result
}
```

### 💻 Задание

Создай параллельный сканер портов:

1. Напиши функцию `scanPort(host string, port int, timeout time.Duration) bool`, которая проверяет, открыт ли порт (используй `net.DialTimeout`)

2. Создай функцию `scanPorts(host string, ports []int)`, которая:
   - Сканирует все порты параллельно (каждый в отдельной горутине)
   - Использует канал для сбора результатов
   - Использует WaitGroup для ожидания завершения всех горутин
   - Выводит список открытых портов

3. Протестируй на localhost с портами [22, 80, 443, 3000, 5432, 8080]

**Подсказка:**
```go
import "net"

conn, err := net.DialTimeout("tcp", 
    fmt.Sprintf("%s:%d", host, port), 
    timeout)
if err != nil {
    return false
}
conn.Close()
return true
```

### 🚀 Бонус (новое)

Улучши сканер:
- Добавь ограничение на количество одновременных подключений (используй буферизованный канал как семафор)
- Добавь таймаут для всей операции сканирования (используй context.WithTimeout)
- Добавь прогресс-бар или счетчик просканированных портов
- Сделай диапазон портов: `scanRange(host string, startPort, endPort int)`

**Подсказка для семафора:**
```go
sem := make(chan struct{}, maxConcurrent)

for _, port := range ports {
    sem <- struct{}{}  // Захватить слот
    go func(p int) {
        defer func() { <-sem }()  // Освободить слот
        // Сканирование
    }(port)
}
```

---

## Модуль 7: Работа с JSON и HTTP (30 минут)

### 🎯 Напоминалка

**JSON:**
```go
import "encoding/json"

// Структура для маршаллинга
type User struct {
    Name  string `json:"name"`
    Email string `json:"email"`
    Age   int    `json:"age,omitempty"`  // Опустить если 0
    Token string `json:"-"`              // Игнорировать
}

// Маршаллинг (Go -> JSON)
user := User{Name: "John", Email: "john@example.com", Age: 30}
data, err := json.Marshal(user)
// Красивое форматирование
data, err := json.MarshalIndent(user, "", "  ")

// Анмаршаллинг (JSON -> Go)
var user User
err := json.Unmarshal(data, &user)

// Работа с map
var result map[string]interface{}
err := json.Unmarshal(data, &result)
```

**HTTP Client:**
```go
import "net/http"

// GET запрос
resp, err := http.Get("https://api.example.com/data")
if err != nil {
    return err
}
defer resp.Body.Close()

body, err := io.ReadAll(resp.Body)

// POST запрос
data := []byte(`{"name":"John"}`)
resp, err := http.Post(
    "https://api.example.com/users",
    "application/json",
    bytes.NewBuffer(data),
)

// Более гибкий способ
client := &http.Client{
    Timeout: 10 * time.Second,
}

req, err := http.NewRequest("GET", url, nil)
req.Header.Add("Authorization", "Bearer token")

resp, err := client.Do(req)
```

**HTTP Server:**
```go
import "net/http"

// Простой обработчик
func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, World!")
}

// Главная функция
func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}

// JSON ответ
func jsonHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]string{
        "status": "ok",
    })
}

// Чтение JSON из запроса
func createUser(w http.ResponseWriter, r *http.Request) {
    var user User
    if err := json.NewDecoder(r.Body).Decode(&user); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    // Обработка user...
}
```

### 💻 Задание

Создай HTTP API для управления задачами (TODO list):

1. Определи структуру:
```go
type Task struct {
    ID          int       `json:"id"`
    Title       string    `json:"title"`
    Description string    `json:"description"`
    Completed   bool      `json:"completed"`
    CreatedAt   time.Time `json:"created_at"`
}
```

2. Реализуй REST API endpoints:
   - `GET /tasks` - список всех задач (вернуть JSON)
   - `POST /tasks` - создать задачу (принять JSON)
   - `GET /tasks/{id}` - получить задачу по ID
   - `DELETE /tasks/{id}` - удалить задачу

3. Используй глобальный слайс для хранения задач (или map)

4. Запусти сервер на порту 8080

### 🚀 Бонус (новое)

Улучши API:
- Добавь middleware для логирования всех запросов
- Добавь обработку ошибок и правильные HTTP статусы
- Создай клиентскую программу, которая делает запросы к твоему API
- Добавь поддержку фильтрации: `GET /tasks?completed=true`

**Подсказка для middleware:**
```go
func loggingMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        next(w, r)
        log.Printf("%s %s %v", r.Method, r.URL.Path, time.Since(start))
    }
}

http.HandleFunc("/tasks", loggingMiddleware(getTasks))
```

---

## Модуль 8: Работа с внешними командами и системой (20 минут)

### 🎯 Напоминалка

**Выполнение команд:**
```go
import "os/exec"

// Простое выполнение
cmd := exec.Command("ls", "-la")
output, err := cmd.Output()
if err != nil {
    return err
}
fmt.Println(string(output))

// С комбинированным выводом (stdout + stderr)
output, err := cmd.CombinedOutput()

// Получить stdout и stderr отдельно
cmd := exec.Command("program")
stdout, err := cmd.StdoutPipe()
stderr, err := cmd.StderrPipe()

if err := cmd.Start(); err != nil {
    return err
}

// Чтение из stdout
scanner := bufio.NewScanner(stdout)
for scanner.Scan() {
    fmt.Println(scanner.Text())
}

cmd.Wait()

// Проверка кода возврата
if err := cmd.Run(); err != nil {
    if exitErr, ok := err.(*exec.ExitError); ok {
        exitCode := exitErr.ExitCode()
        fmt.Printf("Exit code: %d\n", exitCode)
    }
}

// С контекстом и таймаутом
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

cmd := exec.CommandContext(ctx, "long-running-command")
err := cmd.Run()
if ctx.Err() == context.DeadlineExceeded {
    fmt.Println("Command timed out")
}
```

**Переменные окружения:**
```go
import "os"

// Получить
value := os.Getenv("HOME")
value, exists := os.LookupEnv("PATH")

// Установить
os.Setenv("MY_VAR", "value")

// Все переменные
for _, env := range os.Environ() {
    fmt.Println(env)
}

// Для команды
cmd := exec.Command("program")
cmd.Env = append(os.Environ(),
    "CUSTOM_VAR=value",
)
```

**Работа с процессами:**
```go
// Текущий процесс
pid := os.Getpid()
os.Exit(0)

// Сигналы
import "os/signal"

sigChan := make(chan os.Signal, 1)
signal.Notify(sigChan, os.Interrupt, syscall.SIGTERM)

go func() {
    sig := <-sigChan
    fmt.Printf("Received signal: %v\n", sig)
    // Cleanup
    os.Exit(0)
}()
```

### 💻 Задание

Создай программу "System Monitor", которая:

1. Выполняет системные команды и собирает информацию:
   - `uptime` - время работы системы
   - `df -h` - использование дисков
   - `free -h` или `vm_stat` (для macOS) - использование памяти

2. Парсит вывод команд и извлекает полезную информацию

3. Выводит отчет в читаемом формате

4. Обрабатывает таймауты (команды не должны выполняться больше 5 секунд)

### 🚀 Бонус (новое)

Расширь функционал:
- Добавь мониторинг CPU usage (команда `top` или `ps`)
- Добавь проверку работающих Docker контейнеров (`docker ps`)
- Сохрани отчет в JSON файл с временной меткой
- Добавь graceful shutdown по Ctrl+C (используй сигналы)

---

## Модуль 9: Тестирование (30 минут)

### 🎯 Напоминалка

**Базовые тесты:**
```go
// calculator.go
package calculator

func Add(a, b int) int {
    return a + b
}

// calculator_test.go
package calculator

import "testing"

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    expected := 5
    
    if result != expected {
        t.Errorf("Add(2, 3) = %d; want %d", result, expected)
    }
}

// Табличные тесты (table-driven tests)
func TestAddTable(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive numbers", 2, 3, 5},
        {"negative numbers", -2, -3, -5},
        {"mixed", -2, 5, 3},
        {"zeros", 0, 0, 0},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("got %d, want %d", result, tt.expected)
            }
        })
    }
}
```

**Бенчмарки:**
```go
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(2, 3)
    }
}

// Запуск: go test -bench=.
```

**Тестирование с подготовкой:**
```go
func TestMain(m *testing.M) {
    // Setup
    setup()
    
    // Запуск тестов
    code := m.Run()
    
    // Teardown
    teardown()
    
    os.Exit(code)
}

func setup() {
    // Инициализация БД, файлов и т.д.
}

func teardown() {
    // Очистка
}
```

**Моки и интерфейсы:**
```go
// Интерфейс для мокирования
type DataStore interface {
    Get(id string) (string, error)
}

// Реальная реализация
type RealDataStore struct {}

func (r RealDataStore) Get(id string) (string, error) {
    // Реальная логика
}

// Мок для тестов
type MockDataStore struct {
    GetFunc func(id string) (string, error)
}

func (m MockDataStore) Get(id string) (string, error) {
    return m.GetFunc(id)
}

// Тест
func TestWithMock(t *testing.T) {
    mock := MockDataStore{
        GetFunc: func(id string) (string, error) {
            return "test-data", nil
        },
    }
    
    // Используем мок
    result, err := mock.Get("123")
    if err != nil {
        t.Fatal(err)
    }
    // Проверки...
}
```

**Тестирование HTTP:**
```go
import "net/http/httptest"

func TestHTTPHandler(t *testing.T) {
    req := httptest.NewRequest("GET", "/users", nil)
    w := httptest.NewRecorder()
    
    handler(w, req)
    
    if w.Code != http.StatusOK {
        t.Errorf("got %d, want %d", w.Code, http.StatusOK)
    }
    
    // Проверка body
    body := w.Body.String()
    if !strings.Contains(body, "expected") {
        t.Error("response doesn't contain expected text")
    }
}
```

### 💻 Задание

Создай утилиту для валидации email и напиши тесты:

1. Реализуй функцию `IsValidEmail(email string) bool`

2. Напиши табличные тесты с разными кейсами:
   - Валидные email: "user@example.com", "test.user@domain.co.uk"
   - Невалидные: "invalid", "@example.com", "user@", "user @example.com"

3. Создай функцию `ExtractDomain(email string) (string, error)`

4. Напиши тесты для неё

5. Запусти тесты с покрытием: `go test -cover`

### 🚀 Бонус (новое)

Добавь HTTP API для валидации и протестируй его:
```go
POST /validate
{
  "email": "user@example.com"
}

Response:
{
  "valid": true,
  "domain": "example.com"
}
```

Используй `httptest` для тестирования endpoints.

---

## Финальный проект (45-60 минут)

### Задача: CLI утилита для мониторинга и бэкапа

Создай полноценную CLI программу, которая объединяет все изученные концепции.

### Требования:

**1. Функциональность:**
   - Мониторинг системных ресурсов (CPU, память, диск)
   - Создание бэкапов директорий (tar.gz архивы)
   - HTTP API для получения статуса и управления бэкапами
   - Логирование всех операций
   - Конфигурация через JSON файл

**2. CLI команды:**
```bash
# Мониторинг
./app monitor --interval=5s

# Создание бэкапа
./app backup --source=/path/to/dir --dest=/backups

# Запуск HTTP сервера
./app serve --port=8080

# Очистка старых бэкапов
./app cleanup --days=30
```

**3. HTTP API endpoints:**
   - `GET /health` - статус приложения
   - `GET /metrics` - системные метрики
   - `POST /backup` - запустить бэкап
   - `GET /backups` - список бэкапов
   - `DELETE /backups/{id}` - удалить бэкап

**4. Конфигурация (config.json):**
```json
{
  "monitor": {
    "interval": "10s",
    "alert_thresholds": {
      "cpu": 80,
      "memory": 90,
      "disk": 85
    }
  },
  "backup": {
    "source_dirs": ["/etc", "/var/log"],
    "dest_dir": "/backups",
    "retention_days": 30,
    "compression": true
  },
  "server": {
    "port": 8080,
    "log_level": "info"
  }
}
```

**5. Структура проекта:**
```
project/
├── main.go
├── cmd/
│   ├── monitor.go
│   ├── backup.go
│   ├── serve.go
│   └── cleanup.go
├── internal/
│   ├── config/
│   │   └── config.go
│   ├── monitor/
│   │   ├── monitor.go
│   │   └── monitor_test.go
│   ├── backup/
│   │   ├── backup.go
│   │   └── backup_test.go
│   └── server/
│       ├── handlers.go
│       └── handlers_test.go
├── pkg/
│   └── utils/
│       └── utils.go
└── config.json
```

**6. Технические требования:**
   - Используй `cobra` или `flag` для CLI
   - Параллельный мониторинг метрик (горутины)
   - Context для graceful shutdown
   - Логирование в файл и stdout
   - Обработка всех ошибок
   - Минимум 60% покрытие тестами

### Пример структур:

```go
type Config struct {
    Monitor MonitorConfig `json:"monitor"`
    Backup  BackupConfig  `json:"backup"`
    Server  ServerConfig  `json:"server"`
}

type MonitorConfig struct {
    Interval        string           `json:"interval"`
    AlertThresholds AlertThresholds  `json:"alert_thresholds"`
}

type Metrics struct {
    CPU     float64   `json:"cpu"`
    Memory  float64   `json:"memory"`
    Disk    float64   `json:"disk"`
    Uptime  int64     `json:"uptime"`
    Timestamp time.Time `json:"timestamp"`
}

type BackupInfo struct {
    ID        string    `json:"id"`
    Source    string    `json:"source"`
    Dest      string    `json:"dest"`
    Size      int64     `json:"size"`
    CreatedAt time.Time `json:"created_at"`
    Status    string    `json:"status"`
}
```

### 🚀 Бонусные фичи:

1. **Уведомления:**
   - Отправка алертов по email или Slack webhook при превышении порогов
   - Используй goroutine для асинхронной отправки

2. **База данных:**
   - Сохрани историю метрик в SQLite (`database/sql`)
   - Endpoint для получения исторических данных

3. **Dashboard:**
   - Простой HTML dashboard для визуализации метрик
   - WebSocket для real-time обновлений

4. **Распределенность:**
   - Поддержка мониторинга нескольких серверов
   - Центральный сервер собирает данные с агентов

---

## Справочная секция: Быстрые шпаргалки

### Частые ошибки

1. **Забыть обработать ошибку:**
   ```go
   // ❌ Плохо
   result, _ := doSomething()
   
   // ✅ Хорошо
   result, err := doSomething()
   if err != nil {
       return fmt.Errorf("не удалось выполнить операцию: %w", err)
   }
   ```

2. **Не закрыть ресурсы:**
   ```go
   // ✅ Всегда используй defer
   file, err := os.Open("file.txt")
   if err != nil {
       return err
   }
   defer file.Close()
   ```

3. **Гонка данных (data race):**
   ```go
   // ❌ Плохо
   var counter int
   for i := 0; i < 100; i++ {
       go func() { counter++ }()
   }
   
   // ✅ Хорошо
   var mu sync.Mutex
   var counter int
   for i := 0; i < 100; i++ {
       go func() {
           mu.Lock()
           counter++
           mu.Unlock()
       }()
   }
   ```

4. **Замыкание в цикле:**
   ```go
   // ❌ Плохо
   for i := 0; i < 5; i++ {
       go func() {
           fmt.Println(i) // Все выведут 5
       }()
   }
   
   // ✅ Хорошо
   for i := 0; i < 5; i++ {
       go func(n int) {
           fmt.Println(n)
       }(i)
   }
   ```

5. **Nil pointer dereference:**
   ```go
   // ✅ Всегда проверяй на nil
   if obj != nil {
       obj.DoSomething()
   }
   ```

### Полезные пакеты стандартной библиотеки

```go
// Работа с файлами и I/O
os          // Операции с ОС
io          // Базовые интерфейсы I/O
bufio       // Буферизованный I/O
path/filepath // Работа с путями

// Работа со строками и текстом
strings     // Операции со строками
strconv     // Конвертация строк
regexp      // Регулярные выражения
text/template // Шаблоны

// Форматы данных
encoding/json
encoding/xml
encoding/csv
encoding/base64

// Сеть
net         // Сетевые операции
net/http    // HTTP клиент и сервер
net/url     // Парсинг URL

// Конкурентность
sync        // Примитивы синхронизации
context     // Контексты

// Время
time        // Работа со временем

// Системные вызовы
os/exec     // Выполнение команд
os/signal   // Обработка сигналов
syscall     // Системные вызовы

// Тестирование
testing     // Фреймворк тестирования

// Криптография
crypto/md5
crypto/sha256
crypto/rand

// Логирование
log         // Простое логирование
log/slog    // Структурированное логирование (Go 1.21+)

// Флаги
flag        // Парсинг флагов командной строки
```

### Популярные внешние библиотеки

```go
// CLI
github.com/spf13/cobra      // Мощный CLI фреймворк
github.com/urfave/cli       // Альтернатива для CLI

// Конфигурация
github.com/spf13/viper      // Конфигурация

// Логирование
github.com/sirupsen/logrus  // Структурированное логирование
go.uber.org/zap             // Быстрое логирование

// Веб-фреймворки
github.com/gin-gonic/gin    // Веб-фреймворк
github.com/gorilla/mux      // HTTP роутер

// Базы данных
github.com/lib/pq           // PostgreSQL driver
github.com/go-sql-driver/mysql // MySQL driver
gorm.io/gorm                // ORM

// Тестирование
github.com/stretchr/testify // Тестовые утилиты
github.com/golang/mock      // Мокирование

// HTTP
github.com/go-resty/resty   // HTTP клиент

// Конкурентность
golang.org/x/sync/errgroup  // Группы с ошибками
```

### Инструменты разработки

```bash
# Форматирование
go fmt ./...
gofmt -w .

# Линтеры
go vet ./...
golangci-lint run

# Тестирование
go test ./...
go test -v ./...
go test -cover ./...
go test -race ./...
go test -bench=. ./...

# Сборка
go build
go build -o myapp
go install

# Зависимости
go mod init module-name
go mod tidy
go mod vendor
go get package@version

# Документация
go doc package
godoc -http=:6060

# Профилирование
go tool pprof

# Проверка гонок
go build -race
go test -race

# Покрытие
go test -coverprofile=coverage.out
go tool cover -html=coverage.out
```

### Паттерны для DevOps/SysAdmin

**1. Retry с экспоненциальной задержкой:**
```go
func retryWithBackoff(fn func() error, maxRetries int) error {
    for i := 0; i < maxRetries; i++ {
        if err := fn(); err == nil {
            return nil
        }
        time.Sleep(time.Duration(math.Pow(2, float64(i))) * time.Second)
    }
    return fmt.Errorf("превышено количество попыток")
}
```

**2. Graceful shutdown:**
```go
func main() {
    srv := &http.Server{Addr: ":8080"}
    
    go func() {
        if err := srv.ListenAndServe(); err != http.ErrServerClosed {
            log.Fatal(err)
        }
    }()
    
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, os.Interrupt, syscall.SIGTERM)
    <-quit
    
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    if err := srv.Shutdown(ctx); err != nil {
        log.Fatal(err)
    }
}
```

**3. Worker pool:**
```go
func workerPool(jobs <-chan Job, results chan<- Result, workers int) {
    var wg sync.WaitGroup
    
    for i := 0; i < workers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for job := range jobs {
                results <- processJob(job)
            }
        }()
    }
    
    wg.Wait()
    close(results)
}
```

**4. Rate limiting:**
```go
import "golang.org/x/time/rate"

limiter := rate.NewLimiter(rate.Every(time.Second), 10) // 10 req/sec

for {
    if err := limiter.Wait(ctx); err != nil {
        return err
    }
    // Выполнить запрос
}
```

---

## Советы по прохождению курса

1. **Не подглядывай!** Сначала попробуй написать код сам, вспомнив синтаксис. Только потом обращайся к шпаргалкам.

2. **Используй go fmt и go vet.** Привыкай к автоформатированию и проверке кода.

3. **Тестируй в изолированной среде.** Используй Docker контейнер или VM для экспериментов.

4. **Читай документацию.** `go doc` и https://pkg.go.dev - лучшие источники информации.

5. **Пиши тесты.** Даже для простых заданий пиши хотя бы базовые тесты.

6. **Проверяй на race conditions.** Всегда запускай `go test -race` для кода с горутинами.

7. **Изучай чужой код.** Читай популярные Go проекты на GitHub (Docker, Kubernetes, Prometheus).

---

## План повторения

### При первом прохождении (3-4 часа):
- Пройди модули 1-5 обязательно
- Модули 6-9 по возможности
- Выполни хотя бы базовые задания
- Финальный проект начни, но не обязательно заканчивать

### При повторном прохождении (через 6-12 месяцев):
- Фокус на бонусные задания
- Засеки время и улучши результат
- Добавь свои усложнения
- Обязательно доделай финальный проект
- Сравни с предыдущей реализацией

### Для закрепления:
- Автоматизируй рутинные DevOps задачи на Go
- Напиши CLI утилиту для работы
- Создай микросервис для мониторинга
- Изучи код Terraform, kubectl, docker-compose

### Что отслеживать при повторных прохождениях:
- ✅ Забыл ли синтаксис каналов и select?
- ✅ Помню ли правила работы с ошибками?
- ✅ Могу ли быстро написать HTTP сервер?
- ✅ Уверенно ли работаю с JSON?
- ✅ Знаю ли как правильно тестировать?

---

## Чеклист для DevOps/SysAdmin на Go

После прохождения курса ты должен уметь:

- [ ] Написать CLI утилиту с флагами
- [ ] Выполнять системные команды и обрабатывать их вывод
- [ ] Работать с файлами, директориями, путями
- [ ] Парсить JSON, YAML, конфиги
- [ ] Создать HTTP API для автоматизации
- [ ] Делать HTTP запросы к внешним API
- [ ] Использовать горутины для параллельных операций
- [ ] Правильно обрабатывать ошибки
- [ ] Писать тесты для своего кода
- [ ] Работать с контекстом и таймаутами
- [ ] Делать graceful shutdown
- [ ] Логировать операции
- [ ] Читать конфигурацию из файлов
- [ ] Работать с переменными окружения

**Проходи этот курс каждые 6-12 месяцев, и Go станет твоим надежным инструментом для DevOps задач!** 🚀