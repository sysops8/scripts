# 50 Go Scripts для DevOps инженера

## 🔧 Системное администрирование

### 1. Проверка использования диска
```go
package main

import (
    "fmt"
    "syscall"
)

func checkDiskUsage(path string) {
    var stat syscall.Statfs_t
    syscall.Statfs(path, &stat)
    
    total := stat.Blocks * uint64(stat.Bsize)
    free := stat.Bfree * uint64(stat.Bsize)
    used := total - free
    percent := float64(used) / float64(total) * 100
    
    fmt.Printf("Диск: %.2f%% использовано\n", percent)
    if percent > 80 {
        fmt.Println("⚠️ Предупреждение: мало места!")
    }
}

func main() {
    checkDiskUsage("/")
}
```

### 2. Мониторинг CPU и RAM
```go
package main

import (
    "fmt"
    "runtime"
    "time"
)

func systemHealth() {
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    
    fmt.Printf("Alloc: %v MB\n", m.Alloc/1024/1024)
    fmt.Printf("TotalAlloc: %v MB\n", m.TotalAlloc/1024/1024)
    fmt.Printf("Sys: %v MB\n", m.Sys/1024/1024)
    fmt.Printf("NumGC: %v\n", m.NumGC)
    fmt.Printf("Goroutines: %d\n", runtime.NumGoroutine())
}

func main() {
    for {
        systemHealth()
        time.Sleep(5 * time.Second)
    }
}
```

### 3. Проверка работающих процессов
```go
package main

import (
    "fmt"
    "os/exec"
    "strings"
)

func checkProcess(processName string) bool {
    cmd := exec.Command("pgrep", "-f", processName)
    output, err := cmd.Output()
    
    if err == nil && len(output) > 0 {
        fmt.Printf("✓ %s запущен\n", processName)
        return true
    }
    fmt.Printf("✗ %s не найден\n", processName)
    return false
}

func main() {
    checkProcess("nginx")
    checkProcess("docker")
}
```

### 4. Автоматический restart сервиса
```go
package main

import (
    "fmt"
    "os/exec"
)

func restartService(serviceName string) error {
    cmd := exec.Command("systemctl", "restart", serviceName)
    err := cmd.Run()
    
    if err != nil {
        fmt.Printf("✗ Ошибка перезапуска %s: %v\n", serviceName, err)
        return err
    }
    fmt.Printf("✓ %s перезапущен\n", serviceName)
    return nil
}

func main() {
    restartService("nginx")
}
```

### 5. Проверка открытых портов
```go
package main

import (
    "fmt"
    "net"
    "time"
)

func checkPort(host string, port string) bool {
    timeout := 2 * time.Second
    conn, err := net.DialTimeout("tcp", host+":"+port, timeout)
    
    if err != nil {
        fmt.Printf("✗ Порт %s закрыт на %s\n", port, host)
        return false
    }
    defer conn.Close()
    fmt.Printf("✓ Порт %s открыт на %s\n", port, host)
    return true
}

func main() {
    checkPort("localhost", "80")
    checkPort("localhost", "443")
    checkPort("localhost", "8080")
}
```

### 6. Логирование с ротацией
```go
package main

import (
    "log"
    "os"
    "gopkg.in/natefinch/lumberjack.v2"
)

func setupLogger(filename string) *log.Logger {
    logger := &lumberjack.Logger{
        Filename:   filename,
        MaxSize:    10, // MB
        MaxBackups: 5,
        MaxAge:     30, // days
        Compress:   true,
    }
    
    return log.New(logger, "", log.LstdFlags)
}

func main() {
    logger := setupLogger("/var/log/myapp.log")
    logger.Println("Приложение запущено")
    logger.Println("Выполнение операции...")
}
```

### 7. Парсинг логов Nginx
```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "regexp"
)

func parseNginxLog(filename string) {
    file, err := os.Open(filename)
    if err != nil {
        fmt.Println("Ошибка открытия файла:", err)
        return
    }
    defer file.Close()
    
    pattern := regexp.MustCompile(`(\d+\.\d+\.\d+\.\d+).*"(GET|POST) (.+?) HTTP.*" (\d+)`)
    scanner := bufio.NewScanner(file)
    
    for scanner.Scan() {
        line := scanner.Text()
        matches := pattern.FindStringSubmatch(line)
        
        if len(matches) > 4 && matches[4] == "404" {
            fmt.Printf("404: %s -> %s\n", matches[1], matches[3])
        }
    }
}

func main() {
    parseNginxLog("/var/log/nginx/access.log")
}
```

### 8. Мониторинг изменений файлов
```go
package main

import (
    "fmt"
    "log"
    "github.com/fsnotify/fsnotify"
)

func watchFile(filepath string) {
    watcher, err := fsnotify.NewWatcher()
    if err != nil {
        log.Fatal(err)
    }
    defer watcher.Close()
    
    done := make(chan bool)
    
    go func() {
        for {
            select {
            case event, ok := <-watcher.Events:
                if !ok {
                    return
                }
                if event.Op&fsnotify.Write == fsnotify.Write {
                    fmt.Printf("⚠️ Файл %s изменён!\n", event.Name)
                }
            case err, ok := <-watcher.Errors:
                if !ok {
                    return
                }
                log.Println("Ошибка:", err)
            }
        }
    }()
    
    err = watcher.Add(filepath)
    if err != nil {
        log.Fatal(err)
    }
    <-done
}

func main() {
    watchFile("/etc/nginx/nginx.conf")
}
```

---

## 🐳 Docker & Containers

### 9. Список всех контейнеров
```go
package main

import (
    "context"
    "fmt"
    "github.com/docker/docker/api/types"
    "github.com/docker/docker/client"
)

func listContainers() {
    cli, err := client.NewClientWithOpts(client.FromEnv)
    if err != nil {
        panic(err)
    }
    
    containers, err := cli.ContainerList(context.Background(), types.ContainerListOptions{All: true})
    if err != nil {
        panic(err)
    }
    
    for _, container := range containers {
        fmt.Printf("%s: %s\n", container.Names[0], container.State)
    }
}

func main() {
    listContainers()
}
```

### 10. Очистка неиспользуемых образов
```go
package main

import (
    "context"
    "fmt"
    "github.com/docker/docker/api/types"
    "github.com/docker/docker/client"
)

func cleanupDocker() {
    cli, err := client.NewClientWithOpts(client.FromEnv)
    if err != nil {
        panic(err)
    }
    
    // Prune containers
    report, err := cli.ContainersPrune(context.Background(), types.ContainersPruneOptions{})
    if err != nil {
        panic(err)
    }
    fmt.Printf("✓ Удалено контейнеров: %d\n", len(report.ContainersDeleted))
    
    // Prune images
    imgReport, err := cli.ImagesPrune(context.Background(), types.ImagesPruneOptions{})
    if err != nil {
        panic(err)
    }
    fmt.Printf("✓ Освобождено места: %d bytes\n", imgReport.SpaceReclaimed)
}

func main() {
    cleanupDocker()
}
```

### 11. Проверка health контейнеров
```go
package main

import (
    "context"
    "fmt"
    "github.com/docker/docker/api/types"
    "github.com/docker/docker/client"
)

func checkContainerHealth() {
    cli, err := client.NewClientWithOpts(client.FromEnv)
    if err != nil {
        panic(err)
    }
    
    containers, err := cli.ContainerList(context.Background(), types.ContainerListOptions{})
    if err != nil {
        panic(err)
    }
    
    for _, container := range containers {
        inspect, err := cli.ContainerInspect(context.Background(), container.ID)
        if err != nil {
            continue
        }
        
        if inspect.State.Health != nil {
            status := inspect.State.Health.Status
            if status != "healthy" {
                fmt.Printf("⚠️ Нездоровый контейнер: %s (%s)\n", container.Names[0], status)
            }
        }
    }
}

func main() {
    checkContainerHealth()
}
```

### 12. Автоматический restart упавших контейнеров
```go
package main

import (
    "context"
    "fmt"
    "github.com/docker/docker/api/types"
    "github.com/docker/docker/client"
)

func restartStoppedContainers() {
    cli, err := client.NewClientWithOpts(client.FromEnv)
    if err != nil {
        panic(err)
    }
    
    filters := types.ContainerListOptions{
        All:     true,
        Filters: map[string][]string{"status": {"exited"}},
    }
    
    containers, err := cli.ContainerList(context.Background(), filters)
    if err != nil {
        panic(err)
    }
    
    for _, container := range containers {
        err := cli.ContainerStart(context.Background(), container.ID, types.ContainerStartOptions{})
        if err != nil {
            fmt.Printf("✗ Ошибка запуска %s\n", container.Names[0])
        } else {
            fmt.Printf("✓ Перезапущен: %s\n", container.Names[0])
        }
    }
}

func main() {
    restartStoppedContainers()
}
```

### 13. Мониторинг ресурсов Docker
```go
package main

import (
    "context"
    "fmt"
    "github.com/docker/docker/api/types"
    "github.com/docker/docker/client"
)

func dockerStats() {
    cli, err := client.NewClientWithOpts(client.FromEnv)
    if err != nil {
        panic(err)
    }
    
    containers, err := cli.ContainerList(context.Background(), types.ContainerListOptions{})
    if err != nil {
        panic(err)
    }
    
    fmt.Println("Container\t\tCPU\tMemory")
    for _, container := range containers {
        stats, err := cli.ContainerStats(context.Background(), container.ID, false)
        if err != nil {
            continue
        }
        defer stats.Body.Close()
        
        fmt.Printf("%s\t%.2f%%\t%.2fMB\n", 
            container.Names[0], 
            0.0, // Simplified - needs proper calculation
            0.0)
    }
}

func main() {
    dockerStats()
}
```

---

## ☸️ Kubernetes

### 14. Список всех подов
```go
package main

import (
    "context"
    "fmt"
    "k8s.io/client-go/kubernetes"
    "k8s.io/client-go/tools/clientcmd"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

func listPods(namespace string) {
    config, err := clientcmd.BuildConfigFromFlags("", clientcmd.RecommendedHomeFile)
    if err != nil {
        panic(err)
    }
    
    clientset, err := kubernetes.NewForConfig(config)
    if err != nil {
        panic(err)
    }
    
    pods, err := clientset.CoreV1().Pods(namespace).List(context.TODO(), metav1.ListOptions{})
    if err != nil {
        panic(err)
    }
    
    for _, pod := range pods.Items {
        fmt.Printf("%s: %s\n", pod.Name, pod.Status.Phase)
    }
}

func main() {
    listPods("default")
}
```

### 15. Проверка failing подов
```go
package main

import (
    "context"
    "fmt"
    "k8s.io/client-go/kubernetes"
    "k8s.io/client-go/tools/clientcmd"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

func checkFailingPods(namespace string) {
    config, err := clientcmd.BuildConfigFromFlags("", clientcmd.RecommendedHomeFile)
    if err != nil {
        panic(err)
    }
    
    clientset, err := kubernetes.NewForConfig(config)
    if err != nil {
        panic(err)
    }
    
    pods, err := clientset.CoreV1().Pods(namespace).List(context.TODO(), metav1.ListOptions{})
    if err != nil {
        panic(err)
    }
    
    for _, pod := range pods.Items {
        if pod.Status.Phase != "Running" {
            fmt.Printf("⚠️ %s: %s\n", pod.Name, pod.Status.Phase)
        }
    }
}

func main() {
    checkFailingPods("default")
}
```

### 16. Масштабирование deployment
```go
package main

import (
    "context"
    "fmt"
    "k8s.io/client-go/kubernetes"
    "k8s.io/client-go/tools/clientcmd"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

func scaleDeployment(name string, replicas int32, namespace string) {
    config, err := clientcmd.BuildConfigFromFlags("", clientcmd.RecommendedHomeFile)
    if err != nil {
        panic(err)
    }
    
    clientset, err := kubernetes.NewForConfig(config)
    if err != nil {
        panic(err)
    }
    
    deploymentsClient := clientset.AppsV1().Deployments(namespace)
    
    scale, err := deploymentsClient.GetScale(context.TODO(), name, metav1.GetOptions{})
    if err != nil {
        panic(err)
    }
    
    scale.Spec.Replicas = replicas
    _, err = deploymentsClient.UpdateScale(context.TODO(), name, scale, metav1.UpdateOptions{})
    if err != nil {
        panic(err)
    }
    
    fmt.Printf("✓ %s масштабирован до %d реплик\n", name, replicas)
}

func main() {
    scaleDeployment("nginx-deployment", 5, "default")
}
```

### 17. Получение логов пода
```go
package main

import (
    "context"
    "fmt"
    "io"
    "k8s.io/client-go/kubernetes"
    "k8s.io/client-go/tools/clientcmd"
    corev1 "k8s.io/api/core/v1"
)

func getPodLogs(podName, namespace string, lines int64) {
    config, err := clientcmd.BuildConfigFromFlags("", clientcmd.RecommendedHomeFile)
    if err != nil {
        panic(err)
    }
    
    clientset, err := kubernetes.NewForConfig(config)
    if err != nil {
        panic(err)
    }
    
    podLogOpts := corev1.PodLogOptions{
        TailLines: &lines,
    }
    
    req := clientset.CoreV1().Pods(namespace).GetLogs(podName, &podLogOpts)
    podLogs, err := req.Stream(context.TODO())
    if err != nil {
        panic(err)
    }
    defer podLogs.Close()
    
    buf := make([]byte, 2000)
    for {
        numBytes, err := podLogs.Read(buf)
        if err == io.EOF {
            break
        }
        if err != nil {
            panic(err)
        }
        fmt.Print(string(buf[:numBytes]))
    }
}

func main() {
    getPodLogs("my-pod-123", "default", 50)
}
```

### 18. Проверка статуса нод
```go
package main

import (
    "context"
    "fmt"
    "k8s.io/client-go/kubernetes"
    "k8s.io/client-go/tools/clientcmd"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

func checkNodes() {
    config, err := clientcmd.BuildConfigFromFlags("", clientcmd.RecommendedHomeFile)
    if err != nil {
        panic(err)
    }
    
    clientset, err := kubernetes.NewForConfig(config)
    if err != nil {
        panic(err)
    }
    
    nodes, err := clientset.CoreV1().Nodes().List(context.TODO(), metav1.ListOptions{})
    if err != nil {
        panic(err)
    }
    
    for _, node := range nodes.Items {
        for _, condition := range node.Status.Conditions {
            if condition.Type == "Ready" {
                fmt.Printf("%s: Ready=%s\n", node.Name, condition.Status)
            }
        }
    }
}

func main() {
    checkNodes()
}
```

---

## 🔐 Security & Secrets

### 19. Проверка SSL сертификата
```go
package main

import (
    "crypto/tls"
    "fmt"
    "time"
)

func checkSSLExpiry(hostname string) {
    conn, err := tls.Dial("tcp", hostname+":443", &tls.Config{})
    if err != nil {
        fmt.Println("Ошибка подключения:", err)
        return
    }
    defer conn.Close()
    
    certs := conn.ConnectionState().PeerCertificates
    for _, cert := range certs {
        daysLeft := int(time.Until(cert.NotAfter).Hours() / 24)
        fmt.Printf("%s: истекает через %d дней\n", hostname, daysLeft)
        
        if daysLeft < 30 {
            fmt.Println("⚠️ Сертификат скоро истечёт!")
        }
    }
}

func main() {
    checkSSLExpiry("google.com")
}
```

### 20. Сканирование открытых портов
```go
package main

import (
    "fmt"
    "net"
    "sync"
    "time"
)

func scanPorts(host string, startPort, endPort int) {
    var wg sync.WaitGroup
    openPorts := []int{}
    var mu sync.Mutex
    
    for port := startPort; port <= endPort; port++ {
        wg.Add(1)
        go func(p int) {
            defer wg.Done()
            address := fmt.Sprintf("%s:%d", host, p)
            conn, err := net.DialTimeout("tcp", address, 500*time.Millisecond)
            
            if err == nil {
                mu.Lock()
                openPorts = append(openPorts, p)
                mu.Unlock()
                conn.Close()
            }
        }(port)
    }
    
    wg.Wait()
    fmt.Printf("Открытые порты на %s: %v\n", host, openPorts)
}

func main() {
    scanPorts("localhost", 1, 1024)
}
```

### 21. Генерация случайных паролей
```go
package main

import (
    "crypto/rand"
    "fmt"
    "math/big"
)

func generatePassword(length int) string {
    chars := "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!@#$%^&*()"
    password := make([]byte, length)
    
    for i := range password {
        num, _ := rand.Int(rand.Reader, big.NewInt(int64(len(chars))))
        password[i] = chars[num.Int64()]
    }
    
    return string(password)
}

func main() {
    fmt.Printf("Пароль: %s\n", generatePassword(16))
}
```

### 22. Шифрование конфигурационных файлов
```go
package main

import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "fmt"
    "io"
    "os"
)

func encryptFile(filename string, key []byte) error {
    plaintext, err := os.ReadFile(filename)
    if err != nil {
        return err
    }
    
    block, err := aes.NewCipher(key)
    if err != nil {
        return err
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return err
    }
    
    nonce := make([]byte, gcm.NonceSize())
    if _, err := io.ReadFull(rand.Reader, nonce); err != nil {
        return err
    }
    
    ciphertext := gcm.Seal(nonce, nonce, plaintext, nil)
    
    return os.WriteFile(filename+".encrypted", ciphertext, 0644)
}

func main() {
    key := make([]byte, 32) // AES-256
    rand.Read(key)
    
    err := encryptFile("config.yaml", key)
    if err != nil {
        fmt.Println("Ошибка:", err)
    } else {
        fmt.Println("✓ Файл зашифрован")
    }
}
```

---

## 📊 Monitoring & Alerts

### 23. Отправка метрик в Prometheus
```go
package main

import (
    "fmt"
    "net/http"
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    diskUsage = prometheus.NewGauge(prometheus.GaugeOpts{
        Name: "disk_usage_percent",
        Help: "Current disk usage in percent",
    })
)

func init() {
    prometheus.MustRegister(diskUsage)
}

func main() {
    diskUsage.Set(75.5)
    
    http.Handle("/metrics", promhttp.Handler())
    fmt.Println("Metrics server на :2112")
    http.ListenAndServe(":2112", nil)
}
```

### 24. Проверка времени отклика сервиса
```go
package main

import (
    "fmt"
    "net/http"
    "time"
)

func checkResponseTime(url string, threshold time.Duration) {
    start := time.Now()
    
    resp, err := http.Get(url)
    if err != nil {
        fmt.Printf("✗ Ошибка: %v\n", err)
        return
    }
    defer resp.Body.Close()
    
    elapsed := time.Since(start)
    
    fmt.Printf("%s: %v (status: %d)\n", url, elapsed, resp.StatusCode)
    
    if elapsed > threshold {
        fmt.Printf("⚠️ Медленный ответ! Порог: %v\n", threshold)
    }
}

func main() {
    checkResponseTime("https://google.com", 2*time.Second)
}
```

### 25. Мониторинг доступности множества сервисов
```go
package main

import (
    "fmt"
    "net/http"
    "sync"
)

type Service struct {
    Name string
    URL  string
}

func checkServices(services []Service) {
    var wg sync.WaitGroup
    
    for _, service := range services {
        wg.Add(1)
        go func(s Service) {
            defer wg.Done()
            
            resp, err := http.Get(s.URL)
            if err != nil {
                fmt.Printf("✗ %s: %v\n", s.Name, err)
                return
            }
            defer resp.Body.Close()
            
            if resp.StatusCode == 200 {
                fmt.Printf("✓ %s: %d\n", s.Name, resp.StatusCode)
            } else {
                fmt.Printf("✗ %s: %d\n", s.Name, resp.StatusCode)
            }
        }(service)
    }
    
    wg.Wait()
}

func main() {
    services := []Service{
        {"API", "https://api.example.com/health"},
        {"Web", "https://example.com"},
        {"Database", "http://localhost:5432"},
    }
    
    checkServices(services)
}
```

### 26. Telegram уведомления
```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "net/http"
)

func sendTelegram(token, chatID, message string) error {
    url := fmt.Sprintf("https://api.telegram.org/bot%s/sendMessage", token)
    
    payload := map[string]string{
        "chat_id": chatID,
        "text":    message,
    }
    
    jsonData, err := json.Marshal(payload)
    if err != nil {
        return err
    }
    
    resp, err := http.Post(url, "application/json", bytes.NewBuffer(jsonData))
    if err != nil {
        return err
    }
    defer resp.Body.Close()
    
    if resp.StatusCode == 200 {
        fmt.Println("✓ Сообщение отправлено")
    } else {
        fmt.Println("✗ Ошибка отправки")
    }
    
    return nil
}

func main() {
    token := "your_bot_token"
    chatID := "your_chat_id"
    sendTelegram(token, chatID, "⚠️ Сервер упал!")
}
```

### 27. Проверка доступности базы данных
```go
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/lib/pq"
)

func checkPostgres(host, database, user, password string) bool {
    connStr := fmt.Sprintf("host=%s dbname=%s user=%s password=%s sslmode=disable", 
        host, database, user, password)
    
    db, err := sql.Open("postgres", connStr)
    if err != nil {
        fmt.Printf("✗ PostgreSQL %s: %v\n", host, err)
        return false
    }
    defer db.Close()
    
    err = db.Ping()
    if err != nil {
        fmt.Printf("✗ PostgreSQL %s: %v\n", host, err)
        return false
    }
    
    fmt.Printf("✓ PostgreSQL %s доступен\n", host)
    return true
}

func main() {
    checkPostgres("localhost", "mydb", "user", "password")
}
```

---

## 🚀 CI/CD

### 28. Проверка статуса Jenkins job
```go
package main

import (
    "encoding/json"
    "fmt"
    "io"
    "net/http"
)

type JenkinsJob struct {
    Result string `json:"result"`
}

func checkJenkinsJob(url, jobName, user, token string) {
    apiURL := fmt.Sprintf("%s/job/%s/lastBuild/api/json", url, jobName)
    
    client := &http.Client{}
    req, _ := http.NewRequest("GET", apiURL, nil)
    req.SetBasicAuth(user, token)
    
    resp, err := client.Do(req)
    if err != nil {
        fmt.Println("✗ Ошибка:", err)
        return
    }
    defer resp.Body.Close()
    
    if resp.StatusCode == 201 {
        fmt.Println("✓ Pipeline запущен")
    } else {
        fmt.Println("✗ Ошибка запуска pipeline")
    }
}

func main() {
    triggerGitLabPipeline(12345, "your_token", "main")
}
```

### 30. Проверка успешности последнего коммита
```go
package main

import (
    "fmt"
    "os/exec"
    "strings"
)

func checkLastCommit() {
    cmd := exec.Command("git", "log", "-1", "--pretty=format:%h - %s (%an)")
    output, err := cmd.Output()
    
    if err != nil {
        fmt.Println("Ошибка:", err)
        return
    }
    
    fmt.Printf("Последний коммит: %s\n", strings.TrimSpace(string(output)))
}

func main() {
    checkLastCommit()
}
```

### 31. Автоматическое создание релизных тегов
```go
package main

import (
    "fmt"
    "os/exec"
    "time"
)

func createReleaseTag() {
    version := time.Now().Format("v2006.01.02")
    
    tagCmd := exec.Command("git", "tag", "-a", version, "-m", fmt.Sprintf("Release %s", version))
    if err := tagCmd.Run(); err != nil {
        fmt.Println("Ошибка создания тега:", err)
        return
    }
    
    pushCmd := exec.Command("git", "push", "origin", version)
    if err := pushCmd.Run(); err != nil {
        fmt.Println("Ошибка push тега:", err)
        return
    }
    
    fmt.Printf("✓ Создан тег: %s\n", version)
}

func main() {
    createReleaseTag()
}
```

---

## 🌐 Networking

### 32. Проверка DNS резолвинга
```go
package main

import (
    "fmt"
    "net"
)

func checkDNS(hostname string) {
    ips, err := net.LookupIP(hostname)
    if err != nil {
        fmt.Printf("✗ Не удалось разрешить %s: %v\n", hostname, err)
        return
    }
    
    for _, ip := range ips {
        fmt.Printf("%s -> %s\n", hostname, ip)
    }
}

func main() {
    checkDNS("google.com")
    checkDNS("github.com")
}
```

### 33. Ping множества хостов
```go
package main

import (
    "fmt"
    "os/exec"
    "sync"
)

func pingHosts(hosts []string) {
    var wg sync.WaitGroup
    
    for _, host := range hosts {
        wg.Add(1)
        go func(h string) {
            defer wg.Done()
            
            cmd := exec.Command("ping", "-c", "1", "-W", "1", h)
            err := cmd.Run()
            
            if err == nil {
                fmt.Printf("✓ %s\n", h)
            } else {
                fmt.Printf("✗ %s\n", h)
            }
        }(host)
    }
    
    wg.Wait()
}

func main() {
    hosts := []string{"8.8.8.8", "1.1.1.1", "google.com"}
    pingHosts(hosts)
}
```

### 34. Трассировка маршрута
```go
package main

import (
    "fmt"
    "os/exec"
)

func traceroute(host string) {
    cmd := exec.Command("traceroute", "-m", "15", host)
    output, err := cmd.CombinedOutput()
    
    if err != nil {
        fmt.Println("Ошибка:", err)
        return
    }
    
    fmt.Println(string(output))
}

func main() {
    traceroute("google.com")
}
```

### 35. HTTP клиент с таймаутами
```go
package main

import (
    "fmt"
    "io"
    "net/http"
    "time"
)

func httpGetWithTimeout(url string, timeout time.Duration) {
    client := &http.Client{
        Timeout: timeout,
    }
    
    start := time.Now()
    resp, err := client.Get(url)
    elapsed := time.Since(start)
    
    if err != nil {
        fmt.Printf("✗ Ошибка: %v (время: %v)\n", err, elapsed)
        return
    }
    defer resp.Body.Close()
    
    body, _ := io.ReadAll(resp.Body)
    fmt.Printf("✓ Status: %d, Размер: %d bytes, Время: %v\n", 
        resp.StatusCode, len(body), elapsed)
}

func main() {
    httpGetWithTimeout("https://google.com", 5*time.Second)
}
```

---

## 📦 Package Management

### 36. Проверка версии Go модулей
```go
package main

import (
    "fmt"
    "os/exec"
    "strings"
)

func checkModuleVersions() {
    cmd := exec.Command("go", "list", "-m", "-u", "all")
    output, err := cmd.Output()
    
    if err != nil {
        fmt.Println("Ошибка:", err)
        return
    }
    
    lines := strings.Split(string(output), "\n")
    for _, line := range lines {
        if strings.Contains(line, "[") {
            fmt.Println("Доступно обновление:", line)
        }
    }
}

func main() {
    checkModuleVersions()
}
```

### 37. Автоматическое обновление зависимостей
```go
package main

import (
    "fmt"
    "os/exec"
)

func updateDependencies() {
    fmt.Println("Обновление зависимостей...")
    
    cmd := exec.Command("go", "get", "-u", "./...")
    output, err := cmd.CombinedOutput()
    
    if err != nil {
        fmt.Printf("Ошибка: %v\n%s\n", err, output)
        return
    }
    
    tidyCmd := exec.Command("go", "mod", "tidy")
    tidyCmd.Run()
    
    fmt.Println("✓ Зависимости обновлены")
}

func main() {
    updateDependencies()
}
```

### 38. Проверка уязвимостей в зависимостях
```go
package main

import (
    "fmt"
    "os/exec"
)

func securityAudit() {
    cmd := exec.Command("go", "list", "-json", "-m", "all")
    output, err := cmd.Output()
    
    if err != nil {
        fmt.Println("Ошибка:", err)
        return
    }
    
    // Можно интегрировать с govulncheck
    vulnCmd := exec.Command("govulncheck", "./...")
    vulnOutput, _ := vulnCmd.CombinedOutput()
    
    fmt.Println(string(vulnOutput))
}

func main() {
    securityAudit()
}
```

---

## 🗄️ Database Operations

### 39. Бэкап PostgreSQL
```go
package main

import (
    "fmt"
    "os/exec"
    "time"
)

func backupPostgres(host, database, user, outputDir string) {
    timestamp := time.Now().Format("20060102_150405")
    filename := fmt.Sprintf("%s/%s_%s.sql", outputDir, database, timestamp)
    
    cmd := exec.Command("pg_dump",
        "-h", host,
        "-U", user,
        "-d", database,
        "-f", filename,
    )
    
    if err := cmd.Run(); err != nil {
        fmt.Println("✗ Ошибка бэкапа:", err)
        return
    }
    
    fmt.Printf("✓ Бэкап создан: %s\n", filename)
}

func main() {
    backupPostgres("localhost", "mydb", "postgres", "/backups")
}
```

### 40. Проверка размера таблиц
```go
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/lib/pq"
)

func checkTableSizes(host, database, user, password string) {
    connStr := fmt.Sprintf("host=%s dbname=%s user=%s password=%s sslmode=disable",
        host, database, user, password)
    
    db, err := sql.Open("postgres", connStr)
    if err != nil {
        fmt.Println("Ошибка подключения:", err)
        return
    }
    defer db.Close()
    
    query := `
        SELECT tablename, pg_size_pretty(pg_total_relation_size(tablename::regclass))
        FROM pg_tables 
        WHERE schemaname = 'public'
        ORDER BY pg_total_relation_size(tablename::regclass) DESC
    `
    
    rows, err := db.Query(query)
    if err != nil {
        fmt.Println("Ошибка запроса:", err)
        return
    }
    defer rows.Close()
    
    fmt.Println("Таблица\t\t\tРазмер")
    fmt.Println("-----------------------------------")
    for rows.Next() {
        var tableName, size string
        rows.Scan(&tableName, &size)
        fmt.Printf("%s\t\t%s\n", tableName, size)
    }
}

func main() {
    checkTableSizes("localhost", "mydb", "user", "password")
}
```

### 41. Мониторинг медленных запросов
```go
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/lib/pq"
)

func slowQueries(host, database, user, password string, minDuration int) {
    connStr := fmt.Sprintf("host=%s dbname=%s user=%s password=%s sslmode=disable",
        host, database, user, password)
    
    db, err := sql.Open("postgres", connStr)
    if err != nil {
        fmt.Println("Ошибка подключения:", err)
        return
    }
    defer db.Close()
    
    query := `
        SELECT query, calls, total_time, mean_time 
        FROM pg_stat_statements 
        WHERE mean_time > $1
        ORDER BY mean_time DESC 
        LIMIT 10
    `
    
    rows, err := db.Query(query, minDuration)
    if err != nil {
        fmt.Println("Ошибка запроса:", err)
        return
    }
    defer rows.Close()
    
    for rows.Next() {
        var query string
        var calls int
        var totalTime, meanTime float64
        rows.Scan(&query, &calls, &totalTime, &meanTime)
        
        fmt.Printf("Mean: %.2fms | Calls: %d\n", meanTime, calls)
        fmt.Printf("Query: %.100s...\n\n", query)
    }
}

func main() {
    slowQueries("localhost", "mydb", "user", "password", 1000)
}
```

---

## 📈 Performance Testing

### 42. Простой load test
```go
package main

import (
    "fmt"
    "net/http"
    "sync"
    "time"
)

func loadTest(url string, requestsCount, workers int) {
    var wg sync.WaitGroup
    results := make(chan time.Duration, requestsCount)
    
    startTime := time.Now()
    
    for i := 0; i < workers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := 0; j < requestsCount/workers; j++ {
                reqStart := time.Now()
                resp, err := http.Get(url)
                reqDuration := time.Since(reqStart)
                
                if err == nil {
                    resp.Body.Close()
                    results <- reqDuration
                }
            }
        }()
    }
    
    wg.Wait()
    close(results)
    
    totalTime := time.Since(startTime)
    
    var totalDuration time.Duration
    successCount := 0
    for duration := range results {
        totalDuration += duration
        successCount++
    }
    
    fmt.Printf("Load Test Results:\n")
    fmt.Printf("Total time: %v\n", totalTime)
    fmt.Printf("Successful requests: %d/%d\n", successCount, requestsCount)
    fmt.Printf("Average response time: %v\n", totalDuration/time.Duration(successCount))
    fmt.Printf("Requests per second: %.2f\n", float64(successCount)/totalTime.Seconds())
}

func main() {
    loadTest("https://google.com", 100, 10)
}
```

### 43. Бенчмарк API endpoint
```go
package main

import (
    "fmt"
    "net/http"
    "time"
)

func benchmarkAPI(url string, iterations int) {
    times := make([]time.Duration, iterations)
    
    for i := 0; i < iterations; i++ {
        start := time.Now()
        resp, err := http.Get(url)
        elapsed := time.Since(start)
        
        if err == nil {
            resp.Body.Close()
            times[i] = elapsed
        }
    }
    
    var total time.Duration
    var min, max time.Duration
    min = times[0]
    max = times[0]
    
    for _, t := range times {
        total += t
        if t < min {
            min = t
        }
        if t > max {
            max = t
        }
    }
    
    avg := total / time.Duration(iterations)
    
    fmt.Printf("Benchmark Results:\n")
    fmt.Printf("Min: %v\n", min)
    fmt.Printf("Max: %v\n", max)
    fmt.Printf("Avg: %v\n", avg)
}

func main() {
    benchmarkAPI("https://api.github.com", 50)
}
```

---

## 🔄 Automation Helpers

### 44. Выполнение команд на удалённом сервере
```go
package main

import (
    "fmt"
    "golang.org/x/crypto/ssh"
    "io/ioutil"
)

func remoteExecute(host, user, keyPath, command string) {
    key, err := ioutil.ReadFile(keyPath)
    if err != nil {
        fmt.Println("Ошибка чтения ключа:", err)
        return
    }
    
    signer, err := ssh.ParsePrivateKey(key)
    if err != nil {
        fmt.Println("Ошибка парсинга ключа:", err)
        return
    }
    
    config := &ssh.ClientConfig{
        User: user,
        Auth: []ssh.AuthMethod{
            ssh.PublicKeys(signer),
        },
        HostKeyCallback: ssh.InsecureIgnoreHostKey(),
    }
    
    client, err := ssh.Dial("tcp", host+":22", config)
    if err != nil {
        fmt.Println("Ошибка подключения:", err)
        return
    }
    defer client.Close()
    
    session, err := client.NewSession()
    if err != nil {
        fmt.Println("Ошибка создания сессии:", err)
        return
    }
    defer session.Close()
    
    output, err := session.CombinedOutput(command)
    if err != nil {
        fmt.Println("Ошибка выполнения:", err)
        return
    }
    
    fmt.Println(string(output))
}

func main() {
    remoteExecute("192.168.1.100", "user", "/home/user/.ssh/id_rsa", "df -h")
}
```

### 45. Копирование файлов по SSH
```go
package main

import (
    "fmt"
    "io"
    "os"
    "path/filepath"
    "golang.org/x/crypto/ssh"
    "github.com/pkg/sftp"
)

func scpFile(host, user, password, localPath, remotePath string) error {
    config := &ssh.ClientConfig{
        User: user,
        Auth: []ssh.AuthMethod{
            ssh.Password(password),
        },
        HostKeyCallback: ssh.InsecureIgnoreHostKey(),
    }
    
    client, err := ssh.Dial("tcp", host+":22", config)
    if err != nil {
        return err
    }
    defer client.Close()
    
    sftpClient, err := sftp.NewClient(client)
    if err != nil {
        return err
    }
    defer sftpClient.Close()
    
    srcFile, err := os.Open(localPath)
    if err != nil {
        return err
    }
    defer srcFile.Close()
    
    dstFile, err := sftpClient.Create(remotePath)
    if err != nil {
        return err
    }
    defer dstFile.Close()
    
    _, err = io.Copy(dstFile, srcFile)
    if err != nil {
        return err
    }
    
    fmt.Printf("✓ Файл скопирован на %s\n", host)
    return nil
}

func main() {
    scpFile("192.168.1.100", "user", "password", "local.txt", "/tmp/remote.txt")
}
```

### 46. Парсинг конфигурационных файлов
```go
package main

import (
    "encoding/json"
    "fmt"
    "gopkg.in/yaml.v2"
    "io/ioutil"
    "path/filepath"
)

type Config struct {
    Server struct {
        Host string `yaml:"host" json:"host"`
        Port int    `yaml:"port" json:"port"`
    } `yaml:"server" json:"server"`
    Database struct {
        Name string `yaml:"name" json:"name"`
        User string `yaml:"user" json:"user"`
    } `yaml:"database" json:"database"`
}

func parseConfig(filename string) (*Config, error) {
    data, err := ioutil.ReadFile(filename)
    if err != nil {
        return nil, err
    }
    
    var config Config
    ext := filepath.Ext(filename)
    
    switch ext {
    case ".yaml", ".yml":
        err = yaml.Unmarshal(data, &config)
    case ".json":
        err = json.Unmarshal(data, &config)
    default:
        return nil, fmt.Errorf("неподдерживаемый формат: %s", ext)
    }
    
    if err != nil {
        return nil, err
    }
    
    return &config, nil
}

func main() {
    config, err := parseConfig("config.yaml")
    if err != nil {
        fmt.Println("Ошибка:", err)
        return
    }
    
    fmt.Printf("Host: %s:%d\n", config.Server.Host, config.Server.Port)
    fmt.Printf("Database: %s\n", config.Database.Name)
}
```

### 47. Генерация отчётов в HTML
```go
package main

import (
    "fmt"
    "html/template"
    "os"
    "time"
)

type ReportData struct {
    Title     string
    Timestamp time.Time
    Metrics   map[string]string
}

const htmlTemplate = `
<!DOCTYPE html>
<html>
<head>
    <title>{{.Title}}</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #4CAF50; color: white; }
    </style>
</head>
<body>
    <h1>{{.Title}}</h1>
    <p>Generated: {{.Timestamp.Format "2006-01-02 15:04:05"}}</p>
    <table>
        <tr><th>Metric</th><th>Value</th></tr>
        {{range $key, $value := .Metrics}}
        <tr><td>{{$key}}</td><td>{{$value}}</td></tr>
        {{end}}
    </table>
</body>
</html>
`

func generateReport(filename string, data ReportData) error {
    tmpl, err := template.New("report").Parse(htmlTemplate)
    if err != nil {
        return err
    }
    
    file, err := os.Create(filename)
    if err != nil {
        return err
    }
    defer file.Close()
    
    err = tmpl.Execute(file, data)
    if err != nil {
        return err
    }
    
    fmt.Printf("✓ Отчёт создан: %s\n", filename)
    return nil
}

func main() {
    data := ReportData{
        Title:     "System Report",
        Timestamp: time.Now(),
        Metrics: map[string]string{
            "CPU":    "45%",
            "RAM":    "70%",
            "Disk":   "55%",
            "Uptime": "30 days",
        },
    }
    
    generateReport("report.html", data)
}
```

### 48. Ротация логов
```go
package main

import (
    "compress/gzip"
    "fmt"
    "io"
    "os"
    "time"
)

func rotateLogs(logFile string, maxSizeMB int64) error {
    info, err := os.Stat(logFile)
    if err != nil {
        return err
    }
    
    sizeMB := info.Size() / (1024 * 1024)
    
    if sizeMB > maxSizeMB {
        timestamp := time.Now().Format("20060102_150405")
        archivedFile := fmt.Sprintf("%s.%s.gz", logFile, timestamp)
        
        // Открыть исходный файл
        src, err := os.Open(logFile)
        if err != nil {
            return err
        }
        defer src.Close()
        
        // Создать архив
        dst, err := os.Create(archivedFile)
        if err != nil {
            return err
        }
        defer dst.Close()
        
        gzWriter := gzip.NewWriter(dst)
        defer gzWriter.Close()
        
        _, err = io.Copy(gzWriter, src)
        if err != nil {
            return err
        }
        
        // Очистить оригинальный файл
        err = os.Truncate(logFile, 0)
        if err != nil {
            return err
        }
        
        fmt.Printf("✓ Лог заархивирован: %s\n", archivedFile)
    }
    
    return nil
}

func main() {
    rotateLogs("/var/log/app.log", 100)
}
```

### 49. Проверка соответствия версий
```go
package main

import (
    "fmt"
    "os/exec"
    "regexp"
    "strconv"
    "strings"
)

func checkVersion(command []string, requiredVersion string) bool {
    cmd := exec.Command(command[0], command[1:]...)
    output, err := cmd.CombinedOutput()
    if err != nil {
        fmt.Printf("✗ Ошибка выполнения команды: %v\n", err)
        return false
    }
    
    re := regexp.MustCompile(`(\d+)\.(\d+)\.(\d+)`)
    matches := re.FindStringSubmatch(string(output))
    
    if len(matches) < 4 {
        fmt.Println("✗ Не удалось извлечь версию")
        return false
    }
    
    current := parseVersion(matches[1], matches[2], matches[3])
    required := parseVersionString(requiredVersion)
    
    if compareVersions(current, required) >= 0 {
        fmt.Printf("✓ Версия соответствует: %s\n", matches[0])
        return true
    }
    
    fmt.Printf("✗ Требуется обновление: %s < %s\n", matches[0], requiredVersion)
    return false
}

func parseVersion(major, minor, patch string) []int {
    m, _ := strconv.Atoi(major)
    n, _ := strconv.Atoi(minor)
    p, _ := strconv.Atoi(patch)
    return []int{m, n, p}
}

func parseVersionString(version string) []int {
    parts := strings.Split(version, ".")
    result := make([]int, 3)
    for i, p := range parts {
        if i < 3 {
            result[i], _ = strconv.Atoi(p)
        }
    }
    return result
}

func compareVersions(v1, v2 []int) int {
    for i := 0; i < 3; i++ {
        if v1[i] > v2[i] {
            return 1
        } else if v1[i] < v2[i] {
            return -1
        }
    }
    return 0
}

func main() {
    checkVersion([]string{"docker", "--version"}, "20.10.0")
    checkVersion([]string{"kubectl", "version", "--client", "--short"}, "1.20.0")
}
```

### 50. Комплексная проверка инфраструктуры
```go
package main

import (
    "database/sql"
    "fmt"
    "net/http"
    "syscall"
    "time"
    _ "github.com/lib/pq"
)

type HealthCheck struct {
    Name   string
    Status string
    Value  interface{}
}

func infrastructureHealthCheck() []HealthCheck {
    checks := []HealthCheck{}
    
    // Проверка диска
    var stat syscall.Statfs_t
    syscall.Statfs("/", &stat)
    diskPercent := float64(stat.Blocks-stat.Bfree) / float64(stat.Blocks) * 100
    
    diskStatus := "OK"
    if diskPercent > 80 {
        diskStatus = "WARNING"
    }
    
    checks = append(checks, HealthCheck{
        Name:   "Disk",
        Status: diskStatus,
        Value:  fmt.Sprintf("%.2f%%", diskPercent),
    })
    
    // Проверка HTTP сервисов
    services := map[string]string{
        "nginx": "http://localhost:80",
        "api":   "http://localhost:8000/health",
    }
    
    for name, url := range services {
        client := &http.Client{Timeout: 5 * time.Second}
        start := time.Now()
        resp, err := client.Get(url)
        elapsed := time.Since(start)
        
        status := "ERROR"
        value := err
        
        if err == nil {
            resp.Body.Close()
            if resp.StatusCode == 200 {
                status = "OK"
                value = fmt.Sprintf("%v", elapsed)
            }
        }
        
        checks = append(checks, HealthCheck{
            Name:   name,
            Status: status,
            Value:  value,
        })
    }
    
    // Проверка базы данных
    db, err := sql.Open("postgres", "host=localhost user=postgres dbname=mydb sslmode=disable")
    dbStatus := "OK"
    if err != nil || db.Ping() != nil {
        dbStatus = "ERROR"
    }
    if db != nil {
        db.Close()
    }
    
    checks = append(checks, HealthCheck{
        Name:   "Database",
        Status: dbStatus,
        Value:  "PostgreSQL",
    })
    
    return checks
}

func printHealthReport(checks []HealthCheck) {
    fmt.Println("==================================================")
    fmt.Println("INFRASTRUCTURE HEALTH CHECK")
    fmt.Println("==================================================")
    fmt.Printf("Timestamp: %s\n\n", time.Now().Format("2006-01-02 15:04:05"))
    
    for _, check := range checks {
        icon := "✓"
        if check.Status != "OK" {
            icon = "✗"
        }
        
        fmt.Printf("%s %s: %s\n", icon, check.Name, check.Status)
        if check.Value != nil {
            fmt.Printf("  Value: %v\n", check.Value)
        }
    }
    
    fmt.Println("==================================================")
}

func main() {
    checks := infrastructureHealthCheck()
    printHealthReport(checks)
}
```

---

## 🛠️ Установка зависимостей

### go.mod пример
```go
module devops-scripts

go 1.21

require (
    github.com/docker/docker v24.0.7+incompatible
    github.com/fsnotify/fsnotify v1.7.0
    github.com/lib/pq v1.10.9
    github.com/pkg/sftp v1.13.6
    github.com/prometheus/client_golang v1.17.0
    golang.org/x/crypto v0.17.0
    gopkg.in/natefinch/lumberjack.v2 v2.2.1
    gopkg.in/yaml.v2 v2.4.0
    k8s.io/api v0.28.4
    k8s.io/apimachinery v0.28.4
    k8s.io/client-go v0.28.4
)
```

### Установка:
```bash
go mod init devops-scripts
go mod tidy
go get -u ./...
```

---

## 📚 Best Practices

1. **Обработка ошибок**: Всегда проверяйте ошибки
2. **Контекст**: Используйте context для таймаутов
3. **Конкурентность**: Используйте goroutines и sync.WaitGroup
4. **Логирование**: Структурированное логирование (logrus, zap)
5. **Конфигурация**: Используйте ENV переменные
6. **Тестирование**: Пишите unit тесты
7. **Документация**: Комментируйте публичные функции
8. **Версионирование**: Используйте go.mod для управления версиями
9. **Безопасность**: Не храните секреты в коде
10. **Производительность**: Профилируйте код (pprof)

---

## 🚀 Готовые шаблоны для автоматизации

### Универсальный скрипт мониторинга
```go
package main

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "syscall"
    "time"
)

type Config struct {
    CheckInterval   int      `json:"check_interval"`
    CPUThreshold    float64  `json:"cpu_threshold"`
    MemoryThreshold float64  `json:"memory_threshold"`
    DiskThreshold   float64  `json:"disk_threshold"`
    Services        []Service `json:"services"`
    SlackWebhook    string   `json:"slack_webhook"`
}

type Service struct {
    Name string `json:"name"`
    URL  string `json:"url"`
}

type Monitor struct {
    config Config
}

func NewMonitor(configFile string) (*Monitor, error) {
    data, err := ioutil.ReadFile(configFile)
    if err != nil {
        return nil, err
    }
    
    var config Config
    err = json.Unmarshal(data, &config)
    if err != nil {
        return nil, err
    }
    
    return &Monitor{config: config}, nil
}

func (m *Monitor) checkSystemResources() map[string]interface{} {
    var stat syscall.Statfs_t
    syscall.Statfs("/", &stat)
    
    diskPercent := float64(stat.Blocks-stat.Bfree) / float64(stat.Blocks) * 100
    
    return map[string]interface{}{
        "disk": diskPercent,
    }
}

func (m *Monitor) checkServices() []map[string]interface{} {
    results := []map[string]interface{}{}
    
    for _, service := range m.config.Services {
        client := &http.Client{Timeout: 5 * time.Second}
        start := time.Now()
        resp, err := client.Get(service.URL)
        elapsed := time.Since(start)
        
        result := map[string]interface{}{
            "name":     service.Name,
            "url":      service.URL,
            "duration": elapsed.Seconds(),
        }
        
        if err != nil {
            result["status"] = "ERROR"
            result["error"] = err.Error()
        } else {
            resp.Body.Close()
            result["status"] = "OK"
            result["code"] = resp.StatusCode
        }
        
        results = append(results, result)
    }
    
    return results
}

func (m *Monitor) sendAlert(message string) error {
    if m.config.SlackWebhook == "" {
        return nil
    }
    
    payload := map[string]string{"text": message}
    jsonData, _ := json.Marshal(payload)
    
    _, err := http.Post(m.config.SlackWebhook, "application/json", 
        bytes.NewBuffer(jsonData))
    
    return err
}

func (m *Monitor) Run() {
    ticker := time.NewTicker(time.Duration(m.config.CheckInterval) * time.Second)
    defer ticker.Stop()
    
    for {
        fmt.Printf("[%s] Running health checks...\n", time.Now().Format("2006-01-02 15:04:05"))
        
        resources := m.checkSystemResources()
        services := m.checkServices()
        
        alerts := []string{}
        
        // Проверка дисков
        if disk, ok := resources["disk"].(float64); ok {
            if disk > m.config.DiskThreshold {
                alert := fmt.Sprintf("⚠️ High Disk Usage: %.2f%%", disk)
                alerts = append(alerts, alert)
                fmt.Println(alert)
            }
        }
        
        // Проверка сервисов
        for _, service := range services {
            if service["status"] != "OK" {
                alert := fmt.Sprintf("⚠️ Service %s is down!", service["name"])
                alerts = append(alerts, alert)
                fmt.Println(alert)
            }
        }
        
        // Отправка алертов
        if len(alerts) > 0 {
            for _, alert := range alerts {
                m.sendAlert(alert)
            }
        } else {
            fmt.Println("✓ All systems operational")
        }
        
        <-ticker.C
    }
}

func main() {
    monitor, err := NewMonitor("monitor_config.json")
    if err != nil {
        fmt.Println("Error loading config:", err)
        return
    }
    
    monitor.Run()
}
```

### Конфигурационный файл (monitor_config.json)
```json
{
  "check_interval": 60,
  "cpu_threshold": 80.0,
  "memory_threshold": 80.0,
  "disk_threshold": 85.0,
  "services": [
    {
      "name": "webapp",
      "url": "https://example.com"
    },
    {
      "name": "api",
      "url": "https://api.example.com/health"
    }
  ],
  "slack_webhook": "https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
}
```

---

## 🔧 Systemd сервис для Go скрипта

### /etc/systemd/system/monitor.service
```ini
[Unit]
Description=Infrastructure Monitoring Service
After=network.target

[Service]
Type=simple
User=devops
WorkingDirectory=/opt/monitoring
ExecStart=/opt/monitoring/monitor
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

### Команды управления
```bash
# Компиляция
go build -o monitor main.go

# Установка
sudo cp monitor /opt/monitoring/
sudo cp monitor_config.json /opt/monitoring/
sudo systemctl daemon-reload
sudo systemctl enable monitor
sudo systemctl start monitor

# Проверка статуса
sudo systemctl status monitor

# Просмотр логов
sudo journalctl -u monitor -f
```

---

## 📊 Создание Prometheus Exporter

```go
package main

import (
    "fmt"
    "net/http"
    "syscall"
    
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    diskUsageGauge = promauto.NewGauge(prometheus.GaugeOpts{
        Name: "system_disk_usage_percent",
        Help: "Current disk usage percentage",
    })
    
    memoryUsageGauge = promauto.NewGauge(prometheus.GaugeOpts{
        Name: "system_memory_usage_percent",
        Help: "Current memory usage percentage",
    })
    
    serviceHealthGauge = promauto.NewGaugeVec(prometheus.GaugeOpts{
        Name: "service_health_status",
        Help: "Service health status (1=healthy, 0=unhealthy)",
    }, []string{"service"})
)

func recordMetrics() {
    go func() {
        for {
            // Disk usage
            var stat syscall.Statfs_t
            syscall.Statfs("/", &stat)
            diskPercent := float64(stat.Blocks-stat.Bfree) / float64(stat.Blocks) * 100
            diskUsageGauge.Set(diskPercent)
            
            // Memory usage (simplified)
            memoryUsageGauge.Set(65.5) // Replace with actual memory check
            
            // Service health checks
            checkService("api", "http://localhost:8000/health")
            checkService("web", "http://localhost:80")
            
            time.Sleep(10 * time.Second)
        }
    }()
}

func checkService(name, url string) {
    client := &http.Client{Timeout: 5 * time.Second}
    resp, err := client.Get(url)
    
    if err != nil || resp.StatusCode != 200 {
        serviceHealthGauge.WithLabelValues(name).Set(0)
    } else {
        serviceHealthGauge.WithLabelValues(name).Set(1)
        resp.Body.Close()
    }
}

func main() {
    recordMetrics()
    
    http.Handle("/metrics", promhttp.Handler())
    fmt.Println("Exporter running on :2112")
    http.ListenAndServe(":2112", nil)
}
```

---

## 🐳 Dockerfile для Go приложений

```dockerfile
# Multi-stage build
FROM golang:1.21-alpine AS builder

WORKDIR /app

# Копируем go.mod и go.sum
COPY go.mod go.sum ./
RUN go mod download

# Копируем исходный код
COPY . .

# Компиляция с оптимизацией
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Финальный образ
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Копируем бинарник из builder
COPY --from=builder /app/main .
COPY --from=builder /app/config.json .

# Открываем порт
EXPOSE 8080

CMD ["./main"]
```

### Сборка и запуск
```bash
# Сборка образа
docker build -t devops-monitor:latest .

# Запуск контейнера
docker run -d \
  --name monitor \
  -p 8080:8080 \
  -v /var/log:/var/log:ro \
  devops-monitor:latest

# Проверка логов
docker logs -f monitor
```

---

## 🔍 Debugging и Troubleshooting

### 1. Профилирование CPU
```go
package main

import (
    "os"
    "runtime/pprof"
)

func main() {
    f, _ := os.Create("cpu.prof")
    defer f.Close()
    
    pprof.StartCPUProfile(f)
    defer pprof.StopCPUProfile()
    
    // Ваш код здесь
}
```

### 2. Профилирование памяти
```go
package main

import (
    "os"
    "runtime/pprof"
)

func main() {
    // Ваш код здесь
    
    f, _ := os.Create("mem.prof")
    defer f.Close()
    
    pprof.WriteHeapProfile(f)
}
```

### 3. Анализ профилей
```bash
# CPU профиль
go tool pprof cpu.prof

# Memory профиль
go tool pprof mem.prof

# Web интерфейс
go tool pprof -http=:8080 cpu.prof
```

---

## 📝 Логирование с структурированными логами

```go
package main

import (
    "os"
    
    "github.com/sirupsen/logrus"
)

var log = logrus.New()

func setupLogger() {
    log.Out = os.Stdout
    log.SetFormatter(&logrus.JSONFormatter{})
    log.SetLevel(logrus.InfoLevel)
    
    // Ротация логов
    // log.SetOutput(&lumberjack.Logger{
    //     Filename:   "/var/log/app.log",
    //     MaxSize:    10, // MB
    //     MaxBackups: 3,
    //     MaxAge:     28, // days
    // })
}

func main() {
    setupLogger()
    
    log.WithFields(logrus.Fields{
        "service": "api",
        "version": "1.0.0",
    }).Info("Application started")
    
    log.WithFields(logrus.Fields{
        "user":   "admin",
        "action": "login",
    }).Info("User logged in")
    
    log.WithFields(logrus.Fields{
        "error": "connection timeout",
    }).Error("Database connection failed")
}
```

---

## 🧪 Unit тестирование

```go
package main

import (
    "testing"
    "time"
)

func TestCheckPort(t *testing.T) {
    tests := []struct {
        name     string
        host     string
        port     string
        expected bool
    }{
        {"Valid port", "google.com", "80", true},
        {"Invalid port", "google.com", "9999", false},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := checkPort(tt.host, tt.port)
            if result != tt.expected {
                t.Errorf("checkPort(%s, %s) = %v; want %v", 
                    tt.host, tt.port, result, tt.expected)
            }
        })
    }
}

func BenchmarkCheckPort(b *testing.B) {
    for i := 0; i < b.N; i++ {
        checkPort("localhost", "80")
    }
}
```

### Запуск тестов
```bash
# Все тесты
go test ./...

# С покрытием
go test -cover ./...

# Детальное покрытие
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out

# Бенчмарки
go test -bench=. ./...
```

---

## 🔐 Работа с секретами

### Использование environment variables
```go
package main

import (
    "fmt"
    "os"
    
    "github.com/joho/godotenv"
)

type AppConfig struct {
    DBHost     string
    DBUser     string
    DBPassword string
    APIKey     string
}

func loadConfig() (*AppConfig, error) {
    // Загрузка .env файла (опционально)
    godotenv.Load()
    
    config := &AppConfig{
        DBHost:     os.Getenv("DB_HOST"),
        DBUser:     os.Getenv("DB_USER"),
        DBPassword: os.Getenv("DB_PASSWORD"),
        APIKey:     os.Getenv("API_KEY"),
    }
    
    // Валидация
    if config.DBHost == "" {
        return nil, fmt.Errorf("DB_HOST is required")
    }
    
    return config, nil
}

func main() {
    config, err := loadConfig()
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    
    fmt.Printf("Connecting to %s...\n", config.DBHost)
}
```

### .env файл
```env
DB_HOST=localhost
DB_USER=postgres
DB_PASSWORD=secretpassword
API_KEY=your-api-key-here
```

---

## 🎯 Graceful Shutdown

```go
package main

import (
    "context"
    "fmt"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
)

func main() {
    // Создание HTTP сервера
    srv := &http.Server{
        Addr:    ":8080",
        Handler: http.HandlerFunc(handler),
    }
    
    // Запуск сервера в горутине
    go func() {
        fmt.Println("Server starting on :8080")
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            fmt.Printf("Error: %v\n", err)
        }
    }()
    
    // Ожидание сигнала завершения
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    
    fmt.Println("Shutting down server...")
    
    // Graceful shutdown с таймаутом
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    if err := srv.Shutdown(ctx); err != nil {
        fmt.Printf("Server forced to shutdown: %v\n", err)
    }
    
    fmt.Println("Server exited")
}

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, World!")
}
```

---

## 📦 Makefile для автоматизации

```makefile
# Makefile для Go проекта

APP_NAME=devops-monitor
VERSION=1.0.0
BUILD_DIR=build

.PHONY: all build test clean run docker-build docker-run

all: clean build

build:
	@echo "Building $(APP_NAME)..."
	@mkdir -p $(BUILD_DIR)
	@go build -o $(BUILD_DIR)/$(APP_NAME) -ldflags "-X main.Version=$(VERSION)" .

test:
	@echo "Running tests..."
	@go test -v -cover ./...

benchmark:
	@echo "Running benchmarks..."
	@go test -bench=. -benchmem ./...

lint:
	@echo "Running linter..."
	@golangci-lint run

clean:
	@echo "Cleaning..."
	@rm -rf $(BUILD_DIR)
	@go clean

run: build
	@echo "Running $(APP_NAME)..."
	@./$(BUILD_DIR)/$(APP_NAME)

docker-build:
	@echo "Building Docker image..."
	@docker build -t $(APP_NAME):$(VERSION) .

docker-run:
	@echo "Running Docker container..."
	@docker run -d -p 8080:8080 --name $(APP_NAME) $(APP_NAME):$(VERSION)

install-deps:
	@echo "Installing dependencies..."
	@go mod download
	@go mod tidy

release: clean test build
	@echo "Creating release $(VERSION)..."
	@tar -czf $(BUILD_DIR)/$(APP_NAME)-$(VERSION).tar.gz -C $(BUILD_DIR) $(APP_NAME)
```

### Использование
```bash
make build      # Сборка
make test       # Тестирование
make run        # Запуск
make clean      # Очистка
make docker-build  # Docker образ
```

---

## 🚨 Обработка паники и восстановление

```go
package main

import (
    "fmt"
    "log"
    "runtime/debug"
)

func recoverFromPanic() {
    if r := recover(); r != nil {
        log.Printf("Recovered from panic: %v\n", r)
        log.Printf("Stack trace:\n%s", debug.Stack())
    }
}

func riskyOperation() {
    defer recoverFromPanic()
    
    // Код, который может вызвать панику
    panic("something went wrong!")
}

func safeExecute(fn func()) (err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("panic recovered: %v", r)
        }
    }()
    
    fn()
    return nil
}

func main() {
    // Пример 1: С восстановлением
    riskyOperation()
    fmt.Println("Program continues...")
    
    // Пример 2: С возвратом ошибки
    err := safeExecute(func() {
        panic("test panic")
    })
    
    if err != nil {
        fmt.Println("Error:", err)
    }
}
```

---

## ✅ Итоговая структура проекта

```
devops-scripts/
├── cmd/
│   ├── monitor/
│   │   └── main.go
│   ├── backup/
│   │   └── main.go
│   └── healthcheck/
│       └── main.go
├── pkg/
│   ├── config/
│   │   └── config.go
│   ├── logger/
│   │   └── logger.go
│   ├── metrics/
│   │   └── prometheus.go
│   └── alerts/
│       └── slack.go
├── internal/
│   ├── system/
│   │   └── resources.go
│   └── services/
│       └── checker.go
├── configs/
│   ├── config.yaml
│   └── config.json
├── scripts/
│   └── deploy.sh
├── deployments/
│   ├── docker-compose.yml
│   └── kubernetes/
│       ├── deployment.yaml
│       └── service.yaml
├── tests/
│   └── integration_test.go
├── .env.example
├── .gitignore
├── go.mod
├── go.sum
├── Makefile
├── Dockerfile
└── README.md
```

Все 50 скриптов готовы к использованию! 🎉Do(req)
    if err != nil {
        fmt.Println("✗ Ошибка запроса:", err)
        return
    }
    defer resp.Body.Close()
    
    body, _ := io.ReadAll(resp.Body)
    var job JenkinsJob
    json.Unmarshal(body, &job)
    
    fmt.Printf("Job %s: %s\n", jobName, job.Result)
}

func main() {
    checkJenkinsJob("http://jenkins.local", "deploy-prod", "admin", "token")
}
```

### 29. Автоматический деплой через GitLab API
```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "net/http"
)

func triggerGitLabPipeline(projectID int, token, ref string) {
    url := fmt.Sprintf("https://gitlab.com/api/v4/projects/%d/pipeline", projectID)
    
    payload := map[string]string{"ref": ref}
    jsonData, _ := json.Marshal(payload)
    
    client := &http.Client{}
    req, _ := http.NewRequest("POST", url, bytes.NewBuffer(jsonData))
    req.Header.Set("PRIVATE-TOKEN", token)
    req.Header.Set("Content-Type", "application/json")
    
    resp, err := client.