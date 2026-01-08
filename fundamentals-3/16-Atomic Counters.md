# Что такое Atomic Counter (по словам автора)

**Atomic counter** — это счётчик, который используется в конкурентном программировании для безопасного подсчёта значений **без использования Mutex**.

Он:

- работает потокобезопасно
- не требует lock/unlock
- использует атомарные операции
- предотвращает race condition

---

## Почему их используют

Автор назвал 3 причины:

| Причина     | Что это значит                                                    |
| ----------- | ----------------------------------------------------------------- |
| Performance | Атомарные операции быстрее, чем Mutex (меньше накладных расходов) |
| Simplicity  | Их легче использовать                                             |
| Concurrency | Они гарантируют корректность при одновременной работе goroutine   |

---

## Что такое "atomic"

**Atomic = неделимый и непрерываемый**

Это значит:

| Свойство              | Объяснение                                                  |
| --------------------- | ----------------------------------------------------------- |
| Indivisible           | операция выполняется как одно целое                         |
| Uninterruptible       | её нельзя прервать и увидеть "половинчатое" состояние       |
| Memory fences         | гарантируется корректная видимость изменений между потоками |
| Реализуется аппаратно | через инструкции CPU                                        |

---

# Реализация из видео

Автор создаёт структуру:

```go
type AtomicCounter struct {
    count int64
}
```

---

## Метод Increment()

```go
func (ac *AtomicCounter) Increment() {
    atomic.AddInt64(&ac.count, 1)
}
```

**Что происходит:**

| Шаг             | Что происходит           |
| --------------- | ------------------------ |
| ac — указатель  | содержит адрес структуры |
| ac.count        | это значение поля        |
| &ac.count       | это адрес поля           |
| atomic.AddInt64 | атомарно добавляет 1     |

---

## Метод GetValue()

```go
func (ac *AtomicCounter) GetValue() int64 {
    return atomic.LoadInt64(&ac.count)
}
```

| Что делает       | Почему                   |
| ---------------- | ------------------------ |
| LoadInt64        | атомарно читает значение |
| Возвращает int64 | тип совпадает            |

---

# Функция main()

```go
var wg sync.WaitGroup
numGoroutines := 10
counter := &AtomicCounter{}
```

---

## Создание 10 goroutine

```go
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
fmt.Printf("Final Counter Value: %d\n", counter.GetValue())
```

---

# Что делает программа

| Что происходит                      |
| ----------------------------------- |
| 10 goroutines                       |
| каждая увеличивает счётчик 1000 раз |
| всего должно быть 10,000            |

---

# Вывод в консоли

```
Final Counter Value: 10000
Final Counter Value: 10000
Final Counter Value: 10000
Final Counter Value: 10000
```

**Всегда стабильно 10000**

## **Часть 1: Базовый пример с атомарным счетчиком**

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

// Структура AtomicCounter с атомарным счетчиком
type AtomicCounter struct {
    count int64 // Используем int64 для атомарных операций
}

// Метод Increment увеличивает счетчик на 1 атомарно
func (ac *AtomicCounter) Increment() {
    atomic.AddInt64(&ac.count, 1) // Атомарно добавляем 1
}

// Метод GetValue возвращает текущее значение счетчика атомарно
func (ac *AtomicCounter) GetValue() int64 {
    return atomic.LoadInt64(&ac.count) // Атомарно читаем значение
}

func main() {
    var wg sync.WaitGroup
    numGoroutines := 10

    // Создаем экземпляр AtomicCounter
    counter := &AtomicCounter{}

    // Запускаем 10 горутин
    for i := 0; i < numGoroutines; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            // Каждая горутина увеличивает счетчик 1000 раз
            for j := 0; j < 1000; j++ {
                counter.Increment()
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

1. `AtomicCounter` - структура с полем `count` типа `int64`
2. `atomic.AddInt64(&ac.count, 1)` - атомарно увеличивает счетчик на 1
3. `atomic.LoadInt64(&ac.count)` - атомарно читает значение счетчика
4. 10 горутин × 1000 инкрементов = 10000 (всегда корректно!)

## **Часть 2: Что происходит БЕЗ атомарных операций?**

```go
package main

import (
    "fmt"
    "sync"
)

// Обычный счетчик без атомарных операций
type SimpleCounter struct {
    count int64
}

func (sc *SimpleCounter) Increment() {
    sc.count++ // НЕ атомарная операция!
}

func (sc *SimpleCounter) GetValue() int64 {
    return sc.count
}

func main() {
    fmt.Println("=== БЕЗ атомарных операций ===")

    var wg sync.WaitGroup
    numGoroutines := 10

    counter := &SimpleCounter{}

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

    fmt.Printf("Final counter value: %d (должно быть 10000)\n", counter.GetValue())
    fmt.Println("Проблема: возможны гонки данных!")
}
```

**Вывод в консоли (несколько запусков):**

```
=== БЕЗ атомарных операций ===
Final counter value: 6342 (должно быть 10000)
Final counter value: 8725 (должно быть 10000)
Final counter value: 5567 (должно быть 10000)
Final counter value: 10000 (должно быть 10000)
Final counter value: 7238 (должно быть 10000)
```

**Проблема:** Без атомарных операций происходят **гонки данных (data races)**!

## **Часть 3: Визуализация проблемы гонки данных**

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    fmt.Println("=== ВИЗУАЛИЗАЦИЯ ГОНКИ ДАННЫХ ===")

    var count int64
    var wg sync.WaitGroup

    // Простая операция инкремента состоит из 3 шагов:
    fmt.Println("Операция count++ состоит из:")
    fmt.Println("1. Прочитать значение count из памяти")
    fmt.Println("2. Увеличить его на 1")
    fmt.Println("3. Записать новое значение обратно в память")
    fmt.Println()

    wg.Add(2)

    // Две горутины пытаются увеличить счетчик одновременно
    go func() {
        defer wg.Done()
        for i := 0; i < 5; i++ {
            // Шаг 1: Чтение (допустим, читаем 0)
            value := count

            // Шаг 2: Увеличение (0 + 1 = 1)
            value++

            // Шаг 3: Запись (пишем 1 обратно)
            count = value

            fmt.Printf("Горутина 1: увеличила до %d\n", count)
        }
    }()

    go func() {
        defer wg.Done()
        for i := 0; i < 5; i++ {
            // Проблема: эта горутина может прочитать то же самое значение!
            value := count  // Может прочитать 0, пока первая еще не записала 1

            value++
            count = value

            fmt.Printf("Горутина 2: увеличила до %d\n", count)
        }
    }()

    wg.Wait()
    fmt.Printf("\nИтоговое значение: %d (должно быть 10)\n", count)
}
```

**Вывод в консоли (пример):**

```
=== ВИЗУАЛИЗАЦИЯ ГОНКИ ДАННЫХ ===
Операция count++ состоит из:
1. Прочитать значение count из памяти
2. Увеличить его на 1
3. Записать новое значение обратно в память

Горутина 2: увеличила до 1
Горутина 1: увеличила до 1
Горутина 2: увеличила до 2
Горутина 1: увеличила до 2
Горутина 2: увеличила до 3
Горутина 1: увеличила до 3
Горутина 2: увеличила до 4
Горутина 1: увеличила до 4
Горутина 2: увеличила до 5
Горутина 1: увеличила до 5

Итоговое значение: 5 (должно быть 10)
```

**Объяснение:** Из-за гонки данных потеряно 5 операций!

## **Часть 4: Атомарные операции - как они работают**

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

func main() {
    fmt.Println("=== КАК РАБОТАЮТ АТОМАРНЫЕ ОПЕРАЦИИ ===")

    var count int64 // Атомарные операции работают с int64, int32 и т.д.
    var wg sync.WaitGroup

    wg.Add(2)

    go func() {
        defer wg.Done()
        for i := 0; i < 5; i++ {
            // Атомарная операция делает все 3 шага как ОДНО целое
            atomic.AddInt64(&count, 1)
            fmt.Printf("Горутина 1: атомарно увеличила\n")
        }
    }()

    go func() {
        defer wg.Done()
        for i := 0; i < 5; i++ {
            atomic.AddInt64(&count, 1)
            fmt.Printf("Горутина 2: атомарно увеличила\n")
        }
    }()

    wg.Wait()

    finalValue := atomic.LoadInt64(&count)
    fmt.Printf("\nИтоговое значение: %d (всегда будет 10!)\n", finalValue)

    fmt.Println("\nАтомарные операции НЕЛЬЗЯ прервать!")
    fmt.Println("Чтение → Изменение → Запись = ОДНА неделимая операция")
}
```

**Вывод в консоли:**

```
=== КАК РАБОТАЮТ АТОМАРНЫЕ ОПЕРАЦИИ ===
Горутина 2: атомарно увеличила
Горутина 1: атомарно увеличила
Горутина 2: атомарно увеличила
Горутина 1: атомарно увеличила
Горутина 2: атомарно увеличила
Горутина 1: атомарно увеличила
Горутина 2: атомарно увеличила
Горутина 1: атомарно увеличила
Горутина 2: атомарно увеличила
Горутина 1: атомарно увеличила

Итоговое значение: 10 (всегда будет 10!)

Атомарные операции НЕЛЬЗЯ прервать!
Чтение → Изменение → Запись = ОДНА неделимая операция
```

## **Часть 5: Основные атомарные функции**

```go
package main

import (
    "fmt"
    "sync/atomic"
)

func main() {
    fmt.Println("=== ВСЕ ОСНОВНЫЕ АТОМАРНЫЕ ФУНКЦИИ ===")

    var num int64

    // 1. Add - атомарное сложение
    fmt.Println("1. atomic.AddInt64 - атомарное сложение:")
    atomic.AddInt64(&num, 10)
    fmt.Printf("   Добавили 10: %d\n", num)
    atomic.AddInt64(&num, -3)
    fmt.Printf("   Вычли 3: %d\n", num)

    // 2. Load - атомарное чтение
    fmt.Println("\n2. atomic.LoadInt64 - атомарное чтение:")
    currentValue := atomic.LoadInt64(&num)
    fmt.Printf("   Текущее значение: %d\n", currentValue)

    // 3. Store - атомарная запись
    fmt.Println("\n3. atomic.StoreInt64 - атомарная запись:")
    atomic.StoreInt64(&num, 100)
    fmt.Printf("   Установили значение 100: %d\n", atomic.LoadInt64(&num))

    // 4. Swap - атомарная замена
    fmt.Println("\n4. atomic.SwapInt64 - атомарная замена:")
    oldValue := atomic.SwapInt64(&num, 200)
    fmt.Printf("   Заменили %d на 200\n", oldValue)
    fmt.Printf("   Новое значение: %d\n", atomic.LoadInt64(&num))

    // 5. CompareAndSwap - сравнить и заменить
    fmt.Println("\n5. atomic.CompareAndSwapInt64 - сравнить и заменить:")
    success := atomic.CompareAndSwapInt64(&num, 200, 300)
    fmt.Printf("   Пытаемся заменить 200 на 300: %v\n", success)
    fmt.Printf("   Новое значение: %d\n", atomic.LoadInt64(&num))

    success = atomic.CompareAndSwapInt64(&num, 999, 400)
    fmt.Printf("   Пытаемся заменить 999 на 400: %v (не удалось, текущее значение не 999)\n", success)
    fmt.Printf("   Значение осталось: %d\n", atomic.LoadInt64(&num))
}
```

**Вывод в консоли:**

```
=== ВСЕ ОСНОВНЫЕ АТОМАРНЫЕ ФУНКЦИИ ===
1. atomic.AddInt64 - атомарное сложение:
   Добавили 10: 10
   Вычли 3: 7

2. atomic.LoadInt64 - атомарное чтение:
   Текущее значение: 7

3. atomic.StoreInt64 - атомарная запись:
   Установили значение 100: 100

4. atomic.SwapInt64 - атомарная замена:
   Заменили 100 на 200
   Новое значение: 200

5. atomic.CompareAndSwapInt64 - сравнить и заменить:
   Пытаемся заменить 200 на 300: true
   Новое значение: 300
   Пытаемся заменить 999 на 400: false (не удалось, текущее значение не 999)
   Значение осталось: 300
```

## **Часть 6: CompareAndSwap (CAS) - самый мощный инструмент**

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

func main() {
    fmt.Println("=== ATOMIC COMPARE AND SWAP (CAS) ===")

    var value int64 = 0
    var wg sync.WaitGroup

    // 5 горутин пытаются увеличить значение, но только если оно равно ожидаемому
    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()

            for {
                current := atomic.LoadInt64(&value)
                fmt.Printf("Горутина %d: видит значение %d\n", id, current)

                // Пытаемся атомарно заменить current на current+1
                // Только если значение еще равно current (не изменилось другими)
                success := atomic.CompareAndSwapInt64(&value, current, current+1)

                if success {
                    fmt.Printf("Горутина %d: УСПЕШНО увеличила до %d\n", id, current+1)
                    break
                } else {
                    fmt.Printf("Горутина %d: НЕ УДАЛОСЬ, значение изменилось\n", id)
                }
            }
        }(i)
    }

    wg.Wait()
    fmt.Printf("\nФинальное значение: %d\n", atomic.LoadInt64(&value))
}
```

**Вывод в консоли (пример):**

```
=== ATOMIC COMPARE AND SWAP (CAS) ===
Горутина 5: видит значение 0
Горутина 5: УСПЕШНО увеличила до 1
Горутина 4: видит значение 0
Горутина 4: НЕ УДАЛОСЬ, значение изменилось
Горутина 4: видит значение 1
Горутина 4: УСПЕШНО увеличила до 2
Горутина 2: видит значение 0
Горутина 2: НЕ УДАЛОСЬ, значение изменилось
Горутина 2: видит значение 2
Горутина 2: УСПЕШНО увеличила до 3
Горутина 1: видит значение 0
Горутина 1: НЕ УДАЛОСЬ, значение изменилось
Горутина 1: видит значение 3
Горутина 1: УСПЕШНО увеличила до 4
Горутина 3: видит значение 0
Горутина 3: НЕ УДАЛОСЬ, значение изменилось
Горутина 3: видит значение 4
Горутина 3: УСПЕШНО увеличила до 5

Финальное значение: 5
```

**Объяснение CAS:**

- `CompareAndSwap` проверяет, равно ли текущее значение ожидаемому
- Если да - заменяет на новое и возвращает `true`
- Если нет (значение изменилось другой горутиной) - возвращает `false`

## **Часть 7: Атомарные указатели**

```go
package main

import (
    "fmt"
    "sync/atomic"
)

type Config struct {
    Name string
    Port int
}

func main() {
    fmt.Println("=== ATOMIC POINTERS ===")

    // Создаем атомарный указатель
    var config atomic.Pointer[Config]

    // Начальная конфигурация
    initialConfig := &Config{Name: "default", Port: 8080}
    config.Store(initialConfig)

    fmt.Printf("Начальная конфигурация: %+v\n", config.Load())

    // Атомарно обновляем конфигурацию
    newConfig := &Config{Name: "production", Port: 443}
    config.Store(newConfig)

    fmt.Printf("Новая конфигурация: %+v\n", config.Load())

    // Атомарно читаем конфигурацию
    currentConfig := config.Load()
    fmt.Printf("Текущая конфигурация: %s:%d\n", currentConfig.Name, currentConfig.Port)

    // Swap - атомарная замена указателя
    anotherConfig := &Config{Name: "staging", Port: 3000}
    oldConfig := config.Swap(anotherConfig)
    fmt.Printf("\nЗаменили %+v на %+v\n", oldConfig, config.Load())
}
```

**Вывод в консоли:**

```
=== ATOMIC POINTERS ===
Начальная конфигурация: &{Name:default Port:8080}
Новая конфигурация: &{Name:production Port:443}
Текущая конфигурация: production:443

Заменили &{Name:production Port:443} на &{Name:staging Port:3000}
```

## **Часть 8: Сравнение мьютексов и атомарных операций**

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
    "time"
)

func main() {
    fmt.Println("=== СРАВНЕНИЕ МЬЮТЕКСОВ И АТОМАРНЫХ ОПЕРАЦИЙ ===")

    const iterations = 1000000
    const goroutines = 10

    // Тест 1: Мьютекс
    fmt.Println("\n1. Тест с мьютексом:")
    var mutexCounter int64
    var mu sync.Mutex
    var wg sync.WaitGroup

    start := time.Now()
    wg.Add(goroutines)

    for i := 0; i < goroutines; i++ {
        go func() {
            defer wg.Done()
            for j := 0; j < iterations/goroutines; j++ {
                mu.Lock()
                mutexCounter++
                mu.Unlock()
            }
        }()
    }

    wg.Wait()
    mutexTime := time.Since(start)
    fmt.Printf("   Время: %v, Результат: %d\n", mutexTime, mutexCounter)

    // Тест 2: Атомарные операции
    fmt.Println("\n2. Тест с атомарными операциями:")
    var atomicCounter int64
    wg.Add(goroutines)

    start = time.Now()
    for i := 0; i < goroutines; i++ {
        go func() {
            defer wg.Done()
            for j := 0; j < iterations/goroutines; j++ {
                atomic.AddInt64(&atomicCounter, 1)
            }
        }()
    }

    wg.Wait()
    atomicTime := time.Since(start)
    fmt.Printf("   Время: %v, Результат: %d\n", atomicTime, atomicCounter)

    // Тест 3: Без синхронизации (опасно!)
    fmt.Println("\n3. Тест без синхронизации (опасно!):")
    var unsafeCounter int64
    wg.Add(goroutines)

    start = time.Now()
    for i := 0; i < goroutines; i++ {
        go func() {
            defer wg.Done()
            for j := 0; j < iterations/goroutines; j++ {
                unsafeCounter++ // Гонка данных!
            }
        }()
    }

    wg.Wait()
    unsafeTime := time.Since(start)
    fmt.Printf("   Время: %v, Результат: %d (НЕКОРРЕКТНЫЙ!)\n", unsafeTime, unsafeCounter)

    fmt.Println("\n=== ВЫВОД ===")
    fmt.Println("Атомарные операции обычно быстрее мьютексов")
    fmt.Println("Но работают только с простыми типами (int, pointer)")
    fmt.Println("Без синхронизации - быстрее, но НЕКОРРЕКТНО!")
}
```

**Вывод в консоли:**

```
=== СРАВНЕНИЕ МЬЮТЕКСОВ И АТОМАРНЫХ ОПЕРАЦИЙ ===

1. Тест с мьютексом:
   Время: 45.234ms, Результат: 1000000

2. Тест с атомарными операциями:
   Время: 32.567ms, Результат: 1000000

3. Тест без синхронизации (опасно!):
   Время: 12.345ms, Результат: 543210 (НЕКОРРЕКТНЫЙ!)

=== ВЫВОД ===
Атомарные операции обычно быстрее мьютексов
Но работают только с простыми типами (int, pointer)
Без синхронизации - быстрее, но НЕКОРРЕКТНО!
```

## **Итог лекции:**

1. **Атомарные операции** - неделимые операции, которые нельзя прервать
2. **Пакет `sync/atomic`** предоставляет атомарные операции для:

   - Целых чисел: `int32`, `int64`, `uint32`, `uint64`
   - Указателей: `atomic.Pointer[T]`

3. **Основные функции:**

   - `Add` - атомарное сложение/вычитание
   - `Load` - атомарное чтение
   - `Store` - атомарная запись
   - `Swap` - атомарная замена
   - `CompareAndSwap (CAS)` - сравнить и заменить

4. **Преимущества перед мьютексами:**

   - Выше производительность (меньше накладных расходов)
   - Проще в использовании для простых операций
   - Избегают блокировок (lock-free)

5. **Недостатки:**

   - Работают только с простыми типами
   - Не подходят для сложных операций с несколькими переменными

6. **Когда использовать:**

   - Счетчики, флаги, простые состояния
   - Когда нужна максимальная производительность
   - Для обновления указателей на конфигурацию

7. **Когда НЕ использовать:**
   - Сложные операции с несколькими переменными
   - Когда нужны условия или сложная логика
   - Для защиты структур данных (используйте мьютексы)

**Ключевое правило:** Если можно использовать атомарные операции вместо мьютексов - используйте их, они быстрее и проще!
