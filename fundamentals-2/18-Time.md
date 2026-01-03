## Часть 1: Основы работы со временем в Go

### В Go время представлено структурой `time.Time`:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Получаем текущее локальное время
    currentTime := time.Now()
    fmt.Println("Текущее время:", currentTime)

    // Получаем текущее время UTC
    currentTimeUTC := time.Now().UTC()
    fmt.Println("Текущее время UTC:", currentTimeUTC)
}
```

### Вывод:

```
$ go run main.go
Текущее время: 2024-07-08 14:36:45.123456789 +03:00
Текущее время UTC: 2024-07-08 11:36:45.123456789 +0000 UTC
```

**Что здесь происходит:**

- `time.Now()` возвращает текущее время с наносекундной точностью
- Время включает часовой пояс (например, `+03:00` для Москвы)
- `UTC()` преобразует время в часовой пояс UTC (без смещения)

## Часть 2: Создание конкретного времени

### Код:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Создаем конкретное время: 30 июля 2024, 12:00:00 UTC
    specificTime := time.Date(2024, time.July, 30, 12, 0, 0, 0, time.UTC)
    fmt.Println("Конкретное время:", specificTime)

    // Можно использовать числовые значения месяцев
    specificTime2 := time.Date(2024, 7, 30, 12, 0, 0, 0, time.UTC)
    fmt.Println("То же время (числовой месяц):", specificTime2)

    // Время в локальном часовом поясе
    specificTimeLocal := time.Date(2024, time.December, 31, 23, 59, 59, 0, time.Local)
    fmt.Println("Локальное время:", specificTimeLocal)
}
```

### Вывод:

```
$ go run main.go
Конкретное время: 2024-07-30 12:00:00 +0000 UTC
То же время (числовой месяц): 2024-07-30 12:00:00 +0000 UTC
Локальное время: 2024-12-31 23:59:59 +03:00
```

**Объяснение:**

- `time.Date(year, month, day, hour, minute, second, nanosecond, location)`
- `time.July` - это константа, равная 7 (для удобства чтения)
- Можно использовать числа: `7` вместо `time.July`
- `time.UTC` и `time.Local` - предопределенные локации (часовые пояса)

## Часть 3: Парсинг времени из строки

### Код:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Парсинг времени из строки
    // ВАЖНО: Go использует эталонную дату для шаблонов!
    // Эталонная дата: Mon Jan 2 15:04:05 MST 2006 (или 01/02 03:04:05PM '06 -0700)

    // Пример 1: Простой парсинг
    parsedTime, err := time.Parse("2006 January 2", "2020 May 1")
    if err != nil {
        fmt.Println("Ошибка:", err)
        return
    }
    fmt.Println("Пример 1:", parsedTime)

    // Пример 2: Другой формат
    parsedTime2, err := time.Parse("06-01-02", "20-05-01")
    if err != nil {
        fmt.Println("Ошибка:", err)
        return
    }
    fmt.Println("Пример 2:", parsedTime2)

    // Пример 3: С временем
    parsedTime3, err := time.Parse("06-01-02 15:04", "20-05-01 18:03")
    if err != nil {
        fmt.Println("Ошибка:", err)
        return
    }
    fmt.Println("Пример 3:", parsedTime3)

    // Пример 4: НЕПРАВИЛЬНЫЙ формат (покажет проблему)
    parsedTime4, err := time.Parse("06-1-2 15:4", "20-5-1 18:3")
    if err != nil {
        fmt.Println("Пример 4 (ошибка):", err)
    } else {
        fmt.Println("Пример 4:", parsedTime4)
    }
}
```

### Вывод:

```
$ go run main.go
Пример 1: 2020-05-01 00:00:00 +0000 UTC
Пример 2: 2020-05-01 00:00:00 +0000 UTC
Пример 3: 2020-05-01 18:03:00 +0000 UTC
Пример 4 (ошибка): parsing time "20-5-1 18:3" as "06-1-2 15:4": cannot parse "8:3" as "4"
```

**Ключевой момент:** Go использует специфическую эталонную дату для шаблонов!

**Эталонная дата:** Monday, January 2, 15:04:05 MST 2006

- Год: 2006 → `2006` или `06`
- Месяц: January → `Jan` или `01`
- День: 2 → `2` или `02`
- Час: 15 (24-часовой формат) → `15`
- Минута: 04 → `04`
- Секунда: 05 → `05`

## Часть 4: Форматирование времени

### Код:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    t := time.Now()

    // Форматирование времени в различные форматы
    fmt.Println("Текущее время:", t)
    fmt.Println("RFC3339 (стандарт):", t.Format(time.RFC3339))
    fmt.Println("Только дата:", t.Format("2006-01-02"))
    fmt.Println("Только время:", t.Format("15:04:05"))
    fmt.Println("Дата и время:", t.Format("2006-01-02 15:04:05"))

    // Более сложные форматы
    fmt.Println("\nСложные форматы:")
    fmt.Println("С днем недели:", t.Format("Monday, 2006-01-02"))
    fmt.Println("12-часовой формат:", t.Format("3:04:05 PM"))
    fmt.Println("С миллисекундами:", t.Format("15:04:05.000"))

    // Предопределенные форматы Go
    fmt.Println("\nПредопределенные форматы:")
    fmt.Println("ANSIC:", t.Format(time.ANSIC))
    fmt.Println("UnixDate:", t.Format(time.UnixDate))
    fmt.Println("RubyDate:", t.Format(time.RubyDate))
    fmt.Println("RFC822:", t.Format(time.RFC822))
}
```

### Вывод:

```
$ go run main.go
Текущее время: 2024-07-08 14:36:45.123456789 +03:00
RFC3339 (стандарт): 2024-07-08T14:36:45+03:00
Только дата: 2024-07-08
Только время: 14:36:45
Дата и время: 2024-07-08 14:36:45

Сложные форматы:
С днем недели: Monday, 2024-07-08
12-часовой формат: 2:36:45 PM
С миллисекундами: 14:36:45.123

Предопределенные форматы:
ANSIC: Mon Jan  2 15:04:05 2006
UnixDate: Mon Jan  2 15:04:05 MST 2006
RubyDate: Mon Jan 02 15:04:05 +0300 2006
RFC822: 08 Jul 24 14:36 +0300
```

**Важно:** `Format()` работает с уже существующим временем (тип `time.Time`), а `Parse()` преобразует строку во время.

## Часть 5: Работа с продолжительностями (Duration)

### Код:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    t := time.Now()
    fmt.Println("Сейчас:", t.Format("15:04:05"))

    // Добавление продолжительности
    oneHourLater := t.Add(time.Hour)
    fmt.Println("Через час:", oneHourLater.Format("15:04:05"))

    oneDayLater := t.Add(24 * time.Hour)
    fmt.Println("Через день:", oneDayLater.Format("2006-01-02 15:04:05"))

    // Вычитание продолжительности
    twoHoursEarlier := t.Add(-2 * time.Hour)
    fmt.Println("Два часа назад:", twoHoursEarlier.Format("15:04:05"))

    // Разница между двумя временами
    t1 := time.Date(2024, 7, 8, 12, 0, 0, 0, time.UTC)
    t2 := time.Date(2024, 7, 8, 18, 0, 0, 0, time.UTC)
    duration := t2.Sub(t1)

    fmt.Printf("\nРазница между %v и %v:\n", t2.Format("15:04"), t1.Format("15:04"))
    fmt.Printf("Часы: %.1f\n", duration.Hours())
    fmt.Printf("Минуты: %.1f\n", duration.Minutes())
    fmt.Printf("Секунды: %.0f\n", duration.Seconds())

    // Продолжительности в действии
    fmt.Println("\nРазличные продолжительности:")
    fmt.Println("5 секунд:", 5*time.Second)
    fmt.Println("3 минуты:", 3*time.Minute)
    fmt.Println("2.5 часа:", 2*time.Hour + 30*time.Minute)
    fmt.Println("1 день:", 24*time.Hour)
}
```

### Вывод:

```
$ go run main.go
Сейчас: 14:36:45
Через час: 15:36:45
Через день: 2024-07-09 14:36:45
Два часа назад: 12:36:45

Разница между 18:00 и 12:00:
Часы: 6.0
Минуты: 360.0
Секунды: 21600

Различные продолжительности:
5 секунд: 5s
3 минуты: 3m0s
2.5 часа: 2h30m0s
1 день: 24h0m0s
```

**Тип `Duration`:** представляет продолжительность времени в наносекундах (int64). Есть удобные константы:

- `time.Nanosecond`
- `time.Microsecond`
- `time.Millisecond`
- `time.Second`
- `time.Minute`
- `time.Hour`

## Часть 6: Округление и усечение времени

### Код:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Создаем конкретное время для тестов
    t := time.Date(2024, 7, 8, 14, 16, 40, 123456789, time.UTC)
    fmt.Println("Исходное время:", t.Format("2006-01-02 15:04:05.999999999"))

    // Округление
    roundedToHour := t.Round(time.Hour)
    fmt.Println("Округлено до часа:", roundedToHour.Format("15:04:05.999"))

    roundedTo30Min := t.Round(30 * time.Minute)
    fmt.Println("Округлено до 30 минут:", roundedTo30Min.Format("15:04:05"))

    roundedTo15Min := t.Round(15 * time.Minute)
    fmt.Println("Округлено до 15 минут:", roundedTo15Min.Format("15:04:05"))

    // Усечение
    truncatedToHour := t.Truncate(time.Hour)
    fmt.Println("\nУсечено до часа:", truncatedToHour.Format("15:04:05.999"))

    truncatedToDay := t.Truncate(24 * time.Hour)
    fmt.Println("Усечено до дня:", truncatedToDay.Format("2006-01-02 15:04:05"))

    // Разница между Round и Truncate
    testTime := time.Date(2024, 7, 8, 14, 30, 0, 0, time.UTC)
    fmt.Println("\nТестовое время:", testTime.Format("15:04"))
    fmt.Println("Round до часа:", testTime.Round(time.Hour).Format("15:04"))
    fmt.Println("Truncate до часа:", testTime.Truncate(time.Hour).Format("15:04"))
}
```

### Вывод:

```
$ go run main.go
Исходное время: 2024-07-08 14:16:40.123456789
Округлено до часа: 14:00:00.000
Округлено до 30 минут: 14:30:00
Округлено до 15 минут: 14:15:00

Усечено до часа: 14:00:00.123
Усечено до дня: 2024-07-08 00:00:00

Тестовое время: 14:30
Round до часа: 15:00
Truncate до часа: 14:00
```

**Разница между Round и Truncate:**

- `Round()` округляет до ближайшего значения (в обе стороны)
- `Truncate()` всегда округляет вниз (отбрасывает дробную часть)

## Часть 7: Работа с часовыми поясами

### Код:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Загрузка различных часовых поясов
    locations := []string{
        "UTC",
        "Local",
        "America/New_York",
        "Europe/London",
        "Asia/Tokyo",
        "Australia/Sydney",
    }

    now := time.Now()

    fmt.Println("Текущее время в разных часовых поясах:")
    fmt.Println("========================================")

    for _, locName := range locations {
        loc, err := time.LoadLocation(locName)
        if err != nil {
            fmt.Printf("Ошибка загрузки %s: %v\n", locName, err)
            continue
        }

        timeInLoc := now.In(loc)
        fmt.Printf("%-20s: %s (смещение: %s)\n",
            locName,
            timeInLoc.Format("2006-01-02 15:04:05"),
            timeInLoc.Format("-07:00"))
    }

    // Конвертация между поясами
    fmt.Println("\nКонвертация времени:")
    fmt.Println("===================")

    nyLoc, _ := time.LoadLocation("America/New_York")
    tokyoLoc, _ := time.LoadLocation("Asia/Tokyo")

    meetingTimeNY := time.Date(2024, 7, 8, 14, 0, 0, 0, nyLoc)
    meetingTimeTokyo := meetingTimeNY.In(tokyoLoc)

    fmt.Printf("Встреча в Нью-Йорке: %s\n",
        meetingTimeNY.Format("Monday, 2006-01-02 15:04 MST"))
    fmt.Printf("Встреча в Токио:     %s\n",
        meetingTimeTokyo.Format("Monday, 2006-01-02 15:04 MST"))
}
```

### Вывод (будет разным в зависимости от вашего местоположения):

```
$ go run main.go
Текущее время в разных часовых поясах:
========================================
UTC                  : 2024-07-08 11:36:45 (смещение: +00:00)
Local                : 2024-07-08 14:36:45 (смещение: +03:00)
America/New_York     : 2024-07-08 06:36:45 (смещение: -04:00)
Europe/London        : 2024-07-08 12:36:45 (смещение: +01:00)
Asia/Tokyo           : 2024-07-08 20:36:45 (смещение: +09:00)
Australia/Sydney     : 2024-07-08 21:36:45 (смещение: +10:00)

Конвертация времени:
===================
Встреча в Нью-Йорке: Monday, 2024-07-08 14:00 EDT
Встреча в Токио:     Tuesday, 2024-07-09 03:00 JST
```

**Важно:** Для работы с часовыми поясами нужна база данных tzdata. В Go она встроена.

## Часть 8: Сравнение времени и временные операции

### Код:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Создаем времена для сравнения
    t1 := time.Date(2024, 7, 8, 12, 0, 0, 0, time.UTC)
    t2 := time.Date(2024, 7, 8, 18, 0, 0, 0, time.UTC)
    t3 := time.Date(2024, 7, 8, 12, 0, 0, 0, time.UTC) // То же, что t1

    fmt.Println("Время t1:", t1.Format("15:04"))
    fmt.Println("Время t2:", t2.Format("15:04"))
    fmt.Println("Время t3:", t3.Format("15:04"))

    // Сравнение
    fmt.Println("\nСравнение:")
    fmt.Println("t1 == t3?", t1.Equal(t3))   // Equal сравнивает моменты времени
    fmt.Println("t1 == t2?", t1.Equal(t2))
    fmt.Println("t1.Before(t2)?", t1.Before(t2))
    fmt.Println("t2.After(t1)?", t2.After(t1))

    // Проверка на нулевое время
    var zeroTime time.Time
    fmt.Println("\nПроверка нулевого времени:")
    fmt.Println("zeroTime is zero?", zeroTime.IsZero())
    fmt.Println("t1 is zero?", t1.IsZero())

    // Извлечение компонентов времени
    fmt.Println("\nКомпоненты времени t1:")
    fmt.Printf("Год: %d\n", t1.Year())
    fmt.Printf("Месяц: %d (%s)\n", t1.Month(), t1.Month())
    fmt.Printf("День: %d\n", t1.Day())
    fmt.Printf("Час: %d\n", t1.Hour())
    fmt.Printf("Минута: %d\n", t1.Minute())
    fmt.Printf("Секунда: %d\n", t1.Second())
    fmt.Printf("День недели: %s\n", t1.Weekday())
    fmt.Printf("День года: %d\n", t1.YearDay())

    // Добавление дат (лет, месяцев, дней)
    fmt.Println("\nДобавление к датам:")
    futureDate := t1.AddDate(1, 2, 3) // +1 год, +2 месяца, +3 дня
    fmt.Println("t1 + 1 год 2 месяца 3 дня:", futureDate.Format("2006-01-02"))
}
```

### Вывод:

```
$ go run main.go
Время t1: 12:00
Время t2: 18:00
Время t3: 12:00

Сравнение:
t1 == t3? true
t1 == t2? false
t1.Before(t2)? true
t2.After(t1)? true

Проверка нулевого времени:
zeroTime is zero? true
t1 is zero? false

Компоненты времени t1:
Год: 2024
Месяц: 7 (July)
День: 8
Час: 12
Минута: 0
Секунда: 0
День недели: Monday
День года: 190

Добавление к датам:
t1 + 1 год 2 месяца 3 дня: 2025-09-11
```

## Часть 9: Таймеры и тикеры

### Код:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Таймер - одноразовое событие
    fmt.Println("Запускаем таймер на 2 секунды...")
    timer := time.NewTimer(2 * time.Second)

    // Ждем срабатывания таймера
    <-timer.C
    fmt.Println("Таймер сработал!")

    // Тикер - повторяющееся событие
    fmt.Println("\nЗапускаем тикер (каждую секунду, 3 раза)...")
    ticker := time.NewTicker(1 * time.Second)

    // Останавливаем тикер через 3 секунды
    stopTimer := time.NewTimer(3 * time.Second)

    tickCount := 0
    for {
        select {
        case <-ticker.C:
            tickCount++
            fmt.Printf("Тик #%d в %v\n", tickCount, time.Now().Format("15:04:05"))

        case <-stopTimer.C:
            fmt.Println("Останавливаем тикер...")
            ticker.Stop()
            return
        }
    }
}
```

### Вывод:

```
$ go run main.go
Запускаем таймер на 2 секунды...
Таймер сработал!

Запускаем тикер (каждую секунду, 3 раза)...
Тик #1 в 14:36:47
Тик #2 в 14:36:48
Тик #3 в 14:36:49
Останавливаем тикер...
```

## Часть 10: Unix время (timestamp)

### Код:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()

    // Unix время (секунды с 1 января 1970)
    unixTime := now.Unix()
    fmt.Printf("Unix время (секунды): %d\n", unixTime)

    // Unix время с наносекундами
    unixNano := now.UnixNano()
    fmt.Printf("Unix время (наносекунды): %d\n", unixNano)

    // Преобразование Unix времени обратно
    fromUnix := time.Unix(unixTime, 0)
    fmt.Printf("Из Unix времени: %v\n", fromUnix.Format("2006-01-02 15:04:05"))

    // Пример: сколько времени прошло с Unix эпохи
    epoch := time.Unix(0, 0)
    durationSinceEpoch := now.Sub(epoch)
    fmt.Printf("\nВремя с Unix эпохи (1 Jan 1970):\n")
    fmt.Printf("Дней: %.0f\n", durationSinceEpoch.Hours()/24)
    fmt.Printf("Лет: %.1f\n", durationSinceEpoch.Hours()/24/365.25)
}
```

### Вывод:

```
$ go run main.go
Unix время (секунды): 1720442205
Unix время (наносекунды): 1720442205123456789
Из Unix времени: 2024-07-08 14:36:45

Время с Unix эпохи (1 Jan 1970):
Дней: 19913
Лет: 54.5
```

## Итоговая таблица основных методов:

| Метод                      | Описание                     | Пример                                              |
| -------------------------- | ---------------------------- | --------------------------------------------------- |
| `time.Now()`               | Текущее время                | `now := time.Now()`                                 |
| `time.Date()`              | Создание времени             | `t := time.Date(2024, 7, 8, 12, 0, 0, 0, time.UTC)` |
| `time.Parse()`             | Парсинг из строки            | `t, err := time.Parse("2006-01-02", "2024-07-08")`  |
| `Format()`                 | Форматирование в строку      | `t.Format("2006-01-02 15:04:05")`                   |
| `Add()`                    | Добавление продолжительности | `t.Add(2 * time.Hour)`                              |
| `Sub()`                    | Разница между временами      | `diff := t2.Sub(t1)`                                |
| `Before()/After()/Equal()` | Сравнение                    | `t1.Before(t2)`                                     |
| `Round()/Truncate()`       | Округление/усечение          | `t.Round(time.Hour)`                                |
| `In()`                     | Конвертация часового пояса   | `t.In(loc)`                                         |
| `Unix()`                   | Unix timestamp (секунды)     | `t.Unix()`                                          |
| `AddDate()`                | Добавление лет/месяцев/дней  | `t.AddDate(1, 0, 0)`                                |

## Практические советы:

1. **Всегда обрабатывайте ошибки** при парсинге времени
2. **Используйте константы времени** (`time.Hour`, `time.Minute`) вместо "магических чисел"
3. **Учитывайте часовые пояса** при работе с пользователями из разных регионов
4. **Для измерения производительности** используйте `time.Since(start)`
5. **Кэшируйте результаты** `time.LoadLocation()` для часто используемых часовых поясов
