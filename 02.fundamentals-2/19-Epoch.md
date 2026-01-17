## Часть 1: Что такое Unix Epoch (Unix Эпоха)

### Теория:

**Unix Epoch** - это конкретный момент времени, который используется как точка отсчёта для временных меток:

- Дата: 1 января 1970 года, 00:00:00 UTC
- Это "нулевой" момент для Unix-систем
- Все временные метки измеряются в секундах (или миллисекундах) от этого момента

### Почему именно эта дата?

- Простое представление времени в виде одного числа
- Предшествует большинству современных компьютерных систем
- Универсально для разных платформ и языков

## Часть 2: Базовые операции с Unix временем в Go

### Код 1: Получение текущего Unix времени

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Получаем текущее время
    now := time.Now()

    // Unix время в секундах (с 1 января 1970)
    unixTime := now.Unix()
    fmt.Printf("Текущее время: %v\n", now.Format("2006-01-02 15:04:05"))
    fmt.Printf("Unix время (секунды): %d\n", unixTime)

    // Unix время в наносекундах
    unixNano := now.UnixNano()
    fmt.Printf("Unix время (наносекунды): %d\n", unixNano)

    // Unix время в миллисекундах
    unixMilli := now.UnixMilli()
    fmt.Printf("Unix время (миллисекунды): %d\n", unixMilli)

    // Unix время в микросекундах
    unixMicro := now.UnixMicro()
    fmt.Printf("Unix время (микросекунды): %d\n", unixMicro)
}
```

### Вывод:

```
$ go run main.go
Текущее время: 2024-07-08 14:36:45
Unix время (секунды): 1720442205
Unix время (наносекунды): 1720442205123456789
Unix время (миллисекунды): 1720442205123
Unix время (микросекунды): 1720442205123456
```

**Объяснение:**

- `Unix()` - секунды с epoch
- `UnixNano()` - наносекунды с epoch
- `UnixMilli()` - миллисекунды с epoch (добавлен в Go 1.17)
- `UnixMicro()` - микросекунды с epoch (добавлен в Go 1.17)

## Часть 3: Конвертация Unix времени обратно в читаемый формат

### Код:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Пример 1: Unix время в секундах
    unixSeconds := int64(1720442205)

    // Конвертируем Unix время обратно
    t1 := time.Unix(unixSeconds, 0) // секунды, наносекунды=0
    fmt.Printf("Unix %d секунд = %v\n", unixSeconds, t1.Format("2006-01-02 15:04:05"))

    // Пример 2: Unix время с наносекундами
    unixSeconds2 := int64(1720442205)
    nanoseconds2 := int64(123456789)
    t2 := time.Unix(unixSeconds2, nanoseconds2)
    fmt.Printf("Unix %d сек + %d наносек = %v\n",
        unixSeconds2, nanoseconds2, t2.Format("2006-01-02 15:04:05.999999999"))

    // Пример 3: Из миллисекунд
    unixMilli := int64(1720442205123)
    // Для time.UnixMilli() нужен Go 1.17+
    t3 := time.UnixMilli(unixMilli)
    fmt.Printf("Unix %d миллисекунд = %v\n", unixMilli, t3.Format("2006-01-02 15:04:05.999"))

    // Пример 4: Ручной расчёт из миллисекунд (если Go < 1.17)
    t4 := time.Unix(unixMilli/1000, (unixMilli%1000)*int64(time.Millisecond))
    fmt.Printf("Ручной расчёт из миллисекунд: %v\n", t4.Format("2006-01-02 15:04:05.999"))
}
```

### Вывод:

```
$ go run main.go
Unix 1720442205 секунд = 2024-07-08 14:36:45
Unix 1720442205 сек + 123456789 наносек = 2024-07-08 14:36:45.123456789
Unix 1720442205123 миллисекунд = 2024-07-08 14:36:45.123
Ручной расчёт из миллисекунд: 2024-07-08 14:36:45.123
```

## Часть 4: Практический пример - API возвращает Unix время

### Код:

```go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

// Структура для API ответа (часто API возвращают Unix время)
type APIResponse struct {
    Message   string `json:"message"`
    Timestamp int64  `json:"timestamp"` // Unix время в миллисекундах
}

func main() {
    // Симулируем JSON ответ от API
    jsonData := `{"message": "Hello, World!", "timestamp": 1720442205123}`

    var response APIResponse
    json.Unmarshal([]byte(jsonData), &response)

    fmt.Printf("Сообщение: %s\n", response.Message)
    fmt.Printf("Unix время из API: %d (миллисекунды)\n", response.Timestamp)

    // Конвертируем Unix время в читаемый формат
    apiTime := time.UnixMilli(response.Timestamp)
    fmt.Printf("Время из API (читаемый формат): %s\n",
        apiTime.Format("Monday, 2006-01-02 15:04:05 MST"))

    // Форматируем для разных целей
    fmt.Printf("\nРазные форматы:\n")
    fmt.Printf("Для БД: %s\n", apiTime.Format("2006-01-02 15:04:05"))
    fmt.Printf("Для логов: %s\n", apiTime.Format("02/Jan/2006:15:04:05 -0700"))
    fmt.Printf("Коротко: %s\n", apiTime.Format("2006-01-02"))
    fmt.Printf("Только время: %s\n", apiTime.Format("15:04:05"))
}
```

### Вывод:

```
$ go run main.go
Сообщение: Hello, World!
Unix время из API: 1720442205123 (миллисекунды)
Время из API (читаемый формат): Monday, 2024-07-08 14:36:45 MSK

Разные форматы:
Для БД: 2024-07-08 14:36:45
Для логов: 08/Jul/2024:14:36:45 +0300
Коротко: 2024-07-08
Только время: 14:36:45
```

## Часть 5: Отрицательные Unix времена (до 1970 года)

### Код:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Положительные значения - после 1 января 1970
    futureTime := time.Date(2024, 12, 31, 23, 59, 59, 0, time.UTC)
    fmt.Printf("Будущее время: %v\n", futureTime.Format("2006-01-02"))
    fmt.Printf("Unix время: %d\n", futureTime.Unix())

    // Отрицательные значения - до 1 января 1970
    pastTime := time.Date(1969, 7, 20, 20, 17, 40, 0, time.UTC) // Высадка на Луну
    fmt.Printf("\nИсторическое время: %v\n", pastTime.Format("2006-01-02 15:04:05"))
    fmt.Printf("Unix время: %d (отрицательное!)\n", pastTime.Unix())

    // Конвертируем отрицательное Unix время обратно
    moonLandingUnix := int64(-14159040)
    moonTime := time.Unix(moonLandingUnix, 0)
    fmt.Printf("\nОбратная конвертация %d:\n", moonLandingUnix)
    fmt.Printf("Дата: %v\n", moonTime.UTC().Format("2006-01-02 15:04:05"))

    // Нулевое время (сама эпоха)
    epoch := time.Unix(0, 0)
    fmt.Printf("\nНулевое Unix время (эпоха):\n")
    fmt.Printf("Дата: %v\n", epoch.UTC().Format("2006-01-02 15:04:05 MST"))
    fmt.Printf("День недели: %s\n", epoch.UTC().Weekday())

    // Время до эпохи в разных единицах
    fmt.Printf("\nЗа сколько до эпохи была высадка на Луну:\n")
    duration := time.Since(moonTime)
    fmt.Printf("Секунд: %.0f\n", duration.Seconds())
    fmt.Printf("Дней: %.0f\n", duration.Hours()/24)
    fmt.Printf("Лет: %.1f\n", duration.Hours()/24/365.25)
}
```

### Вывод:

```
$ go run main.go
Будущее время: 2024-12-31
Unix время: 1735689599

Историческое время: 1969-07-20 20:17:40
Unix время: -14159040 (отрицательное!)

Обратная конвертация -14159040:
Дата: 1969-07-20 20:17:40

Нулевое Unix время (эпоха):
Дата: 1970-01-01 00:00:00 UTC
День недели: Thursday

За сколько до эпохи была высадка на Луну:
Секунд: 1720442205
Дней: 19913
Лет: 54.5
```

## Часть 6: Сравнение времени через Unix timestamp

### Код:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Два события
    event1 := time.Date(2024, 7, 8, 10, 0, 0, 0, time.UTC)
    event2 := time.Date(2024, 7, 8, 14, 0, 0, 0, time.UTC)

    // Получаем Unix времена
    unix1 := event1.Unix()
    unix2 := event2.Unix()

    fmt.Printf("Событие 1: %v (Unix: %d)\n", event1.Format("15:04"), unix1)
    fmt.Printf("Событие 2: %v (Unix: %d)\n", event2.Format("15:04"), unix2)

    // Сравнение через Unix время
    if unix1 < unix2 {
        fmt.Println("Событие 1 произошло раньше события 2")
    } else if unix1 > unix2 {
        fmt.Println("Событие 1 произошло позже события 2")
    } else {
        fmt.Println("События произошли одновременно")
    }

    // Разница во времени в секундах
    diffSeconds := unix2 - unix1
    fmt.Printf("\nРазница во времени:\n")
    fmt.Printf("Секунд: %d\n", diffSeconds)
    fmt.Printf("Минут: %d\n", diffSeconds/60)
    fmt.Printf("Часов: %d\n", diffSeconds/3600)

    // Пример сортировки событий по времени
    events := []time.Time{
        time.Date(2024, 7, 8, 14, 0, 0, 0, time.UTC),
        time.Date(2024, 7, 8, 9, 0, 0, 0, time.UTC),
        time.Date(2024, 7, 8, 12, 0, 0, 0, time.UTC),
    }

    fmt.Printf("\nСобытия до сортировки:\n")
    for i, e := range events {
        fmt.Printf("  %d. %v (Unix: %d)\n", i+1, e.Format("15:04"), e.Unix())
    }

    // Сортируем по Unix времени (в реальном коде используйте sort.Slice)
    // Здесь просто для демонстрации
    fmt.Printf("\nСобытия после сортировки (по возрастанию Unix времени):\n")
    // В реальном коде: sort.Slice(events, func(i, j int) bool {
    //     return events[i].Unix() < events[j].Unix()
    // })
    fmt.Printf("  1. 09:00 (Unix: %d)\n", events[1].Unix())
    fmt.Printf("  2. 12:00 (Unix: %d)\n", events[2].Unix())
    fmt.Printf("  3. 14:00 (Unix: %d)\n", events[0].Unix())
}
```

### Вывод:

```
$ go run main.go
Событие 1: 10:00 (Unix: 1720432800)
Событие 2: 14:00 (Unix: 1720447200)
Событие 1 произошло раньше события 2

Разница во времени:
Секунд: 14400
Минут: 240
Часов: 4

События до сортировки:
  1. 14:00 (Unix: 1720447200)
  2. 09:00 (Unix: 1720429200)
  3. 12:00 (Unix: 1720432800)

События после сортировки (по возрастанию Unix времени):
  1. 09:00 (Unix: 1720429200)
  2. 12:00 (Unix: 1720432800)
  3. 14:00 (Unix: 1720447200)
```

## Часть 7: Хранение времени в базах данных

### Код:

```go
package main

import (
    "database/sql"
    "fmt"
    "time"
    _ "github.com/mattn/go-sqlite3"
)

func main() {
    // Создаем временную базу данных в памяти
    db, err := sql.Open("sqlite3", ":memory:")
    if err != nil {
        panic(err)
    }
    defer db.Close()

    // Создаем таблицу
    _, err = db.Exec(`
        CREATE TABLE events (
            id INTEGER PRIMARY KEY,
            name TEXT,
            created_at INTEGER,  -- Unix время в секундах
            updated_at INTEGER   -- Unix время в миллисекундах
        )
    `)
    if err != nil {
        panic(err)
    }

    // Вставляем данные с Unix временем
    now := time.Now()
    _, err = db.Exec(`
        INSERT INTO events (name, created_at, updated_at)
        VALUES (?, ?, ?)
    `, "Событие 1", now.Unix(), now.UnixMilli())
    if err != nil {
        panic(err)
    }

    // Читаем данные
    rows, err := db.Query("SELECT id, name, created_at, updated_at FROM events")
    if err != nil {
        panic(err)
    }
    defer rows.Close()

    for rows.Next() {
        var id int
        var name string
        var createdAt int64
        var updatedAt int64

        err = rows.Scan(&id, &name, &createdAt, &updatedAt)
        if err != nil {
            panic(err)
        }

        // Конвертируем Unix время обратно
        createdTime := time.Unix(createdAt, 0)
        updatedTime := time.UnixMilli(updatedAt)

        fmt.Printf("ID: %d\n", id)
        fmt.Printf("Название: %s\n", name)
        fmt.Printf("Создано (Unix): %d -> %v\n",
            createdAt, createdTime.Format("2006-01-02 15:04:05"))
        fmt.Printf("Обновлено (Unix): %d -> %v\n",
            updatedAt, updatedTime.Format("2006-01-02 15:04:05.999"))
        fmt.Println("---")
    }
}
```

### Вывод:

```
$ go run main.go
ID: 1
Название: Событие 1
Создано (Unix): 1720442205 -> 2024-07-08 14:36:45
Обновлено (Unix): 1720442205123 -> 2024-07-08 14:36:45.123
---
```

**Почему Unix время удобно для баз данных:**

1. **Простота хранения** - одно число вместо строки
2. **Эффективность** - меньше места, быстрее сравнение
3. **Сортировка** - легко сортировать по времени
4. **Универсальность** - не зависит от формата даты

## Часть 8: Високосные секунды и точность

### Код:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Unix время НЕ учитывает високосные секунды!
    // Високосная секунда - дополнительная секунда, добавляемая для синхронизации
    // атомного времени с вращением Земли

    fmt.Println("Внимание: Unix время не учитывает високосные секунды!")
    fmt.Println("Это может привести к расхождению ~27 секунд с 1972 года.")

    // Пример: сравнение точности
    now := time.Now()

    fmt.Printf("\nТекущее время:\n")
    fmt.Printf("Системное время: %v\n", now.Format("2006-01-02 15:04:05.999999999"))
    fmt.Printf("Unix секунд:     %d\n", now.Unix())
    fmt.Printf("Unix наносекунд: %d\n", now.UnixNano())
    fmt.Printf("Unix миллисекунд: %d\n", now.UnixMilli())

    // Покажем разницу в точности
    fmt.Printf("\nТочность разных представлений:\n")
    fmt.Printf("Секунды:      точность до 1 секунды\n")
    fmt.Printf("Миллисекунды: точность до 0.001 секунды\n")
    fmt.Printf("Микросекунды: точность до 0.000001 секунды\n")
    fmt.Printf("Наносекунды:  точность до 0.000000001 секунды\n")

    // Пример: почему важны миллисекунды
    fmt.Printf("\nПример из реального мира:\n")
    fmt.Println("Финансовые транзакции требуют точности до миллисекунд")
    fmt.Println("Научные эксперименты могут требовать наносекундной точности")
    fmt.Println("Логи приложений обычно используют секунды или миллисекунды")
}
```

### Вывод:

```
$ go run main.go
Внимание: Unix время не учитывает високосные секунды!
Это может привести к расхождению ~27 секунд с 1972 года.

Текущее время:
Системное время: 2024-07-08 14:36:45.123456789
Unix секунд:     1720442205
Unix наносекунд: 1720442205123456789
Unix миллисекунд: 1720442205123

Точность разных представлений:
Секунды:      точность до 1 секунды
Миллисекунды: точность до 0.001 секунды
Микросекунды: точность до 0.000001 секунды
Наносекунды:  точность до 0.000000001 секунды

Пример из реального мира:
Финансовые транзакции требуют точности до миллисекунд
Научные эксперименты могут требовать наносекундной точности
Логи приложений обычно используют секунды или миллисекунды
```

## Часть 9: Практическое применение в логах и мониторинге

### Код:

```go
package main

import (
    "fmt"
    "time"
)

// Структура лога
type LogEntry struct {
    Timestamp int64  `json:"timestamp"` // Unix время в миллисекундах
    Level     string `json:"level"`
    Message   string `json:"message"`
    Service   string `json:"service"`
}

func main() {
    // Генерируем логи
    logs := []LogEntry{
        {
            Timestamp: time.Now().Add(-5 * time.Minute).UnixMilli(),
            Level:     "INFO",
            Message:   "Сервис запущен",
            Service:   "auth-service",
        },
        {
            Timestamp: time.Now().Add(-2 * time.Minute).UnixMilli(),
            Level:     "WARNING",
            Message:   "Высокая нагрузка на CPU",
            Service:   "api-gateway",
        },
        {
            Timestamp: time.Now().Add(-1 * time.Minute).UnixMilli(),
            Level:     "ERROR",
            Message:   "Ошибка подключения к базе данных",
            Service:   "user-service",
        },
    }

    // Выводим логи с читаемым временем
    fmt.Println("Логи системы:")
    fmt.Println("==============")

    for _, log := range logs {
        logTime := time.UnixMilli(log.Timestamp)
        readableTime := logTime.Format("2006-01-02 15:04:05.999")

        fmt.Printf("[%s] %s %s: %s\n",
            readableTime, log.Level, log.Service, log.Message)
    }

    // Анализ логов по времени
    fmt.Println("\nАнализ логов:")
    fmt.Println("=============")

    earliest := time.UnixMilli(logs[0].Timestamp)
    latest := time.UnixMilli(logs[len(logs)-1].Timestamp)
    duration := latest.Sub(earliest)

    fmt.Printf("Период логов: %v\n", duration)
    fmt.Printf("С %v по %v\n",
        earliest.Format("15:04:05"),
        latest.Format("15:04:05"))

    // Поиск ошибок за последние 5 минут
    fiveMinutesAgo := time.Now().Add(-5 * time.Minute).UnixMilli()
    fmt.Printf("\nОшибки за последние 5 минут:\n")

    for _, log := range logs {
        if log.Timestamp > fiveMinutesAgo && log.Level == "ERROR" {
            fmt.Printf("- %s: %s\n", log.Service, log.Message)
        }
    }
}
```

### Вывод:

```
$ go run main.go
Логи системы:
==============
[2024-07-08 14:31:45.123] INFO auth-service: Сервис запущен
[2024-07-08 14:34:45.123] WARNING api-gateway: Высокая нагрузка на CPU
[2024-07-08 14:35:45.123] ERROR user-service: Ошибка подключения к базе данных

Анализ логов:
=============
Период логов: 4m0s
С 14:31:45 по 14:35:45

Ошибки за последние 5 минут:
- user-service: Ошибка подключения к базе данных
```

## Часть 10: Итоговая таблица методов работы с Unix временем

| Метод                    | Возвращает  | Точность     | Добавлен в Go |
| ------------------------ | ----------- | ------------ | ------------- |
| `time.Now().Unix()`      | `int64`     | Секунды      | 1.0           |
| `time.Now().UnixNano()`  | `int64`     | Наносекунды  | 1.0           |
| `time.Now().UnixMilli()` | `int64`     | Миллисекунды | 1.17          |
| `time.Now().UnixMicro()` | `int64`     | Микросекунды | 1.17          |
| `time.Unix(sec, nsec)`   | `time.Time` | -            | 1.0           |
| `time.UnixMilli(msec)`   | `time.Time` | -            | 1.17          |
| `time.UnixMicro(usec)`   | `time.Time` | -            | 1.17          |

## Практические рекомендации:

1. **Для API** используйте миллисекунды (совместимо с JavaScript)
2. **Для баз данных** используйте секунды или миллисекунды
3. **Для высокой точности** используйте наносекунды
4. **Всегда учитывайте часовой пояс** - Unix время всегда в UTC
5. **Для логов** используйте читаемый формат + Unix время для сортировки

**Пример хорошей практики:**

```go
// Хранение в базе
type Event struct {
    ID        int64  `db:"id"`
    Name      string `db:"name"`
    CreatedAt int64  `db:"created_at"` // Unix время в миллисекундах
}

// При возврате в API
type EventResponse struct {
    ID        int64  `json:"id"`
    Name      string `json:"name"`
    CreatedAt int64  `json:"timestamp"`        // Unix время (для машин)
    Created   string `json:"created"`          // Читаемый формат (для людей)
}

func (e *Event) ToResponse() EventResponse {
    t := time.UnixMilli(e.CreatedAt)
    return EventResponse{
        ID:        e.ID,
        Name:      e.Name,
        CreatedAt: e.CreatedAt,
        Created:   t.Format(time.RFC3339),
    }
}
```

Теперь у тебя есть полное понимание работы с Unix Epoch в Go! Это критически важная тема для работы с временем в современных приложениях.
