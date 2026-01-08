## **Часть 1: Базовый пример с Mutex и структурой**

```go
package main

import (
    "fmt"
    "sync"
)

// Структура Counter с Mutex для защиты общего ресурса
type Counter struct {
    mu    sync.Mutex
    count int
}

// Метод Increment увеличивает счетчик на 1
func (c *Counter) Increment() {
    c.mu.Lock()         // Блокируем доступ к счетчику
    defer c.mu.Unlock() // Разблокируем при выходе из функции
    c.count++
}

// Метод GetValue возвращает текущее значение счетчика
func (c *Counter) GetValue() int {
    c.mu.Lock()         // Блокируем для безопасного чтения
    defer c.mu.Unlock() // Разблокируем
    return c.count
}

func main() {
    var wg sync.WaitGroup
    counter := &Counter{} // Создаем указатель на Counter

    numGoroutines := 10

    // Создаем 10 горутин, каждая увеличивает счетчик 1000 раз
    for i := 0; i < numGoroutines; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := 0; j < 1000; j++ {
                counter.Increment() // Увеличиваем счетчик
            }
        }()
    }

    wg.Wait() // Ждем завершения всех горутин

    fmt.Printf("Final counter value: %d\n", counter.GetValue())
}
```

**Вывод в консоли:**

```
Final counter value: 10000
```

**Объяснение:**

1. `Counter` - структура с полями `mu` (мьютекс) и `count` (счетчик)
2. `Increment()` блокирует мьютекс перед увеличением счетчика
3. `defer c.mu.Unlock()` гарантирует разблокировку даже при панике
4. 10 горутин × 1000 инкрементов = 10000 (всегда!)

## **Часть 2: Что происходит БЕЗ мьютекса?**

```go
package main

import (
    "fmt"
    "sync"
)

type Counter struct {
    count int // БЕЗ мьютекса!
}

func (c *Counter) Increment() {
    c.count++ // ОПАСНО: несколько горутин могут читать/писать одновременно
}

func (c *Counter) GetValue() int {
    return c.count
}

func main() {
    var wg sync.WaitGroup
    counter := &Counter{}

    numGoroutines := 10

    for i := 0; i < numGoroutines; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := 0; j < 1000; j++ {
                counter.Increment()
            }
        }()
    }

    wg.Wait()

    fmt.Printf("Final counter value: %d\n", counter.GetValue())
}
```

**Вывод в консоли (примеры нескольких запусков):**

```
Final counter value: 6342
```

**Объяснение:**

- Без мьютекса горутины могут одновременно читать и писать в `count`
- Это приводит к **гонкам данных (race conditions)**
- Результат непредсказуем и меньше ожидаемого (иногда 10000, но часто меньше)

# Что такое Mutex (Mutual Exclusion)

**Mutex (mutual exclusion)** — это механизм синхронизации, который:

> не позволяет нескольким goroutine одновременно обращаться к общему ресурсу.

То есть:

- есть **общая переменная / файл / БД**
- есть **много goroutine**
- Mutex гарантирует, что **в каждый момент времени только одна goroutine работает с этим ресурсом**

Это защищает от:

- **race condition**
- **потери данных**
- **некорректных вычислений**

---

# Принцип mutual exclusion

> Mutual exclusion — это правило: только один поток/процесс/функция может работать с общим ресурсом одновременно.

Если есть несколько функций, которые пытаются менять одну переменную:

- Mutex заставляет их **выстраиваться в очередь**
- пока одна работает — остальные **ждут**

---

# Почему Mutex нужен

Автор выделяет 3 причины:

1. **Data integrity** — защита данных от одновременной записи
2. **Synchronization** — можно блокировать целые участки кода
3. **Avoid race conditions** — предотвращение гонок

---

# Тип mutex в Go

```go
sync.Mutex
```

Методы:

| Метод     | Что делает                                      |
| --------- | ----------------------------------------------- |
| Lock()    | Захватывает mutex (блокирует остальных)         |
| Unlock()  | Освобождает mutex                               |
| TryLock() | Пытается захватить mutex, возвращает true/false |

Lock — **блокирующий**: если mutex занят, goroutine ждёт.

---

# Практический пример (из видео)

## Структура counter

```go
type Counter struct {
	mu    sync.Mutex
	count int
}
```

- `mu` — mutex
- `count` — общая переменная

---

## Метод Increment

```go
func (c *Counter) Increment() {
	c.mu.Lock()
	defer c.mu.Unlock()
	c.count++
}
```

Что происходит:

1. `Lock()` — закрывает доступ другим goroutine
2. `defer Unlock()` — гарантирует, что разблокировка будет выполнена
3. `count++` — увеличение происходит **между Lock и Unlock**

---

## Метод GetValue

```go
func (c *Counter) GetValue() int {
	c.mu.Lock()
	defer c.mu.Unlock()
	return c.count
}
```

Также защищён mutex.

---

# main()

```go
var wg sync.WaitGroup
counter := &Counter{}
numGoroutines := 10
```

Создаём:

- WaitGroup
- Counter
- 10 goroutine

---

## Запуск goroutine

Каждая goroutine увеличивает значение **1000 раз**:

```go
wg.Add(1)
go func() {
	defer wg.Done()
	for i := 0; i < 1000; i++ {
		counter.Increment()
	}
}()
```

Всего операций:

> 10 goroutine × 1000 = **10 000**

---

## Ожидание и вывод

```go
wg.Wait()
fmt.Printf("Final counter value: %d\n", counter.GetValue())
```

### Консоль:

```
Final counter value: 10000
```

**Всегда 10000.**

---

# Что если убрать mutex

Если делать:

```go
counter.count++
```

то результат **ломается**:

### Консоль:

```
Final counter value: 6396
```

→ **Непредсказуемо**

---

# Почему ломается (механика)

Без mutex:

- 4 goroutine читают `count = 0`
- все делают `0 + 1`
- все записывают обратно `1`

**Вместо 4 → становится 1**

→ Операции теряются
→ Процессорные ресурсы тратятся впустую
→ Результат некорректен

## **Часть 3: Как работает гонка данных?**

Давайте визуализируем, что происходит без мьютекса:

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    fmt.Println("=== ДЕМОНСТРАЦИЯ ГОНКИ ДАННЫХ ===")

    var count int
    var wg sync.WaitGroup

    // Симулируем 2 горутины, которые пытаются увеличить счетчик
    wg.Add(2)

    go func() {
        defer wg.Done()
        // Горутина 1 читает count (допустим, count = 0)
        value := count
        time.Sleep(1 * time.Millisecond) // Имитируем задержку
        // Пишет обратно (0 + 1 = 1)
        count = value + 1
        fmt.Println("Горутина 1 записала:", value+1)
    }()

    go func() {
        defer wg.Done()
        // Горутина 2 ТАКЖЕ читает count (все еще 0!)
        value := count
        time.Sleep(1 * time.Millisecond)
        // Тоже пишет (0 + 1 = 1)
        count = value + 1
        fmt.Println("Горутина 2 записала:", value+1)
    }()

    wg.Wait()
    fmt.Printf("Итоговое значение: %d (должно быть 2!)\n", count)
}
```

**Вывод в консоли:**

```
=== ДЕМОНСТРАЦИЯ ГОНКИ ДАННЫХ ===
Горутина 2 записала: 1
Горутина 1 записала: 1
Итоговое значение: 1 (должно быть 2!)
```

**Объяснение гонки:**

1. Обе горутины читают `count = 0`
2. Обе увеличивают прочитанное значение на 1
3. Обе записывают `1` обратно
4. Вместо `2` получаем `1` - операция потеряна!

## **Часть 4: Mutex без структуры**

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    fmt.Println("=== MUTEX БЕЗ СТРУКТУРЫ ===")

    var counter int
    var mu sync.Mutex
    var wg sync.WaitGroup

    numGoroutines := 5

    // Функция для инкремента (анонимная функция)
    increment := func() {
        defer wg.Done()
        for i := 0; i < 1000; i++ {
            mu.Lock()   // Начало критической секции
            counter++   // Операция под защитой мьютекса
            mu.Unlock() // Конец критической секции
        }
    }

    // Запускаем горутины
    for i := 0; i < numGoroutines; i++ {
        wg.Add(1)
        go increment()
    }

    wg.Wait()

    fmt.Printf("Final counter value: %d\n", counter)
}
```

**Вывод в консоли:**

```
=== MUTEX БЕЗ СТРУКТУРЫ ===
Final counter value: 5000
```

**Объяснение:**

1. Мьютекс можно использовать независимо от структур
2. `mu.Lock()` и `mu.Unlock()` определяют **критическую секцию**
3. Только одна горутина может выполнять код между `Lock()` и `Unlock()`

## **Часть 5: Использование defer для разблокировки**

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    fmt.Println("=== ПРАВИЛЬНОЕ ИСПОЛЬЗОВАНИЕ DEFER ===")

    var counter int
    var mu sync.Mutex
    var wg sync.WaitGroup

    wg.Add(2)

    go func() {
        defer wg.Done()

        mu.Lock()
        defer mu.Unlock() // defer гарантирует разблокировку!

        // Критическая секция
        for i := 0; i < 1000; i++ {
            counter++
        }
        fmt.Println("Горутина 1 завершила работу")

        // Если бы здесь была паника, defer mu.Unlock() все равно бы выполнился
        // panic("ошибка!") // Раскомментируйте для теста
    }()

    go func() {
        defer wg.Done()

        mu.Lock()
        defer mu.Unlock() // Всегда используйте defer!

        for i := 0; i < 1000; i++ {
            counter++
        }
        fmt.Println("Горутина 2 завершила работу")
    }()

    wg.Wait()
    fmt.Printf("Итоговое значение: %d\n", counter)
}
```

**Вывод в консоли:**

```
=== ПРАВИЛЬНОЕ ИСПОЛЬЗОВАНИЕ DEFER ===
Горутина 2 завершила работу
Горутина 1 завершила работу
Итоговое значение: 2000
```

**Важность defer:**

- `defer mu.Unlock()` гарантирует разблокировку даже при панике
- Без defer можно забыть вызвать `Unlock()`, что приведет к дедлоку

## **Часть 6: TryLock - попытка заблокировать без ожидания**

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    fmt.Println("=== DEMO TryLock ===")

    var mu sync.Mutex
    var counter int

    // Горутина 1: захватывает мьютекс на 2 секунды
    go func() {
        mu.Lock()
        defer mu.Unlock()

        fmt.Println("Горутина 1: захватила мьютекс")
        time.Sleep(2 * time.Second)
        counter = 100
        fmt.Println("Горутина 1: отпустила мьютекс")
    }()

    // Даем время первой горутине захватить мьютекс
    time.Sleep(100 * time.Millisecond)

    // Горутина 2: пытается использовать TryLock
    go func() {
        fmt.Println("Горутина 2: пытается TryLock...")

        if mu.TryLock() {
            defer mu.Unlock()
            fmt.Println("Горутина 2: удалось захватить мьютекс!")
            counter++
        } else {
            fmt.Println("Горутина 2: не удалось захватить мьютекс (занято)")
        }
    }()

    // Горутина 3: обычный Lock (будет ждать)
    go func() {
        fmt.Println("Горутина 3: пытается Lock (будет ждать)...")
        mu.Lock()
        defer mu.Unlock()
        fmt.Println("Горутина 3: наконец-то захватила мьютекс!")
        counter = 200
    }()

    // Ждем завершения
    time.Sleep(3 * time.Second)
    fmt.Printf("Итоговое значение counter: %d\n", counter)
}
```

**Вывод в консоли:**

```
=== DEMO TryLock ===
Горутина 1: захватила мьютекс
Горутина 2: пытается TryLock...
Горутина 2: не удалось захватить мьютекс (занято)
Горутина 3: пытается Lock (будет ждать)...
Горутина 1: отпустила мьютекс
Горутина 3: наконец-то захватила мьютекс!
Итоговое значение counter: 200
```

**Разница между Lock и TryLock:**

- `Lock()` - блокирует выполнение, пока мьютекс не освободится
- `TryLock()` - пытается захватить, но не блокирует; возвращает `true/false`

## **Часть 7: Deadlock (взаимная блокировка)**

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    fmt.Println("=== ДЕМОНСТРАЦИЯ DEADLOCK ===")

    var mu1, mu2 sync.Mutex
    var wg sync.WaitGroup

    wg.Add(2)

    // Горутина 1: захватывает mu1, потом mu2
    go func() {
        defer wg.Done()

        mu1.Lock()
        defer mu1.Unlock()
        fmt.Println("Горутина 1: захватила mu1")

        time.Sleep(100 * time.Millisecond) // Имитируем работу

        mu2.Lock()
        defer mu2.Unlock()
        fmt.Println("Горутина 1: захватила mu2")
    }()

    // Горутина 2: захватывает mu2, потом mu1 (ОПАСНО!)
    go func() {
        defer wg.Done()

        mu2.Lock()
        defer mu2.Unlock()
        fmt.Println("Горутина 2: захватила mu2")

        time.Sleep(100 * time.Millisecond)

        mu1.Lock() // ← ЗАВИСНЕТ ЗДЕСЬ!
        defer mu1.Unlock()
        fmt.Println("Горутина 2: захватила mu1")
    }()

    // Пытаемся подождать, но будет deadlock
    done := make(chan bool)
    go func() {
        wg.Wait()
        done <- true
    }()

    select {
    case <-done:
        fmt.Println("Все горутины завершились")
    case <-time.After(2 * time.Second):
        fmt.Println("DEADLOCK! Программа зависла на 2 секунды")
    }
}
```

**Вывод в консоли:**

```
=== ДЕМОНСТРАЦИЯ DEADLOCK ===
Горутина 1: захватила mu1
Горутина 2: захватила mu2
DEADLOCK! Программа зависла на 2 секунды
```

**Что такое deadlock:**

1. Горутина 1 ждет mu2 (который у горутины 2)
2. Горутина 2 ждет mu1 (который у горутины 1)
3. Обе ждут друг друга вечно!

## **Часть 8: Правильный порядок блокировок**

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    fmt.Println("=== ПРАВИЛЬНЫЙ ПОРЯДОК БЛОКИРОВОК ===")

    var mu1, mu2 sync.Mutex
    var wg sync.WaitGroup

    wg.Add(2)

    // ОБА берут мьютексы в ОДНОМ порядке: сначала mu1, потом mu2
    go func() {
        defer wg.Done()

        mu1.Lock()
        defer mu1.Unlock()
        fmt.Println("Горутина 1: захватила mu1")

        time.Sleep(100 * time.Millisecond)

        mu2.Lock()
        defer mu2.Unlock()
        fmt.Println("Горутина 1: захватила mu2")
    }()

    go func() {
        defer wg.Done()

        mu1.Lock() // ТАКОЙ ЖЕ порядок: mu1 сначала!
        defer mu1.Unlock()
        fmt.Println("Горутина 2: захватила mu1")

        time.Sleep(100 * time.Millisecond)

        mu2.Lock()
        defer mu2.Unlock()
        fmt.Println("Горутина 2: захватила mu2")
    }()

    wg.Wait()
    fmt.Println("Все горутины завершились без deadlock!")
}
```

**Вывод в консоли:**

```
=== ПРАВИЛЬНЫЙ ПОРЯДОК БЛОКИРОВОК ===
Горутина 2: захватила mu1
Горутина 2: захватила mu2
Горутина 1: захватила mu1
Горутина 1: захватила mu2
Все горутины завершились без deadlock!
```

**Правило:** Всегда захватывайте мьютексы в одинаковом порядке!

## **Итог лекции:**

1. **Мьютекс (mutex)** - Mutual Exclusion (взаимное исключение)
2. **Зачем нужен:** Защищает общие ресурсы от одновременного доступа горутин
3. **Основные методы:**

   - `Lock()` - блокирует мьютекс (ждет, если занят)
   - `Unlock()` - разблокирует мьютекс
   - `TryLock()` - пытается захватить без ожидания

4. **Критическая секция:** Код между `Lock()` и `Unlock()`

5. **Проблемы без мьютекса:**

   - Гонки данных (race conditions)
   - Непредсказуемые результаты
   - Потеря обновлений

6. **Лучшие практики:**

   - Всегда используйте `defer mu.Unlock()`
   - Держите критические секции короткими
   - Захватывайте мьютексы в одинаковом порядке
   - Избегайте вложенных блокировок

7. **Типичные ошибки:**
   - Забыть разблокировать мьютекс
   - Deadlock (взаимная блокировка)
   - Долгие критические секции (contention)

**Ключевой вывод:** Мьютекс гарантирует, что только одна горутина в момент времени может выполнять критическую секцию, защищая общие данные от гонок.
