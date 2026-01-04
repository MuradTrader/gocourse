### **1. Простое логирование с пакетом `log`**

**Код:**

```go
package main

import "log"

func main() {
    log.Println("This is a log message")
}
```

**Объяснение:**

- Импортируем стандартный пакет `log`.
- `log.Println` выводит сообщение в стандартный вывод (консоль) с добавлением временной метки по умолчанию.
- Функция автоматически добавляет символ новой строки.

**Вывод в консоли:**

```
2023/10/05 10:30:45 This is a log message
```

---

### **2. Добавление префикса к логам**

**Код:**

```go
package main

import "log"

func main() {
    log.SetPrefix("INFO: ")
    log.Println("This is an info message")
}
```

**Объяснение:**

- `log.SetPrefix("INFO: ")` устанавливает строку, которая будет добавляться в начало каждого сообщения лога.
- Префикс отображается перед временной меткой.

**Вывод в консоли:**

```
INFO: 2023/10/05 10:30:45 This is an info message
```

---

### **3. Использование флагов для формата лога**

**Код:**

```go
package main

import "log"

func main() {
    log.SetFlags(log.Ldate)
    log.Println("This is a log message with only date information")
}
```

**Объяснение:**

- `log.SetFlags(log.Ldate)` изменяет формат временной метки, оставляя только дату.
- Пакет `log` предоставляет константы для настройки: `Ldate`, `Ltime`, `Lshortfile` и другие.

**Вывод в консоли:**

```
2023/10/05 This is a log message with only date information
```

**Добавим несколько флагов:**

**Код:**

```go
package main

import "log"

func main() {
    log.SetFlags(log.Ldate | log.Ltime | log.Lshortfile)
    log.Println("This is a log message with date, time, and file name")
}
```

**Объяснение:**

- Флаги комбинируются с помощью оператора `|` (побитовое ИЛИ).
- `log.Lshortfile` добавляет имя файла и номер строки, где был вызван лог.

**Вывод в консоли:**

```
2023/10/05 10:30:45 logging.go:10: This is a log message with date, time, and file name
```

---

### **4. Создание пользовательских логгеров с уровнями (info, warn, error)**

**Код:**

```go
package main

import (
    "log"
    "os"
)

var (
    infoLogger  *log.Logger
    warnLogger  *log.Logger
    errorLogger *log.Logger
)

func init() {
    infoLogger = log.New(os.Stdout, "INFO: ", log.Ldate|log.Ltime|log.Lshortfile)
    warnLogger = log.New(os.Stdout, "WARN: ", log.Ldate|log.Ltime|log.Lshortfile)
    errorLogger = log.New(os.Stdout, "ERROR: ", log.Ldate|log.Ltime|log.Lshortfile)
}

func main() {
    infoLogger.Println("This is an info message")
    warnLogger.Println("This is a warning message")
    errorLogger.Println("This is an error message")
}
```

**Объяснение:**

- Мы объявляем три переменные типа `*log.Logger` для каждого уровня логирования.
- В функции `init()` (которая выполняется автоматически при старте программы) инициализируем логгеры с помощью `log.New`.
- `log.New` принимает три аргумента:
  1. `io.Writer` (куда писать логи) — `os.Stdout` (консоль).
  2. `prefix` (префикс для каждого сообщения).
  3. `flags` (флаги формата).
- В `main()` вызываем `Println` у каждого логгера.

**Вывод в консоли:**

```
INFO: 2023/10/05 10:30:45 logging.go:20: This is an info message
WARN: 2023/10/05 10:30:45 logging.go:21: This is a warning message
ERROR: 2023/10/05 10:30:45 logging.go:22: This is an error message
```

---

### **5. Запись логов в файл**

**Код:**

```go
package main

import (
    "log"
    "os"
)

func main() {
    // Открываем файл для записи логов. Если файл не существует, он будет создан.
    file, err := os.OpenFile("app.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
    if err != nil {
        log.Fatalf("Failed to open log file: %v", err)
    }
    defer file.Close() // Закрываем файл при завершении функции.

    // Создаем логгер, который пишет в файл.
    fileLogger := log.New(file, "INFO: ", log.Ldate|log.Ltime|log.Lshortfile)

    fileLogger.Println("This is a log message written to a file")
}
```

**Объяснение:**

- `os.OpenFile` открывает (или создает) файл `app.log` с флагами:
  - `os.O_CREATE`: создать файл, если не существует.
  - `os.O_WRONLY`: открыть только для записи.
  - `os.O_APPEND`: добавлять новые данные в конец файла.
- `0666` — права доступа к файлу (чтение и запись для всех).
- Обрабатываем ошибку: если не удалось открыть файл, выводим сообщение и завершаем программу с помощью `log.Fatalf`.
- `defer file.Close()` гарантирует закрытие файла после завершения функции.
- Создаем новый логгер `fileLogger`, который пишет в файл вместо консоли.

**Содержимое файла `app.log` после запуска:**

```
INFO: 2023/10/05 10:30:45 logging.go:20: This is a log message written to a file
```

---

### **6. Использование стороннего пакета `logrus`**

**Предварительно инициализируем модуль Go и устанавливаем `logrus`:**

```bash
go mod init go-course
go get github.com/sirupsen/logrus
```

**Код (файл `logrus_logging.go`):**

```go
package main

import (
    "github.com/sirupsen/logrus"
)

func main() {
    log := logrus.New()

    // Устанавливаем уровень логирования.
    log.SetLevel(logrus.InfoLevel)

    // Устанавливаем формат вывода в JSON.
    log.SetFormatter(&logrus.JSONFormatter{})

    log.Info("This is an info message")
    log.Warn("This is a warning message")
    log.Error("This is an error message")

    // Логирование с дополнительными полями.
    log.WithFields(logrus.Fields{
        "username": "John Doe",
        "method":   "GET",
    }).Info("User logged in")
}
```

**Объяснение:**

- Создаем экземпляр логгера `logrus.New()`.
- Устанавливаем уровень логирования (сообщения ниже этого уровня не выводятся).
- Устанавливаем JSON-форматер для структурированного вывода.
- Методы `Info`, `Warn`, `Error` выводят сообщения соответствующего уровня.
- `WithFields` добавляет к логу дополнительные поля в виде пар ключ-значение.

**Вывод в консоли:**

```json
{"level":"info","msg":"This is an info message","time":"2023-10-05T10:30:45+03:00"}
{"level":"warning","msg":"This is a warning message","time":"2023-10-05T10:30:45+03:00"}
{"level":"error","msg":"This is an error message","time":"2023-10-05T10:30:45+03:00"}
{"level":"info","method":"GET","msg":"User logged in","time":"2023-10-05T10:30:45+03:00","username":"John Doe"}
```

---

### **7. Использование стороннего пакета `zap`**

**Устанавливаем `zap`:**

```bash
go get go.uber.org/zap
```

**Код (файл `zap_log.go`):**

```go
package main

import (
    "log"
    "go.uber.org/zap"
)

func main() {
    // Инициализируем логгер в конфигурации для production.
    logger, err := zap.NewProduction()
    if err != nil {
        log.Fatal("Error initializing zap logger:", err)
    }
    defer logger.Sync() // Сбрасываем буферы перед выходом.

    logger.Info("This is an info message")
    logger.Warn("This is a warning message")
    logger.Error("This is an error message")

    // Логирование с дополнительными полями.
    logger.Info("User logged in",
        zap.String("username", "John Doe"),
        zap.String("method", "GET"),
    )
}
```

**Объяснение:**

- `zap.NewProduction()` создает логгер, настроенный для использования в production (выводит логи в JSON).
- Логгер возвращает ошибку, которую необходимо обработать.
- `defer logger.Sync()` гарантирует, что все буферизованные логи будут записаны перед выходом.
- Методы `Info`, `Warn`, `Error` аналогичны `logrus`.
- Дополнительные поля добавляются с помощью функций `zap.String`, `zap.Int` и т.д.

**Вывод в консоли:**

```json
{"level":"info","ts":1696491045.1234567,"caller":"zap_log.go:14","msg":"This is an info message"}
{"level":"warn","ts":1696491045.1234567,"caller":"zap_log.go:15","msg":"This is a warning message"}
{"level":"error","ts":1696491045.1234567,"caller":"zap_log.go:16","msg":"This is an error message"}
{"level":"info","ts":1696491045.1234567,"caller":"zap_log.go:19","msg":"User logged in","username":"John Doe","method":"GET"}
```

---

### **Итоговые выводы из лекции:**

1. **Стандартный пакет `log`** предоставляет базовые возможности логирования.
2. **Кастомные логгеры** можно создавать через `log.New` для разделения уровней логирования.
3. **Логи можно писать в файлы**, управляя их открытием, закрытием и форматом.
4. **Сторонние пакеты** (`logrus`, `zap`) предлагают расширенные возможности: уровни логирования, структурированный вывод (JSON), производительность.
5. **Лучшие практики**:
   - Использовать уровни логирования (Debug, Info, Warn, Error).
   - Применять структурированное логирование (JSON) для удобства анализа.
   - Добавлять контекст (поля) к логам.
   - Настраивать ротацию логов для экономии места на диске.
   - Использовать централизованные системы сбора логов (ELK, Grafana, Splunk).
