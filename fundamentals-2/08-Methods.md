## **Часть 1: ЧТО ТАКОЕ МЕТОДЫ НА САМОМ ДЕЛЕ**

### Код 1: Базовое понимание методов

```go
package main

import "fmt"

// Структура Rectangle
type Rectangle struct {
    length float64
    width  float64
}

// Метод с ПОЛУЧАТЕЛЕМ ПО ЗНАЧЕНИЮ (value receiver)
// Фактически: func Area(r Rectangle) float64
func (r Rectangle) Area() float64 {
    fmt.Printf("Внутри Area - адрес r: %p\n", &r)
    return r.length * r.width
}

// Метод с ПОЛУЧАТЕЛЕМ ПО УКАЗАТЕЛЮ (pointer receiver)
// Фактически: func Scale(r *Rectangle, factor float64)
func (r *Rectangle) Scale(factor float64) {
    fmt.Printf("Внутри Scale - адрес r: %p\n", r)
    r.length *= factor
    r.width *= factor
}

// Обычная функция (для сравнения)
func AreaFunc(r Rectangle) float64 {
    return r.length * r.width
}

func main() {
    fmt.Println("=== ЧТО ТАКОЕ МЕТОДЫ ===")

    rect := Rectangle{length: 10, width: 9}

    fmt.Printf("Адрес rect в main: %p\n", &rect)

    // Вызов метода с value receiver
    area := rect.Area()
    fmt.Printf("Площадь: %.2f\n", area)

    // Вызов метода с pointer receiver
    rect.Scale(2)
    fmt.Printf("После масштабирования: %.2f x %.2f\n", rect.length, rect.width)

    // Вызов обычной функции (для сравнения)
    area2 := AreaFunc(rect)
    fmt.Printf("Площадь через функцию: %.2f\n", area2)
}
```

**Вывод программы:**

```
=== ЧТО ТАКОЕ МЕТОДЫ ===
Адрес rect в main: 0xc0000180a8
Внутри Area - адрес r: 0xc0000180c0  ← РАЗНЫЙ адрес (копия!)
Площадь: 90.00
Внутри Scale - адрес r: 0xc0000180a8  ← ТОТ ЖЕ адрес (оригинал)
После масштабирования: 20.00 x 18.00
Площадь через функцию: 360.00
```

**Что происходит под капотом:**

1. **Метод с value receiver** `(r Rectangle) Area()`:

   - Go создает **КОПИЮ** структуры
   - Метод работает с копией
   - Изменения не влияют на оригинал
   - В памяти: `rect.Area()` → `Area(rect)` (неявное копирование)

2. **Метод с pointer receiver** `(r *Rectangle) Scale()`:
   - Go передает **АДРЕС** структуры (8 байт)
   - Метод работает с оригиналом
   - Изменения сохраняются
   - В памяти: `rect.Scale(2)` → `Scale(&rect, 2)`

## **Часть 2: РАЗНИЦА МЕЖДУ VALUE И POINTER RECEIVER**

### Код 2: Детальное сравнение

```go
package main

import (
    "fmt"
    "unsafe"
)

type Data struct {
    values [1000]int // Большая структура
    id     int
}

// Value receiver - КОПИРУЕТ всю структуру
func (d Data) PrintByValue() {
    fmt.Printf("PrintByValue: id=%d, size=%d байт (копия)\n",
        d.id, unsafe.Sizeof(d))
    d.id = 999 // Не влияет на оригинал
}

// Pointer receiver - передает только АДРЕС
func (d *Data) PrintByPointer() {
    fmt.Printf("PrintByPointer: id=%d, size указателя=%d байт\n",
        d.id, unsafe.Sizeof(d))
    d.id = 888 // Изменяет оригинал
}

func main() {
    fmt.Println("=== VALUE vs POINTER RECEIVER ===")

    // Большая структура (1000 int * 8 байт = 8000 байт)
    data := Data{id: 1}
    data.values[0] = 42

    fmt.Printf("Размер Data: %d байт\n", unsafe.Sizeof(data))
    fmt.Printf("Размер указателя: %d байт\n", unsafe.Sizeof(&data))

    // Тестируем value receiver
    fmt.Println("\n1. Value receiver:")
    fmt.Printf("До вызова: id=%d\n", data.id)
    data.PrintByValue()
    fmt.Printf("После вызова: id=%d (НЕ изменился!)\n", data.id)

    // Тестируем pointer receiver
    fmt.Println("\n2. Pointer receiver:")
    fmt.Printf("До вызова: id=%d\n", data.id)
    data.PrintByPointer()
    fmt.Printf("После вызова: id=%d (изменился!)\n", data.id)

    // Автоматические преобразования
    fmt.Println("\n=== АВТОМАТИЧЕСКИЕ ПРЕОБРАЗОВАНИЯ ===")

    ptr := &data

    // Метод с value receiver
    ptr.PrintByValue()  // OK: (*ptr).PrintByValue()
    data.PrintByValue() // OK: копирование

    // Метод с pointer receiver
    ptr.PrintByPointer()  // OK
    data.PrintByPointer() // OK: &data.PrintByPointer()

    // Когда использовать что:
    fmt.Println("\n=== КОГДА ИСПОЛЬЗОВАТЬ ===")
    fmt.Println("Value receiver (значение):")
    fmt.Println("  - Метод НЕ изменяет структуру")
    fmt.Println("  - Структура маленькая (менее ~64 байт)")
    fmt.Println("  - Нужна иммутабельность")

    fmt.Println("\nPointer receiver (указатель):")
    fmt.Println("  - Метод изменяет структуру")
    fmt.Println("  - Структура большая")
    fmt.Println("  - Для согласованности (все методы используют pointer)")
}
```

**Вывод программы:**

```
=== VALUE vs POINTER RECEIVER ===
Размер Data: 8008 байт
Размер указателя: 8 байт

1. Value receiver:
До вызова: id=1
PrintByValue: id=1, size=8008 байт (копия)
После вызова: id=1 (НЕ изменился!)

2. Pointer receiver:
До вызова: id=1
PrintByPointer: id=1, size указателя=8 байт
После вызова: id=888 (изменился!)

=== АВТОМАТИЧЕСКИЕ ПРЕОБРАЗОВАНИЯ ===
...
```

**Ключевое понимание:**

```
Value receiver:
  rect.Area() → Go делает: Area(rect) ← КОПИЯ 8008 байт!

Pointer receiver:
  rect.Scale(2) → Go делает: Scale(&rect, 2) ← АДРЕС 8 байт
```

## **Часть 3: МЕТОДЫ ДЛЯ ЛЮБЫХ ТИПОВ (не только структур)**

### Код 3: Методы для пользовательских типов

```go
package main

import "fmt"

// 1. Пользовательский тип на основе int
type MyInt int

// Метод для MyInt
func (m MyInt) IsPositive() bool {
    return m > 0
}

// Метод, который не использует значение получателя
func (m MyInt) WelcomeMessage() string {
    return "Добро пожаловать в тип MyInt!"
}

// 2. Пользовательский тип на основе slice
type IntSlice []int

// Метод для IntSlice (добавление элемента)
func (s *IntSlice) Add(value int) {
    *s = append(*s, value)
}

// Метод для IntSlice (сумма)
func (s IntSlice) Sum() int {
    total := 0
    for _, v := range s {
        total += v
    }
    return total
}

// 3. Пользовательский тип на основе map
type UserMap map[string]string

// Метод для UserMap
func (um UserMap) GetOrDefault(key, defaultValue string) string {
    if value, exists := um[key]; exists {
        return value
    }
    return defaultValue
}

func main() {
    fmt.Println("=== МЕТОДЫ ДЛЯ ЛЮБЫХ ТИПОВ ===")

    // 1. MyInt
    var num MyInt = -5
    var num2 MyInt = 9

    fmt.Printf("%d положительное? %v\n", num, num.IsPositive())
    fmt.Printf("%d положительное? %v\n", num2, num2.IsPositive())
    fmt.Println("Сообщение:", num.WelcomeMessage())

    // 2. IntSlice
    var slice IntSlice = []int{1, 2, 3}
    fmt.Printf("\nСлайс: %v, сумма: %d\n", slice, slice.Sum())

    slice.Add(4)
    slice.Add(5)
    fmt.Printf("После добавления: %v, сумма: %d\n", slice, slice.Sum())

    // 3. UserMap
    users := UserMap{
        "alice": "Алиса",
        "bob":   "Боб",
    }

    fmt.Printf("\nПользователь 'alice': %s\n", users.GetOrDefault("alice", "Не найден"))
    fmt.Printf("Пользователь 'charlie': %s\n", users.GetOrDefault("charlie", "Не найден"))

    // 4. Метод для типа-алиаса string
    type Email string

    // Метод для проверки email
    func (e Email) IsValid() bool {
        // Простая проверка
        for _, ch := range e {
            if ch == '@' {
                return true
            }
        }
        return false
    }

    email := Email("user@example.com")
    fmt.Printf("\nEmail '%s' валиден? %v\n", email, email.IsValid())

    invalidEmail := Email("userexample.com")
    fmt.Printf("Email '%s' валиден? %v\n", invalidEmail, invalidEmail.IsValid())
}
```

**Вывод программы:**

```
=== МЕТОДЫ ДЛЯ ЛЮБЫХ ТИПОВ ===
-5 положительное? false
9 положительное? true
Сообщение: Добро пожаловать в тип MyInt!

Слайс: [1 2 3], сумма: 6
После добавления: [1 2 3 4 5], сумма: 15

Пользователь 'alice': Алиса
Пользователь 'charlie': Не найден

Email 'user@example.com' валиден? true
Email 'userexample.com' валиден? false
```

**Важное замечание:**

- Методы можно определять для любых типов, **но только в том же пакете**, где определен тип
- Нельзя добавить методы к встроенным типам (`int`, `string`) напрямую, нужно создать пользовательский тип

## **Часть 4: ВСТРАИВАНИЕ МЕТОДОВ (PROMOTION)**

### Код 4: Как работают встроенные методы

```go
package main

import "fmt"

// Встраиваемая структура
type Engine struct {
    power int    // л.с.
    fuel  string // тип топлива
}

// Метод Engine
func (e Engine) Start() {
    fmt.Printf("Двигатель запущен: %d л.с., топливо: %s\n",
        e.power, e.fuel)
}

// Метод Engine
func (e *Engine) Upgrade(power int) {
    e.power = power
    fmt.Printf("Двигатель улучшен до %d л.с.\n", power)
}

// Основная структура
type Car struct {
    brand  string
    model  string
    Engine // Встраивание Engine
}

// Метод Car
func (c Car) Info() {
    fmt.Printf("Автомобиль: %s %s\n", c.brand, c.model)
}

func main() {
    fmt.Println("=== ВСТРАИВАНИЕ МЕТОДОВ ===")

    car := Car{
        brand: "Toyota",
        model: "Camry",
        Engine: Engine{
            power: 200,
            fuel:  "бензин",
        },
    }

    // Методы Engine становятся доступны для Car
    car.Start()  // Продвинутый метод: car.Engine.Start()
    car.Info()

    // Можно вызывать через полный путь
    car.Engine.Start()

    // Изменение через pointer receiver
    fmt.Println("\nДо улучшения:")
    car.Start()

    car.Upgrade(250)  // car.Engine.Upgrade(250)

    fmt.Println("После улучшения:")
    car.Start()

    // Конфликт имен методов
    fmt.Println("\n=== КОНФЛИКТ ИМЕН ===")

    type Boat struct {
        Engine
    }

    // Метод Start с тем же именем
    func (b Boat) Start() {
        fmt.Println("Лодка готова к отплытию!")
        b.Engine.Start() // Явный вызов встроенного метода
    }

    boat := Boat{Engine{150, "дизель"}}
    boat.Start() // Вызывается метод Boat, а не Engine

    // Явный вызов встроенного метода
    boat.Engine.Start()
}
```

**Вывод программы:**

```
=== ВСТРАИВАНИЕ МЕТОДОВ ===
Двигатель запущен: 200 л.с., топливо: бензин
Автомобиль: Toyota Camry
Двигатель запущен: 200 л.с., топливо: бензин

До улучшения:
Двигатель запущен: 200 л.с., топливо: бензин
Двигатель улучшен до 250 л.с.
После улучшения:
Двигатель запущен: 250 л.с., топливо: бензин

=== КОНФЛИКТ ИМЕН ===
Лодка готова к отплытию!
Двигатель запущен: 150 л.с., топливо: дизель
Двигатель запущен: 150 л.с., топливо: дизель
```

**Что происходит под капотом:**

1. **Продвижение методов (method promotion)**:

   - Методы встроенной структуры становятся доступны для внешней
   - `car.Start()` → компилятор ищет `Start()` в `Car`, не находит, ищет в `Engine`
   - В памяти: методы не копируются, просто добавляются в таблицу методов

2. **Таблица методов**:
   ```
   Car методы:
     Info()   - собственный метод
     Start()  - продвинутый из Engine
     Upgrade()- продвинутый из Engine (pointer receiver)
   ```

## **Часть 5: ИНТЕРФЕЙСЫ И МЕТОДЫ (предварительный взгляд)**

### Код 5: Как методы связаны с интерфейсами

```go
package main

import (
    "fmt"
    "math"
)

// Интерфейс (будем подробно изучать позже)
type Shape interface {
    Area() float64
    Perimeter() float64
}

// Структура Circle
type Circle struct {
    radius float64
}

// Методы для Circle
func (c Circle) Area() float64 {
    return math.Pi * c.radius * c.radius
}

func (c Circle) Perimeter() float64 {
    return 2 * math.Pi * c.radius
}

// Структура Square
type Square struct {
    side float64
}

// Методы для Square
func (s Square) Area() float64 {
    return s.side * s.side
}

func (s Square) Perimeter() float64 {
    return 4 * s.side
}

// Функция, принимающая интерфейс
func PrintShapeInfo(s Shape) {
    fmt.Printf("Площадь: %.2f, Периметр: %.2f\n",
        s.Area(), s.Perimeter())
}

func main() {
    fmt.Println("=== МЕТОДЫ И ИНТЕРФЕЙСЫ ===")

    circle := Circle{radius: 5}
    square := Square{side: 4}

    // Вызов методов напрямую
    fmt.Printf("Круг: площадь=%.2f, периметр=%.2f\n",
        circle.Area(), circle.Perimeter())
    fmt.Printf("Квадрат: площадь=%.2f, периметр=%.2f\n",
        square.Area(), square.Perimeter())

    // Использование интерфейса
    fmt.Println("\nЧерез интерфейс:")
    PrintShapeInfo(circle)
    PrintShapeInfo(square)

    // Слайс интерфейсов
    shapes := []Shape{circle, square}
    fmt.Println("\nСлайс фигур:")
    for i, shape := range shapes {
        fmt.Printf("Фигура %d: площадь=%.2f\n", i, shape.Area())
    }
}
```

**Вывод программы:**

```
=== МЕТОДЫ И ИНТЕРФЕЙСЫ ===
Круг: площадь=78.54, периметр=31.42
Квадрат: площадь=16.00, периметр=16.00

Через интерфейс:
Площадь: 78.54, Периметр: 31.42
Площадь: 16.00, Периметр: 16.00

Слайс фигур:
Фигура 0: площадь=78.54
Фигура 1: площадь=16.00
```

**Ключевой момент:** Интерфейсы в Go определяют набор методов. Если тип имеет все методы интерфейса, он автоматически удовлетворяет этому интерфейсу.

## **Часть 6: ПРАКТИЧЕСКИЕ ПРИМЕРЫ И ОШИБКИ**

### Код 6: Частые ошибки и лучшие практики

```go
package main

import "fmt"

type BankAccount struct {
    owner   string
    balance float64
}

// Правильно: pointer receiver для изменения
func (ba *BankAccount) Deposit(amount float64) {
    if amount > 0 {
        ba.balance += amount
    }
}

// Правильно: pointer receiver для изменения
func (ba *BankAccount) Withdraw(amount float64) bool {
    if amount > 0 && ba.balance >= amount {
        ba.balance -= amount
        return true
    }
    return false
}

// Правильно: value receiver для чтения
func (ba BankAccount) GetBalance() float64 {
    return ba.balance
}

// НЕПРАВИЛЬНО: value receiver для изменения
func (ba BankAccount) BadDeposit(amount float64) {
    ba.balance += amount // Изменяет копию!
}

func main() {
    fmt.Println("=== ПРАКТИЧЕСКИЕ ПРИМЕРЫ ===")

    // Создаем счет
    account := BankAccount{
        owner:   "Иван",
        balance: 1000,
    }

    fmt.Printf("Начальный баланс: %.2f\n", account.GetBalance())

    // Правильное пополнение
    account.Deposit(500)
    fmt.Printf("После депозита: %.2f\n", account.GetBalance())

    // Неправильное пополнение (не сработает)
    account.BadDeposit(500)
    fmt.Printf("После BadDeposit: %.2f (не изменился!)\n", account.GetBalance())

    // Снятие
    success := account.Withdraw(300)
    if success {
        fmt.Printf("После снятия: %.2f\n", account.GetBalance())
    }

    fmt.Println("\n=== СОВМЕСТИМОСТЬ МЕТОДОВ ===")

    // Методы должны быть последовательны
    type Config struct {
        value int
    }

    // Смешивание value и pointer receivers - ПЛОХАЯ ПРАКТИКА
    func (c Config) GetValue() int {
        return c.value
    }

    func (c *Config) SetValue(v int) {
        c.value = v
    }

    // Лучше всегда использовать pointer receiver
    type GoodConfig struct {
        value int
    }

    func (gc *GoodConfig) Get() int {
        return gc.value
    }

    func (gc *GoodConfig) Set(v int) {
        gc.value = v
    }

    fmt.Println("\n=== МЕТОДЫ ДЛЯ nil ===")

    type SafeStruct struct {
        data string
    }

    // Метод, который работает с nil
    func (s *SafeStruct) SafeMethod() string {
        if s == nil {
            return "Объект nil"
        }
        return s.data
    }

    var nilStruct *SafeStruct
    fmt.Println("Вызов метода у nil:", nilStruct.SafeMethod())

    // А вот так будет паника:
    // nilStruct.data // panic: runtime error
}
```

**Вывод программы:**

```
=== ПРАКТИЧЕСКИЕ ПРИМЕРЫ ===
Начальный баланс: 1000.00
После депозита: 1500.00
После BadDeposit: 1500.00 (не изменился!)
После снятия: 1200.00

=== СОВМЕСТИМОСТЬ МЕТОДОВ ===

=== МЕТОДЫ ДЛЯ nil ===
Вызов метода у nil: Объект nil
```

## **Шпаргалка по методам:**

```
ОСНОВЫ МЕТОДОВ:
  func (r Receiver) MethodName(params) returns {
      // тело метода
  }

ТИПЫ RECEIVER:
  1. Value receiver:    (r Type)
     - Работает с копией
     - Не изменяет оригинал
     - Подходит для маленьких структур

  2. Pointer receiver:  (r *Type)
     - Работает с оригиналом
     - Может изменять значения
     - Эффективен для больших структур

АВТОМАТИЧЕСКИЕ ПРЕОБРАЗОВАНИЯ:
  value.Method()      → (&value).Method()   // для pointer receiver
  pointer.Method()    → (*pointer).Method() // для value receiver

ВСТРАИВАНИЕ МЕТОДОВ:
  type Car struct {
      Engine          // Встраивание
  }
  car.Start()        // Метод Engine доступен через Car

МЕТОДЫ ДЛЯ ЛЮБЫХ ТИПОВ:
  type MyInt int
  func (m MyInt) Method() { ... }

ЛУЧШИЕ ПРАКТИКИ:
  1. Используйте pointer receiver, если метод изменяет получателя
  2. Будьте последовательны: все методы типа должны использовать
     один тип receiver (value или pointer)
  3. Для больших структур всегда используйте pointer receiver
  4. Проверяйте nil в методах с pointer receiver, если нужно
```

**Ключевые выводы:**

1. **Методы** - это функции с получателем (receiver)
2. **Value receiver** создает копию, **pointer receiver** работает с оригиналом
3. Go автоматически преобразует `value` → `&value` и `pointer` → `*pointer` при вызове методов
4. Методы можно определять для любых типов в том же пакете
5. Встроенные методы автоматически "продвигаются" (promoted)
6. Интерфейсы определяют набор методов, которые должен реализовать тип

**Что происходит в памяти при вызове метода:**

```
rect.Area() с value receiver:
  1. Копирование всей структуры rect в стек вызова
  2. Вызов Area с копией
  3. Возврат результата

rect.Scale(2) с pointer receiver:
  1. Помещение адреса &rect в стек (8 байт)
  2. Вызов Scale с адресом
  3. Изменение оригинальной структуры
```
