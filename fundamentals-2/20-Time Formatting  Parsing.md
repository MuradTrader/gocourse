## Часть 1: Основы форматирования времени в Go

### Ключевая концепция: эталонная дата

Go использует специфическую дату как шаблон для форматирования:
**Monday, January 2, 15:04:05 MST 2006**

Это можно запомнить как **"01/02 03:04:05PM '06 -0700"**:

- 01 (January) - месяц
- 02 - день
- 03 (или 15) - час
- 04 - минута
- 05 - секунда
- 2006 - год
- MST - часовой пояс

### Код с различными форматами:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()

    fmt.Println("=== Различные форматы времени ===")
    fmt.Println()

    // Предопределенные форматы
    fmt.Println("Предопределенные форматы:")
    fmt.Printf("RFC3339:      %s\n", now.Format(time.RFC3339))
    fmt.Printf("RFC822:       %s\n", now.Format(time.RFC822))
    fmt.Printf("RFC1123:      %s\n", now.Format(time.RFC1123))
    fmt.Printf("ANSIC:        %s\n", now.Format(time.ANSIC))
    fmt.Printf("UnixDate:     %s\n", now.Format(time.UnixDate))

    fmt.Println("\nПользовательские форматы:")

    // Дата
    fmt.Printf("Только дата:                %s\n", now.Format("2006-01-02"))
    fmt.Printf("Дата с точками:             %s\n", now.Format("02.01.2006"))
    fmt.Printf("Дата со словами:            %s\n", now.Format("Monday, January 2, 2006"))
    fmt.Printf("Короткая дата:              %s\n", now.Format("Jan 02, 2006"))

    // Время
    fmt.Printf("\nТолько время (24ч):         %s\n", now.Format("15:04:05"))
    fmt.Printf("Время (12ч):                %s\n", now.Format("3:04:05 PM"))
    fmt.Printf("С миллисекундами:           %s\n", now.Format("15:04:05.000"))

    // Комбинированные
    fmt.Printf("\nДата и время:               %s\n", now.Format("2006-01-02 15:04:05"))
    fmt.Printf("Для логов:                  %s\n", now.Format("2006/01/02 15:04:05"))
    fmt.Printf("Для файлов:                 %s\n", now.Format("20060102_150405"))
    fmt.Printf("Полный формат:              %s\n", now.Format("Monday, January 2, 2006 at 3:04:05 PM MST"))

    // Часовой пояс
    fmt.Printf("\nС часовым поясом:           %s\n", now.Format("2006-01-02 15:04:05 -07:00"))
    fmt.Printf("Сокращенный часовой пояс:   %s\n", now.Format("2006-01-02 15:04:05 MST"))

    // Примеры для разных стран
    fmt.Println("\nМеждународные форматы:")
    fmt.Printf("Европейский:                %s\n", now.Format("02/01/2006 15:04"))
    fmt.Printf("Американский:               %s\n", now.Format("01/02/2006 03:04 PM"))
    fmt.Printf("Японский:                   %s\n", now.Format("2006年01月02日 15時04分"))
    fmt.Printf("ISO 8601:                   %s\n", now.Format("2006-01-02T15:04:05Z07:00"))
}
```

### Вывод:

```
$ go run main.go
=== Различные форматы времени ===

Предопределенные форматы:
RFC3339:      2024-07-08T15:04:05+03:00
RFC822:       08 Jul 24 15:04 +0300
RFC1123:      Mon, 08 Jul 2024 15:04:05 +0300
ANSIC:        Mon Jan  8 15:04:05 2024
UnixDate:     Mon Jan  8 15:04:05 +0300 2024

Пользовательские форматы:
Только дата:                2024-07-08
Дата с точками:             08.07.2024
Дата со словами:            Monday, July 8, 2024
Короткая дата:              Jul 08, 2024

Только время (24ч):         15:04:05
Время (12ч):                3:04:05 PM
С миллисекундами:           15:04:05.123

Дата и время:               2024-07-08 15:04:05
Для логов:                  2024/07/08 15:04:05
Для файлов:                 20240708_150405
Полный формат:              Monday, July 8, 2024 at 3:04:05 PM +0300

С часовым поясом:           2024-07-08 15:04:05 +03:00
Сокращенный часовой пояс:   2024-07-08 15:04:05 +0300

Международные форматы:
Европейский:                08/07/2024 15:04
Американский:               07/08/2024 03:04 PM
Японский:                   2024年07月08日 15時04分
ISO 8601:                   2024-07-08T15:04:05+03:00
```

## Часть 2: Парсинг времени (разбор строк)

### Код с примерами парсинга:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("=== Парсинг времени из строк ===")

    // Пример 1: ISO 8601 формат
    layoutISO := "2006-01-02T15:04:05Z07:00"
    strISO := "2024-07-04T14:30:18Z"

    t1, err := time.Parse(layoutISO, strISO)
    if err != nil {
        fmt.Printf("Ошибка парсинга ISO: %v\n", err)
    } else {
        fmt.Printf("Пример 1 (ISO 8601):\n")
        fmt.Printf("  Строка: %s\n", strISO)
        fmt.Printf("  Результат: %v\n", t1)
        fmt.Printf("  Форматировано: %s\n\n", t1.Format("January 2, 2006 at 15:04"))
    }

    // Пример 2: Пользовательский формат
    layoutCustom := "January 02, 2006 03:04 PM"
    strCustom := "July 03, 2024 03:18 PM"

    t2, err := time.Parse(layoutCustom, strCustom)
    if err != nil {
        fmt.Printf("Ошибка парсинга пользовательского формата: %v\n", err)
    } else {
        fmt.Printf("Пример 2 (пользовательский):\n")
        fmt.Printf("  Строка: %s\n", strCustom)
        fmt.Printf("  Результат: %v\n", t2)
        fmt.Printf("  В 24-часовом формате: %s\n\n", t2.Format("15:04"))
    }

    // Пример 3: Разные форматы дат
    formats := []struct {
        layout string
        value  string
        desc   string
    }{
        {"2006-01-02", "2024-12-31", "Простая дата"},
        {"02/01/2006", "31/12/2024", "Европейский формат"},
        {"01/02/2006", "12/31/2024", "Американский формат"},
        {"2006.01.02", "2024.12.31", "Дата с точками"},
        {"02-Jan-2006", "31-Dec-2024", "Сокращенный месяц"},
        {"Monday, January 2, 2006", "Tuesday, December 31, 2024", "Полная дата"},
    }

    fmt.Println("Пример 3: Разные форматы дат")
    for _, f := range formats {
        t, err := time.Parse(f.layout, f.value)
        if err != nil {
            fmt.Printf("  %s: ОШИБКА - %v\n", f.desc, err)
        } else {
            fmt.Printf("  %s: %v → %s\n", f.desc, f.value, t.Format("2006-01-02"))
        }
    }

    // Пример 4: Время с часовым поясом
    fmt.Println("\nПример 4: Время с часовыми поясами")
    timeWithZone := "2024-07-08 10:30:00 +02:00"
    layoutWithZone := "2006-01-02 15:04:05 -07:00"

    t3, err := time.Parse(layoutWithZone, timeWithZone)
    if err != nil {
        fmt.Printf("  Ошибка: %v\n", err)
    } else {
        fmt.Printf("  Строка: %s\n", timeWithZone)
        fmt.Printf("  Результат: %v\n", t3)
        fmt.Printf("  В UTC: %v\n", t3.UTC())
        fmt.Printf("  В локальном времени: %v\n", t3.Local())
    }
}
```

### Вывод:

```
$ go run main.go
=== Парсинг времени из строк ===

Пример 1 (ISO 8601):
  Строка: 2024-07-04T14:30:18Z
  Результат: 2024-07-04 14:30:18 +0000 UTC
  Форматировано: July 4, 2024 at 14:30

Пример 2 (пользовательский):
  Строка: July 03, 2024 03:18 PM
  Результат: 2024-07-03 15:18:00 +0000 UTC
  В 24-часовом формате: 15:18

Пример 3: Разные форматы дат
  Простая дата: 2024-12-31 → 2024-12-31
  Европейский формат: 31/12/2024 → 2024-12-31
  Американский формат: 12/31/2024 → 2024-12-31
  Дата с точками: 2024.12.31 → 2024-12-31
  Сокращенный месяц: 31-Dec-2024 → 2024-12-31
  Полная дата: Tuesday, December 31, 2024 → 2024-12-31

Пример 4: Время с часовыми поясами
  Строка: 2024-07-08 10:30:00 +02:00
  Результат: 2024-07-08 10:30:00 +0200 +0200
  В UTC: 2024-07-08 08:30:00 +0000 UTC
  В локальном времени: 2024-07-08 11:30:00 +0300 +0300
```

## Часть 3: Объединение Epoch, форматирования и парсинга

### Код:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("=== Полный цикл: Epoch → Форматирование → Парсинг ===")

    // 1. Получаем текущее время
    now := time.Now()
    fmt.Printf("1. Текущее время: %v\n", now)

    // 2. Конвертируем в Unix время
    unixTime := now.Unix()
    unixMilli := now.UnixMilli()
    fmt.Printf("2. Unix время:\n")
    fmt.Printf("   Секунды: %d\n", unixTime)
    fmt.Printf("   Миллисекунды: %d\n", unixMilli)

    // 3. Форматируем в разные форматы
    fmt.Printf("3. Форматирование:\n")
    fmt.Printf("   Для БД: %s\n", now.Format("2006-01-02 15:04:05"))
    fmt.Printf("   Для логов: %s\n", now.Format("2006/01/02 15:04:05.000"))
    fmt.Printf("   Для пользователя: %s\n", now.Format("Monday, January 2, 2006 at 3:04 PM"))
    fmt.Printf("   Для API (ISO 8601): %s\n", now.Format(time.RFC3339Nano))

    // 4. Конвертируем Unix время обратно
    fromUnix := time.Unix(unixTime, 0)
    fmt.Printf("4. Из Unix обратно: %v\n", fromUnix.Format("2006-01-02 15:04:05"))

    // 5. Парсим строку с временем
    apiResponse := `{"timestamp": "2024-07-08T15:30:45+03:00", "event": "user_login"}`
    timestampStr := "2024-07-08T15:30:45+03:00"

    parsedTime, err := time.Parse(time.RFC3339, timestampStr)
    if err != nil {
        fmt.Printf("5. Ошибка парсинга: %v\n", err)
    } else {
        fmt.Printf("5. Парсинг API ответа:\n")
        fmt.Printf("   Строка: %s\n", timestampStr)
        fmt.Printf("   Результат: %v\n", parsedTime)
        fmt.Printf("   Unix время: %d\n", parsedTime.Unix())

        // 6. Конвертируем в другой часовой пояс
        nyLoc, _ := time.LoadLocation("America/New_York")
        timeInNY := parsedTime.In(nyLoc)
        fmt.Printf("6. В Нью-Йорке: %s\n", timeInNY.Format("2006-01-02 15:04:05 MST"))

        // 7. Разница во времени
        diff := now.Sub(parsedTime)
        fmt.Printf("7. Разница с сейчас: %v\n", diff.Round(time.Second))
    }

    // 8. Работа с продолжительностями
    fmt.Println("\n8. Работа с продолжительностями:")
    duration, _ := time.ParseDuration("2h30m15s")
    futureTime := now.Add(duration)
    fmt.Printf("   Сейчас: %s\n", now.Format("15:04:05"))
    fmt.Printf("   +2h30m15s: %s\n", futureTime.Format("15:04:05"))

    // 9. Форматирование продолжительности
    fmt.Printf("   Продолжительность: %v\n", duration)
    fmt.Printf("   Часы: %.2f\n", duration.Hours())
    fmt.Printf("   Минуты: %.2f\n", duration.Minutes())
    fmt.Printf("   Секунды: %.2f\n", duration.Seconds())
}
```

### Вывод:

```
$ go run main.go
=== Полный цикл: Epoch → Форматирование → Парсинг ===

1. Текущее время: 2024-07-08 15:04:05.123456789 +03:00
2. Unix время:
   Секунды: 1720443845
   Миллисекунды: 1720443845123
3. Форматирование:
   Для БД: 2024-07-08 15:04:05
   Для логов: 2024/07/08 15:04:05.123
   Для пользователя: Monday, July 8, 2024 at 3:04 PM
   Для API (ISO 8601): 2024-07-08T15:04:05.123456789+03:00
4. Из Unix обратно: 2024-07-08 15:04:05
5. Парсинг API ответа:
   Строка: 2024-07-08T15:30:45+03:00
   Результат: 2024-07-08 15:30:45 +0300 +0300
   Unix время: 1720445445
6. В Нью-Йорке: 2024-07-08 08:30:45 EDT
7. Разница с сейчас: -26m39s
8. Работа с продолжительностями:
   Сейчас: 15:04:05
   +2h30m15s: 17:34:20
   Продолжительность: 2h30m15s
   Часы: 2.50
   Минуты: 150.25
   Секунды: 9015.00
```

## Часть 4: Обработка ошибок и лучшие практики

### Код:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("=== Обработка ошибок при парсинге ===")

    // Неправильные форматы дат
    testCases := []struct {
        name   string
        layout string
        value  string
    }{
        {"Неправильный месяц", "2006-01-02", "2024-13-01"},
        {"Неправильный день", "2006-01-02", "2024-02-30"},
        {"Несоответствие формата", "2006-01-02", "01/02/2024"},
        {"Неправильный час", "15:04:05", "25:61:61"},
        {"Пустая строка", "2006-01-02", ""},
        {"Смешанные разделители", "2006/01/02", "2024-01-02"},
    }

    for _, tc := range testCases {
        _, err := time.Parse(tc.layout, tc.value)
        if err != nil {
            fmt.Printf("✓ %s: Ошибка (ожидаемо) - %v\n", tc.name, err)
        } else {
            fmt.Printf("✗ %s: Нет ошибки (неожиданно)\n", tc.name)
        }
    }

    // Лучшие практики: безопасный парсинг
    fmt.Println("\n=== Безопасный парсинг с несколькими форматами ===")

    parseSafe := func(dateStr string) (time.Time, error) {
        // Пробуем несколько форматов
        layouts := []string{
            time.RFC3339,
            "2006-01-02T15:04:05",
            "2006-01-02 15:04:05",
            "02/01/2006 15:04",
            "January 2, 2006 3:04 PM",
            "2006-01-02",
        }

        for _, layout := range layouts {
            t, err := time.Parse(layout, dateStr)
            if err == nil {
                return t, nil
            }
        }
        return time.Time{}, fmt.Errorf("не удалось распознать формат даты: %s", dateStr)
    }

    testDates := []string{
        "2024-07-08T15:30:45+03:00",
        "2024-07-08 15:30:45",
        "08/07/2024 15:30",
        "July 8, 2024 3:30 PM",
        "2024-07-08",
    }

    for _, dateStr := range testDates {
        t, err := parseSafe(dateStr)
        if err != nil {
            fmt.Printf("Ошибка для '%s': %v\n", dateStr, err)
        } else {
            fmt.Printf("Успех: '%s' → %s\n", dateStr, t.Format("2006-01-02 15:04:05"))
        }
    }

    // Работа с временными зонами
    fmt.Println("\n=== Важность учета часовых поясов ===")

    // Без указания зоны
    t1, _ := time.Parse("2006-01-02 15:04:05", "2024-07-08 12:00:00")
    fmt.Printf("Без зоны (по умолчанию UTC): %v\n", t1)

    // С указанием зоны
    t2, _ := time.Parse("2006-01-02 15:04:05 -07:00", "2024-07-08 12:00:00 +03:00")
    fmt.Printf("С зоной +03:00: %v\n", t2)

    // Конвертация между зонами
    loc, _ := time.LoadLocation("America/Los_Angeles")
    t3 := t2.In(loc)
    fmt.Printf("В Лос-Анджелесе: %v\n", t3.Format("2006-01-02 15:04:05 MST"))

    // Разница
    fmt.Printf("Разница: %v часов\n", t2.Sub(t1).Hours())
}
```

### Вывод:

```
$ go run main.go
=== Обработка ошибок при парсинге ===
✓ Неправильный месяц: Ошибка (ожидаемо) - parsing time "2024-13-01": month out of range
✓ Неправильный день: Ошибка (ожидаемо) - parsing time "2024-02-30": day out of range
✓ Несоответствие формата: Ошибка (ожидаемо) - parsing time "01/02/2024" as "2006-01-02": cannot parse "/02/2024" as "-"
✓ Неправильный час: Ошибка (ожидаемо) - parsing time "25:61:61": hour out of range
✓ Пустая строка: Ошибка (ожидаемо) - parsing time "": extra text: ""
✓ Смешанные разделители: Ошибка (ожидаемо) - parsing time "2024-01-02" as "2006/01/02": cannot parse "-01-02" as "/"

=== Безопасный парсинг с несколькими форматами ===
Успех: '2024-07-08T15:30:45+03:00' → 2024-07-08 15:30:45
Успех: '2024-07-08 15:30:45' → 2024-07-08 15:30:45
Успех: '08/07/2024 15:30' → 2024-07-08 15:30:00
Успех: 'July 8, 2024 3:30 PM' → 2024-07-08 15:30:00
Успех: '2024-07-08' → 2024-07-08 00:00:00

=== Важность учета часовых поясов ===
Без зоны (по умолчанию UTC): 2024-07-08 12:00:00 +0000 UTC
С зоной +03:00: 2024-07-08 12:00:00 +0300 +0300
В Лос-Анджелесе: 2024-07-08 02:00:00 PDT
Разница: -3 часов
```

## Часть 5: Итоговая сводка по трём лекциям

### Таблица: Всё о времени в Go

| Концепция             | Методы/Функции                            | Пример                     | Когда использовать             |
| --------------------- | ----------------------------------------- | -------------------------- | ------------------------------ |
| **Получение времени** | `time.Now()`<br>`time.Date()`             | `now := time.Now()`        | Текущее время                  |
| **Unix Epoch**        | `Unix()`<br>`UnixNano()`<br>`UnixMilli()` | `unix := now.Unix()`       | Для хранения, API, БД          |
| **Форматирование**    | `Format()`                                | `now.Format("2006-01-02")` | Для вывода пользователю        |
| **Парсинг**           | `Parse()`<br>`ParseInLocation()`          | `t, _ := time.Parse(...)`  | Чтение ввода пользователя, API |
| **Продолжительность** | `time.Duration`<br>`Add()`<br>`Sub()`     | `dur := 2*time.Hour`       | Таймеры, интервалы             |
| **Сравнение**         | `Before()`<br>`After()`<br>`Equal()`      | `t1.Before(t2)`            | Сортировка, проверки           |
| **Часовые пояса**     | `LoadLocation()`<br>`In()`<br>`UTC()`     | `t.In(loc)`                | Международные приложения       |
| **Округление**        | `Round()`<br>`Truncate()`                 | `t.Round(time.Hour)`       | Агрегация данных               |
| **Компоненты**        | `Year()`<br>`Month()`<br>`Hour()`         | `year := now.Year()`       | Извлечение частей даты         |

### Лучшие практики:

1. **Всегда указывайте часовой пояс** при парсинге
2. **Используйте Unix время** для хранения в БД и передачи по API
3. **Обрабатывайте ошибки** при парсинге
4. **Используйте константы времени**: `time.Hour`, `time.Minute`
5. **Для API** используйте RFC3339 формат
6. **Кэшируйте** `time.LoadLocation()` для часто используемых зон

### Пример реального приложения:

```go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

// Модель для API
type Event struct {
    ID        int64  `json:"id"`
    Title     string `json:"title"`
    CreatedAt int64  `json:"created_at"` // Unix миллисекунды
    StartsAt  string `json:"starts_at"`  // RFC3339 для API
}

// Для отображения пользователю
type EventResponse struct {
    ID        int64  `json:"id"`
    Title     string `json:"title"`
    CreatedAt string `json:"created_at"` // Читаемый формат
    StartsAt  string `json:"starts_at"`  // В часовом поясе пользователя
    UnixTime  int64  `json:"unix_time"`  // Для клиентских вычислений
}

func (e *Event) ToResponse(userTimezone string) (*EventResponse, error) {
    // Загружаем часовой пояс пользователя
    loc, err := time.LoadLocation(userTimezone)
    if err != nil {
        return nil, err
    }

    // Парсим время начала
    startsAt, err := time.Parse(time.RFC3339, e.StartsAt)
    if err != nil {
        return nil, err
    }

    // Конвертируем в зону пользователя
    startsAtLocal := startsAt.In(loc)

    return &EventResponse{
        ID:        e.ID,
        Title:     e.Title,
        CreatedAt: time.UnixMilli(e.CreatedAt).Format("January 2, 2006 at 3:04 PM"),
        StartsAt:  startsAtLocal.Format("Monday, January 2, 2006 at 3:04 PM MST"),
        UnixTime:  startsAt.UnixMilli(),
    }, nil
}

func main() {
    // Создаем событие
    event := Event{
        ID:        1,
        Title:     "Встреча команды",
        CreatedAt: time.Now().UnixMilli(),
        StartsAt:  time.Now().Add(24 * time.Hour).Format(time.RFC3339),
    }

    // Конвертируем для пользователя в Нью-Йорке
    response, err := event.ToResponse("America/New_York")
    if err != nil {
        panic(err)
    }

    // Выводим как JSON
    jsonData, _ := json.MarshalIndent(response, "", "  ")
    fmt.Println(string(jsonData))
}
```

**Вывод:**

```json
{
  "id": 1,
  "title": "Встреча команды",
  "created_at": "July 8, 2024 at 3:04 PM",
  "starts_at": "Tuesday, July 9, 2024 at 8:04 AM EDT",
  "unix_time": 1720523045123
}
```

## Заключение

Работа со временем - критически важный навык для любого разработчика. В Go пакет `time` предоставляет:

1. **Мощные инструменты** для всех операций со временем
2. **Простую, но гибкую систему** форматирования через эталонную дату
3. **Надежную работу** с часовыми поясами
4. **Высокую точность** до наносекунд
5. **Удобную интеграцию** с Unix временем

Запомните ключевые моменты:

- Unix время - для машин
- Форматированное время - для людей
- Всегда учитывайте часовой пояс
- Обрабатывайте ошибки при парсинге

Теперь у вас есть полное понимание работы со временем в Go!
